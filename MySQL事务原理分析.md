# 事务四大特性
## 1. A — Atomicity（原子性）
### 含义
- 事务是数据库操作的最小单元，要么全部成功，要么全部失败回滚，不允许只完成一部分
- 如果事务中途失败（断电、异常、SQL 错误），已经执行的修改必须撤销到事务开始前的状态
### 作用
- 保证业务逻辑的完整执行，例如转账时，A 账户扣款和 B 账户加款必须同时成功或同时失败
### InnoDB 实现方式
Undo Log（回滚日志）：
- 在修改数据前，先把被修改的数据的旧值记录到 Undo Log 中
- 事务回滚时，按 Undo Log 把数据恢复到修改前的状态<br>

回滚可以是显式（ROLLBACK）或隐式（事务失败/崩溃恢复时自动执行）

## 2. C — Consistency（一致性）
### 含义 
- 事务执行前后，数据库必须从一个一致状态转变到另一个一致状态
- 一致性不仅指单表数据约束（比如唯一性、外键、检查约束），还包括业务逻辑层面的完整性规则
### 作用
- 保证数据的正确性，不会出现违反约束或逻辑错误的数据状态
### InnoDB 实现方式
- 依赖 原子性、隔离性、持久性共同保证
- 约束与触发器：在执行事务时即刻检查并阻止违规数据写入
- 外部应用逻辑配合，确保业务规则正确实现

## 3. I — Isolation（隔离性）
### 含义
- 多个事务并发执行时，一个事务的中间状态对其他事务是不可见的
- 不同隔离级别决定了事务之间可见性的强弱，防止脏读、不可重复读、幻读等问题
### SQL 标准隔离级别（从低到高）
对于所有的隔离级别,写操作都是加锁的
#### 1. READ UNCOMMITTED（读未提交）
- 可能出现脏读（读到未提交的数据）

#### 2. READ COMMITTED（读已提交）
- 每次读都只能读到其他事务已经提交的数据（Oracle 默认）

#### 3. REPEATABLE READ（可重复读）
- 同一事务中多次读取同一行结果一致（InnoDB 默认，还额外解决了幻读）

#### 4. SERIALIZABLE（可串行化）
- 强制事务串行执行，完全避免并发冲突，但性能最差

## 4. D — Durability（持久性）
### 含义
- 一旦事务提交，其结果就必须被永久保存，即使系统宕机或掉电也不能丢失
### 作用
- 保障数据的最终安全性，使提交后的数据可恢复
### InnoDB 实现方式
Redo Log（重做日志）：
- 在事务提交前，把变更以物理日志形式顺序写入 redo log 并 fsync 落盘
- 宕机后，重启时用 redo log 重放最近提交的事务，恢复到最新状态<br>

Doublewrite Buffer：
- 防止页半写导致 redo 无法正确应用<br>

持久化策略参数：
- innodb_flush_log_at_trx_commit=1（最安全，每次提交都 fsync redo log）
- sync_binlog=1（每次提交都 fsync binlog，主从复制安全）


# 不同的隔离级别并发异常
## 1. 脏读（Dirty Read）
### 1.1 现象
读到别的事务还没提交的数据
### 1.2 出现的隔离级别
只会出现在<mark>读未提交READ UNCOMMITTED</mark>隔离级别
## 2. 不可重复读（Non-Repeatable Read）
### 2.1 现象
同一事务内两次读同一行，结果不同（因为别的事务提交了对该行的修改/删除）
### 2.2 出现的隔离级别
会出现在<mark>读未提交READ UNCOMMITTED和读已提交 READ COMMITTED</mark>隔离级别
## 3. 幻读（Phantom Read）
### 3.1 现象
同一事务内按某个条件读出一批行，后续再读同一范围内的记录多了或少了行（因为别的事务插入/删除了“新行”满足该条件）<br>
也就是当前读和快照读的不一致
### 3.2 出现的隔离级别
会出现在<mark>读未提交READ UNCOMMITTED和读已提交 READ COMMITTED</mark>隔离级别,对于<mark>可重复读REPEATABLE READ</mark>, 在事务开始时锁定读 FOR UPDATE/SHARE或 LOCK IN SHARE MODE可以避免幻读

# MVCC多版本并发控制

## 1. Read View（读视图）

### 1.1 m_ids
当前系统中所有未提交的活跃事务的 ID 列表,用来判断某个数据版本是不是由这些活跃事务产生的，如果是，就不可见

### 1.2 min_trx_id
活跃事务列表里最小的事务 ID。<mark>小于 min_trx_id 的版本一定已经提交 → 可见</mark>

### 1.3 max_trx_id
生成 Read View 时，系统分配的下一个事务 ID。<mark>大于或等于 max_trx_id 的版本，肯定是未来事务产生的 → 不可见。</mark>

### 1.4 creator_trx_id
创建这个 Read View 的事务 ID。<mark>事务应该能“看到自己修改过的数据”，所以特殊对待：即使 creator_trx_id 在活跃事务里，也对自己可见。</mark>


## 2. InnoDB 行记录的隐藏字段

### 2.1 DB_ROW_ID
如果表里没有显式定义主键，InnoDB 会自动生成一个 6 字节的行 ID，单调递增,作为该行的内部唯一标识；被用来构造聚簇索引（Clustered Index）的主键

### 2.2 DB_TRX_ID
最近一次修改（INSERT/UPDATE/DELETE）该行的事务 ID，占 6 字节,<mark>MVCC 中用来判断该版本是否对某个 Read View 可见；</mark>事务提交时由 InnoDB 更新，保证最新版本能区分创建它的事务。

### 2.3 DB_ROLL_PTR
指向 Undo Log（回滚段）的指针，占 7 字节;可以通过它找到这条记录的历史版本；实现 MVCC 的“版本链”：当前版本 → Undo Log → 更旧的版本

<img width="1124" height="250" alt="image" src="https://github.com/user-attachments/assets/15902eb1-14b6-4802-be27-e6652154a069" />

## 3. 事务可见性问题
假设某条记录的版本号是 DB_TRX_ID：
### 3.1 事务可以看到本事务自身的修改
#### DB_TRX_ID == creator_trx_id
✅可见, 依上所述,事务应该能“看到自己修改过的数据”，所以特殊对待：即使 creator_trx_id 在活跃事务里，也对自己可见。

### 3.2 事务间的可见性

####  DB_TRX_ID < min_trx_id
✅ 可见, 这个版本对应的事务早就提交了

#### DB_TRX_ID >= max_trx_id
❌ 不可见, 这个版本来自未来事务,当前版本不可见，就沿着 DB_ROLL_PTR 找 Undo Log 里的上一个版本，继续判断，直到找到可见版本为止

#### DB_TRX_ID ∈ m_ids
❌ 不可见, 该事务未提交

#### 其余情况
✅ 可见, 刚好在本事务开始后提交的其他事务

## 4. READ COMMITTED和REPEATABLE READ 的MVCC
### 4.1 相同点
- READ COMMITTED (RC) 和 REPEATABLE READ (RR) 在 InnoDB 下都使用 MVCC（多版本并发控制） 来实现一致性读（快照读，普通 SELECT）。
- 两者都会利用：DB_TRX_ID（行的最后修改事务 ID）,DB_ROLL_PTR（Undo Log 指针，找到旧版本）,Read View（活跃事务 ID 列表）来判断当前事务能看到哪个版本。

### 4.2 核心区别：Read View 的生成时机
- READ COMMITTED (RC)每次执行快照读（SELECT）时，都会新生成一个 Read View。同一事务中，多次查询可能看到不同的已提交数据 → 可能出现不可重复读
- REPEATABLE READ (RR), 事务第一次执行快照读时生成 Read View，整个事务期间复用这个视图。事务期间读到的数据始终一致，即使别的事务提交了新版本，也看不到 → 避免不可重复读

## 5 锁
### 5.1 全局锁（Global Lock）
#### FLUSH TABLES WITH READ LOCK (FTWRL)
- 锁住整个实例，所有库、所有表只能读，不能写。
- 用于全库备份（逻辑备份时保证一致性）

### 5.2 数据库/表级锁
#### 表锁（Table Lock）
- LOCK TABLES ... READ/WRITE
- 整张表加锁。读锁可以并发读，写锁独占
- 开销小、粒度大，适合 MyISAM 或管理操作

#### 元数据锁（Metadata Lock, MDL）
- 自动加锁，不需要手动操作
- 作用：保证 DML 和 DDL 的并发安全

#### 意向锁（Intention Lock）
快速判断表里是否有行锁冲突，提高加锁效率

### 5.3 行级锁（Row Lock, InnoDB 核心）
InnoDB 基于索引来加行锁，若条件不走索引，可能升级为表锁。
#### 记录锁（Record Lock）
锁定索引上的某一条记录,如：SELECT ... FOR UPDATE WHERE id=10; → 锁住 id=10 这一行。
- 共享锁（S Lock）：允许事务读取一行，阻止其他事务修改。SQL 写法：SELECT ... LOCK IN SHARE MODE（MySQL 8.0 推荐 FOR SHARE）
- 排他锁（X Lock）：允许事务修改/删除一行，阻止其他事务读写。SQL 写法：SELECT ... FOR UPDATE
#### 间隙锁（Gap Lock）
- <mark>锁定一个范围的“间隙”，不包含已有记录,也就是左开右开 (),从而避免幻读现象</mark>
- 如：WHERE id > 10 AND id < 20 FOR UPDATE; → 会锁住 (10,20) 区间，阻止插入新行。
#### 临键锁（Next-Key Lock）
-  <mark>记录锁 + 间隙锁的组合。</mark>
-  <mark>锁定一个记录以及它之前的间隙，用于防止幻读。既防止了“改已有行”，又防止了“插新行” → 彻底避免幻读。</mark>
- 是 InnoDB 在 REPEATABLE READ 下的默认加锁方式。
### 查询
#### MVCC
参考DB_TRX_ID >= max_trx_id时的情况
#### S锁
#### X锁
见行级锁里的记录锁
#### 不做任何处理
读未提交隔离级别使用

### 删除\更新
自动加X锁

### 插入
#### 插入意向锁（Insert Intention Lock）
在插入位置所在的间隙上，加一个特殊的间隙锁:
- 表示“我打算在这个间隙插入数据”。
- 多个事务在同一个间隙里插入不同的值时，它们的插入意向锁不会互相冲突，因此可以并发插入
- 如果该间隙已被别的事务加了 Next-Key Lock/Gap Lock（比如范围查询 FOR UPDATE），插入意向锁就会被阻塞，从而防止幻读

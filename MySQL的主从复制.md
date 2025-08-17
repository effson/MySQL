## 1.主从复制的原理
### 主库（Master）
- 在事务提交时，把数据变更写入 binlog（二进制日志）
- binlog 记录了对数据库进行的所有修改操作（如 INSERT/UPDATE/DELETE），但不会记录 SELECT
### 从库（Slave/Replica）
- 通过 I/O 线程 从主库拉取 binlog，写入本地 relay log（中继日志）
- SQL 线程 读取 relay log 并在从库上重放这些操作，从而达到和主库一致的数据

## 2. 复制的三种模式
### 基于语句的复制（SBR, Statement-Based Replication）
- 在 binlog 中记录 SQL 语句
- 优点：日志小
- 缺点：某些不确定性函数（如 NOW()、UUID()）可能导致主从结果不一致
### 基于行的复制（RBR, Row-Based Replication）
- 在 binlog 中记录行的变化（哪一行被改成了什么值）
- 优点：精确，不存在不一致问题
- 缺点：日志量大

### 混合模式复制（MBR, Mixed-Based Replication）
- 默认采用语句复制，遇到不确定语句时切换到行复制

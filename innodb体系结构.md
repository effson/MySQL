<img width="1171" height="648" alt="image" src="https://github.com/user-attachments/assets/214ff604-2c56-4e76-a100-eba3dca80640" />

# 1.  Adaptive Hash Index（自适应哈希索引）
## 1.1 介绍
Innodb 自适应哈希索引（Adaptive Hash Index, AHI）是 MySQL Innodb 存储引擎的一个独特功能，旨在提高查询性能。它是一种内部机制，由 Innodb 引擎自动创建和管理
## 1.2 工作原理
### 1.2.1 监控访问模式
Innodb 引擎会持续监控对表索引的访问模式。
### 1.2.2 自动创建
如果 Innodb 检测到某个索引的某个前缀（通常是完整的索引键）被频繁访问，它就会将该键值和指向其在缓冲池中对应页面的指针，添加到哈希索引中。
### 1.2.3 高效查找
下次查询时，如果查询条件能匹配到这个哈希索引中的键，Innodb 就能直接通过哈希表查找数据页的内存地址，而不需要从 B-tree 的根节点开始一层层往下搜索。这在某些情况下能将查询时间从 O(log n) 降至 O(1)
# 2. 内存结构（Memory Structures）
## 2.1 Buffer Pool
### 2.1.1 作用
主要作用是缓存表数据和索引数据（聚集索引B+树缓存在Buffer Pool），MySQL 需要读取或写入数据时，它不会直接操作磁盘上的文件，而是先将数据从磁盘加载到内存中的 Buffer Pool。后续对相同数据的访问，就可以直接在内存中进行，从而避免了耗时的磁盘 I/O 操作，极大地提高了数据库的性能
### 2.1.2 工作原理
#### 2.1.2.1 数据读取
当 Innodb 需要读取某个数据页（例如，执行 SELECT 查询）时，它会首先检查 Buffer Pool 中是否已经存在这个数据页
- 命中（Hit）: 如果数据页已经在 Buffer Pool 中，Innodb 会直接从内存中读取，这非常快
- 未命中（Miss）: 如果数据页不在 Buffer Pool 中，Innodb 会从磁盘加载这个数据页到 Buffer Pool 的一个空闲页中。如果 Buffer Pool 已满，它会使用一种淘汰算法（通常是改进的 LRU 算法）来淘汰掉一个最不常用或最老的数据页，然后将新的数据页加载进来
#### 2.1.2.2 数据写入
当 Innodb 需要修改某个数据页时，它会先在 Buffer Pool 中找到这个数据页，并对其进行修改。此时，这个数据页被称为“脏页”（dirty page），因为它在内存中的内容与磁盘上的内容不一致
- 异步刷盘: 为了不阻塞当前的写入操作，Innodb 不会立即将脏页写回磁盘。相反，它会将这些脏页标记出来，并在后台的某个时机（例如，Buffer Pool 空间不足、Innodb 引擎关闭、或者定期刷盘机制）异步地将它们刷回磁盘，以保证数据的一致性和持久性
#### 2.1.2.3 LRU（Least Recently Used）淘汰算法
- LRU 列表分成了两个子列表：新列表（New sublist）和老列表（Old sublist）。新读入的数据页不会直接放在列表头部（新列表），而是先放在老列表的头部。只有当一个数据页在老列表中被再次访问时，它才会被移动到新列表的头部

<img width="365" height="610" alt="image" src="https://github.com/user-attachments/assets/3ca6bb06-8dda-4666-8835-98ab5963b12d" />

## 2.2 Change BUffer
### 2.2.1 定义与作用
- Change Buffer 是 InnoDB Buffer Pool 里的一个特殊区域，用来<mark>缓存对二级索引页（secondary index page）</mark>的修改操作（插入、更新、删除），而不是立即去磁盘随机写。
- 主要目的：减少随机 I/O，把多个分散的修改合并成顺序 I/O，提高性能。
- 主键索引（聚簇索引）不使用 Change Buffer，因为它访问频率高且会被频繁缓存。
### 2.2.2 工作原理
#### 2.2.2.1 写入流程
当需要修改一个不在 Buffer Pool 的二级索引页：
- 不直接从磁盘读取该页到内存
- 把变更记录插入到 Change Buffer<br>
后台线程或读请求需要该页时，才会触发 Merge（合并）：

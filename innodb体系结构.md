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
缓存数据页（Data Page）、索引页（Index Page）、Undo 页等，减少磁盘 I/O
### 2.1.2 特点

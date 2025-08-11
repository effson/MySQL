## 1. 定义
结构化查询语言(Structured Query Language),简称SQL,是一种用于管理关系型数据库的标准化语言。简单来说，SQL 是一种与数据库进行交流的语言
## 2. DQL
 Data Query Language，即数据查询语言。SQL 中，DQL 的核心命令就是 SELECT 语句。虽然 SELECT 语句经常与其他 DML (Data Manipulation Language) 命令（如 FROM、WHERE）一起使用，但从功能上严格划分，SELECT 命令本身就是 DQL 的代表
## 3. DML
DML (Data Manipulation Language) 数据操作语言，常用 DML 命令：
- INSERT: 向表中插入新数据。
- UPDATE: 修改表中已有的数据。
- DELETE: 从表中删除数据。
## 4. DDL
DDL (Data Definition Language)数据定义语言，DDL 语句用于定义和修改数据库对象的结构，例如表、视图、索引等。执行 DDL 命令后，这些结构的变化会立即生效，并且无法撤销。DDL 操作会自动触发一次 COMMIT，即永久保存更改。
常用 DDL 命令：
- CREATE: 创建新的数据库、表、视图、索引等。
- ALTER: 修改已存在的数据库对象。
- DROP: 删除数据库对象。
- TRUNCATE: 快速删除表中的所有数据，并重置表结构，但保留表本身。
## 5. DCL
Data Control Language，即数据控制语言，

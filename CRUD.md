## 1. 删除创建数据库
### 1.1 创建数据库
```SQL
CREATE DATABASE `数据库名` DEFAULT CHARACTER SET utf8; #字符集设置为utf8
```
### 1.2 删除数据库
```SQL
DROP DATABASE `数据库名`;
DROP DATABASE IF EXISTS `数据库名`;
```
### 1.3 选择数据库
```SQL
USE `数据库名`;
```
## 2. 删除创建表
### 2.1  创建表
```SQL
CREATE TABLE `table_name` (
    column1 datatype constraints,
    column2 datatype constraints,
    column3 datatype constraints,
    ...
);
```
```SQL
CREATE TABLE IF NOT EXISTS  `employees` (
    `id` INT UNSIGNED AUTO_INCREMENT COMMENT '编号',
    `course` VARCHAR(100) NOT NULL COMMENT '课程',
    `teacher` VARCHAR(100) NOT NULL COMMENT '讲师',
    `price` DECIMAL(8,2) NOT NULL COMMENT '价格',
    PRIMARY KEY (`id`)
)ENGINE=innodb DEFAULT CHARSET=utf8mb4 COMMENT='课程表';
```
#### constraints（约束）
是用来限制表中的数据，以确保数据的准确性和完整性
- NOT NULL 约束强制一列中的数据不能为 NULL（空值）
- UNIQUE 约束确保一列中的所有值都是唯一的
- PRIMARY KEY（主键）是表中唯一标识每一行的列或一组列。它实际上是 NOT NULL 和 UNIQUE 的组合
- FOREIGN KEY（外键）是用于在两个表之间建立链接的键。它指向另一个表中的 PRIMARY KEY
- DEFAULT 约束用于为一列设置默认值。如果 INSERT 语句没有为该列提供值，则会自动使用默认值
- CHECK 约束用于确保一列中的值满足特定的条件：age INT CHECK (age >= 18)
- AUTO_INCREMENT（或 SQL Server 中的 IDENTITY）约束会自动为新插入的行生成一个唯一的、递增的整数值
### 2.2  删除表
```SQL
DROP TABLE `table_name`;
DROP FROM `table_name`;
```
DDL语言，不能回滚，同时删除数据和表
### 2.3 清空数据表
```SQL
TRUNCATE TABLE table_name;
```
```SQL
DELETE TABLE table_name;
```
#### 执行方式与速度
- DELETE: 属于 DML (数据操作语言)。它会逐行删除数据，并且会为每一行记录删除操作，这个过程相对较慢
- TRUNCATE: 属于 DDL (数据定义语言)。它会先删除整个表，然后重建一个一模一样的空表。这个过程非常快，因为它不涉及逐行删除<br>
#### 事务与回滚
- DELETE支持事务,可以通过 ROLLBACK 命令撤销;
- TRUNCATE不支持事务
#### 自增列 (AUTO_INCREMENT)
- DELETE: 删除数据后，自增列的计数器不会重置。新插入的数据会接着之前的计数继续递增。
- TRUNCATE: 删除数据后，自增列的计数器会重置为初始值（通常是 1）
#### 锁定
- DELETE: 默认是行级锁
- TRUNCATE: 是表级锁
## 3. 增
```SQL
INSERT INTO `table_name` (`field1`, `field2`, ...,`fieldn`) VALUES(value1, value2, ..., valuen);
```
## 4. 改
```SQL
UPDATE `table_name` SET field1=value1, field2=value2, ...,fieldn=valuen;
```
```SQL
UPDATE `employees` SET `name` = '张三', age = 30 WHERE id = 1;
```
```SQL
UPDATE `employees` SET `name` = '张三', `age` = `age` + 1 WHERE id = 2;
```

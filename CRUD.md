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
### 2.2  删除表

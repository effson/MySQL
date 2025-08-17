## 1.MySQL配置
### 添加配置
```bash
root@worker02:/home/jeff# vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
添加：
```
server-id                       = 1
log_bin                         = mysql-bin
binlog_format                   = ROW
```
### 添加用户账户
```mysql
CREATE USER IF NOT EXISTS 'gmt'@'127.0.0.1'
  IDENTIFIED WITH mysql_native_password BY 'StrongPass#2025';
```



## go_mysql_transfer

<img width="607" height="229" alt="image" src="https://github.com/user-attachments/assets/46ac5815-8460-4c48-b3a9-569dc3c6626f" />


### 安装
git clone https://github.com/wj596/go-mysql-transfer.git

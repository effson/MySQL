## MySQL配置
```bash
root@worker02:/home/jeff# vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
添加：
```
server-id               = 1
#log_bin                        = /var/log/mysql/mysql-bin.log
log_bin                         = mysql-bin
binlog_format = ROW
```

## go_mysql_transfer

<img width="607" height="229" alt="image" src="https://github.com/user-attachments/assets/46ac5815-8460-4c48-b3a9-569dc3c6626f" />


### 安装
git clone https://github.com/wj596/go-mysql-transfer.git

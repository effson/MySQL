## 1.MySQL配置
```bash
root@worker02:/home/jeff# mysql -ugmt -p'StrongPass#2025' -h127.0.0.1 -P3306
root@worker02:/home/jeff# mysql -uroot -p'RootStrong#2025'
```
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
### 添加测试账户
```mysql
CREATE USER IF NOT EXISTS 'gmt'@'127.0.0.1'
  IDENTIFIED WITH mysql_native_password BY 'StrongPass#2025';

FLUSH PRIVILEGES;
```

## go_mysql_transfer安装与使用

<img width="607" height="229" alt="image" src="https://github.com/user-attachments/assets/46ac5815-8460-4c48-b3a9-569dc3c6626f" />


### 安装
git clone https://github.com/wj596/go-mysql-transfer.git

### 更新、安装依赖包
```bash
go mod tidy
```

```bash
go get -u github.com/json-iterator/go@v1.1.12
go get -u github.com/modern-go/reflect2@v1.0.2
```
```bash
go build
```
### 放回缺失的静态文件
```bash
cd /home/jeff/go-mysql-transfer

mkdir -p statics

cat > statics/index.html <<'HTML'
<!doctype html><meta charset="utf-8">
<title>go-mysql-transfer</title>
<body>go-mysql-transfer is running.</body>
HTML
```
### app.yml
```bash
# mysql配置
addr: 127.0.0.1:3306
user: "gmt"
pass: "StrongPass#2025"
charset : utf8
slave_id: 1001 #slave ID
flavor: mysql #mysql or mariadb,默认mysql

#web admin相关配置
enable_web_admin: true #是否启用web admin，默认false
web_admin_port: 8060 #web监控端口,默认8060

#目标类型
target: redis # 支持redis、mongodb、elasticsearch、rocketmq、kafka、rabbitmq

#redis连接配置
redis_addrs: 127.0.0.1:6379 #redis地址，多个用逗号分隔
#redis_group_type: cluster   # 集群类型 sentinel或者cluster
#redis_master_name: mymaster # Master节点名称,如果group_type为sentinel则此项不能为空，为cluster此项无效
redis_pass: 123321 #redis密码
#redis_database: 0  #redis数据库 0-16,默认0。如果group_type为cluster此项无效

rule:
  -
    schema: db1 #数据库名称
    table: users #表名称
    order_by_column: id #排序字段，存量数据同步时不能为空
    #column_lower_case:false #列名称转为小写,默认为false
    #column_upper_case:false#列名称转为大写,默认为false
    column_underscore_to_camel: true #列名称下划线转驼峰,默认为false
    # 包含的列，多值逗号分隔，如：id,name,age,area_id  为空时表示包含全部列
    #include_columns: ID,USER_NAME,PASSWORD
    #exclude_columns: BIRTHDAY,MOBIE # 排除掉的列，多值逗号分隔，如：id,name,age,area_id  默认为空
    #column_mappings: USER_NAME=account    #列名称映射，多个映射关系用逗号分隔，如：USER_NAME=account 表示将字段名USER_NAME映射为account
    #default_column_values: area_name=合肥  #默认的列-值，多个用逗号分隔，如：source=binlog,area_name=合肥
    #date_formatter: yyyy-MM-dd #date类型格式化， 不填写默认yyyy-MM-dd
    #datetime_formatter: yyyy-MM-dd HH:mm:ss #datetime、timestamp类型格式化，不填写默认yyyy-MM-dd HH:mm:ss
    #lua_file_path: lua/t_user.lua   #lua脚本文件
    #lua_script:   #lua 脚本
    value_encoder: json  #值编码，支持json、kv-commas、v-commas；默认为json
    #value_formatter: '{{.ID}}|{{.USER_NAME}}' # 值格式化表达式，如：{{.ID}}|{{.USER_NAME}},{{.ID}}表示ID字段的值、{{.USER_NAME}}表示USER_NAME字段的值

    #redis相关
    redis_structure: string # 数据类型。 支持string、hash、list、set、sortedset类型(与redis的数据类型一致)
    redis_key_prefix: "user:" #key的前缀
    redis_key_column: username #使用哪个列的值作为key，不填写默认使用主键
```
### 全量＋增量
```bash
root@worker02:/home/jeff/go-mysql-transfer# ./go-mysql-transfer -stock
2025-08-17 09:11:13.105599 I | process id: 12263
2025-08-17 09:11:13.105625 I | GOMAXPROCS :8
2025-08-17 09:11:13.105629 I | source  mysql(127.0.0.1:3306)
2025-08-17 09:11:13.105636 I | destination redis(127.0.0.1:6379)
2025-08-17 09:11:13.111817 I | bulk size: 100
2025-08-17 09:11:13.111841 I | 开始导出 db1.users
2025-08-17 09:11:13.112145 I | db1.users 共 3 条数据
2025-08-17 09:11:13.113591 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.113615 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.113795 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.113999 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.114157 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.114482 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.114734 I | db1.users 导入数据 3 条
2025-08-17 09:11:13.114944 I | db1.users 导入数据 3 条
共耗时 ：3（毫秒）
表： db1.users，共：3 条数据，成功导入：3 条
root@worker02:/home/jeff/go-mysql-transfer# ./go-mysql-transfer -stock -config app.yml
2025-08-17 09:14:14.498511 I | process id: 12323
2025-08-17 09:14:14.498533 I | GOMAXPROCS :8
2025-08-17 09:14:14.498536 I | source  mysql(127.0.0.1:3306)
2025-08-17 09:14:14.498541 I | destination redis(127.0.0.1:6379)
2025-08-17 09:14:14.503439 I | bulk size: 100
2025-08-17 09:14:14.503499 I | 开始导出 db1.users
2025-08-17 09:14:14.503759 I | db1.users 共 3 条数据
2025-08-17 09:14:14.504490 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.504554 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.504732 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.504872 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.505056 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.505271 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.505462 I | db1.users 导入数据 3 条
2025-08-17 09:14:14.505673 I | db1.users 导入数据 3 条
共耗时 ：2（毫秒）
表： db1.users，共：3 条数据，成功导入：3 条
```
redis结果：
```bash
127.0.0.1:6379> keys *
1) "user:alice"
2) "user:carl"
3) "user:bob"
```

### 测一条数据
```sql
INSERT INTO db1.users (username,email,age,status) VALUES ('swan','dora@example.com',30,1);
```
redis结果：
```bash
127.0.0.1:6379> keys *
1) "user:swan"
2) "user:alice"
3) "user:carl"
4) "user:bob"
```

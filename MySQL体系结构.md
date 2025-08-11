## 1. 总体架构图
<img width="763" height="756" alt="image" src="https://github.com/user-attachments/assets/8ec1502c-037f-4c70-8461-3290acb73119" />

## 2. server层
MySQL 的 Server 层 是 MySQL 体系结构中的核心部分，主要负责解析 SQL、优化执行计划、管理连接与权限等“数据库的大脑”功能，它不直接处理底层存储数据（那是存储引擎层的工作），而是完成SQL 从文本到结果的全过程管理
### 2.1 连接管理（Connection Management）
#### 管理客户端与 MySQL 之间的连接
- 处理 TCP/IP 或 Socket 连接请求
- 验证用户身份（用户名、密码、Host 等）
- 建立会话上下文（Session Context），保存当前连接的状态（事务、变量等）

### 2.2 SQL 解析（SQL Parser）
#### 将客户端发送的 SQL 文本解析成 MySQL 能理解的数据结构（语法解析树）
- 预处理：检查 SQL 中的表名、列名是否存在，校验权限等
- 语法解析：将 SQL 转换成语法解析树（Parse Tree）

### 2.3 查询优化（Query Optimizer）
#### 根据解析树生成高效的执行计划
- 选择合适的索引（Index Selection）
- 选择 Join 顺序与方式（Nested Loop、Hash Join 等）
- 决定使用何种访问方法（全表扫描、索引扫描、范围扫描等）
## 3. 引擎层

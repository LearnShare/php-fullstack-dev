# 1.2.2 客户端工具与命令

## 概述

选择合适的客户端工具和掌握常用命令对于 MySQL 管理至关重要。本章介绍常用的客户端工具和命令。

## 客户端工具

### 命令行工具

| 工具 | 适用场景 | 特点 |
| :--- | :------- | :--- |
| MySQL CLI (`mysql`) | 快速执行 SQL、编写脚本 | 配合 `history`、`tee` 保留执行记录 |
| `mysqldump` | 数据库备份 | 逻辑备份，适合小到中等数据库 |
| `mysqladmin` | 管理操作 | 查看状态、关闭服务等 |
| `mysql_config_editor` | 凭据管理 | 安全保存登录信息 |

### 图形化工具

| 工具 | 适用场景 | 特点 |
| :--- | :------- | :--- |
| TablePlus | 可视化管理、ER 图 | 支持 SSH Tunnel、连接模板、数据比对 |
| DataGrip | 专业数据库 IDE | JetBrains 出品，功能强大 |
| DBeaver | 跨数据库教学场景 | 内置 ER 反向工程、数据导入导出向导 |
| MySQL Workbench | 新手向、图形化建模 | 提供查询面板、性能监控图表 |

> 新手可先使用 GUI 观察数据库结构，再对照 CLI 输出记忆命令。

## 常用命令

### mysql - 客户端连接

**语法**：`mysql -h host -P port -u user -p`

**关键参数**：
- `-h`：主机地址
- `-P`：端口号
- `-u`：用户名
- `-p`：提示输入密码
- `-e`：直接执行 SQL 语句
- `--default-character-set=utf8mb4`：显式设置客户端编码

**示例**：

```bash
# 基本连接
mysql -uroot -p

# 指定主机和端口
mysql -h127.0.0.1 -P3306 -uroot -p

# 直接执行 SQL
mysql -uroot -p -e "SHOW DATABASES;"

# 执行 SQL 文件
mysql -uroot -p < script.sql

# 设置字符集
mysql -uroot -p --default-character-set=utf8mb4
```

### mysqldump - 数据库备份

**语法**：`mysqldump [options] db [tables]`

**关键参数**：
- `--single-transaction`：避免锁表，适合 InnoDB
- `--routines`：包含存储过程与函数
- `--triggers`：包含触发器
- `--events`：包含事件
- `--set-gtid-purged=OFF`：开启 GTID 时必备
- `--no-data`：只导出结构，不导出数据
- `--where`：条件导出

**示例**：

```bash
# 备份整个数据库
mysqldump --single-transaction demo > backup.sql

# 备份特定表
mysqldump --single-transaction demo users posts > backup.sql

# 备份结构和数据
mysqldump --single-transaction --routines --triggers demo > backup.sql

# 只备份结构
mysqldump --no-data demo > structure.sql

# 条件备份
mysqldump --single-transaction --where="created_at > '2024-01-01'" demo users > recent_users.sql
```

### mysql_config_editor - 凭据管理

**语法**：`mysql_config_editor set --login-path=name --user=user --host=host --password`

**关键参数**：
- `set`：设置登录信息
- `print`：查看已保存的登录信息
- `remove`：删除指定 login-path

**示例**：

```bash
# 保存登录信息
mysql_config_editor set \
  --login-path=local \
  --user=demo_app \
  --host=127.0.0.1 \
  --password

# 查看保存的信息
mysql_config_editor print --all

# 使用保存的登录信息
mysql --login-path=local

# 删除登录信息
mysql_config_editor remove --login-path=local
```

### mysqladmin - 管理命令

**语法**：`mysqladmin -u user -p command`

**常用命令**：
- `status`：查看当前连接、运行时间、吞吐量
- `variables`：导出运行时变量
- `processlist`：查看进程列表
- `shutdown`：优雅关闭实例（需权限）
- `ping`：检查服务器是否运行

**示例**：

```bash
# 查看状态
mysqladmin -uroot -p status

# 查看变量
mysqladmin -uroot -p variables

# 查看进程
mysqladmin -uroot -p processlist

# 检查服务器
mysqladmin -uroot -p ping
```

## SQL 常用命令

### 数据库操作

```sql
-- 创建数据库
CREATE DATABASE demo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 删除数据库
DROP DATABASE demo;

-- 查看所有数据库
SHOW DATABASES;

-- 使用数据库
USE demo;
```

### 表操作

```sql
-- 创建表
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 查看表结构
DESCRIBE users;
SHOW CREATE TABLE users;

-- 查看所有表
SHOW TABLES;
```

### 用户和权限

```sql
-- 创建用户
CREATE USER 'app_user'@'%' IDENTIFIED BY 'password';

-- 授权
GRANT SELECT, INSERT, UPDATE, DELETE ON demo.* TO 'app_user'@'%';

-- 查看权限
SHOW GRANTS FOR 'app_user'@'%';

-- 撤销权限
REVOKE DELETE ON demo.* FROM 'app_user'@'%';

-- 刷新权限
FLUSH PRIVILEGES;
```

## 常见问题排查

### 端口被占用

```bash
# Windows
netstat -ano | findstr 3306

# macOS/Linux
lsof -iTCP:3306

# 或修改 MySQL 端口为 3307
```

### 字符集不一致

```sql
-- 查看字符集配置
SHOW VARIABLES LIKE 'character_set%';
SHOW VARIABLES LIKE 'collation%';

-- 修改字符集
ALTER DATABASE demo CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 权限错误

```sql
-- 查看用户权限
SHOW GRANTS FOR 'app_user'@'%';

-- 重新授权
GRANT ALL PRIVILEGES ON demo.* TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

### 连接拒绝

```bash
# 检查 bind-address
# 在 my.cnf 中设置
bind-address = 0.0.0.0

# Docker 场景需映射端口
docker run -p 3306:3306 mysql:8.0
```

### 慢查询

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- 查看慢查询
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;
```

## 完整示例

### 管理脚本

```bash
#!/bin/bash
# mysql-admin.sh

# 备份数据库
backup() {
    mysqldump --single-transaction --routines demo > "backup_$(date +%Y%m%d_%H%M%S).sql"
    echo "Backup completed"
}

# 恢复数据库
restore() {
    if [ -z "$1" ]; then
        echo "Usage: restore <backup_file.sql>"
        return 1
    fi
    mysql demo < "$1"
    echo "Restore completed"
}

# 查看状态
status() {
    mysqladmin -uroot -p status
}

# 使用
case "$1" in
    backup)
        backup
        ;;
    restore)
        restore "$2"
        ;;
    status)
        status
        ;;
    *)
        echo "Usage: $0 {backup|restore|status}"
        exit 1
        ;;
esac
```

## 注意事项

1. **安全配置**：生产环境必须配置安全策略，禁用 root 远程访问。

2. **备份策略**：建立定期备份机制，测试恢复流程。

3. **字符集**：统一使用 `utf8mb4`，避免字符集问题。

4. **性能监控**：定期检查慢查询日志，优化性能。

5. **文档记录**：记录常用命令和配置，便于团队协作。

## 练习

1. 使用命令行工具连接 MySQL，执行基本操作。

2. 使用 `mysqldump` 备份和恢复数据库。

3. 配置 `mysql_config_editor`，安全保存登录信息。

4. 使用图形化工具（TablePlus 或 DBeaver）管理数据库。

5. 排查常见的连接和权限问题。

# 1.3.2 客户端工具与命令

## 概述

选择合适的客户端工具和掌握常用命令是高效管理 MySQL 数据库的关键。本节介绍命令行工具和图形化工具的使用，以及常用的 MySQL 命令和 SQL 命令。

## 客户端工具选择

### 命令行工具

**优点**：
- 适合脚本自动化
- 服务器管理方便
- 批量操作高效

**缺点**：
- 学习曲线较陡
- 不适合复杂查询

### 图形化工具

**优点**：
- 界面友好，易于使用
- 可视化数据查看
- 表结构设计方便

**缺点**：
- 需要图形界面
- 不适合服务器环境

## 命令行工具

### mysql

**连接数据库**：

```bash
mysql -u username -p
```

**执行 SQL 文件**：

```bash
mysql -u username -p database < file.sql
```

**导出数据**：

```bash
mysql -u username -p -e "SELECT * FROM table" database
```

### mysqldump

**备份数据库**：

```bash
# 备份单个数据库
mysqldump -u username -p database > backup.sql

# 备份所有数据库
mysqldump -u username -p --all-databases > all_backup.sql

# 只备份结构
mysqldump -u username -p --no-data database > structure.sql

# 只备份数据
mysqldump -u username -p --no-create-info database > data.sql
```

**恢复数据库**：

```bash
mysql -u username -p database < backup.sql
```

### mysql_config_editor

**保存登录凭据**：

```bash
mysql_config_editor set --login-path=local --host=localhost --user=root --password
```

**使用保存的凭据**：

```bash
mysql --login-path=local
```

### mysqladmin

**检查服务器状态**：

```bash
mysqladmin -u root -p status
```

**创建数据库**：

```bash
mysqladmin -u root -p create database_name
```

**删除数据库**：

```bash
mysqladmin -u root -p drop database_name
```

## 常用 SQL 命令

### 数据库操作

```sql
-- 创建数据库
CREATE DATABASE testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 删除数据库
DROP DATABASE testdb;

-- 显示所有数据库
SHOW DATABASES;

-- 选择数据库
USE testdb;
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

-- 删除表
DROP TABLE users;

-- 修改表结构
ALTER TABLE users ADD COLUMN age INT;

-- 显示所有表
SHOW TABLES;

-- 显示表结构
DESCRIBE users;
-- 或
SHOW CREATE TABLE users;
```

### 数据操作

```sql
-- 查询数据
SELECT * FROM users;
SELECT id, name FROM users WHERE id = 1;

-- 插入数据
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');

-- 更新数据
UPDATE users SET name = 'Jane' WHERE id = 1;

-- 删除数据
DELETE FROM users WHERE id = 1;
```

### 用户和权限

```sql
-- 创建用户
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'password';

-- 授权
GRANT SELECT, INSERT, UPDATE, DELETE ON testdb.* TO 'appuser'@'localhost';

-- 撤销权限
REVOKE DELETE ON testdb.* FROM 'appuser'@'localhost';

-- 显示用户权限
SHOW GRANTS FOR 'appuser'@'localhost';

-- 刷新权限
FLUSH PRIVILEGES;
```

## 图形化工具

### phpMyAdmin

**特点**：
- Web 界面，易于使用
- 适合初学者
- 功能全面

**安装**（Docker）：

```bash
docker run -d \
  --name phpmyadmin \
  -e PMA_HOST=mysql \
  -e PMA_PORT=3306 \
  -p 8080:80 \
  phpmyadmin/phpmyadmin
```

### MySQL Workbench

**特点**：
- 官方工具，功能强大
- 支持数据库设计和管理
- 跨平台支持

**下载**：https://dev.mysql.com/downloads/workbench/

### DBeaver

**特点**：
- 开源工具，支持多种数据库
- 功能丰富，界面友好
- 跨平台支持

**下载**：https://dbeaver.io/

### TablePlus

**特点**：
- 现代化界面
- 支持多种数据库
- 付费工具，有免费版本

**下载**：https://tableplus.com/

## 完整示例

### 示例 1：使用命令行工具

```bash
# 1. 连接数据库
$ mysql -u root -p
Enter password: 

# 2. 创建数据库
mysql> CREATE DATABASE testdb CHARACTER SET utf8mb4;
Query OK, 1 row affected (0.01 sec)

# 3. 使用数据库
mysql> USE testdb;
Database changed

# 4. 创建表
mysql> CREATE TABLE users (
    -> id INT AUTO_INCREMENT PRIMARY KEY,
    -> name VARCHAR(100) NOT NULL,
    -> email VARCHAR(100) UNIQUE
    -> );
Query OK, 0 rows affected (0.02 sec)

# 5. 插入数据
mysql> INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
Query OK, 1 row affected (0.01 sec)

# 6. 查询数据
mysql> SELECT * FROM users;
+----+------+------------------+
| id | name | email            |
+----+------+------------------+
|  1 | John | john@example.com |
+----+------+------------------+
1 row in set (0.00 sec)
```

### 示例 2：使用 mysqldump 备份

```bash
# 备份数据库
$ mysqldump -u root -p testdb > backup.sql
Enter password: 

# 查看备份文件
$ head -20 backup.sql
-- MySQL dump 10.13  Distrib 8.0.35
-- Host: localhost    Database: testdb
-- ------------------------------------------------------
-- Server version	8.0.35

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
...

# 恢复数据库
$ mysql -u root -p testdb < backup.sql
Enter password: 
```

### 示例 3：使用 mysql_config_editor

```bash
# 保存登录凭据
$ mysql_config_editor set --login-path=local --host=localhost --user=root --password
Enter password: 

# 使用保存的凭据连接
$ mysql --login-path=local
Welcome to the MySQL monitor.
```

## 使用场景

- **命令行工具**：适合脚本自动化、服务器管理、批量操作
- **图形化工具**：适合日常开发、数据查看、表结构设计
- **mysqldump**：适合数据库备份和迁移
- **mysqladmin**：适合服务器管理和监控

## 注意事项

- **密码安全**：不要在命令行中直接输入密码，使用 `-p` 参数交互输入
- **权限管理**：确保用户有足够的权限执行操作
- **备份策略**：定期备份，测试恢复流程
- **工具选择**：根据场景选择合适的工具

## 常见问题

### 问题 1：无法连接到数据库

**原因**：用户名密码错误、权限不足、服务未启动

**解决方法**：

```bash
# 检查服务状态
sudo systemctl status mysql

# 检查用户权限
mysql> SHOW GRANTS FOR 'user'@'localhost';
```

### 问题 2：mysqldump 备份失败

**原因**：权限不足、磁盘空间不足

**解决方法**：

```bash
# 检查磁盘空间
df -h

# 检查权限
mysql> SHOW GRANTS FOR 'user'@'localhost';
```

### 问题 3：图形化工具连接超时

**原因**：网络问题、防火墙阻止、MySQL 配置

**解决方法**：

```bash
# 检查网络连接
ping mysql_host

# 检查防火墙
sudo ufw status

# 检查 MySQL 远程连接配置
mysql> SELECT user, host FROM mysql.user;
```

## 最佳实践

- 掌握命令行工具，便于脚本自动化
- 使用图形化工具提高开发效率
- 使用 `mysql_config_editor` 保存常用连接
- 定期备份数据库，测试恢复流程
- 记录常用命令，建立命令库

## 相关章节

- **阶段六：数据库与缓存系统**：深入学习 PDO、ORM、数据库操作等详细内容

## 练习任务

1. 使用 `mysql` 命令行工具连接数据库，并执行基本的 SQL 命令。
2. 使用 `mysqldump` 备份数据库，并测试恢复流程。
3. 使用 `mysql_config_editor` 保存登录凭据，并使用保存的凭据连接。
4. 安装并配置一个图形化工具（如 phpMyAdmin 或 DBeaver），使用图形化工具管理数据库。
5. 创建一个数据库、表，并执行基本的增删改查操作。

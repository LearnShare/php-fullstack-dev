# 1.2.1 MySQL 安装与配置

## 概述

MySQL 是最流行的关系型数据库之一。本章介绍 MySQL 8.0+ 的安装和配置方法。

## 版本与发行版

### 推荐版本

- **MySQL Community 8.0**：默认选项，提供 InnoDB、GTID 复制
- **Percona Server / MariaDB**：仅在明确需求（XtraBackup、兼容 MySQL 5.7）时使用
- 建议统一 `utf8mb4` 字符集、`InnoDB` 存储引擎

## 安装方式

### Docker 方式（推荐）

#### 基本安装

```bash
docker run -d \
  --name mysql8 \
  -e MYSQL_ROOT_PASSWORD=secret \
  -p 3306:3306 \
  -v ./mysql-data:/var/lib/mysql \
  mysql:8.0
```

#### 完整配置

```bash
docker run -d \
  --name mysql8 \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  -e MYSQL_USER=app_user \
  -e MYSQL_PASSWORD=app_password \
  -p 3306:3306 \
  -v ./mysql-data:/var/lib/mysql \
  -v ./my.cnf:/etc/mysql/conf.d/custom.cnf \
  mysql:8.0
```

#### 使用 docker-compose

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:-root}
      MYSQL_DATABASE: ${DB_DATABASE:-myapp}
      MYSQL_USER: ${DB_USERNAME:-user}
      MYSQL_PASSWORD: ${DB_PASSWORD:-password}
    ports:
      - "${DB_PORT:-3306}:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf

volumes:
  mysql-data:
```

### 本地安装

#### Windows

1. 访问 [MySQL 官网](https://dev.mysql.com/downloads/mysql/)，下载 Windows 安装器
2. 运行安装器，选择 "Developer Default"
3. 在服务面板中启动 `MySQL80` 服务

#### macOS

```bash
# 使用 Homebrew
brew install mysql@8.0

# 启动服务
brew services start mysql@8.0

# 第一次启动会提示默认 root 密码，建议立即修改
mysql_secure_installation
```

#### Linux

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install mysql-server

# 安全设置
sudo mysql_secure_installation

# 启动服务
sudo systemctl start mysql
sudo systemctl enable mysql
```

## 配置

### 字符集配置

```ini
# my.cnf
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4
```

### 基本配置

```ini
# my.cnf
[mysqld]
# 字符集
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# 连接
max_connections=200
max_connect_errors=10

# 缓冲区
innodb_buffer_pool_size=1G
innodb_log_file_size=256M

# 日志
slow_query_log=1
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2
```

## 第一个数据库

### 创建数据库

```sql
-- 创建数据库并设置字符集
CREATE DATABASE demo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 查看数据库
SHOW DATABASES;

-- 使用数据库
USE demo;
```

### 创建用户

```sql
-- 创建应用账号
CREATE USER 'demo_app'@'%' IDENTIFIED BY 'P@ssw0rd!';

-- 授权
GRANT ALL PRIVILEGES ON demo.* TO 'demo_app'@'%';

-- 刷新权限
FLUSH PRIVILEGES;
```

### 保存脚本

将以上命令保存到 `docs/sql/init-demo.sql`，便于日后复用。

## 连接安全

### 安全建议

1. **禁用 root 远程访问**
   ```sql
   -- 删除 root 远程访问
   DROP USER 'root'@'%';
   ```

2. **强制 TLS**
   ```sql
   -- 开启 TLS
   ALTER USER 'app_user'@'%' REQUIRE SSL;
   ```

3. **使用 mysql_config_editor**
   ```bash
   mysql_config_editor set \
     --login-path=local \
     --user=app_user \
     --host=127.0.0.1 \
     --password
   ```

## 数据目录与备份

### 数据目录位置

- **Docker**：`./mysql-data`（由 `-v` 参数决定）
- **Windows Installer**：`C:\ProgramData\MySQL\MySQL Server 8.0\Data\`
- **Homebrew**：`/opt/homebrew/var/mysql/`

### 备份

#### 逻辑备份

```bash
# 备份数据库
mysqldump --single-transaction --routines demo > backup.sql

# 恢复数据库
mysql demo < backup.sql
```

#### 物理备份

使用 `Percona XtraBackup` 或 `mydumper`，适合大数据量与在线备份。

## 完整示例

### 初始化脚本

```bash
#!/bin/bash
# init-mysql.sh

# 创建数据库
mysql -uroot -p <<EOF
CREATE DATABASE IF NOT EXISTS demo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'demo_app'@'%' IDENTIFIED BY 'P@ssw0rd!';
GRANT ALL PRIVILEGES ON demo.* TO 'demo_app'@'%';
FLUSH PRIVILEGES;
EOF

echo "Database initialized"
```

## 注意事项

1. **字符集**：统一使用 `utf8mb4`，支持完整的 Unicode。

2. **安全配置**：生产环境必须配置安全策略。

3. **备份策略**：建立定期备份机制。

4. **性能优化**：根据实际负载调整配置参数。

5. **版本兼容**：确保客户端和服务器版本兼容。

## 练习

1. 使用 Docker 安装 MySQL 8.0。

2. 创建数据库和用户，配置字符集。

3. 配置 MySQL 安全策略。

4. 实现数据库备份和恢复脚本。

5. 验证字符集配置是否正确。

# 1.3.1 MySQL 安装与配置

## 概述

MySQL 是最流行的关系型数据库之一。本节介绍 MySQL 8.0+ 的安装和配置方法，包括 Docker 方式（推荐）和本地安装方式。让零基础学员能够成功运行 MySQL，并理解每一步在做什么。

## 版本选择

**推荐版本**：

- **MySQL Community 8.0+**：默认选项，提供 InnoDB、GTID 复制
- **Percona Server / MariaDB**：仅在明确需求时使用
- 建议统一 `utf8mb4` 字符集、`InnoDB` 存储引擎

## 安装方法

### Docker 方式（推荐）

**基本安装**：

```bash
# 拉取 MySQL 镜像
docker pull mysql:8.0

# 运行 MySQL 容器
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  -p 3306:3306 \
  mysql:8.0
```

**使用 Docker Compose**：

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: testdb
      MYSQL_USER: testuser
      MYSQL_PASSWORD: testpass
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./my.cnf:/etc/mysql/conf.d/my.cnf

volumes:
  mysql_data:
```

启动：

```bash
docker-compose up -d
```

**验证安装**：

```bash
# 检查容器状态
docker ps | grep mysql

# 连接测试
docker exec -it mysql mysql -u root -p
```

### 本地安装

#### Windows

**安装步骤**：

1. 访问 MySQL 官网：https://dev.mysql.com/downloads/mysql/
2. 下载 MySQL 安装器（MySQL Installer）
3. 运行安装向导，选择 "Developer Default"
4. 设置 root 密码
5. 启动 MySQL 服务

**验证安装**：

```bash
mysql --version
```

#### macOS

**使用 Homebrew**：

```bash
# 安装 MySQL
brew install mysql@8.0

# 启动服务
brew services start mysql@8.0

# 验证安装
mysql --version
```

#### Linux（Debian/Ubuntu）

**安装步骤**：

```bash
# 更新包列表
sudo apt update

# 安装 MySQL
sudo apt install mysql-server

# 启动服务
sudo systemctl start mysql
sudo systemctl enable mysql

# 安全配置
sudo mysql_secure_installation

# 验证安装
mysql --version
```

## 基本配置

### 字符集配置

**设置默认字符集为 utf8mb4**：

编辑 MySQL 配置文件（`/etc/mysql/my.cnf` 或 `/etc/my.cnf`）：

```ini
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4
```

**重启 MySQL 服务**：

```bash
# Linux
sudo systemctl restart mysql

# macOS
brew services restart mysql@8.0

# Docker
docker-compose restart mysql
```

### 创建第一个数据库

**连接 MySQL**：

```bash
mysql -u root -p
```

**创建数据库**：

```sql
-- 创建数据库
CREATE DATABASE testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 创建用户
CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'testpass';

-- 授权
GRANT ALL PRIVILEGES ON testdb.* TO 'testuser'@'localhost';
FLUSH PRIVILEGES;

-- 使用数据库
USE testdb;
```

**验证**：

```sql
-- 查看数据库
SHOW DATABASES;

-- 查看用户
SELECT user, host FROM mysql.user;
```

## 连接安全

### 用户管理

**创建专用用户**：

```sql
-- 创建应用用户（不要使用 root）
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'strong_password';

-- 授予最小权限
GRANT SELECT, INSERT, UPDATE, DELETE ON testdb.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;
```

### 远程连接配置

**允许远程连接**（谨慎使用）：

```sql
-- 创建远程用户
CREATE USER 'remoteuser'@'%' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON testdb.* TO 'remoteuser'@'%';
FLUSH PRIVILEGES;
```

**配置防火墙**：

```bash
# 允许 MySQL 端口（3306）
sudo ufw allow 3306/tcp
```

## 数据目录与备份

### 数据目录位置

**查看数据目录**：

```sql
SHOW VARIABLES LIKE 'datadir';
```

**常见位置**：

- Linux：`/var/lib/mysql`
- macOS：`/usr/local/var/mysql`
- Windows：`C:\ProgramData\MySQL\MySQL Server 8.0\Data`
- Docker：`/var/lib/mysql`

### 备份方法

**使用 mysqldump 备份**：

```bash
# 备份单个数据库
mysqldump -u root -p testdb > backup.sql

# 备份所有数据库
mysqldump -u root -p --all-databases > all_backup.sql

# 只备份结构
mysqldump -u root -p --no-data testdb > structure.sql
```

**恢复数据库**：

```bash
mysql -u root -p testdb < backup.sql
```

## 完整示例

### 示例 1：Docker 安装和配置

```bash
# 1. 创建 docker-compose.yml
cat > docker-compose.yml << EOF
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: myapp
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
volumes:
  mysql_data:
EOF

# 2. 启动 MySQL
docker-compose up -d

# 3. 连接测试
docker exec -it mysql mysql -u root -prootpass

# 4. 创建数据库和用户
mysql> CREATE DATABASE myapp CHARACTER SET utf8mb4;
mysql> CREATE USER 'appuser'@'%' IDENTIFIED BY 'apppass';
mysql> GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'%';
mysql> FLUSH PRIVILEGES;
```

### 示例 2：本地安装验证

```bash
# 检查 MySQL 版本
$ mysql --version
mysql  Ver 8.0.35 for Linux on x86_64

# 连接 MySQL
$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.

# 查看数据库
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

## 注意事项

- **密码安全**：设置强密码，不要使用默认密码
- **数据持久化**：使用 Docker 时注意数据卷配置
- **字符集**：统一使用 `utf8mb4`，避免中文乱码
- **权限管理**：不要使用 root 用户连接应用，创建专用用户
- **备份策略**：定期备份数据库，测试恢复流程

## 常见问题

### 问题 1：无法连接到 MySQL

**原因**：服务未启动、端口被占用、防火墙阻止

**解决方法**：

```bash
# 检查服务状态
sudo systemctl status mysql

# 检查端口占用
netstat -an | grep 3306

# 检查防火墙
sudo ufw status
```

### 问题 2：字符集乱码

**原因**：字符集配置不正确

**解决方法**：

```sql
-- 检查数据库字符集
SHOW CREATE DATABASE testdb;

-- 修改数据库字符集
ALTER DATABASE testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 问题 3：权限不足

**原因**：用户权限配置不正确

**解决方法**：

```sql
-- 查看用户权限
SHOW GRANTS FOR 'testuser'@'localhost';

-- 重新授权
GRANT ALL PRIVILEGES ON testdb.* TO 'testuser'@'localhost';
FLUSH PRIVILEGES;
```

### 问题 4：Docker 容器数据丢失

**原因**：未配置数据卷

**解决方法**：在 `docker-compose.yml` 中配置数据卷：

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

## 最佳实践

- 使用 Docker 方式安装，便于环境管理
- 统一使用 `utf8mb4` 字符集和 `InnoDB` 存储引擎
- 创建专用数据库用户，不要使用 root
- 配置定期备份策略
- 记录安装和配置步骤，便于环境复现

## 相关章节

- **阶段六：数据库与缓存系统**：深入学习数据库操作、PDO、ORM 等详细内容

## 练习任务

1. 使用 Docker 方式安装 MySQL，并创建测试数据库。
2. 配置 MySQL 字符集为 `utf8mb4`，并验证配置。
3. 创建一个应用用户，授予最小权限，并使用该用户连接数据库。
4. 使用 `mysqldump` 备份数据库，并测试恢复流程。
5. 查看 MySQL 数据目录位置，了解数据存储结构。

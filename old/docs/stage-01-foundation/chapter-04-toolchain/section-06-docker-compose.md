# 1.3.6 容器与环境管理

## 概述

Docker 和 Docker Compose 可以帮助我们快速搭建一致的开发环境，避免"在我机器上能跑"的问题。

## Docker 基础

### 安装

#### macOS

```bash
# 使用 Homebrew
brew install --cask docker

# 或下载 Docker Desktop
# https://www.docker.com/products/docker-desktop
```

#### Linux

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install docker.io docker-compose

# 启动服务
sudo systemctl start docker
sudo systemctl enable docker
```

### 验证安装

```bash
docker --version
docker compose version
```

## Docker Compose

### 基本概念

- **服务（Service）**：一个容器实例
- **网络（Network）**：服务间的网络连接
- **数据卷（Volume）**：持久化存储

### 基本配置

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  php-fpm:
    image: php:8.2-fpm
    volumes:
      - .:/var/www/html
    working_dir: /var/www/html

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - .:/var/www/html
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - php-fpm

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

## 常用命令

### docker compose up

**语法**：`docker compose up [service] [-d] [--build]`

**参数**：
- `-d`：后台运行
- `--build`：在启动前重新构建镜像

**示例**：

```bash
# 启动所有服务
docker compose up

# 后台启动
docker compose up -d

# 启动特定服务
docker compose up php-fpm nginx

# 重新构建并启动
docker compose up --build
```

### docker compose down

**语法**：`docker compose down [--volumes] [--remove-orphans]`

**参数**：
- `--volumes`：连同数据卷一起删除
- `--remove-orphans`：删除未定义的服务

**示例**：

```bash
# 停止并删除容器
docker compose down

# 删除数据卷
docker compose down --volumes
```

### docker compose logs

**语法**：`docker compose logs [-f] [service]`

**参数**：
- `-f`：实时跟随输出
- `--tail <number>`：显示最后 N 行

**示例**：

```bash
# 查看所有日志
docker compose logs

# 实时查看
docker compose logs -f

# 查看特定服务
docker compose logs -f php-fpm

# 查看最后 100 行
docker compose logs --tail 100
```

### docker compose exec

**语法**：`docker compose exec [service] [command]`

**参数**：
- `-T`：关闭 pseudo-tty，适合 CI

**示例**：

```bash
# 进入容器
docker compose exec php-fpm bash

# 执行命令
docker compose exec php-fpm php -v

# CI 环境
docker compose exec -T php-fpm composer install
```

### docker compose ps

**语法**：`docker compose ps`

**说明**：查看运行中的服务。

**示例**：

```bash
docker compose ps
```

### docker compose restart

**语法**：`docker compose restart [service]`

**示例**：

```bash
# 重启所有服务
docker compose restart

# 重启特定服务
docker compose restart php-fpm
```

## 完整配置示例

### docker-compose.yml

```yaml
version: '3.8'

services:
  php-fpm:
    build:
      context: .
      dockerfile: Dockerfile.php
    volumes:
      - .:/var/www/html
    working_dir: /var/www/html
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - .:/var/www/html
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./docker/nginx/ssl:/etc/nginx/ssl
    depends_on:
      - php-fpm
    networks:
      - app-network

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
    networks:
      - app-network

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

  mailpit:
    image: axllent/mailpit
    ports:
      - "1025:1025"
      - "8025:8025"
    networks:
      - app-network

volumes:
  mysql-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### Dockerfile.php

```dockerfile
FROM php:8.2-fpm

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www/html

# 复制文件
COPY . .

# 安装依赖
RUN composer install --no-dev --optimize-autoloader

# 设置权限
RUN chown -R www-data:www-data /var/www/html
```

### nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## 环境变量

### .env 文件

创建 `.env` 文件：

```env
DB_ROOT_PASSWORD=root
DB_DATABASE=myapp
DB_USERNAME=user
DB_PASSWORD=password
DB_PORT=3306
```

### 在 docker-compose.yml 中使用

```yaml
services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
```

## 数据卷管理

### 创建数据卷

```bash
# 创建命名卷
docker volume create mysql-data

# 查看所有卷
docker volume ls

# 查看卷详情
docker volume inspect mysql-data
```

### 备份和恢复

```bash
# 备份 MySQL 数据
docker compose exec mysql mysqldump -u root -p myapp > backup.sql

# 恢复数据
docker compose exec -T mysql mysql -u root -p myapp < backup.sql
```

## 网络管理

### 创建网络

```bash
# 创建网络
docker network create app-network

# 查看网络
docker network ls

# 查看网络详情
docker network inspect app-network
```

### 在 docker-compose.yml 中使用

```yaml
services:
  php-fpm:
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

## 完整示例

### 项目结构

```
my-project/
├── docker-compose.yml
├── Dockerfile.php
├── .env
├── docker/
│   └── nginx/
│       └── nginx.conf
└── src/
    └── index.php
```

### 使用流程

1. **启动服务**
   ```bash
   docker compose up -d
   ```

2. **查看日志**
   ```bash
   docker compose logs -f
   ```

3. **进入容器**
   ```bash
   docker compose exec php-fpm bash
   ```

4. **执行命令**
   ```bash
   docker compose exec php-fpm composer install
   docker compose exec php-fpm php artisan migrate
   ```

5. **停止服务**
   ```bash
   docker compose down
   ```

## 注意事项

1. **数据持久化**：使用命名卷保存数据，避免容器删除后数据丢失。

2. **端口冲突**：确保端口不冲突，可以在 `.env` 中配置。

3. **性能优化**：使用 `:cached` 挂载选项（macOS）提高文件系统性能。

4. **资源限制**：为服务设置资源限制，避免占用过多资源。

5. **安全配置**：生产环境不要暴露敏感端口，使用内部网络。

## 练习

1. 创建一个完整的 docker-compose.yml，包含 PHP-FPM、Nginx、MySQL。

2. 配置数据卷，确保数据持久化。

3. 使用环境变量配置服务参数。

4. 创建 Dockerfile，自定义 PHP 镜像。

5. 配置网络，实现服务间通信。

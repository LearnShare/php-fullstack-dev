# 8.2.1 docker-compose 基础

## 概述

docker-compose 是定义和运行多容器 Docker 应用的工具。本节介绍 docker-compose 语法、服务定义、网络配置、数据卷等内容。

## docker-compose 语法

### 基础配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:80"
    environment:
      - APP_ENV=local
    depends_on:
      - db
  
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

## 服务定义

### PHP 应用服务

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: my-app:latest
    container_name: my-app
    restart: unless-stopped
    working_dir: /var/www/html
    volumes:
      - .:/var/www/html
      - ./storage:/var/www/html/storage
    environment:
      - APP_ENV=${APP_ENV:-local}
      - DB_HOST=db
    depends_on:
      - db
      - redis
```

### 数据库服务

```yaml
services:
  db:
    image: mysql:8.0
    container_name: my-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD:-root}
      MYSQL_DATABASE: ${DB_DATABASE:-myapp}
      MYSQL_USER: ${DB_USERNAME:-user}
      MYSQL_PASSWORD: ${DB_PASSWORD:-password}
    ports:
      - "${DB_PORT:-3306}:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./docker/mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## 网络配置

### 自定义网络

```yaml
services:
  app:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

## 数据卷

### 命名卷

```yaml
services:
  app:
    volumes:
      - app_data:/var/www/html/storage
  
  db:
    volumes:
      - db_data:/var/lib/mysql

volumes:
  app_data:
    driver: local
  db_data:
    driver: local
```

### 绑定挂载

```yaml
services:
  app:
    volumes:
      - .:/var/www/html
      - ./storage:/var/www/html/storage
```

## 完整示例

```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: my-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./public:/var/www/html/public
    depends_on:
      - app
    networks:
      - frontend

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: my-app
    restart: unless-stopped
    working_dir: /var/www/html
    volumes:
      - .:/var/www/html
    environment:
      - APP_ENV=${APP_ENV:-local}
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - frontend
      - backend

  db:
    image: mysql:8.0
    container_name: my-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD:-root}
      MYSQL_DATABASE: ${DB_DATABASE:-myapp}
    ports:
      - "${DB_PORT:-3306}:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    container_name: my-redis
    restart: unless-stopped
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - redis_data:/data
    networks:
      - backend

volumes:
  db_data:
  redis_data:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

## 最佳实践

1. **使用环境变量**：配置通过环境变量管理
2. **健康检查**：为服务配置健康检查
3. **依赖管理**：使用 depends_on 管理服务依赖
4. **网络隔离**：使用网络隔离服务

## 注意事项

1. 注意服务启动顺序
2. 合理使用数据卷
3. 配置健康检查
4. 使用环境变量文件

## 练习

1. 创建一个 docker-compose.yml，包含 PHP、MySQL、Redis。

2. 配置服务网络，实现网络隔离。

3. 使用数据卷持久化数据。

4. 配置健康检查和服务依赖。

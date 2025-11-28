# 8.2 docker-compose 与本地开发环境

## 目标

- 掌握 docker-compose 的配置和使用。
- 能够搭建完整的本地开发环境。
- 理解生产与开发环境的差异配置。

## docker-compose 基础

### 基础配置

```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/var/www
    depends_on:
      - db
      - redis
  
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql
  
  redis:
    image: redis:alpine
  
volumes:
  db_data:
```

## 完整开发环境

### docker-compose.yml

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./public:/var/www/public
    depends_on:
      - php
  
  php:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - .:/var/www
    depends_on:
      - db
      - redis
  
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
      MYSQL_USER: app
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
  
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
  
  mailpit:
    image: axllent/mailpit
    ports:
      - "1025:1025"
      - "8025:8025"

volumes:
  db_data:
  redis_data:
```

## 练习

1. 创建一个完整的 docker-compose 开发环境。

2. 配置不同环境的 compose 文件（开发、测试、生产）。

3. 实现服务的健康检查和自动重启。

4. 配置网络和卷管理。

5. 创建开发环境的初始化脚本。

6. 实现服务的日志收集和查看。

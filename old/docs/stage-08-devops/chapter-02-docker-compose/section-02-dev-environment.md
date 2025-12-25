# 8.2.2 本地开发环境

## 概述

使用 docker-compose 可以快速搭建完整的本地开发环境。本节介绍开发环境配置、热重载、调试配置、多服务编排等内容。

## 开发环境配置

### 开发专用配置

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/var/www/html
      - ./vendor:/var/www/html/vendor  # 挂载 vendor 目录
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
      - XDEBUG_MODE=debug
    ports:
      - "9000:9000"
      - "9003:9003"  # Xdebug 端口
```

### 热重载配置

```yaml
services:
  app:
    volumes:
      - .:/var/www/html
    environment:
      - APP_ENV=local
    # 使用开发镜像，支持热重载
    command: php-fpm -d opcache.enable=0
```

## 调试配置

### Xdebug 配置

```dockerfile
# Dockerfile.dev
FROM php:8.2-fpm

# 安装 Xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# 配置 Xdebug
RUN echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.start_with_request=yes" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_port=9003" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
```

### PHPStorm 配置

```yaml
# docker-compose.yml
services:
  app:
    environment:
      - XDEBUG_CONFIG=client_host=host.docker.internal client_port=9003
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

## 多服务编排

### 完整开发环境

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./public:/var/www/html/public
    depends_on:
      - app

  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/var/www/html
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./docker/mysql/init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI

volumes:
  db_data:
  redis_data:
```

## 完整示例

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: dev-nginx
    ports:
      - "${APP_PORT:-80}:80"
    volumes:
      - ./docker/nginx/dev.conf:/etc/nginx/nginx.conf
      - ./public:/var/www/html/public
    depends_on:
      - app
    networks:
      - dev-network

  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: dev-app
    volumes:
      - .:/var/www/html
      - ./vendor:/var/www/html/vendor
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
      - XDEBUG_MODE=debug
      - DB_HOST=db
      - REDIS_HOST=redis
    ports:
      - "9000:9000"
      - "9003:9003"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - dev-network
    command: php-fpm -d opcache.enable=0

  db:
    image: mysql:8.0
    container_name: dev-db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    ports:
      - "${DB_PORT:-3306}:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 10
    networks:
      - dev-network

  redis:
    image: redis:7-alpine
    container_name: dev-redis
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - redis_data:/data
    networks:
      - dev-network

  mailhog:
    image: mailhog/mailhog
    container_name: dev-mailhog
    ports:
      - "1025:1025"
      - "8025:8025"
    networks:
      - dev-network

volumes:
  db_data:
  redis_data:

networks:
  dev-network:
    driver: bridge
```

## 最佳实践

1. **环境分离**：开发和生产环境分离
2. **热重载**：开发环境支持代码热重载
3. **调试工具**：集成 Xdebug 等调试工具
4. **服务完整**：包含所有开发需要的服务

## 注意事项

1. 开发环境不要暴露敏感端口
2. 注意数据卷挂载权限
3. 配置健康检查
4. 使用环境变量管理配置

## 练习

1. 创建一个完整的开发环境 docker-compose 配置。

2. 配置 Xdebug，支持远程调试。

3. 实现代码热重载，修改代码自动生效。

4. 集成 MailHog 等开发工具。

# 8.1.2 多阶段构建

## 概述

多阶段构建可以显著减小镜像大小，分离构建和运行环境。本节介绍多阶段构建的原理、实现方式、优化技巧。

## 多阶段构建原理

### 为什么需要多阶段构建

传统构建方式会将构建工具和依赖都包含在最终镜像中，导致镜像过大。多阶段构建允许在构建阶段使用完整工具链，在运行阶段只保留运行时需要的文件。

### 基础多阶段构建

```dockerfile
# 构建阶段
FROM php:8.2-cli AS builder
WORKDIR /app

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 复制依赖文件
COPY composer.json composer.lock ./

# 安装依赖
RUN composer install --no-dev --optimize-autoloader

# 复制源代码
COPY . .

# 运行阶段
FROM php:8.2-fpm-alpine
WORKDIR /var/www/html

# 只复制构建产物
COPY --from=builder /app /var/www/html

# 设置权限
RUN chown -R www-data:www-data /var/www/html

CMD ["php-fpm"]
```

## 复杂多阶段构建

### 包含前端构建

```dockerfile
# Node.js 构建阶段
FROM node:18-alpine AS frontend
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# PHP 构建阶段
FROM php:8.2-cli AS php-builder
WORKDIR /app
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader
COPY . .

# 运行阶段
FROM php:8.2-fpm-alpine
WORKDIR /var/www/html

# 复制 PHP 依赖
COPY --from=php-builder /app/vendor /var/www/html/vendor
COPY --from=php-builder /app /var/www/html

# 复制前端构建产物
COPY --from=frontend /app/public/build /var/www/html/public/build

CMD ["php-fpm"]
```

## 优化技巧

### 分离依赖安装

```dockerfile
# 阶段 1: 安装 Composer 依赖
FROM php:8.2-cli AS composer
WORKDIR /app
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader

# 阶段 2: 运行环境
FROM php:8.2-fpm-alpine
WORKDIR /var/www/html
COPY . .
COPY --from=composer /app/vendor ./vendor
```

### 使用 Alpine 镜像

```dockerfile
# 使用 Alpine 减小镜像大小
FROM php:8.2-fpm-alpine

# Alpine 使用 apk 包管理器
RUN apk add --no-cache \
    git \
    curl \
    && rm -rf /var/cache/apk/*
```

## 完整示例

```dockerfile
# 多阶段构建完整示例
# 阶段 1: Composer 依赖
FROM composer:latest AS composer
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction

# 阶段 2: 应用构建
FROM php:8.2-cli AS app-builder
WORKDIR /app
COPY . .
COPY --from=composer /app/vendor ./vendor
RUN php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache

# 阶段 3: 运行环境
FROM php:8.2-fpm-alpine
WORKDIR /var/www/html

# 安装运行时依赖
RUN apk add --no-cache \
    libpng \
    libzip \
    oniguruma

# 安装 PHP 扩展
RUN docker-php-ext-install \
    pdo_mysql \
    mbstring \
    zip \
    gd

# 复制应用文件
COPY --from=app-builder /app /var/www/html

# 设置权限
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/storage

EXPOSE 9000
CMD ["php-fpm"]
```

## 最佳实践

1. **分离构建和运行**：构建工具不进入最终镜像
2. **使用 Alpine**：减小镜像大小
3. **缓存优化**：合理利用构建缓存
4. **最小化层数**：合并 RUN 命令

## 注意事项

1. 确保构建产物完整
2. 注意文件权限
3. 测试多阶段构建
4. 监控镜像大小

## 练习

1. 实现一个多阶段构建的 Dockerfile。

2. 优化多阶段构建，减小镜像大小。

3. 包含前端构建的多阶段 Dockerfile。

4. 对比单阶段和多阶段构建的镜像大小。

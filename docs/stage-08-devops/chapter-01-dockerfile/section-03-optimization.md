# 8.1.3 优化技巧

## 概述

Dockerfile 优化可以提升构建速度、减小镜像大小、提高安全性。本节介绍镜像大小优化、构建速度优化、缓存利用、安全优化等技巧。

## 镜像大小优化

### 使用 Alpine 镜像

```dockerfile
# 不推荐：使用完整镜像
FROM php:8.2-fpm

# 推荐：使用 Alpine 镜像
FROM php:8.2-fpm-alpine
```

### 清理缓存和临时文件

```dockerfile
FROM php:8.2-fpm-alpine

# 合并命令，减少层数
RUN apk add --no-cache \
    git \
    curl \
    && docker-php-ext-install pdo_mysql \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*
```

### 使用 .dockerignore

```dockerfile
# .dockerignore
.git
.gitignore
.env
node_modules
vendor
*.log
.DS_Store
tests/
docs/
```

## 构建速度优化

### 利用缓存层

```dockerfile
# 先复制依赖文件，利用缓存
COPY composer.json composer.lock ./
RUN composer install --no-dev

# 再复制源代码
COPY . .
```

### 并行构建

```dockerfile
# 使用 BuildKit 并行构建
# syntax=docker/dockerfile:1.4
FROM php:8.2-fpm-alpine AS base

# 并行安装扩展
RUN --mount=type=cache,target=/var/cache/apk \
    apk add --no-cache libpng-dev libzip-dev

RUN --mount=type=cache,target=/root/.composer \
    composer install
```

## 安全优化

### 使用非 root 用户

```dockerfile
FROM php:8.2-fpm-alpine

# 创建非 root 用户
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# 切换到非 root 用户
USER appuser

WORKDIR /var/www/html
```

### 最小权限原则

```dockerfile
FROM php:8.2-fpm-alpine

# 只安装必要的包
RUN apk add --no-cache \
    --virtual .build-deps \
    $PHPIZE_DEPS \
    && docker-php-ext-install pdo_mysql \
    && apk del .build-deps
```

## 完整示例

```dockerfile
# 优化的 Dockerfile
FROM php:8.2-fpm-alpine AS base

# 安装系统依赖（合并命令）
RUN apk add --no-cache \
    git \
    curl \
    libpng-dev \
    libzip-dev \
    oniguruma-dev \
    && docker-php-ext-install \
    pdo_mysql \
    mbstring \
    zip \
    gd \
    && apk del libpng-dev libzip-dev oniguruma-dev \
    && rm -rf /var/cache/apk/*

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www/html

# 复制依赖文件（利用缓存）
COPY composer.json composer.lock ./

# 安装依赖（缓存 composer 数据）
RUN --mount=type=cache,target=/root/.composer \
    composer install --no-dev --optimize-autoloader --no-interaction

# 复制应用代码
COPY . .

# 设置权限
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/storage

# 切换到非 root 用户
USER www-data

EXPOSE 9000
CMD ["php-fpm"]
```

## 最佳实践

1. **使用 Alpine**：减小镜像大小
2. **合并命令**：减少镜像层数
3. **利用缓存**：优化构建速度
4. **安全配置**：使用非 root 用户

## 注意事项

1. 测试优化后的镜像
2. 监控镜像大小变化
3. 注意构建时间
4. 保持 Dockerfile 可读性

## 练习

1. 优化一个 Dockerfile，减小镜像大小。

2. 实现构建缓存优化，提升构建速度。

3. 配置安全优化，使用非 root 用户。

4. 对比优化前后的镜像大小和构建时间。

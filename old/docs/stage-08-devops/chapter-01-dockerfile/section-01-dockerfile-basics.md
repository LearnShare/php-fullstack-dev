# 8.1.1 Dockerfile 基础

## 概述

Dockerfile 是构建 Docker 镜像的配置文件。本节介绍 Dockerfile 语法、基础镜像选择、构建命令、最佳实践等内容。

## Dockerfile 语法

### 基础指令

```dockerfile
# 指定基础镜像
FROM php:8.2-fpm

# 设置工作目录
WORKDIR /var/www/html

# 复制文件
COPY . /var/www/html

# 安装依赖
RUN apt-get update && apt-get install -y \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 设置环境变量
ENV APP_ENV=production

# 暴露端口
EXPOSE 9000

# 启动命令
CMD ["php-fpm"]
```

### 指令详解

```dockerfile
# FROM: 指定基础镜像
FROM php:8.2-fpm-alpine

# LABEL: 添加元数据
LABEL maintainer="your-email@example.com"
LABEL version="1.0"

# RUN: 执行命令
RUN composer install --no-dev --optimize-autoloader

# COPY: 复制文件（保留权限）
COPY composer.json composer.lock ./

# ADD: 复制文件（支持 URL 和解压）
ADD https://example.com/file.tar.gz /tmp/

# ENV: 设置环境变量
ENV APP_ENV=production \
    APP_DEBUG=false

# ARG: 构建参数
ARG PHP_VERSION=8.2
FROM php:${PHP_VERSION}-fpm

# EXPOSE: 声明端口
EXPOSE 80 443

# VOLUME: 挂载卷
VOLUME ["/var/www/html/storage"]

# CMD: 默认命令
CMD ["php-fpm"]

# ENTRYPOINT: 入口点
ENTRYPOINT ["docker-php-entrypoint"]
```

## 基础镜像选择

### PHP 官方镜像

```dockerfile
# PHP-FPM
FROM php:8.2-fpm

# PHP-FPM Alpine（更小）
FROM php:8.2-fpm-alpine

# PHP CLI
FROM php:8.2-cli
```

### 自定义基础镜像

```dockerfile
# 基于 Ubuntu
FROM ubuntu:22.04

# 安装 PHP
RUN apt-get update && apt-get install -y \
    php8.2 \
    php8.2-fpm \
    php8.2-mysql \
    && rm -rf /var/lib/apt/lists/*
```

## 构建命令

### 基础构建

```bash
# 构建镜像
docker build -t my-app:latest .

# 指定 Dockerfile
docker build -f Dockerfile.prod -t my-app:prod .

# 构建参数
docker build --build-arg PHP_VERSION=8.2 -t my-app:latest .
```

### 多阶段构建

```dockerfile
# 构建阶段
FROM php:8.2-cli AS builder
WORKDIR /app
COPY . .
RUN composer install --no-dev --optimize-autoloader

# 运行阶段
FROM php:8.2-fpm-alpine
WORKDIR /var/www/html
COPY --from=builder /app /var/www/html
```

## 完整示例

```dockerfile
# Dockerfile
FROM php:8.2-fpm-alpine AS base

# 安装系统依赖
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
    gd

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www/html

# 复制依赖文件
COPY composer.json composer.lock ./

# 安装 PHP 依赖
RUN composer install --no-dev --optimize-autoloader --no-interaction

# 复制应用代码
COPY . .

# 设置权限
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/storage

# 暴露端口
EXPOSE 9000

# 启动 PHP-FPM
CMD ["php-fpm"]
```

## 最佳实践

1. **使用官方镜像**：优先使用官方维护的镜像
2. **多阶段构建**：减小镜像大小
3. **层缓存**：合理排序指令，利用缓存
4. **最小权限**：使用非 root 用户运行

## 注意事项

1. 避免在 RUN 中执行过多命令
2. 及时清理临时文件
3. 使用 .dockerignore 排除文件
4. 合理使用缓存层

## 练习

1. 创建一个基础的 PHP-FPM Dockerfile。

2. 实现多阶段构建，优化镜像大小。

3. 配置 .dockerignore，排除不必要的文件。

4. 优化 Dockerfile，提升构建速度。

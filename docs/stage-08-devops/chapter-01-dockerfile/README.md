# 8.1 专业 Dockerfile

## 目标

- 掌握 Dockerfile 的最佳实践。
- 理解多阶段构建（multi-stage build）。
- 能够优化镜像大小和构建速度。
- 熟悉 FPM + Nginx 分离容器架构。

## 基础 Dockerfile

### PHP-FPM Dockerfile

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
    unzip \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www

# 复制应用代码
COPY . .

# 安装依赖
RUN composer install --no-dev --optimize-autoloader

# 设置权限
RUN chown -R www-data:www-data /var/www

EXPOSE 9000
CMD ["php-fpm"]
```

## 多阶段构建

### 优化示例

```dockerfile
# 阶段一：构建阶段
FROM php:8.2-cli AS builder

WORKDIR /app

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 复制依赖文件
COPY composer.json composer.lock ./

# 安装依赖
RUN composer install --no-dev --optimize-autoloader --no-scripts

# 复制源代码
COPY . .

# 运行构建脚本
RUN composer run-script post-install-cmd

# 阶段二：生产阶段
FROM php:8.2-fpm-alpine

# 安装运行时依赖
RUN apk add --no-cache \
    libpng \
    libjpeg-turbo \
    freetype \
    && docker-php-ext-install pdo_mysql opcache

# 从构建阶段复制文件
COPY --from=builder /app /var/www

# 配置 OPcache
RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini

WORKDIR /var/www

EXPOSE 9000
CMD ["php-fpm"]
```

## FPM + Nginx 分离

### PHP-FPM Dockerfile

```dockerfile
FROM php:8.2-fpm-alpine

RUN apk add --no-cache \
    libpng \
    libjpeg-turbo \
    freetype \
    && docker-php-ext-install pdo_mysql opcache gd

COPY php.ini /usr/local/etc/php/conf.d/custom.ini
COPY www.conf /usr/local/etc/php-fpm.d/www.conf

WORKDIR /var/www

EXPOSE 9000
CMD ["php-fpm"]
```

### Nginx Dockerfile

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## 镜像优化

### 减小镜像大小

```dockerfile
# 使用 Alpine 基础镜像
FROM php:8.2-fpm-alpine

# 合并 RUN 命令，减少层数
RUN apk add --no-cache \
    libpng \
    libjpeg-turbo \
    && docker-php-ext-install gd \
    && apk del .build-deps

# 清理缓存
RUN rm -rf /var/cache/apk/* /tmp/*
```

### 构建缓存优化

```dockerfile
# 先复制依赖文件，利用缓存
COPY composer.json composer.lock ./
RUN composer install --no-dev

# 再复制源代码
COPY . .
```

## 练习

1. 编写一个优化的 PHP-FPM Dockerfile，使用多阶段构建。

2. 创建 FPM + Nginx 分离架构的 docker-compose 配置。

3. 优化 Dockerfile，减小镜像大小并提升构建速度。

4. 配置 OPcache 和 PHP 优化选项。

5. 实现一个 CI/CD 流程，自动构建和推送 Docker 镜像。

6. 创建不同环境的 Dockerfile（开发、测试、生产）。

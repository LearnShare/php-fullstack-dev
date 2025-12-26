# 1.5.2 扩展管理

## 概述

PHP 扩展提供额外的功能，如数据库连接、图像处理、加密等。本节介绍如何检查、安装和管理 PHP 扩展。

## 检查扩展

### 查看已安装的扩展

**命令行方式**：

```bash
php -m
```

**输出示例**：

```
[PHP Modules]
Core
ctype
curl
date
...
pdo_mysql
mysqli
```

**代码方式**：

```php
<?php
// 获取所有已加载的扩展
$extensions = get_loaded_extensions();
print_r($extensions);

// 检查特定扩展是否已加载
if (extension_loaded('pdo_mysql')) {
    echo "PDO MySQL extension is loaded\n";
}
```

### 检查扩展信息

```bash
php -i | grep extension
```

## 安装扩展

### Windows

**使用 PECL**：

```bash
pecl install extension_name
```

**手动安装**：
1. 下载扩展 DLL 文件
2. 复制到 PHP 扩展目录
3. 在 php.ini 中启用扩展

### macOS

**使用 Homebrew**：

```bash
# 安装扩展
brew install php@8.3-mysql
brew install php@8.3-redis

# 启用扩展
echo "extension=mysql.so" >> /opt/homebrew/etc/php/8.3/php.ini
```

### Linux

**Debian/Ubuntu**：

```bash
# 安装扩展
sudo apt install php8.3-mysql
sudo apt install php8.3-redis
sudo apt install php8.3-xml
sudo apt install php8.3-mbstring

# 重启 PHP-FPM
sudo systemctl restart php-fpm
```

**CentOS/RHEL**：

```bash
# 安装扩展
sudo yum install php-mysql
sudo yum install php-redis
```

### Docker

**使用官方 PHP 镜像**：

```dockerfile
FROM php:8.3-fpm

# 安装扩展
RUN docker-php-ext-install pdo_mysql mysqli
RUN pecl install redis && docker-php-ext-enable redis

# 验证
RUN php -m
```

## 常用扩展

### PDO MySQL

**功能**：数据库访问接口

**安装**：

```bash
# Linux
sudo apt install php8.3-mysql

# macOS
brew install php@8.3-mysql
```

**启用**：

```ini
extension=pdo_mysql
```

### Redis

**功能**：Redis 客户端

**安装**：

```bash
# 使用 PECL
pecl install redis

# Linux
sudo apt install php8.3-redis
```

**启用**：

```ini
extension=redis
```

### cURL

**功能**：HTTP 客户端

**安装**：

```bash
# Linux
sudo apt install php8.3-curl
```

**启用**：

```ini
extension=curl
```

### mbstring

**功能**：多字节字符串处理

**安装**：

```bash
# Linux
sudo apt install php8.3-mbstring
```

**启用**：

```ini
extension=mbstring
```

## 扩展配置

### 在 php.ini 中配置

```ini
; 启用扩展
extension=pdo_mysql
extension=mysqli
extension=redis

; 扩展配置
[redis]
redis.host = 127.0.0.1
redis.port = 6379
```

### 使用独立的配置文件

**创建扩展配置文件**：

```ini
; /etc/php/8.3/mods-available/mysql.ini
extension=pdo_mysql
extension=mysqli
```

**启用配置**：

```bash
# Debian/Ubuntu
sudo phpenmod mysql
```

## 完整示例

### 示例 1：检查扩展

```php
<?php
// 检查扩展是否已加载
$required_extensions = ['pdo_mysql', 'mysqli', 'curl', 'mbstring'];

foreach ($required_extensions as $ext) {
    if (extension_loaded($ext)) {
        echo "✓ {$ext} is loaded\n";
    } else {
        echo "✗ {$ext} is NOT loaded\n";
    }
}
```

### 示例 2：安装扩展（Docker）

```dockerfile
FROM php:8.3-fpm

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev

# 安装扩展
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    mysqli \
    gd \
    opcache

# 安装 Redis 扩展
RUN pecl install redis && docker-php-ext-enable redis

# 验证
RUN php -m
```

### 示例 3：扩展配置

```ini
; php.ini

; 数据库扩展
extension=pdo_mysql
extension=mysqli

; Redis 扩展
extension=redis

; Redis 配置
[redis]
redis.host = 127.0.0.1
redis.port = 6379
redis.timeout = 0
```

## 注意事项

- **版本兼容**：确保扩展版本与 PHP 版本兼容
- **依赖关系**：某些扩展需要系统库支持
- **性能影响**：只加载必要的扩展
- **配置验证**：安装后验证扩展是否正常工作

## 常见问题

### 问题 1：扩展未加载

**原因**：扩展未安装、配置错误、路径问题

**解决方法**：

```bash
# 检查扩展文件是否存在
ls -la /usr/lib/php/8.3/extensions/

# 检查 php.ini 配置
php --ini

# 检查扩展加载
php -m | grep extension_name
```

### 问题 2：扩展功能不可用

**原因**：扩展版本不匹配、依赖缺失

**解决方法**：

```bash
# 检查扩展信息
php -i | grep extension_name

# 重新安装扩展
sudo apt remove php8.3-extension
sudo apt install php8.3-extension
```

## 最佳实践

- 只安装必要的扩展
- 定期更新扩展版本
- 使用 Docker 管理扩展依赖
- 记录扩展安装步骤
- 验证扩展功能

## 相关章节

- **阶段六：数据库与缓存系统**：深入学习 PDO、Redis 等扩展的使用

## 练习任务

1. 使用 `php -m` 查看已安装的扩展列表。
2. 安装一个常用扩展（如 Redis），并验证安装。
3. 在 php.ini 中配置扩展，并重启 PHP 服务验证。
4. 使用代码检查扩展是否已加载。
5. 创建一个 Dockerfile，安装多个常用扩展。

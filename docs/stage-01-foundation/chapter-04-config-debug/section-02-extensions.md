# 1.4.2 扩展管理

## 概述

PHP 扩展提供了额外的功能，如数据库连接、图像处理、加密等。本章介绍如何安装、配置和管理 PHP 扩展。

## 扩展类型

### 核心扩展

PHP 内置的扩展，通常默认启用：
- `pdo`：数据库抽象层
- `json`：JSON 处理
- `mbstring`：多字节字符串
- `openssl`：加密和 SSL/TLS

### PECL 扩展

通过 PECL（PHP Extension Community Library）安装的扩展：
- `xdebug`：调试工具
- `redis`：Redis 客户端
- `imagick`：图像处理

### 第三方扩展

通过源码编译安装的扩展：
- `swoole`：异步网络框架
- `opcache`：操作码缓存

## 检查扩展

### 查看已安装扩展

```bash
# 列出所有已安装的扩展
php -m

# 查看特定扩展信息
php --ri extension_name

# 查看扩展配置
php -i | grep extension_name
```

### 在代码中检查

```php
<?php
declare(strict_types=1);

// 检查扩展是否加载
if (extension_loaded('pdo_mysql')) {
    echo "PDO MySQL extension is loaded\n";
} else {
    echo "PDO MySQL extension is not loaded\n";
}

// 获取已加载的扩展列表
$extensions = get_loaded_extensions();
print_r($extensions);
```

## 安装扩展

### Windows

#### 方式一：修改 php.ini

1. 下载扩展 DLL 文件
2. 将 DLL 文件放到 PHP 扩展目录
3. 在 `php.ini` 中添加：
   ```ini
   extension=pdo_mysql
   ```

#### 方式二：使用包管理器

```bash
# 使用 Chocolatey
choco install php-extension-name
```

### macOS

#### 使用 Homebrew

```bash
# 安装扩展
brew install php@8.2-mysql
brew install php@8.2-xdebug
brew install php@8.2-imagick

# 启用扩展
# 扩展会自动添加到 php.ini
```

#### 使用 PECL

```bash
# 安装 PECL 扩展
pecl install xdebug
pecl install redis

# 在 php.ini 中添加
# extension=xdebug.so
# extension=redis.so
```

### Linux

#### 使用包管理器

```bash
# Ubuntu/Debian
sudo apt install php8.2-mysql
sudo apt install php8.2-xdebug
sudo apt install php8.2-mbstring

# RHEL/CentOS
sudo yum install php-mysql
sudo yum install php-xdebug
```

#### 使用 PECL

```bash
# 安装 PECL
sudo apt install php-pear php8.2-dev

# 安装扩展
sudo pecl install xdebug
sudo pecl install redis

# 在 php.ini 中添加
# extension=xdebug.so
# extension=redis.so
```

### Docker 环境

#### 使用 docker-php-ext-install

```dockerfile
FROM php:8.2-fpm

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install gd
RUN docker-php-ext-install mbstring

# 安装 PECL 扩展
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug
```

## 常用扩展

### 数据库扩展

#### PDO

```bash
# 安装
# Ubuntu/Debian
sudo apt install php8.2-pdo php8.2-pdo-mysql

# macOS
brew install php@8.2-pdo_mysql
```

**使用**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO(
    'mysql:host=localhost;dbname=test',
    'user',
    'password'
);
```

#### MySQLi

```bash
# 安装
sudo apt install php8.2-mysqli
```

**使用**：

```php
<?php
declare(strict_types=1);

$mysqli = new mysqli('localhost', 'user', 'password', 'test');
```

### 字符串处理

#### mbstring

```bash
# 安装
sudo apt install php8.2-mbstring
```

**使用**：

```php
<?php
declare(strict_types=1);

$length = mb_strlen('你好世界', 'UTF-8');
echo $length; // 4
```

### 图像处理

#### GD

```bash
# 安装
sudo apt install php8.2-gd
```

**使用**：

```php
<?php
declare(strict_types=1);

$image = imagecreate(200, 200);
$color = imagecolorallocate($image, 255, 0, 0);
imagefill($image, 0, 0, $color);
imagepng($image, 'output.png');
imagedestroy($image);
```

#### Imagick

```bash
# 安装
sudo apt install php8.2-imagick
# 或
pecl install imagick
```

### 加密和哈希

#### OpenSSL

```bash
# 通常默认安装
# 检查
php -m | grep openssl
```

**使用**：

```php
<?php
declare(strict_types=1);

$data = 'sensitive data';
$encrypted = openssl_encrypt($data, 'AES-256-CBC', $key);
$decrypted = openssl_decrypt($encrypted, 'AES-256-CBC', $key);
```

### 缓存扩展

#### OPcache

```bash
# 通常默认安装
# 检查
php -m | grep opcache
```

**配置**（`php.ini`）：

```ini
opcache.enable=1
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

## 扩展配置

### 在 php.ini 中配置

```ini
; 启用扩展
extension=pdo_mysql
extension=mbstring
extension=gd

; 扩展特定配置
[redis]
redis.session.locking_enabled = 1
redis.session.lock_expire = 0
redis.session.lock_wait_time = 200000
```

### 使用独立的配置文件

在 `conf.d` 目录创建独立配置文件：

```ini
; /etc/php/8.2/fpm/conf.d/20-xdebug.ini
zend_extension=xdebug.so
xdebug.mode=debug
xdebug.client_host=host.docker.internal
xdebug.client_port=9003
```

## 验证安装

### 命令行验证

```bash
# 检查扩展是否加载
php -m | grep extension_name

# 查看扩展详细信息
php --ri extension_name

# 查看扩展配置
php -i | grep extension_name
```

### 代码验证

```php
<?php
declare(strict_types=1);

function checkExtension(string $name): bool
{
    if (!extension_loaded($name)) {
        echo "Extension '{$name}' is not loaded\n";
        return false;
    }
    
    echo "Extension '{$name}' is loaded\n";
    return true;
}

// 检查常用扩展
$extensions = ['pdo_mysql', 'mbstring', 'gd', 'openssl', 'json'];
foreach ($extensions as $ext) {
    checkExtension($ext);
}
```

## 完整示例

### Dockerfile 示例

```dockerfile
FROM php:8.2-fpm

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    libicu-dev \
    libxml2-dev

# 安装 PHP 扩展
RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    mbstring \
    gd \
    zip \
    intl \
    opcache

# 安装 PECL 扩展
RUN pecl install xdebug redis && \
    docker-php-ext-enable xdebug redis

# 配置 OPcache
RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.memory_consumption=128" >> /usr/local/etc/php/conf.d/opcache.ini

# 清理
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
```

### 扩展检查脚本

```php
<?php
declare(strict_types=1);

class ExtensionChecker
{
    private array $required = [
        'pdo_mysql',
        'mbstring',
        'gd',
        'openssl',
        'json',
        'curl'
    ];
    
    private array $optional = [
        'xdebug',
        'redis',
        'imagick'
    ];
    
    public function check(): void
    {
        echo "=== Required Extensions ===\n";
        foreach ($this->required as $ext) {
            $this->checkExtension($ext, true);
        }
        
        echo "\n=== Optional Extensions ===\n";
        foreach ($this->optional as $ext) {
            $this->checkExtension($ext, false);
        }
    }
    
    private function checkExtension(string $name, bool $required): void
    {
        $loaded = extension_loaded($name);
        $status = $loaded ? '✓' : '✗';
        
        echo "{$status} {$name}";
        
        if ($loaded) {
            $version = phpversion($name) ?: 'unknown';
            echo " (version: {$version})";
        } else {
            echo " (not loaded)";
            if ($required) {
                echo " [REQUIRED]";
            }
        }
        
        echo "\n";
    }
}

$checker = new ExtensionChecker();
$checker->check();
```

## 注意事项

1. **扩展兼容性**：确保扩展版本与 PHP 版本兼容。

2. **依赖关系**：某些扩展需要系统库支持，需要先安装依赖。

3. **性能影响**：只安装需要的扩展，避免加载不必要的扩展。

4. **配置验证**：安装扩展后使用 `php -m` 验证是否加载。

5. **生产环境**：生产环境只安装必需的扩展，禁用调试扩展。

## 练习

1. 检查当前 PHP 环境已安装的扩展。

2. 安装常用的 PHP 扩展（pdo_mysql、mbstring、gd）。

3. 创建扩展检查脚本，验证必需扩展是否安装。

4. 在 Docker 环境中安装和配置 PHP 扩展。

5. 配置 OPcache，优化 PHP 性能。

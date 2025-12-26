# 1.2.1 PHP 安装与版本管理

## 概述

PHP 的安装和版本管理是开发的第一步。本节介绍在不同平台上安装 PHP 的方法，以及如何管理多个 PHP 版本。

## 版本策略

| 场景 | 推荐版本 | 说明 |
|:-----|:---------|:-----|
| 本地开发 | 8.2+ | 与主流框架兼容，扩展生态成熟 |
| 新项目 | 8.3+ | 获得最新语法特性 |
| 旧项目维护 | 8.1+ | 如依赖旧扩展，可暂时保留 |

## 安装方法

### Windows 平台

#### 方法一：Herd（推荐）

**步骤**：

1. 访问 Herd 官网：https://herd.laravel.com
2. 下载并安装 Herd
3. 打开控制面板，选择 PHP 版本
4. 启动服务

**验证**：

```bash
php -v
```

**输出示例**：

```
PHP 8.3.0 (cli) (built: Dec 7 2023 10:30:00) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.3.0, Copyright (c) Zend Technologies
```

#### 方法二：Laragon

**步骤**：

1. 访问 Laragon 官网：https://laragon.org
2. 下载并安装 Laragon
3. 在控制面板中选择 PHP 版本
4. 启动服务

### macOS 平台

#### 使用 Homebrew（推荐）

**安装步骤**：

```bash
# 安装 Homebrew（如果未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装 PHP
brew install php@8.3

# 链接到系统路径
brew link --overwrite php@8.3

# 配置环境变量（添加到 ~/.zshrc 或 ~/.bash_profile）
echo 'export PATH="/opt/homebrew/opt/php@8.3/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 验证安装
php -v
```

**多版本管理**：

```bash
# 安装多个版本
brew install php@8.2
brew install php@8.3

# 切换版本
brew unlink php@8.2
brew link php@8.3
```

### Linux 平台

#### Debian/Ubuntu

**安装步骤**：

```bash
# 添加 PPA 仓库
sudo add-apt-repository ppa:ondrej/php
sudo apt update

# 安装 PHP
sudo apt install php8.3

# 安装常用扩展
sudo apt install php8.3-mysql php8.3-xml php8.3-curl php8.3-mbstring

# 验证安装
php -v
```

#### CentOS/RHEL

**安装步骤**：

```bash
# 添加 Remi 仓库
sudo yum install epel-release
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-8.rpm

# 安装 PHP
sudo yum install php82

# 验证安装
php -v
```

### Docker 安装

**使用官方 PHP 镜像**：

```bash
# 拉取 PHP 镜像
docker pull php:8.3-cli

# 运行容器
docker run -it --rm php:8.3-cli php -v
```

**创建 Dockerfile**：

```dockerfile
FROM php:8.3-fpm

# 安装扩展
RUN docker-php-ext-install pdo_mysql

# 验证
RUN php -v
```

## 运行验证

### 验证命令

**查看版本**：

```bash
php -v
```

**查看已安装的扩展**：

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
```

**查看配置信息**：

```bash
php -i
```

**查看配置文件路径**：

```bash
php --ini
```

**输出示例**：

```
Configuration File (php.ini) Path: /etc/php/8.3/cli
Loaded Configuration File:         /etc/php/8.3/cli/php.ini
```

### 常用命令

**运行 PHP 脚本**：

```bash
php script.php
```

**启动内置 Web 服务器**：

```bash
php -S localhost:8000
```

**执行单行代码**：

```bash
php -r "echo 'Hello World';"
```

**输出**：

```
Hello World
```

## 完整示例

### 示例 1：Windows 安装验证

```bash
# 检查 PHP 版本
C:\> php -v
PHP 8.3.0 (cli) (built: Dec 7 2023 10:30:00) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.3.0, Copyright (c) Zend Technologies

# 检查已安装的扩展
C:\> php -m
[PHP Modules]
Core
ctype
curl
date
...
```

### 示例 2：macOS 安装验证

```bash
# 检查 PHP 版本
$ php -v
PHP 8.3.0 (cli) (built: Dec 7 2023 10:30:00) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.3.0, Copyright (c) Zend Technologies

# 检查配置文件路径
$ php --ini
Configuration File (php.ini) Path: /opt/homebrew/etc/php/8.3
Loaded Configuration File:         /opt/homebrew/etc/php/8.3/php.ini
```

### 示例 3：Linux 安装验证

```bash
# 检查 PHP 版本
$ php -v
PHP 8.3.0 (cli) (built: Dec 7 2023 10:30:00) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.3.0, Copyright (c) Zend Technologies

# 检查已安装的扩展
$ php -m | grep mysql
pdo_mysql
mysqli
```

## 注意事项

- **版本一致性**：确保 CLI、FPM、容器镜像标签保持一致
- **扩展兼容性**：不同 PHP 版本可能需要不同的扩展版本
- **路径配置**：确保 PHP 可执行文件在系统 PATH 中
- **权限问题**：某些操作可能需要管理员权限
- **环境变量**：正确配置环境变量，避免版本冲突

## 常见问题

### 问题 1：找不到 php 命令

**原因**：PHP 未添加到系统 PATH

**解决方法**：

1. 检查 PHP 安装路径
2. 将 PHP 可执行文件路径添加到系统 PATH
3. 重启终端或命令行窗口

**Windows 示例**：

```bash
# 检查 PHP 路径
where php

# 如果未找到，手动添加到 PATH
set PATH=%PATH%;C:\php
```

**macOS/Linux 示例**：

```bash
# 检查 PHP 路径
which php

# 如果未找到，添加到 PATH（添加到 ~/.zshrc 或 ~/.bash_profile）
export PATH="/opt/homebrew/opt/php@8.3/bin:$PATH"
```

### 问题 2：版本不匹配

**原因**：多个 PHP 版本安装，PATH 优先级问题

**解决方法**：

1. 检查 PATH 中的 PHP 版本顺序
2. 使用完整路径或版本别名
3. 使用版本管理工具（如 phpenv、phpbrew）

**示例**：

```bash
# 检查当前使用的 PHP 版本
php -v

# 使用完整路径
/opt/homebrew/opt/php@8.2/bin/php -v
/opt/homebrew/opt/php@8.3/bin/php -v
```

### 问题 3：扩展未加载

**原因**：扩展未安装或配置不正确

**解决方法**：

1. 检查扩展是否已安装
2. 检查 php.ini 配置文件
3. 确保扩展已启用

**示例**：

```bash
# 检查扩展是否已安装
php -m | grep mysql

# 检查 php.ini 配置
php --ini

# 编辑 php.ini，确保扩展已启用
# extension=pdo_mysql
```

## 最佳实践

- 使用版本管理工具（如 phpenv、phpbrew）管理多个版本
- 在项目中明确指定 PHP 版本要求（使用 `composer.json` 的 `php` 字段）
- 使用 Docker 确保开发和生产环境一致
- 定期更新 PHP 版本，但要注意兼容性
- 记录安装步骤，便于环境复现

## 练习任务

1. 在本地环境安装 PHP 8.3，并使用 `php -v` 验证安装。
2. 使用 `php -m` 查看已安装的扩展，并记录扩展列表。
3. 使用 `php --ini` 查看配置文件路径，并打开配置文件查看内容。
4. 创建一个简单的 PHP 脚本 `test.php`，内容为 `<?php echo "Hello PHP"; ?>`，并使用 `php test.php` 运行。
5. 如果系统中有多个 PHP 版本，尝试切换版本并验证。

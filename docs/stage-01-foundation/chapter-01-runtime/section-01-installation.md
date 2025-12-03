# 1.1.1 PHP 安装与版本管理

## 概述

PHP 的安装和版本管理是开发的第一步。本章介绍在不同平台上安装 PHP 的方法，以及如何管理多个 PHP 版本。

## 版本策略

### 推荐版本

| 场景 | 推荐版本 | 说明 |
| :--- | :------- | :--- |
| 本地开发 | 8.2 LTS | 与主流框架兼容，扩展生态成熟 |
| 新项目 | 8.3 | 获得最新语法（`readonly class`、改进的 `json_validate` 等） |
| 旧项目维护 | 8.1 | 如依赖旧扩展，可暂时保留，但需规划升级窗口 |

> 提示：确保 CLI `php -v`、FPM `php-fpm -v`、容器镜像标签保持一致，否则会出现"线上/线下不一致"问题。

## 安装方式

### Windows：Herd / Laragon

#### Herd

1. 访问 [Herd 官网](https://herd.laravel.com)，下载安装包
2. 运行安装向导，完成安装
3. 打开控制面板，勾选所需 PHP 版本与 Nginx/MySQL 组件
4. 点击 "Start All" 启动服务

#### Laragon

1. 访问 [Laragon 官网](https://laragon.org)，下载安装包
2. 运行安装向导，完成安装
3. 在控制面板中选择 PHP 版本
4. 启动服务

#### 多版本管理

```bash
# 在 Herd 中添加额外版本（如 8.3）
# 将 C:\Program Files\Herd\php83\ 添加到 PATH
# 或创建 php83.bat 指向该目录

# 使用
php82 -v
php83 -v
```

### macOS：Homebrew / php-tap

#### 安装 Homebrew

```bash
# 安装 Homebrew（如果未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 安装 PHP

```bash
# 安装 PHP 8.3
brew install php@8.3

# 链接到系统路径
brew link --overwrite php@8.3

# 验证安装
php -v
```

#### 配置环境变量

```bash
# 添加到 ~/.zshrc 或 ~/.bash_profile
export PATH="/opt/homebrew/opt/php@8.3/bin:$PATH"
export PATH="/opt/homebrew/opt/php@8.3/sbin:$PATH"
```

#### 多版本管理

```bash
# 安装多个版本
brew install php@8.2
brew install php@8.3

# 切换版本
brew unlink php@8.2
brew link php@8.3
```

### Linux：包管理器 / 源码 / Docker

#### Debian/Ubuntu

```bash
# 添加 PPA
sudo add-apt-repository ppa:ondrej/php
sudo apt update

# 安装 PHP 8.2
sudo apt install php8.2 php8.2-fpm php8.2-cli

# 安装常用扩展
sudo apt install php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl
```

#### RHEL/CentOS

```bash
# 添加 Remi 仓库
sudo yum install epel-release
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-8.rpm

# 安装 PHP 8.2
sudo yum install php82 php82-php-fpm php82-php-cli
```

#### Docker

```bash
# 使用官方镜像
docker run --rm -it php:8.3-cli php -v

# 或使用 docker-compose
```

## 多版本管理

### update-alternatives (Linux)

```bash
# 配置多个 PHP 版本
sudo update-alternatives --install /usr/bin/php php /usr/bin/php8.2 1
sudo update-alternatives --install /usr/bin/php php /usr/bin/php8.3 2

# 切换版本
sudo update-alternatives --config php
```

### asdf / mise

```bash
# 安装 asdf
git clone https://github.com/asdf-vm/asdf.git ~/.asdf

# 安装 PHP 插件
asdf plugin add php

# 安装 PHP 版本
asdf install php 8.2.0
asdf install php 8.3.0

# 切换版本
asdf global php 8.3.0
```

## 运行验证

### 基本验证

```bash
# 查看版本
php -v

# 查看配置
php -i | head -n 20

# 查看扩展
php -m

# 查看配置文件位置
php --ini
```

### 创建测试文件

```php
<?php
// hello.php
declare(strict_types=1);

echo "PHP Version: " . PHP_VERSION . "\n";
echo "SAPI: " . php_sapi_name() . "\n";
echo "Environment OK!\n";
```

```bash
# 运行测试
php hello.php
```

## 常用命令速查

| 命令 | 作用 | 关键参数 |
| :--- | :--- | :------- |
| `php -v` | 显示当前 CLI 版本与构建信息 | 无 |
| `php -i` | 查看完整配置，与 `phpinfo()` 一致 | 可结合 `grep` 过滤 |
| `php -m` | 列出已启用扩展 | 无 |
| `php-fpm -v` | 查看 FPM 版本 | 配置多个版本时用于核对 |
| `where php` / `which php` | 找到实际可执行文件路径 | Windows 使用 `where`，Unix 使用 `which` |
| `php -S host:port -t public` | 启动内置服务器并指定根目录 | `-t` 指定文档根 |

> 所有命令建议记录在 `docs/cheatsheet-runtime.md`，并标注"适用平台+操作人+日期"，便于团队溯源。

## 常见故障

### 缺少扩展

```bash
# 通过包管理器安装
# Ubuntu/Debian
sudo apt install php8.2-xml

# macOS
brew install php@8.3-xml

# 或使用 PECL
pecl install xdebug
```

### 路径冲突

```bash
# 检查 PHP 路径
where php  # Windows
which php  # Unix

# 移除旧版本路径
# 编辑 PATH 环境变量
```

### 无法切换 FPM 版本

```nginx
# 在 Nginx 配置中显式指定
fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
```

### 权限问题

```bash
# Windows：以管理员身份运行安装器
# macOS/Linux：检查权限
sudo chown -R $USER /usr/local/php
```

### 证书错误

```bash
# 更新系统证书
# 或使用 git config
git config --global http.sslBackend schannel
```

## 完整示例

### 安装脚本（macOS）

```bash
#!/bin/bash
# install-php.sh

# 安装 Homebrew（如果未安装）
if ! command -v brew &> /dev/null; then
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
fi

# 安装 PHP 8.3
brew install php@8.3

# 链接到系统路径
brew link --overwrite php@8.3

# 验证安装
php -v

# 安装常用扩展
brew install php@8.3-mysql php@8.3-mbstring php@8.3-xml
```

### 安装脚本（Linux）

```bash
#!/bin/bash
# install-php.sh

# 添加 PPA
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# 安装 PHP 8.2
sudo apt install php8.2 php8.2-fpm php8.2-cli -y

# 安装常用扩展
sudo apt install php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl -y

# 验证安装
php -v
php-fpm -v
```

## 注意事项

1. **版本一致性**：确保开发、测试、生产环境使用相同的 PHP 版本。

2. **扩展兼容性**：确保扩展版本与 PHP 版本兼容。

3. **路径配置**：正确配置 PATH 环境变量，避免版本冲突。

4. **文档记录**：记录安装步骤和配置，便于团队复现。

5. **定期更新**：定期更新 PHP 版本，获得安全补丁和新特性。

## 练习

1. 在本地环境安装 PHP 8.2 或 8.3。

2. 配置多版本 PHP，能够切换不同版本。

3. 验证 PHP 安装，检查常用扩展是否已安装。

4. 创建安装脚本，记录安装步骤。

5. 解决常见的安装问题，如路径冲突、权限问题等。

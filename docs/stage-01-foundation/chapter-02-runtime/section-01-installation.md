# 1.2.1 PHP 安装与版本管理

## 概述

PHP 的安装和版本管理是开发的第一步。本节介绍在不同平台上安装 PHP 的方法，以及如何管理多个 PHP 版本。确保 CLI、FPM、容器镜像标签保持一致，避免"线上/线下不一致"问题。

**章节类型**：配置性章节

**主要内容**：
- 版本策略和推荐版本
- Windows 平台安装（Herd、Laragon）
- macOS 平台安装（Homebrew）
- Linux 平台安装（包管理器、源码、Docker）
- Docker 安装方式
- 多版本管理方法
- 运行验证和常用命令
- 常见故障排查

## 特性

- **跨平台支持**：Windows、macOS、Linux 均有完善的安装方案
- **多版本管理**：支持在同一系统上安装和管理多个 PHP 版本
- **容器化支持**：支持 Docker 容器化部署
- **版本切换**：可以方便地在不同版本间切换

## 配置步骤/语法

### 版本策略

| 场景 | 推荐版本 | 说明 |
| :--- | :------- | :--- |
| 本地开发 | 8.2 LTS | 与主流框架兼容，扩展生态成熟 |
| 新项目 | 8.3 | 获得最新语法特性 |
| 旧项目维护 | 8.1 | 如依赖旧扩展，可暂时保留 |

### Windows：Herd / Laragon

**Herd 安装步骤**：
1. 访问 Herd 官网，下载安装包
2. 运行安装向导，完成安装
3. 打开控制面板，勾选所需 PHP 版本与 Nginx/MySQL 组件
4. 点击 "Start All" 启动服务

**Laragon 安装步骤**：
1. 访问 Laragon 官网，下载安装包
2. 运行安装向导，完成安装
3. 在控制面板中选择 PHP 版本
4. 启动服务

**多版本管理**：
- 在 Herd 中添加额外版本
- 创建不同版本的批处理文件
- 使用 `php82 -v`、`php83 -v` 验证版本

### macOS：Homebrew

**安装步骤**：
1. 安装 Homebrew（如果未安装）
2. 使用 `brew install php@8.3` 安装 PHP
3. 使用 `brew link --overwrite php@8.3` 链接到系统路径
4. 配置环境变量（添加到 ~/.zshrc 或 ~/.bash_profile）
5. 使用 `php -v` 验证安装

**多版本管理**：
- 安装多个版本：`brew install php@8.2`、`brew install php@8.3`
- 切换版本：`brew unlink php@8.2`、`brew link php@8.3`

### Linux：包管理器 / 源码 / Docker

**Debian/Ubuntu**：
- 添加 PPA：`sudo add-apt-repository ppa:ondrej/php`
- 更新包列表：`sudo apt update`
- 安装 PHP：`sudo apt install php8.2`
- 安装扩展：`sudo apt install php8.2-mysql php8.2-xml`

**CentOS/RHEL**：
- 添加 Remi 仓库
- 安装 PHP：`sudo yum install php82`

**Docker 安装**：
- 使用官方 PHP 镜像
- 创建 Dockerfile
- 运行容器并验证

### 运行验证

**验证命令**：
- `php -v`：查看 PHP 版本
- `php -m`：查看已安装的扩展
- `php -i`：查看 PHP 配置信息
- `php --ini`：查看配置文件路径

**常用命令**：
- `php script.php`：运行 PHP 脚本
- `php -S localhost:8000`：启动内置 Web 服务器
- `php -r "echo 'Hello World';"`：执行单行代码

## 注意事项

- **版本一致性**：确保 CLI、FPM、容器镜像标签保持一致
- **扩展兼容性**：不同 PHP 版本可能需要不同的扩展版本
- **路径配置**：确保 PHP 可执行文件在系统 PATH 中
- **权限问题**：某些操作可能需要管理员权限
- **环境变量**：正确配置环境变量，避免版本冲突

## 常见问题

### 问题 1：找不到 php 命令
- **原因**：PHP 未添加到系统 PATH
- **解决**：检查环境变量配置，确保 PHP 可执行文件路径在 PATH 中

### 问题 2：版本不匹配
- **原因**：多个 PHP 版本安装，PATH 优先级问题
- **解决**：检查 PATH 顺序，使用完整路径或版本别名

### 问题 3：扩展未加载
- **原因**：扩展未安装或配置不正确
- **解决**：检查 php.ini 配置，确保扩展已启用

### 问题 4：Docker 容器中 PHP 版本不一致
- **原因**：镜像标签或 Dockerfile 配置问题
- **解决**：检查 Dockerfile，确保使用正确的 PHP 版本标签

## 输出结果说明

**安装验证示例**：
```bash
$ php -v
PHP 8.3.0 (cli) (built: Dec 7 2023 10:30:00) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.3.0, Copyright (c) Zend Technologies
```

**扩展列表示例**：
```bash
$ php -m
[PHP Modules]
Core
ctype
curl
...
```

## 最佳实践

- 使用版本管理工具（如 phpenv、phpbrew）管理多个版本
- 在项目中明确指定 PHP 版本要求
- 使用 Docker 确保开发和生产环境一致
- 定期更新 PHP 版本，但要注意兼容性
- 记录安装步骤，便于环境复现

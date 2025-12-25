# 1.4.1 Composer 基础

## 概述

Composer 是 PHP 的依赖管理工具，类似于 Node.js 的 npm 或 Python 的 pip。它不仅可以管理项目依赖，还提供了强大的自动加载功能。掌握 Composer 的使用是现代 PHP 开发的基础。

**章节类型**：工具性章节

**主要内容**：
- Composer 的定义和为什么需要 Composer
- Composer 安装（Windows、Linux/macOS）
- 基本使用（初始化项目、安装依赖、更新依赖）
- 常用命令（require、update、install、remove、show、search）
- 版本约束（精确版本、范围版本、通配符、稳定性标志）
- composer.json 配置详解
- 与 npm 的对比
- 常见问题排查

## 特性

- **依赖管理**：自动管理项目依赖和版本
- **自动加载**：自动生成类加载器
- **版本控制**：灵活的版本约束机制
- **包管理**：支持发布和分享自己的包
- **跨平台**：支持 Windows、macOS、Linux

## 基本用法/命令

### 安装 Composer

**Windows**：
- 使用安装程序（推荐）
- 手动安装方法

**Linux/macOS**：
- 全局安装方法
- 验证安装

### 基本使用

**初始化项目**：
- `composer init`：交互式创建 composer.json
- 手动创建 composer.json

**安装依赖**：
- `composer install`：安装 composer.json 中的依赖
- `composer require vendor/package`：添加新包并安装
- `composer update`：更新所有依赖

**常用命令**：
- `composer require`：添加依赖
- `composer remove`：移除依赖
- `composer update`：更新依赖
- `composer show`：显示已安装的包
- `composer search`：搜索包
- `composer dump-autoload`：重新生成自动加载文件

### 版本约束

**精确版本**：`"vendor/package": "1.2.3"`
**范围版本**：`"vendor/package": "^1.2"`、`"vendor/package": "~1.2.3"`
**通配符**：`"vendor/package": "1.2.*"`
**稳定性标志**：`"vendor/package": "dev-master"`、`"vendor/package": "@dev"`

### composer.json 配置

**基本结构**：
- name：包名
- description：描述
- type：包类型
- require：生产依赖
- require-dev：开发依赖
- autoload：自动加载配置
- scripts：脚本配置

## 配置选项

**composer.json 配置项**：
- 包信息配置
- 依赖配置
- 自动加载配置
- 脚本配置
- 仓库配置

**composer 全局配置**：
- `composer config`：配置命令
- 全局配置文件位置
- 常用配置项

## 使用场景

- **新项目**：使用 `composer init` 初始化项目
- **现有项目**：使用 `composer require` 添加依赖
- **团队协作**：提交 composer.json 和 composer.lock
- **生产部署**：使用 `composer install --no-dev` 安装生产依赖

## 注意事项

- **composer.lock**：应该提交到版本控制，确保团队使用相同版本
- **版本约束**：合理使用版本约束，避免过于严格或过于宽松
- **开发依赖**：区分生产依赖和开发依赖
- **自动加载**：确保在代码中引入 `vendor/autoload.php`

## 常见问题

### 问题 1：composer install 失败
- **原因**：网络问题、版本冲突、权限问题
- **解决**：检查网络、解决版本冲突、检查权限

### 问题 2：版本冲突
- **原因**：依赖包版本要求冲突
- **解决**：更新依赖、使用 `composer why-not` 查看冲突原因

### 问题 3：自动加载不工作
- **原因**：未引入 autoload.php、命名空间配置错误
- **解决**：检查 autoload.php 引入、检查命名空间配置

## 最佳实践

- 使用精确版本或范围版本，避免使用 `*`
- 提交 composer.lock 到版本控制
- 区分生产依赖和开发依赖
- 定期更新依赖，但要注意兼容性
- 使用 `composer validate` 验证 composer.json

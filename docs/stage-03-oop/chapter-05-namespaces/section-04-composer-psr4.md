# 3.5.4 Composer 自动加载与 PSR-4

## 概述

PSR-4 是标准的自动加载规范，Composer 提供了 PSR-4 自动加载支持。本节介绍 PSR-4 规则、Composer 自动加载配置、目录结构示例、自动加载优化、调试自动加载、命名空间最佳实践等内容。

**章节类型**：配置性章节

**主要内容**：
- PSR-4 自动加载规则
- Composer 自动加载配置（composer.json）
- 目录结构与命名空间的对应关系
- 自动加载优化（classmap、files）
- 调试自动加载问题
- 命名空间最佳实践
- 生产环境优化

## 特性

- **标准规范**：遵循 PSR-4 标准
- **自动生成**：Composer 自动生成加载器
- **性能优化**：支持多种优化方式

## 配置步骤/语法

### composer.json 配置

**语法**：
```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

### 生成自动加载器

**命令**：`composer dump-autoload`
**作用**：生成 `vendor/autoload.php`

### 使用自动加载器

**语法**：`require 'vendor/autoload.php';`
**位置**：项目入口文件

## 基本用法

### composer.json 示例

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Controller\\": "src/Controller/"
        }
    }
}
```

### 目录结构示例

```
project/
  src/
    App/
      Model/
        User.php  // namespace App\Model;
      Controller/
        UserController.php  // namespace App\Controller;
  vendor/
    autoload.php
```

### 使用示例

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

use App\Model\User;

$user = new User();
```

## 使用场景

- **标准项目**：遵循 PSR-4 标准的项目
- **框架开发**：框架的自动加载机制
- **库开发**：库的自动加载配置

## 注意事项

- **目录结构**：命名空间必须与目录结构对应
- **类文件**：一个文件一个类
- **命名规范**：遵循 PSR-4 命名规范

## 常见问题

### 问题 1：类找不到
- **原因**：命名空间与目录结构不匹配
- **解决**：检查命名空间和目录结构

### 问题 2：自动加载器未更新
- **原因**：修改配置后未重新生成
- **解决**：运行 `composer dump-autoload`

## 最佳实践

- 遵循 PSR-4 标准
- 命名空间与目录结构对应
- 使用 Composer 管理自动加载
- 定期优化自动加载器

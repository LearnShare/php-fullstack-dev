# 3.5.3 自动加载基础

## 概述

自动加载是按需加载类文件的机制，避免手动 `require` 每个类文件。本节介绍传统方式的问题、`spl_autoload_register`、简单自动加载器示例、多个自动加载器等内容。

**章节类型**：工具性章节

**主要内容**：
- 传统方式的问题（手动 require）
- `spl_autoload_register()` 函数的使用
- 简单自动加载器的实现
- 多个自动加载器的注册
- 自动加载器的优先级
- 自动加载最佳实践

## 特性

- **按需加载**：只在需要时加载类文件
- **性能优化**：避免加载不需要的类
- **简化代码**：不需要手动 require

## 基本用法/命令

### spl_autoload_register()

**语法**：`spl_autoload_register(callable $autoloader, bool $throw = true, bool $prepend = false): bool`
**功能**：注册自动加载器
**参数**：
- `$autoloader`：自动加载函数
- `$throw`：是否在加载失败时抛出异常
- `$prepend`：是否添加到队列前面

## 基本用法

### 简单自动加载器示例

```php
<?php
declare(strict_types=1);

spl_autoload_register(function ($className) {
    $file = __DIR__ . '/' . str_replace('\\', '/', $className) . '.php';
    if (file_exists($file)) {
        require $file;
    }
});
```

### 多个自动加载器示例

```php
<?php
declare(strict_types=1);

spl_autoload_register(function ($className) {
    // 自动加载器 1
});

spl_autoload_register(function ($className) {
    // 自动加载器 2
}, false, true);  // prepend = true，添加到前面
```

## 使用场景

- **大型项目**：组织大型项目代码
- **框架开发**：框架自动加载机制
- **库开发**：库的自动加载

## 注意事项

- **性能考虑**：自动加载器不应过于复杂
- **错误处理**：处理类文件不存在的情况
- **优先级**：理解自动加载器的调用顺序

## 常见问题

### 问题 1：类文件找不到
- **原因**：自动加载器逻辑错误
- **解决**：检查自动加载器逻辑和文件路径

### 问题 2：性能问题
- **原因**：自动加载器过于复杂
- **解决**：优化自动加载器逻辑，使用缓存

## 最佳实践

- 使用 `spl_autoload_register` 注册自动加载器
- 实现简单高效的自动加载逻辑
- 处理类文件不存在的情况
- 考虑使用 Composer 自动加载（PSR-4）

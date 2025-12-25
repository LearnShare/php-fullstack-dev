# 2.13.2 PHPDoc

## 概述

PHPDoc 是 PHP 的文档注释标准。本节介绍 PHPDoc 基本语法、常用标签（`@param`、`@return`、`@throws`、`@var`、`@deprecated`）、类和方法文档、完整示例。

**章节类型**：参考性章节

**主要内容**：
- PHPDoc 基本语法
- 常用标签的使用
- 类文档注释
- 方法文档注释
- 属性文档注释
- 完整示例
- 工具支持

## 特性

- **文档生成**：可以生成 API 文档
- **IDE 支持**：IDE 可以提供代码提示
- **类型提示**：提供类型信息

## 语法/定义

### PHPDoc 语法

**语法**：`/** ... */`
**位置**：函数、类、属性之前
**格式**：多行注释

### 常用标签

**@param**：参数说明
**@return**：返回值说明
**@throws**：异常说明
**@var**：变量类型
**@deprecated**：已废弃标记

## 基本用法

### 函数文档示例

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 * 
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 * @throws InvalidArgumentException 当参数无效时
 */
function add(int $a, int $b): int
{
    return $a + $b;
}
```

### 类文档示例

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 * 
 * @package App\Service
 * @author John Doe
 * @since 1.0.0
 */
class UserService
{
    /**
     * 用户名称
     * 
     * @var string
     */
    private string $name;
}
```

## 使用场景

- **API 文档**：生成 API 文档
- **代码提示**：IDE 代码提示
- **类型检查**：静态分析工具使用

## 注意事项

- **格式规范**：遵循 PHPDoc 格式规范
- **及时更新**：代码变更时更新文档
- **完整性**：为公共 API 编写完整文档

## 常见问题

### 问题 1：文档格式不规范
- **原因**：不理解 PHPDoc 格式
- **解决**：学习 PHPDoc 规范

### 问题 2：文档过期
- **原因**：代码变更后未更新文档
- **解决**：及时更新文档

## 最佳实践

- 为公共 API 编写完整文档
- 遵循 PHPDoc 格式规范
- 及时更新文档
- 使用工具生成文档

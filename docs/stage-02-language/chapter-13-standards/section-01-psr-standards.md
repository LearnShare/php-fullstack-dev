# 2.13.1 PSR 标准

## 概述

PSR（PHP Standards Recommendations）是 PHP 社区制定的编码标准。本节介绍 PSR-1 基本编码标准、PSR-4 自动加载标准、PSR-12 编码风格扩展、命名规范及完整示例。

**章节类型**：参考性章节

**主要内容**：
- PSR-1 基本编码标准
- PSR-4 自动加载标准
- PSR-12 编码风格扩展
- 命名规范
- 代码示例
- 工具支持

## 核心内容

### PSR-1 基本编码标准

**文件要求**：PHP 标签、编码、行结束符
**类命名**：`StudlyCase`
**方法命名**：`camelCase`
**常量命名**：`UPPER_SNAKE_CASE`

### PSR-4 自动加载标准

**命名空间**：与目录结构对应
**类文件**：一个文件一个类
**自动加载**：Composer 自动加载

### PSR-12 编码风格扩展

**缩进**：4 个空格
**行长度**：建议不超过 120 字符
**关键字**：小写
**控制结构**：花括号位置

## 基本用法

### PSR-1 示例

```php
<?php
declare(strict_types=1);

namespace App\Example;

class UserService
{
    public const MAX_USERS = 100;
    
    public function getUserName(): string
    {
        return "John";
    }
}
```

### PSR-4 示例

```
app/
  Example/
    UserService.php  // namespace App\Example;
```

## 使用场景

- **团队协作**：统一编码标准
- **代码质量**：提高代码质量
- **工具支持**：IDE 和工具支持

## 注意事项

- **严格遵循**：严格遵循 PSR 标准
- **工具检查**：使用工具检查代码规范
- **持续改进**：持续改进代码质量

## 常见问题

### 问题 1：命名不规范
- **原因**：不理解 PSR 命名规范
- **解决**：学习并遵循 PSR 标准

### 问题 2：自动加载配置错误
- **原因**：PSR-4 配置不正确
- **解决**：检查 `composer.json` 配置

## 最佳实践

- 严格遵循 PSR 标准
- 使用工具检查代码规范
- 持续改进代码质量
- 参考 PSR 官方文档

# 2.14.3 PHP 8.4 新特性与示例

## 概述

PHP 8.4 引入了属性钩子、不对称可见性、`array_first/array_last`、`#[\NoDiscard]`、DOM API 改进等新特性。

**章节类型**：参考性章节

**主要内容**：
- 属性钩子（Property Hooks）
- 不对称可见性（Asymmetric Visibility）
- `array_first()` 和 `array_last()` 函数
- `#[\NoDiscard]` 属性
- DOM API 改进
- 其他新特性
- 迁移指南

## 特性

- **属性钩子**：属性的 getter 和 setter
- **不对称可见性**：属性的读写可见性不同
- **array_first/last**：获取数组首尾元素
- **#[\NoDiscard]**：标记不应丢弃的返回值

## 基本用法

### 属性钩子示例

```php
<?php
declare(strict_types=1);

class User
{
    private string $name {
        get {
            return $this->name;
        }
        set(string $value) {
            $this->name = trim($value);
        }
    }
}
```

### array_first/last 示例

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
$first = array_first($arr);  // 1
$last = array_last($arr);    // 3
```

### #[\NoDiscard] 示例

```php
<?php
declare(strict_types=1);

#[\NoDiscard]
function process(): Result
{
    // ...
}
```

## 使用场景

- **属性控制**：使用属性钩子控制属性访问
- **数组操作**：使用 `array_first/last` 简化代码
- **返回值检查**：使用 `#[\NoDiscard]` 提醒检查返回值

## 注意事项

- **版本要求**：需要 PHP 8.4+
- **兼容性**：注意向后兼容性
- **迁移成本**：评估迁移成本

## 常见问题

### 问题 1：属性钩子语法
- **语法**：新的语法需要适应
- **解决**：参考官方文档和示例

### 问题 2：array_first/last 性能
- **性能**：性能良好
- **使用**：替代手动获取首尾元素

## 最佳实践

- 了解新特性的适用场景
- 评估迁移成本
- 逐步采用新特性
- 参考官方文档

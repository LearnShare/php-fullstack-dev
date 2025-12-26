# 5.15.3 Respect/Validation

## 概述

Respect/Validation 是简洁的 PHP 验证库。本节介绍 Respect/Validation 的使用方法，包括安装配置、验证规则、链式验证等，帮助零基础学员掌握 Respect/Validation。

**章节类型**：工具性章节

**主要内容**：
- Respect/Validation 概述
- 安装配置
- 验证规则
- 链式验证
- 错误处理
- 自定义规则
- 完整示例

## 核心内容

### Respect/Validation 概述

- 库的特点
- 库的优势
- 使用场景

### 验证规则

- 基础规则
- 组合规则
- 条件规则
- 规则链

### 链式验证

- 链式语法
- 规则组合
- 错误收集
- 验证结果

### 自定义规则

- 规则编写
- 规则注册
- 规则使用
- 规则扩展

## 基本用法

### Respect/Validation 示例

```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

$validator = v::email()->notEmpty()->length(5, 50);
if ($validator->validate($email)) {
    // 验证通过
} else {
    // 验证失败
    $errors = $validator->getMessages();
}
```

## 使用场景

- 简单验证
- 链式验证
- 快速验证
- 轻量级应用

## 注意事项

- 规则链的顺序
- 错误消息
- 性能考虑
- 规则组合

## 常见问题

- 如何安装 Respect/Validation？
- 如何使用链式验证？
- 如何处理验证错误？
- 如何创建自定义规则？

## 最佳实践

- 使用链式验证提高可读性
- 提供清晰的错误消息
- 创建可复用的验证器
- 组合使用规则

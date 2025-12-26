# 5.15.2 Symfony Validator

## 概述

Symfony Validator 是强大的验证组件。本节介绍 Symfony Validator 的使用方法，包括安装配置、验证规则、验证执行等，帮助零基础学员掌握 Symfony Validator。

**章节类型**：工具性章节

**主要内容**：
- Symfony Validator 概述
- 安装配置
- 验证规则定义
- 验证执行
- 错误处理
- 自定义约束
- 完整示例

## 核心内容

### Symfony Validator 概述

- 组件功能
- 组件优势
- 使用场景

### 验证规则定义

- 注解方式
- YAML 配置
- PHP 配置
- 规则类型

### 验证执行

- Validator 使用
- 验证方法
- 错误收集
- 错误格式化

### 自定义约束

- 约束类编写
- 约束注册
- 约束使用
- 约束验证器

## 基本用法

### Symfony Validator 示例

```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Validation;
use Symfony\Component\Validator\Constraints as Assert;

$validator = Validation::createValidator();
$constraints = new Assert\Collection([
    'email' => new Assert\Email(),
    'age' => new Assert\Range(['min' => 18, 'max' => 100])
]);

$violations = $validator->validate($data, $constraints);
```

## 使用场景

- 表单验证
- API 验证
- 数据验证
- 复杂验证规则

## 注意事项

- 组件安装
- 规则定义
- 性能考虑
- 错误处理

## 常见问题

- 如何安装 Symfony Validator？
- 如何定义验证规则？
- 如何处理验证错误？
- 如何创建自定义约束？

## 最佳实践

- 使用注解简化规则定义
- 实现统一的验证机制
- 提供清晰的错误信息
- 创建可复用的约束

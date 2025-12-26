# 7.4.1 异常体系层次

## 概述

异常体系层次是异常设计的基础。本节介绍异常体系的概念、层次设计、基础异常类等，帮助零基础学员设计合理的异常体系。

**章节类型**：概念性章节

**主要内容**：
- 异常体系概念
- 异常层次设计原则
- 基础异常类设计
- 异常继承关系
- 异常分类
- 完整示例

## 核心内容

### 异常体系概念

- 什么是异常体系
- 体系的作用
- 层次的重要性

### 异常层次设计

- 基础异常类
- 特定异常类
- 继承关系
- 层次深度

### 基础异常类

- 应用异常基类
- 业务异常基类
- 系统异常基类
- 异常基类设计

### 异常分类

- 按类型分类
- 按层次分类
- 按场景分类
- 分类原则

## 基本用法

### 异常体系示例

```php
<?php
declare(strict_types=1);

// 基础异常类
abstract class AppException extends Exception {
    protected string $errorCode;
    
    public function getErrorCode(): string {
        return $this->errorCode;
    }
}

// 业务异常
class BusinessException extends AppException {
    protected string $errorCode = 'BUSINESS_ERROR';
}

// 验证异常
class ValidationException extends BusinessException {
    protected string $errorCode = 'VALIDATION_ERROR';
}
```

## 使用场景

- 所有应用开发
- 框架设计
- 异常管理
- 错误处理

## 注意事项

- 层次深度
- 异常粒度
- 继承关系
- 命名规范

## 常见问题

- 如何设计异常体系？
- 异常层次如何划分？
- 基础异常类如何设计？
- 异常分类的原则？

## 最佳实践

- 设计清晰的层次
- 使用基础异常类
- 合理分类异常
- 保持命名一致

# 7.4.2 自定义异常

## 概述

自定义异常能够更好地表达业务逻辑中的异常情况。本节介绍自定义异常的设计方法、异常属性、异常消息等，帮助零基础学员创建有意义的异常类型。

**章节类型**：实践性章节

**主要内容**：
- 自定义异常设计
- 异常属性设计
- 异常消息设计
- 异常上下文
- 异常分类
- 完整示例

## 核心内容

### 自定义异常设计

- 异常类设计
- 异常命名
- 异常属性
- 异常方法

### 异常属性

- 错误码
- 错误消息
- 上下文数据
- 时间戳

### 异常消息

- 用户友好消息
- 技术消息
- 消息国际化
- 消息模板

### 异常上下文

- 上下文信息
- 堆栈信息
- 请求信息
- 环境信息

## 基本用法

### 自定义异常示例

```php
<?php
declare(strict_types=1);

class ValidationException extends BusinessException {
    private array $errors;
    
    public function __construct(
        string $message,
        array $errors = [],
        int $code = 0,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous);
        $this->errors = $errors;
    }
    
    public function getErrors(): array {
        return $this->errors;
    }
}
```

## 使用场景

- 业务逻辑异常
- 数据验证异常
- 权限异常
- 系统异常

## 注意事项

- 异常设计
- 属性选择
- 消息设计
- 性能考虑

## 常见问题

- 如何设计自定义异常？
- 异常应该包含哪些属性？
- 如何设计异常消息？
- 异常上下文如何传递？

## 最佳实践

- 创建有意义的异常类型
- 包含足够的上下文信息
- 提供清晰的错误消息
- 实现异常序列化

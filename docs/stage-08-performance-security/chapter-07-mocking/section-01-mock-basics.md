# 8.7.1 Mock 基础

## 概述

Mock 是测试中模拟对象的重要技术。本节介绍 Mock 的概念、Mock 使用、PHPUnit Mock，帮助零基础学员理解如何在测试中使用 Mock。

**章节类型**：实践性章节

**主要内容**：
- Mock 概述
- Mock 概念
- Mock 使用
- PHPUnit Mock
- 最佳实践
- 完整示例

## 核心内容

### Mock 概述

- Mock 的作用
- 与真实对象的区别
- 优势特点

### Mock 概念

- Mock 对象
- Mock 方法
- Mock 返回值
- Mock 验证

### Mock 使用

- 创建 Mock 对象
- 配置 Mock 行为
- 验证 Mock 调用
- Mock 异常

### PHPUnit Mock

- createMock()
- getMockBuilder()
- Mock 方法配置
- Mock 验证方法

## 基本用法

### Mock 示例

```php
// Mock 示例
```

## 完整示例

### 示例：Mock 实践

```php
// 完整示例代码
```

## 注意事项

- 只 Mock 外部依赖
- 验证 Mock 调用
- 避免过度 Mock
- 保持测试简单

## 常见问题

### Q: 什么时候使用 Mock？

A: 当需要隔离被测试代码的外部依赖时使用 Mock，如数据库、API、文件系统等。

### Q: 如何验证 Mock 调用？

A: 使用 PHPUnit 的 expect() 方法验证 Mock 方法的调用次数和参数。

## 最佳实践

- 只 Mock 外部依赖
- 验证 Mock 调用
- 避免过度 Mock
- 保持测试简单

## 对比分析

### Mock vs Stub vs Fakes

- 功能对比
- 使用场景对比

## 实践任务

1. 使用 Mock 编写测试
2. 验证 Mock 调用
3. 优化 Mock 使用

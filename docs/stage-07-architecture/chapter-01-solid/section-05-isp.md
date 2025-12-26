# 7.1.5 接口隔离原则（ISP）

## 概述

接口隔离原则要求客户端不应依赖它不需要的接口。本节介绍 ISP 的概念、接口设计、接口拆分等，帮助零基础学员掌握接口隔离原则的应用。

**章节类型**：概念性章节

**主要内容**：
- ISP 概念
- 接口设计原则
- 接口拆分方法
- 违反 ISP 的示例
- 符合 ISP 的重构示例
- 完整示例

## 核心内容

### ISP 概念

- 什么是接口隔离
- 客户端的概念
- 依赖的含义

### 接口设计

- 接口的粒度
- 接口的职责
- 接口的组合

### 接口拆分

- 大接口拆分
- 按职责拆分
- 按客户端拆分

### 违反示例

- 违反 ISP 的代码
- 问题分析
- 重构方向

## 基本用法

### ISP 应用示例

```php
<?php
declare(strict_types=1);

// 违反 ISP：接口包含不需要的方法
interface Worker {
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
}

// 符合 ISP：接口按职责拆分
interface Workable {
    public function work(): void;
}

interface Eatable {
    public function eat(): void;
}

interface Sleepable {
    public function sleep(): void;
}
```

## 使用场景

- 接口设计
- 服务接口
- API 设计
- 框架设计

## 注意事项

- 接口粒度
- 拆分程度
- 实际需求
- 维护成本

## 常见问题

- 如何设计接口？
- 接口如何拆分？
- 如何避免接口过大？
- ISP 与 SRP 的关系？

## 最佳实践

- 设计细粒度接口
- 按职责拆分接口
- 避免接口污染
- 考虑客户端需求

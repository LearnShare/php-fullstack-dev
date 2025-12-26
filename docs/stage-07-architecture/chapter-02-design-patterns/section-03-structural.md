# 7.2.3 结构型模式

## 概述

结构型模式关注类和对象的组合方式。本节介绍常见的结构型模式，包括适配器模式、装饰器模式、代理模式等，帮助零基础学员掌握结构型模式的应用。

**章节类型**：语法性章节

**主要内容**：
- 结构型模式概述
- 适配器模式（Adapter Pattern）
- 装饰器模式（Decorator Pattern）
- 代理模式（Proxy Pattern）
- 外观模式（Facade Pattern）
- 组合模式（Composite Pattern）
- 完整示例

## 核心内容

### 适配器模式

- 适配器概念
- 接口适配
- 类适配器
- 对象适配器

### 装饰器模式

- 装饰器概念
- 功能增强
- 动态组合
- 应用场景

### 代理模式

- 代理概念
- 虚拟代理
- 保护代理
- 远程代理

### 外观模式

- 外观概念
- 子系统封装
- 简化接口
- 应用场景

## 基本用法

### 结构型模式示例

```php
<?php
declare(strict_types=1);

// 适配器模式
class OldService {
    public function oldMethod(): string {
        return 'old';
    }
}

class ServiceAdapter implements NewInterface {
    public function __construct(
        private OldService $oldService
    ) {}
    
    public function newMethod(): string {
        return $this->oldService->oldMethod();
    }
}
```

## 使用场景

- 接口适配
- 功能增强
- 系统封装
- 对象组合

## 注意事项

- 模式选择
- 性能影响
- 复杂度增加
- 维护成本

## 常见问题

- 适配器和装饰器的区别？
- 何时使用代理模式？
- 外观模式的作用？
- 结构型模式如何选择？

## 最佳实践

- 理解模式本质
- 根据场景选择
- 注意性能影响
- 保持设计简洁

# 7.1.6 依赖倒置原则（DIP）

## 概述

依赖倒置原则要求依赖抽象而不是具体实现。本节介绍 DIP 的概念、依赖注入、控制反转等，帮助零基础学员掌握依赖倒置原则的应用。

**章节类型**：概念性章节

**主要内容**：
- DIP 概念
- 依赖注入（DI）
- 控制反转（IoC）
- 依赖注入容器
- 违反 DIP 的示例
- 符合 DIP 的重构示例
- 完整示例

## 核心内容

### DIP 概念

- 什么是依赖倒置
- 高层的含义
- 低层的含义

### 依赖注入

- 构造函数注入
- 方法注入
- 属性注入
- 注入方式选择

### 控制反转

- IoC 概念
- IoC 容器
- 服务定位器
- 依赖解析

### 违反示例

- 违反 DIP 的代码
- 问题分析
- 重构方向

## 基本用法

### DIP 应用示例

```php
<?php
declare(strict_types=1);

// 违反 DIP：依赖具体实现
class UserService {
    private MySQLDatabase $db;
    
    public function __construct() {
        $this->db = new MySQLDatabase(); // 依赖具体类
    }
}

// 符合 DIP：依赖抽象
interface Database {
    public function query(string $sql): array;
}

class UserService {
    public function __construct(
        private Database $db // 依赖接口
    ) {}
}
```

## 使用场景

- 服务设计
- 框架设计
- 测试友好
- 解耦设计

## 注意事项

- 抽象层次
- 依赖管理
- 容器使用
- 性能考虑

## 常见问题

- 什么是依赖倒置？
- 如何实现依赖注入？
- IoC 容器的作用？
- DIP 与测试的关系？

## 最佳实践

- 依赖抽象接口
- 使用依赖注入
- 使用 IoC 容器
- 实现解耦设计

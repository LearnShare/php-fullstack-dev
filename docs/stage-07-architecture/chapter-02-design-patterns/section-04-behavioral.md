# 7.2.4 行为型模式

## 概述

行为型模式关注对象间的通信和职责分配。本节介绍常见的行为型模式，包括观察者模式、策略模式、命令模式等，帮助零基础学员掌握行为型模式的应用。

**章节类型**：语法性章节

**主要内容**：
- 行为型模式概述
- 观察者模式（Observer Pattern）
- 策略模式（Strategy Pattern）
- 命令模式（Command Pattern）
- 责任链模式（Chain of Responsibility）
- 模板方法模式（Template Method）
- 完整示例

## 核心内容

### 观察者模式

- 观察者概念
- 发布订阅
- 事件通知
- 应用场景

### 策略模式

- 策略概念
- 算法封装
- 策略选择
- 应用场景

### 命令模式

- 命令概念
- 请求封装
- 撤销操作
- 应用场景

### 责任链模式

- 责任链概念
- 请求传递
- 处理链
- 应用场景

## 基本用法

### 行为型模式示例

```php
<?php
declare(strict_types=1);

// 观察者模式
interface Observer {
    public function update(string $event): void;
}

class Subject {
    private array $observers = [];
    
    public function attach(Observer $observer): void {
        $this->observers[] = $observer;
    }
    
    public function notify(string $event): void {
        foreach ($this->observers as $observer) {
            $observer->update($event);
        }
    }
}
```

## 使用场景

- 事件处理
- 算法选择
- 请求处理
- 职责分配

## 注意事项

- 模式选择
- 性能考虑
- 复杂度管理
- 维护成本

## 常见问题

- 观察者和发布订阅的区别？
- 策略模式的应用场景？
- 命令模式的优势？
- 行为型模式如何选择？

## 最佳实践

- 理解模式本质
- 根据场景选择
- 注意性能影响
- 保持设计清晰

# 7.2.1 设计模式概述

## 概述

设计模式是软件设计中常见问题的可重用解决方案。它们是经过时间检验的最佳实践，由四位著名软件工程师 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 在 1994 年出版的《设计模式：可复用面向对象软件的基础》一书中系统化地提出。这本书因此被称为"四人帮"（Gang of Four，简称 GoF）著作。

设计模式不是具体的代码库或框架，而是一种思维方式，一种解决特定类型问题的模板。它们帮助开发者避免重复发明轮子，利用前人的经验和智慧来解决相似的问题。掌握设计模式是成为高级软件工程师的必经之路，它能够显著提升代码质量、降低维护成本、促进团队协作。

然而，设计模式也不是万能的。过度使用或不当使用设计模式可能导致代码过度复杂，难以理解和维护。因此，理解每种模式的适用场景和优缺点至关重要。本节将详细介绍设计模式的概念、分类、以及如何在实际开发中正确应用。

**主要内容**：
- 设计模式的定义和历史
- 设计模式的三大分类
- 23 种经典设计模式概览
- 设计模式的作用和价值
- 如何选择和使用设计模式

## 设计模式的定义

### 什么是设计模式

设计模式（Design Pattern）是针对软件设计中常见问题的可重用解决方案。它们是经过实践验证的代码结构和方法，能够帮助开发者解决特定类型的设计问题。

设计模式具有以下特点：
- **经过验证**：已经在实际项目中被证明有效
- **可重用**：可以在不同的项目中应用
- **上下文相关**：适用于特定的问题场景
- **提供共享词汇**：为团队提供共同的设计语言

### 设计模式的组成

一个完整的设计模式通常包含以下要素：

1. **模式名称（Name）**：模式的简明描述
2. **问题（Problem）**：模式要解决的问题
3. **解决方案（Solution）**：模式的通用设计
4. **后果（Consequences）**：应用模式的利弊权衡

## 设计模式的历史

### GoF 模式的诞生

1994 年，Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 四位作者出版了《设计模式：可复用面向对象软件的基础》一书，系统总结了 23 种经典的设计模式。这本书奠定了现代软件设计模式的基础，至今仍是软件工程领域的经典著作。

### 模式的演进

自 GoF 模式诞生以来，设计模式的概念不断扩展：
- 出现了更多针对特定领域的模式
- 一些模式被重新解释和优化
- 新的编程范式催生了新的模式

## 设计模式的分类

### 创建型模式（Creational Patterns）

创建型模式关注对象的创建机制，旨在封装对象创建的过程，使代码与具体类的创建解耦。

**包含的模式**：
- 简单工厂模式（Simple Factory）
- 工厂方法模式（Factory Method）
- 抽象工厂模式（Abstract Factory）
- 建造者模式（Builder）
- 原型模式（Prototype）
- 单例模式（Singleton）

### 结构型模式（Structural Patterns）

结构型模式关注对象之间的组合关系，旨在通过建立对象之间的结构来解决复杂性问题。

**包含的模式**：
- 适配器模式（Adapter）
- 桥接模式（Bridge）
- 组合模式（Composite）
- 装饰器模式（Decorator）
- 外观模式（Facade）
- 代理模式（Proxy）

### 行为型模式（Behavioral Patterns）

行为型模式关注对象之间的通信和职责分配，旨在解决对象之间的交互问题。

**包含的模式**：
- 职责链模式（Chain of Responsibility）
- 命令模式（Command）
- 迭代器模式（Iterator）
- 中介者模式（Mediator）
- 备忘录模式（Memento）
- 观察者模式（Observer）
- 状态模式（State）
- 策略模式（Strategy）
- 模板方法模式（Template Method）
- 访问者模式（Visitor）

## 23 种经典设计模式概览

### 创建型模式详解

| 模式 | 描述 | 适用场景 |
|:-----|:-----|:---------|
| 简单工厂 | 根据参数创建不同对象 | 对象创建逻辑简单 |
| 工厂方法 | 子类决定创建哪种对象 | 需要扩展对象创建 |
| 抽象工厂 | 创建一系列相关对象 | 需要创建产品族 |
| 建造者 | 分步构建复杂对象 | 对象创建步骤多 |
| 原型 | 克隆现有对象 | 对象创建成本高 |
| 单例 | 类只有一个实例 | 需要全局唯一实例 |

### 结构型模式详解

| 模式 | 描述 | 适用场景 |
|:-----|:-----|:---------|
| 适配器 | 转换接口以适配客户 | 已有接口不兼容 |
| 桥接 | 分离抽象与实现 | 抽象与实现独立变化 |
| 组合 | 树形结构统一处理 | 部分-整体层次结构 |
| 装饰器 | 动态添加功能 | 需要动态添加职责 |
| 外观 | 统一复杂子系统 | 简化复杂系统接口 |
| 代理 | 控制对象访问 | 需要延迟加载或访问控制 |

### 行为型模式详解

| 模式 | 描述 | 适用场景 |
|:-----|:-----|:---------|
| 职责链 | 对象链式处理请求 | 多个对象处理请求 |
| 命令 | 请求封装为对象 | 需要撤销/重做功能 |
| 迭代器 | 顺序访问集合元素 | 统一遍历不同集合 |
| 中介者 | 对象间通信中介 | 对象间依赖复杂 |
| 备忘录 | 保存和恢复对象状态 | 需要撤销功能 |
| 观察者 | 一对多依赖通知 | 状态变化通知 |
| 状态 | 对象状态影响行为 | 行为随状态变化 |
| 策略 | 封装可互换算法 | 多种算法可选 |
| 模板方法 | 算法骨架子类实现 | 算法步骤固定 |
| 访问者 | 操作分离到访问者 | 数据结构稳定 |

## 设计模式的作用

### 解决常见设计问题

设计模式提供了针对常见设计问题的标准解决方案：
- 对象的创建和管理
- 对象之间的关系和通信
- 复杂系统的简化
- 代码的可扩展性和可维护性

### 促进代码复用

通过设计模式：
- 开发者可以利用经过验证的解决方案
- 减少重复设计和实现
- 提高开发效率

### 提供共享词汇

设计模式为团队提供了共同的设计语言：
- 开发者可以用模式名称交流
- 代码更易理解
- 文档更简洁

### 提高代码质量

遵循设计模式的代码通常：
- 更加灵活
- 更容易维护
- 更容易扩展
- 更好地遵循 SOLID 原则

## 基本用法

### 示例：使用工厂模式创建对象

```php
<?php
declare(strict_types=1);

// 产品接口
interface Notification
{
    public function send(string $message): void;
}

// 具体产品
class EmailNotification implements Notification
{
    public function send(string $message): void
    {
        echo "Email: {$message}\n";
    }
}

class SMSNotification implements Notification
{
    public function send(string $message): void
    {
        echo "SMS: {$message}\n";
    }
}

class PushNotification implements Notification
{
    public function send(string $message): void
    {
        echo "Push: {$message}\n";
    }
}

// 工厂类
class NotificationFactory
{
    public static function create(string $type): Notification
    {
        return match($type) {
            'email' => new EmailNotification(),
            'sms' => new SMSNotification(),
            'push' => new PushNotification(),
            default => throw new InvalidArgumentException(
                "Unknown notification type: {$type}"
            ),
        };
    }
}

// 使用示例
$email = NotificationFactory::create('email');
$email->send('Hello via Email');

$sms = NotificationFactory::create('sms');
$sms->send('Hello via SMS');
```

**输出**：

```
Email: Hello via Email
SMS: Hello via SMS
```

### 示例：使用单例模式确保唯一实例

```php
<?php
declare(strict_types=1);

// 数据库连接单例
class DatabaseConnection
{
    private static ?DatabaseConnection $instance = null;
    private PDO $pdo;
    
    // 私有构造函数，防止直接实例化
    private function __construct()
    {
        $this->pdo = new PDO(
            'mysql:host=localhost;dbname=test',
            'root',
            '',
            [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
        );
    }
    
    // 获取唯一实例
    public static function getInstance(): DatabaseConnection
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        
        return self::$instance;
    }
    
    // 防止克隆
    private function __clone() {}
    
    // 防止反序列化
    public function __wakeup()
    {
        throw new Exception("Cannot unserialize singleton");
    }
    
    public function query(string $sql): array
    {
        $stmt = $this->pdo->query($sql);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}

// 使用示例
$db1 = DatabaseConnection::getInstance();
$db2 = DatabaseConnection::getInstance();

echo "Same instance: " . ($db1 === $db2 ? 'Yes' : 'No') . "\n";
```

**输出**：

```
Same instance: Yes
```

## 使用场景

### 何时使用设计模式

- 遇到常见的设计问题
- 需要创建可扩展的架构
- 代码需要长期维护
- 团队需要共同的设计语言

### 何时不使用设计模式

- 问题简单，模式过于复杂
- 项目是短期的原型
- 过度设计会影响开发效率

## 注意事项

### 避免模式滥用

常见问题：
- 为小问题强行使用模式
- 堆砌多个模式导致复杂化
- 不理解模式的核心思想

### 理解模式本质

每种模式都有其核心思想：
- 工厂模式：封装对象创建
- 单例模式：控制实例数量
- 策略模式：封装算法
- 观察者模式：解耦通知

### 结合实际场景

选择模式时考虑：
- 问题的具体场景
- 团队的技术水平
- 项目的维护周期

## 常见问题

### 什么是设计模式？

设计模式是针对软件设计中常见问题的可重用解决方案。它们是经过实践验证的代码结构和最佳实践。

### 为什么需要设计模式？

设计模式帮助开发者：
- 利用前人的经验
- 解决常见设计问题
- 编写更易维护的代码
- 促进团队协作

### 如何选择设计模式？

1. 分析问题类型（创建、结构、行为）
2. 识别问题的具体场景
3. 比较类似模式的适用性
4. 权衡模式的利弊

### 如何避免模式滥用？

- 理解模式的核心思想，而不是表面形式
- 只在真正需要时使用模式
- 优先考虑简单的解决方案
- 不要为了使用模式而使用模式

## 最佳实践

1. **从基础开始**：先掌握 SOLID 原则，再学习设计模式

2. **理解模式本质**：理解每种模式解决的问题，而不仅仅是代码结构

3. **在实践中应用**：在真实项目中应用模式，而不是纸上谈兵

4. **避免教条主义**：模式是工具，不是教条

5. **持续学习**：设计模式是一个庞大的知识体系，需要持续学习

## 练习任务

1. **分析现有代码**：找出你过去项目中使用过的设计模式，分析其作用

2. **选择模式解决问题**：为一个具体的设计问题选择合适的模式，并实现

3. **对比模式**：比较工厂方法和抽象工厂模式的适用场景

4. **重构练习**：将一个不使用模式的代码重构为使用适当模式

5. **模式组合**：分析一个真实项目中使用多个设计模式的案例

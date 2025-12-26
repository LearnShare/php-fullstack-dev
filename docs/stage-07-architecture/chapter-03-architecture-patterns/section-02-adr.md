# 7.3.2 ADR 架构模式

## 概述

ADR 是 MVC 的改进架构模式。本节介绍 ADR 的概念、组成部分、与 MVC 的区别等，帮助零基础学员理解 ADR 架构模式。

**章节类型**：概念性章节

**主要内容**：
- ADR 概念
- 动作（Action）层
- 域（Domain）层
- 响应器（Responder）层
- ADR 实现
- ADR vs MVC
- 完整示例

## 核心内容

### ADR 概念

- 什么是 ADR
- ADR 的组成部分
- ADR 的优势

### 动作层

- 动作的作用
- 请求处理
- 动作设计
- 单一职责

### 域层

- 域的作用
- 业务逻辑
- 领域模型
- 域设计

### 响应器层

- 响应器的作用
- 响应格式化
- 响应器设计
- 响应类型

## 基本用法

### ADR 实现示例

```php
<?php
declare(strict_types=1);

// Action
class ShowUserAction {
    public function __construct(
        private UserDomain $domain,
        private UserResponder $responder
    ) {}
    
    public function __invoke(int $id): Response {
        $user = $this->domain->findUser($id);
        return $this->responder->respond($user);
    }
}

// Domain
class UserDomain {
    public function findUser(int $id): User {
        // 业务逻辑
    }
}

// Responder
class UserResponder {
    public function respond(User $user): Response {
        // 响应格式化
    }
}
```

## 使用场景

- API 开发
- 现代 Web 应用
- 单一职责要求
- 清晰架构

## 注意事项

- 动作粒度
- 域设计
- 响应器设计
- 架构一致性

## 常见问题

- ADR 和 MVC 的区别？
- 何时使用 ADR？
- 如何设计动作？
- ADR 的优势？

## 最佳实践

- 单一动作职责
- 清晰的域设计
- 灵活的响应器
- 保持架构一致

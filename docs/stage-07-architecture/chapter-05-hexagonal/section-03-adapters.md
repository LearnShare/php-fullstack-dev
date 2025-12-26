# 7.5.3 Adapters（适配器）

## 概述

适配器实现了端口与外部世界的连接。本节介绍适配器的概念、输入适配器、输出适配器的实现方法，帮助零基础学员掌握适配器设计。

**章节类型**：实践性章节

**主要内容**：
- 适配器概念
- 输入适配器（HTTP、CLI、消息队列）
- 输出适配器（数据库、文件系统、外部 API）
- 适配器实现
- 适配器测试
- 完整示例

## 核心内容

### 适配器概念

- 什么是适配器
- 适配器的作用
- 适配器的类型

### 输入适配器

- HTTP 适配器
- CLI 适配器
- 消息队列适配器
- 适配器实现

### 输出适配器

- 数据库适配器
- 文件系统适配器
- 外部 API 适配器
- 适配器实现

### 适配器实现

- 接口实现
- 数据转换
- 错误处理
- 适配器测试

## 基本用法

### 适配器实现示例

```php
<?php
declare(strict_types=1);

// 输入适配器（HTTP）
class HttpUserController {
    public function __construct(
        private CreateUserUseCase $useCase
    ) {}
    
    public function create(Request $request): Response {
        $user = $this->useCase->execute(
            new CreateUserRequest($request->get('name'))
        );
        return new Response(['user' => $user], 201);
    }
}

// 输出适配器（数据库）
class DatabaseUserRepository implements UserRepository {
    public function save(User $user): void {
        // 数据库保存
    }
}
```

## 使用场景

- 技术集成
- 外部系统对接
- 数据源切换
- 测试隔离

## 注意事项

- 适配器职责
- 数据转换
- 错误处理
- 性能考虑

## 常见问题

- 什么是适配器？
- 如何实现适配器？
- 适配器如何测试？
- 适配器的职责？

## 最佳实践

- 实现端口接口
- 处理数据转换
- 实现错误处理
- 保持适配器简单

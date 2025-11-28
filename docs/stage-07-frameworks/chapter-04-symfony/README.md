# 7.4 Symfony 深度应用

## 目标

- 了解 Symfony 的核心组件和架构。
- 掌握 Messenger（消息队列和总线）。
- 熟悉 Workflow（业务流程状态机）。
- 了解 API Platform 的使用。
- 了解 Symfony 7.0 的新特性和改进。

## Symfony 版本说明

- **Symfony 7.0**：最新稳定版本（2024 年 11 月发布）
  - 要求 PHP 8.2+
  - 性能优化和改进
  - 新的组件和功能
- **Symfony 6.x**：长期支持版本（LTS）
  - 支持至 2027 年
  - 适合生产环境使用

## Symfony 核心

### HttpFoundation

```php
<?php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

$request = Request::createFromGlobals();
$response = new Response('Hello World', 200);
$response->send();
```

### Dependency Injection

```php
<?php
// services.yaml
services:
    App\Service\UserService:
        arguments:
            $repository: '@App\Repository\UserRepository'
```

## Messenger

### 消息定义

```php
<?php
class CreateUserMessage
{
    public function __construct(
        public string $name,
        public string $email
    ) {
    }
}
```

### 消息处理

```php
<?php
use Symfony\Component\Messenger\Handler\MessageHandlerInterface;

class CreateUserHandler implements MessageHandlerInterface
{
    public function __invoke(CreateUserMessage $message): void
    {
        // 处理消息
        createUser($message->name, $message->email);
    }
}
```

## Workflow

### 状态机定义

```yaml
# config/packages/workflow.yaml
framework:
    workflows:
        order:
            type: 'state_machine'
            marking_store:
                type: 'method'
                property: 'status'
            supports:
                - App\Entity\Order
            initial_marking: pending
            places:
                - pending
                - processing
                - shipped
                - delivered
            transitions:
                process:
                    from: pending
                    to: processing
                ship:
                    from: processing
                    to: shipped
                deliver:
                    from: shipped
                    to: delivered
```

## API Platform

### 实体定义

```php
<?php
use ApiPlatform\Metadata\ApiResource;

#[ApiResource]
class User
{
    public int $id;
    public string $name;
    public string $email;
}
```

## Symfony 7.0 新特性

### 性能改进

- 更快的组件加载
- 优化的依赖注入容器
- 改进的路由匹配性能

### 新的组件和功能

- 改进的 Messenger 组件
- 增强的 Workflow 组件
- 更新的 API Platform 集成

### PHP 8.2+ 特性支持

- 充分利用 PHP 8.2+ 的新特性
- 更好的类型推断
- 改进的错误处理

## 练习

1. 创建一个 Symfony 7.0 项目，实现 RESTful API。

2. 使用 Messenger 实现异步任务处理。

3. 配置 Workflow 管理订单状态流转。

4. 使用 API Platform 自动生成 API 文档。

5. 实现一个完整的 Symfony 应用。

6. 对比 Laravel 和 Symfony 的设计差异。

7. 升级现有 Symfony 6.x 项目到 Symfony 7.0。

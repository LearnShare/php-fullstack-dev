# 3.7.2 ADR 架构模式

## 概述

ADR（Action-Domain-Responder）是 MVC 的改进版本，通过更清晰的职责划分解决了 MVC 的一些问题。ADR 强调每个 Action 处理单个请求，Domain 包含业务逻辑，Responder 负责构建响应。

## ADR 核心概念

### 三个组成部分

ADR（Action-Domain-Responder）是 MVC 的改进版本：

- **Action（动作）**：处理单个 HTTP 请求，协调 Domain 和 Responder。
- **Domain（领域）**：包含业务逻辑，不依赖框架。
- **Responder（响应器）**：构建 HTTP 响应（JSON、HTML 等）。

```
HTTP 请求
    ↓
Action（处理请求、调用 Domain、使用 Responder）
    ↓
Domain（业务逻辑）
    ↓
Responder（构建响应）
    ↓
HTTP 响应
```

## ADR 实现示例

### Domain 层

```php
namespace App\Domain;

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {
    }

    public function getUserById(int $id): User
    {
        $user = $this->repository->find($id);
        
        if ($user === null) {
            throw new UserNotFoundException($id);
        }

        return $user;
    }

    public function createUser(string $name, string $email): User
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($email);
        }

        $user = new User(null, $name, $email);
        $this->repository->save($user);
        
        return $user;
    }
}
```

### Action 层

```php
namespace App\Actions;

use App\Domain\UserService;
use App\Responders\JsonResponder;
use App\Responders\HtmlResponder;

class ShowUserAction
{
    public function __construct(
        private UserService $userService,
        private JsonResponder $jsonResponder,
        private HtmlResponder $htmlResponder
    ) {
    }

    public function __invoke(int $id, string $format = 'json'): Response
    {
        try {
            $user = $this->userService->getUserById($id);
            
            return match ($format) {
                'json' => $this->jsonResponder->respond($user),
                'html' => $this->htmlResponder->respond($user),
                default => throw new InvalidArgumentException("Unsupported format: {$format}"),
            };
        } catch (UserNotFoundException $e) {
            return $this->jsonResponder->error($e->getMessage(), 404);
        }
    }
}
```

### Responder 层

```php
namespace App\Responders;

class JsonResponder
{
    public function respond(mixed $data): Response
    {
        return new Response(
            json_encode($data, JSON_UNESCAPED_UNICODE),
            200,
            ['Content-Type' => 'application/json']
        );
    }

    public function error(string $message, int $code = 500): Response
    {
        return new Response(
            json_encode(['error' => $message], JSON_UNESCAPED_UNICODE),
            $code,
            ['Content-Type' => 'application/json']
        );
    }
}

class HtmlResponder
{
    public function respond(User $user): Response
    {
        $html = "
            <!DOCTYPE html>
            <html>
            <head><title>User Profile</title></head>
            <body>
                <h1>{$user->getName()}</h1>
                <p>Email: {$user->getEmail()}</p>
            </body>
            </html>
        ";

        return new Response($html, 200, ['Content-Type' => 'text/html']);
    }
}
```

## ADR vs MVC

### 对比

| 特性           | MVC                    | ADR                        |
| :------------- | :--------------------- | :------------------------- |
| Controller     | 处理多个动作           | 每个 Action 处理单个请求   |
| 响应构建       | 通常在 Controller 中   | 独立的 Responder 层        |
| 职责划分       | 可能模糊               | 更清晰                     |
| 测试           | 需要模拟 View          | 更容易测试                 |
| 适用场景       | 传统 Web 应用          | API 和 Web 应用            |

### ADR 优势

- **单一职责**：每个 Action 只处理一个请求。
- **响应解耦**：Responder 独立，可以轻松切换响应格式。
- **领域独立**：Domain 层不依赖框架，易于测试。

## 完整示例

```php
<?php
declare(strict_types=1);

// Domain
namespace App\Domain;

class OrderService
{
    public function __construct(
        private OrderRepository $repository
    ) {
    }

    public function getOrder(int $id): Order
    {
        $order = $this->repository->find($id);
        if ($order === null) {
            throw new OrderNotFoundException($id);
        }
        return $order;
    }
}

// Action
namespace App\Actions;

use App\Domain\OrderService;
use App\Responders\JsonResponder;

class ShowOrderAction
{
    public function __construct(
        private OrderService $orderService,
        private JsonResponder $responder
    ) {
    }

    public function __invoke(int $id): Response
    {
        try {
            $order = $this->orderService->getOrder($id);
            return $this->responder->respond($order->toArray());
        } catch (OrderNotFoundException $e) {
            return $this->responder->error($e->getMessage(), 404);
        }
    }
}

// Responder
namespace App\Responders;

class JsonResponder
{
    public function respond(mixed $data): Response
    {
        return new Response(
            json_encode($data),
            200,
            ['Content-Type' => 'application/json']
        );
    }

    public function error(string $message, int $code): Response
    {
        return new Response(
            json_encode(['error' => $message]),
            $code,
            ['Content-Type' => 'application/json']
        );
    }
}
```

## 注意事项

1. **单一职责**：每个 Action 只处理一个请求，保持简洁。

2. **Domain 独立性**：Domain 层不应该依赖框架或 HTTP 相关代码。

3. **Responder 复用**：Responder 可以在多个 Action 中复用。

4. **错误处理**：在 Action 层处理异常，使用 Responder 返回错误响应。

5. **测试友好**：ADR 结构更容易进行单元测试和集成测试。

## 练习

1. 实现一个完整的 ADR 应用，包含 User 的 Domain、Action 和 Responder。

2. 创建一个支持多种响应格式（JSON、HTML、XML）的 Responder。

3. 实现一个 Order ADR 应用，包含创建、查看、更新功能。

4. 设计一个路由系统，将请求分发到对应的 Action。

5. 创建一个可测试的 ADR 结构，各层可以独立测试。

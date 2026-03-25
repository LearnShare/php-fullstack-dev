# 7.3.2 ADR 架构模式

## 概述

ADR（Action-Domain-Responder）是由 Paul M. Jones 在 2013 年提出的一种 Web 应用架构模式。它是对传统 MVC 模式的一种改进和简化，旨在解决 MVC 在现代 Web 应用开发中的一些问题。

ADR 的核心思想是将传统的 MVC 模式简化为三个更清晰的层次：动作（Action）负责协调请求处理，域（Domain）负责业务逻辑，响应器（Responder）负责生成响应。这种分离使得代码更加清晰、更容易理解和测试。

ADR 特别适合构建 API 驱动的应用和现代 Web 服务。在 PHP 社区中，ADR 已经被多个框架采用作为核心架构模式。

**主要内容**：
- ADR 架构模式的定义和背景
- Action（动作）层的职责和实现
- Domain（域）层的职责和实现
- Responder（响应器）层的职责和实现
- ADR 与 MVC 的对比

## ADR 架构详解

### ADR 的起源

ADR 是对 MVC 模式的一种演进：
- 2013 年：Paul M. Jones 提出 ADR 概念
- 针对问题：MVC 在现代 Web 应用中的局限性
- 核心理念：简化层次，明确职责

### ADR 与 MVC 的对比

| 方面 | MVC | ADR |
|:-----|:-----|:-----|
| 层次数量 | 3 层 | 3 层 |
| 控制器职责 | 处理请求 + 选择视图 | 仅协调 |
| 模型职责 | 数据 + 业务逻辑 | 主要是业务逻辑 |
| 视图职责 | 渲染输出 | 生成响应 |
| 适用场景 | 传统 Web 应用 | API 应用 |

## 动作层（Action）

动作层负责接收 HTTP 请求并协调域和响应器。动作应该是单一的、专注的，每个动作对应一个用户操作。

### 动作的设计

```php
<?php
declare(strict_types=1);

// 动作基类
abstract class Action
{
    abstract public function __invoke(Request $request): Response;
}

// 用户列表动作
class UserListAction
{
    private UserDomain $domain;
    private UserResponder $responder;
    
    public function __construct(UserDomain $domain, UserResponder $responder)
    {
        $this->domain = $domain;
        $this->responder = $responder;
    }
    
    public function __invoke(Request $request): Response
    {
        // 获取查询参数
        $page = (int)($request->get('page', 1));
        $limit = (int)($request->get('limit', 20));
        
        // 调用域层
        $users = $this->domain->findUsers($page, $limit);
        $total = $this->domain->countUsers();
        
        // 返回响应
        return $this->responder->respond([
            'users' => $users,
            'pagination' => [
                'page' => $page,
                'limit' => $limit,
                'total' => $total,
            ]
        ]);
    }
}

// 用户详情动作
class UserShowAction
{
    private UserDomain $domain;
    private UserResponder $responder;
    private ErrorResponder $errorResponder;
    
    public function __construct(
        UserDomain $domain,
        UserResponder $responder,
        ErrorResponder $errorResponder
    ) {
        $this->domain = $domain;
        $this->responder = $responder;
        $this->errorResponder = $errorResponder;
    }
    
    public function __invoke(Request $request): Response
    {
        $id = (int)$request->get('id');
        
        if ($id <= 0) {
            return $this->errorResponder->respondBadRequest('Invalid user ID');
        }
        
        $user = $this->domain->findUser($id);
        
        if ($user === null) {
            return $this->errorResponder->respondNotFound('User not found');
        }
        
        return $this->responder->respond($user);
    }
}

// 用户创建动作
class UserCreateAction
{
    private UserDomain $domain;
    private UserResponder $responder;
    private ErrorResponder $errorResponder;
    
    public function __construct(
        UserDomain $domain,
        UserResponder $responder,
        ErrorResponder $errorResponder
    ) {
        $this->domain = $domain;
        $this->responder = $responder;
        $this->errorResponder = $errorResponder;
    }
    
    public function __invoke(Request $request): Response
    {
        $data = $request->getBody();
        
        // 验证输入
        $errors = $this->validate($data);
        if (!empty($errors)) {
            return $this->errorResponder->respondValidationError($errors);
        }
        
        // 创建用户
        $user = $this->domain->createUser($data);
        
        return $this->responder->respondCreated($user);
    }
    
    private function validate(array $data): array
    {
        $errors = [];
        
        if (empty($data['name'])) {
            $errors['name'] = 'Name is required';
        }
        
        if (empty($data['email'])) {
            $errors['email'] = 'Email is required';
        } elseif (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors['email'] = 'Invalid email format';
        }
        
        return $errors;
    }
}
```

## 域层（Domain）

域层负责所有的业务逻辑。在 ADR 中，域类似于 MVC 中的 Model，但更加纯粹，主要关注业务逻辑而非数据访问。

### 域的设计

```php
<?php
declare(strict_types=1);

// 域基类
abstract class Domain
{
    // 通用域逻辑
}

// 用户域
class UserDomain extends Domain
{
    private UserRepository $repository;
    private EmailService $emailService;
    
    public function __construct(UserRepository $repository, EmailService $emailService)
    {
        $this->repository = $repository;
        $this->emailService = $emailService;
    }
    
    public function findUser(int $id): ?User
    {
        if ($id <= 0) {
            return null;
        }
        
        return $this->repository->findById($id);
    }
    
    public function findUsers(int $page = 1, int $limit = 20): array
    {
        $offset = ($page - 1) * $limit;
        return $this->repository->findAll($limit, $offset);
    }
    
    public function countUsers(): int
    {
        return $this->repository->count();
    }
    
    public function createUser(array $data): User
    {
        // 业务逻辑：创建用户
        $user = new User(
            $data['name'],
            $data['email']
        );
        
        // 生成激活码
        $user->generateActivationCode();
        
        // 保存
        $this->repository->save($user);
        
        // 发送激活邮件
        $this->emailService->sendActivationEmail($user);
        
        return $user;
    }
    
    public function activateUser(int $id, string $activationCode): bool
    {
        $user = $this->repository->findById($id);
        
        if ($user === null) {
            return false;
        }
        
        if ($user->getActivationCode() !== $activationCode) {
            return false;
        }
        
        $user->activate();
        $this->repository->save($user);
        
        return true;
    }
    
    public function deleteUser(int $id): bool
    {
        $user = $this->repository->findById($id);
        
        if ($user === null) {
            return false;
        }
        
        $this->repository->delete($id);
        
        return true;
    }
}

// 订单域
class OrderDomain extends Domain
{
    private OrderRepository $repository;
    private InventoryService $inventory;
    private PaymentGateway $payment;
    
    public function __construct(
        OrderRepository $repository,
        InventoryService $inventory,
        PaymentGateway $payment
    ) {
        $this->repository = $repository;
        $this->inventory = $inventory;
        $this->payment = $payment;
    }
    
    public function createOrder(array $data): Order
    {
        // 验证库存
        foreach ($data['items'] as $item) {
            if (!$this->inventory->checkAvailability($item['product_id'], $item['quantity'])) {
                throw new DomainException(
                    "Product {$item['product_id']} is out of stock"
                );
            }
        }
        
        // 创建订单
        $order = new Order(
            $data['customer_id'],
            $data['items']
        );
        
        // 扣减库存
        foreach ($data['items'] as $item) {
            $this->inventory->reserve($item['product_id'], $item['quantity']);
        }
        
        // 处理支付
        $paymentResult = $this->payment->process($order->getTotal());
        
        if (!$paymentResult->isSuccess()) {
            // 释放库存
            foreach ($data['items'] as $item) {
                $this->inventory->release($item['product_id'], $item['quantity']);
            }
            
            throw new DomainException("Payment failed: " . $paymentResult->getMessage());
        }
        
        // 保存订单
        $this->repository->save($order);
        
        return $order;
    }
}
```

## 响应器层（Responder）

响应器层负责生成 HTTP 响应。在 ADR 中，响应器完全负责响应的格式和内容，使得响应格式可以独立于业务逻辑进行变化。

### 响应器的设计

```php
<?php
declare(strict_types=1);

// 响应器接口
interface Responder
{
    public function respond(mixed $data): Response;
}

// 成功响应器
class SuccessResponder implements Responder
{
    public function respond(mixed $data): Response
    {
        return Response::json([
            'success' => true,
            'data' => $data
        ], 200);
    }
}

// 用户响应器
class UserResponder implements Responder
{
    public function respond(mixed $data): Response
    {
        if ($data instanceof User) {
            return Response::json([
                'success' => true,
                'data' => $data->toArray()
            ], 200);
        }
        
        if (is_array($data)) {
            return Response::json([
                'success' => true,
                'data' => array_map(
                    fn(User $user) => $user->toArray(),
                    $data
                )
            ], 200);
        }
        
        return Response::json(['success' => true], 200);
    }
    
    public function respondCreated(User $user): Response
    {
        return Response::json([
            'success' => true,
            'data' => $user->toArray()
        ], 201);
    }
}

// 错误响应器
class ErrorResponder implements Responder
{
    public function respond(mixed $data): Response
    {
        return Response::json([
            'success' => false,
            'error' => $data
        ], 400);
    }
    
    public function respondBadRequest(string $message): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'BAD_REQUEST',
                'message' => $message
            ]
        ], 400);
    }
    
    public function respondNotFound(string $message): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'NOT_FOUND',
                'message' => $message
            ]
        ], 404);
    }
    
    public function respondValidationError(array $errors): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'VALIDATION_ERROR',
                'message' => 'Validation failed',
                'details' => $errors
            ]
        ], 422);
    }
    
    public function respondServerError(string $message): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'INTERNAL_ERROR',
                'message' => $message
            ]
        ], 500);
    }
}
```

## ADR 实现示例

```php
<?php
declare(strict_types=1);

// 完整的 ADR 实现

// 依赖注入容器
class Container
{
    private array $bindings = [];
    
    public function bind(string $abstract, mixed $concrete): void
    {
        $this->bindings[$abstract] = $concrete;
    }
    
    public function make(string $abstract): mixed
    {
        if (isset($this->bindings[$abstract])) {
            $binding = $this->bindings[$abstract];
            
            if (is_callable($binding)) {
                return $binding($this);
            }
            
            return $binding;
        }
        
        throw new RuntimeException("No binding for: {$abstract}");
    }
}

// 路由配置
class Router
{
    private array $routes = [];
    
    public function get(string $path, array $handler): self
    {
        $this->routes['GET'][$path] = $handler;
        return $this;
    }
    
    public function post(string $path, array $handler): self
    {
        $this->routes['POST'][$path] = $handler;
        return $this;
    }
    
    public function dispatch(string $method, string $path, Container $container): Response
    {
        $handler = $this->routes[$method][$path] ?? null;
        
        if ($handler === null) {
            return Response::json([
                'success' => false,
                'error' => ['code' => 'NOT_FOUND', 'message' => 'Route not found']
            ], 404);
        }
        
        [$actionClass, $method] = $handler;
        
        $action = $container->make($actionClass);
        $request = new Request($_GET, $_POST, $_SERVER, getallheaders());
        
        return $action->$method($request);
    }
}

// 使用示例
$container = new Container();

// 绑定依赖
$container->bind(UserDomain::class, function($c) {
    return new UserDomain(
        $c->make(UserRepository::class),
        $c->make(EmailService::class)
    );
});

$container->bind(UserResponder::class, new UserResponder());
$container->bind(ErrorResponder::class, new ErrorResponder());

// 配置路由
$router = new Router();
$router->get('/users', [UserListAction::class, '__invoke']);
$router->get('/users/{id}', [UserShowAction::class, '__invoke']);
$router->post('/users', [UserCreateAction::class, '__invoke']);

// 运行
$response = $router->dispatch(
    $_SERVER['REQUEST_METHOD'],
    $_SERVER['REQUEST_URI'],
    $container
);

$response->send();
```

## ADR vs MVC

### 主要区别

| 方面 | MVC | ADR |
|:-----|:-----|:-----|
| Controller 职责 | 接收请求 + 选择视图 | 仅协调（调用域） |
| Model 职责 | 数据 + 业务逻辑 | 仅业务逻辑 |
| View 职责 | 渲染视图 | 生成响应 |
| 层次清晰度 | 较模糊 | 更清晰 |

### ADR 的优势

1. **更清晰的职责分离**：三层各司其职
2. **更好的可测试性**：每个组件都可以独立测试
3. **更灵活的响应**：响应器可以独立变化
4. **更适合 API**：特别适合构建 REST API

### 何时使用 ADR

- 构建 RESTful API
- 需要清晰的代码组织
- 需要高可测试性
- 团队对 ADR 有共识

## 练习任务

1. **实现 ADR 框架**：使用 PHP 实现一个简单的 ADR 框架

2. **重构 MVC 为 ADR**：将一个 MVC 项目重构为 ADR 架构

3. **设计响应器系统**：设计一个支持多种响应格式的响应器系统

4. **对比分析**：分析 ADR 和 MVC 在实际项目中的优缺点

5. **实践域设计**：为一个业务领域设计完整的域层

# 3.12 微服务架构

## 目标

- 理解微服务架构的核心概念和设计原则。
- 掌握服务拆分策略和边界划分。
- 熟悉服务间通信方式（同步和异步）。
- 了解微服务架构的挑战和解决方案。

## 什么是微服务

### 微服务定义

- **微服务**：将单一应用拆分为多个小型、独立的服务
- **特点**：每个服务独立部署、独立扩展、独立技术栈
- **原则**：单一职责、高内聚低耦合、自治性

### 微服务 vs 单体架构

| 特性           | 单体架构                     | 微服务架构                       |
| :------------- | :--------------------------- | :------------------------------- |
| **部署**       | 单一部署单元                 | 多个独立服务                     |
| **扩展**       | 整体扩展                     | 按服务扩展                       |
| **技术栈**     | 统一技术栈                   | 可混合技术栈                     |
| **复杂度**     | 相对简单                     | 更复杂（服务治理、监控等）       |
| **开发速度**   | 初期快速                     | 需要更多基础设施                 |
| **故障隔离**   | 单点故障影响整体             | 故障隔离，影响范围小             |
| **数据一致性** | 事务保证                     | 最终一致性                       |

## 服务拆分策略

### 按业务领域拆分

```php
<?php
declare(strict_types=1);

// 电商系统拆分示例

// 用户服务
namespace UserService;

class UserController
{
    public function register(array $data): User
    {
        // 用户注册逻辑
    }
    
    public function login(string $email, string $password): Token
    {
        // 用户登录逻辑
    }
}

// 商品服务
namespace ProductService;

class ProductController
{
    public function listProducts(array $filters): array
    {
        // 商品列表逻辑
    }
    
    public function getProduct(string $id): Product
    {
        // 商品详情逻辑
    }
}

// 订单服务
namespace OrderService;

class OrderController
{
    public function createOrder(array $items): Order
    {
        // 创建订单逻辑
        // 需要调用商品服务验证库存
        // 需要调用用户服务验证用户
    }
}
```

### 按数据模型拆分

```php
<?php
declare(strict_types=1);

// 按数据模型拆分

// 用户服务：管理用户数据
class UserService
{
    private UserRepository $userRepository;
    
    public function getUser(string $id): User
    {
        return $this->userRepository->find($id);
    }
}

// 订单服务：管理订单数据
class OrderService
{
    private OrderRepository $orderRepository;
    
    public function getOrder(string $id): Order
    {
        return $this->orderRepository->find($id);
    }
}
```

### 拆分原则

1. **单一职责**：每个服务只负责一个业务领域
2. **高内聚**：相关功能放在同一服务
3. **低耦合**：服务间依赖最小化
4. **数据独立**：每个服务有独立数据库
5. **可独立部署**：服务可独立发布和回滚

## 服务间通信

### 同步通信：HTTP/REST

```php
<?php
declare(strict_types=1);

namespace OrderService;

use GuzzleHttp\Client;

class OrderService
{
    private Client $httpClient;
    
    public function __construct()
    {
        $this->httpClient = new Client([
            'base_uri' => getenv('USER_SERVICE_URL'),
            'timeout' => 5.0,
        ]);
    }
    
    public function createOrder(array $items, string $userId): Order
    {
        // 1. 验证用户（调用用户服务）
        $user = $this->validateUser($userId);
        
        // 2. 验证库存（调用商品服务）
        $this->validateInventory($items);
        
        // 3. 创建订单
        $order = new Order($userId, $items);
        $this->orderRepository->save($order);
        
        return $order;
    }
    
    private function validateUser(string $userId): array
    {
        try {
            $response = $this->httpClient->get("/users/{$userId}");
            return json_decode($response->getBody()->getContents(), true);
        } catch (\Exception $e) {
            throw new \RuntimeException("User validation failed: {$e->getMessage()}");
        }
    }
    
    private function validateInventory(array $items): void
    {
        $response = $this->httpClient->post('/products/validate-inventory', [
            'json' => ['items' => $items],
        ]);
        
        if ($response->getStatusCode() !== 200) {
            throw new \RuntimeException('Inventory validation failed');
        }
    }
}
```

### 异步通信：消息队列

```php
<?php
declare(strict_types=1);

namespace OrderService;

use Redis;

class OrderService
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function createOrder(array $items, string $userId): Order
    {
        // 创建订单
        $order = new Order($userId, $items);
        $this->orderRepository->save($order);
        
        // 发送异步事件
        $this->publishEvent('order.created', [
            'order_id' => $order->getId(),
            'user_id' => $userId,
            'items' => $items,
        ]);
        
        return $order;
    }
    
    private function publishEvent(string $event, array $data): void
    {
        $message = json_encode([
            'event' => $event,
            'data' => $data,
            'timestamp' => time(),
        ]);
        
        // 使用 Redis Stream 发布事件
        $this->redis->xAdd('events', '*', [
            'type' => $event,
            'payload' => $message,
        ]);
    }
}
```

### 事件消费者

```php
<?php
declare(strict_types=1);

namespace NotificationService;

use Redis;

class EventConsumer
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function consume(): void
    {
        while (true) {
            // 从 Redis Stream 读取事件
            $messages = $this->redis->xRead(['events' => '$'], 1, 1000);
            
            foreach ($messages as $stream => $streamMessages) {
                foreach ($streamMessages as $id => $message) {
                    $this->handleEvent($message);
                    
                    // 确认处理
                    $this->redis->xAck('events', 'notification-group', [$id]);
                }
            }
        }
    }
    
    private function handleEvent(array $message): void
    {
        $event = json_decode($message['payload'], true);
        
        match ($event['type']) {
            'order.created' => $this->sendOrderConfirmation($event['data']),
            'order.shipped' => $this->sendShippingNotification($event['data']),
            default => null,
        };
    }
    
    private function sendOrderConfirmation(array $data): void
    {
        // 发送订单确认邮件
        // ...
    }
}
```

## API 网关

### 什么是 API 网关

- **API 网关**：统一入口，路由请求到后端服务
- **功能**：路由、认证、限流、监控、负载均衡

### 简单 API 网关实现

```php
<?php
declare(strict_types=1);

namespace Gateway;

use GuzzleHttp\Client;

class ApiGateway
{
    private array $routes = [
        '/api/users' => 'http://user-service:8001',
        '/api/products' => 'http://product-service:8002',
        '/api/orders' => 'http://order-service:8003',
    ];
    
    private Client $httpClient;
    
    public function __construct()
    {
        $this->httpClient = new Client();
    }
    
    public function handle(array $request): void
    {
        $path = parse_url($request['REQUEST_URI'], PHP_URL_PATH);
        $method = $request['REQUEST_METHOD'];
        
        // 路由匹配
        $serviceUrl = $this->route($path);
        
        if ($serviceUrl === null) {
            http_response_code(404);
            echo json_encode(['error' => 'Not found']);
            return;
        }
        
        // 转发请求
        try {
            $response = $this->httpClient->request($method, $serviceUrl . $path, [
                'headers' => $this->extractHeaders($request),
                'body' => file_get_contents('php://input'),
            ]);
            
            // 返回响应
            http_response_code($response->getStatusCode());
            foreach ($response->getHeaders() as $name => $values) {
                header("{$name}: " . implode(', ', $values));
            }
            echo $response->getBody()->getContents();
        } catch (\Exception $e) {
            http_response_code(500);
            echo json_encode(['error' => $e->getMessage()]);
        }
    }
    
    private function route(string $path): ?string
    {
        foreach ($this->routes as $prefix => $serviceUrl) {
            if (str_starts_with($path, $prefix)) {
                return $serviceUrl;
            }
        }
        
        return null;
    }
    
    private function extractHeaders(array $request): array
    {
        $headers = [];
        
        foreach ($request as $key => $value) {
            if (str_starts_with($key, 'HTTP_')) {
                $headerName = str_replace('_', '-', substr($key, 5));
                $headers[$headerName] = $value;
            }
        }
        
        return $headers;
    }
}
```

## 服务发现

### 服务注册

```php
<?php
declare(strict_types=1);

namespace ServiceRegistry;

use Redis;

class ServiceRegistry
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function register(string $serviceName, string $url, int $ttl = 60): void
    {
        $key = "service:{$serviceName}";
        $this->redis->setex($key, $ttl, $url);
        
        // 添加到服务列表
        $this->redis->sAdd('services', $serviceName);
    }
    
    public function discover(string $serviceName): ?string
    {
        $key = "service:{$serviceName}";
        $url = $this->redis->get($key);
        
        return $url !== false ? $url : null;
    }
    
    public function heartbeat(string $serviceName, string $url): void
    {
        // 定期更新服务状态
        $this->register($serviceName, $url);
    }
}
```

## 数据一致性

### 最终一致性

```php
<?php
declare(strict_types=1);

namespace OrderService;

// 订单服务创建订单后，异步更新用户服务的订单统计
class OrderService
{
    public function createOrder(array $items, string $userId): Order
    {
        // 1. 创建订单（本地事务）
        $order = new Order($userId, $items);
        $this->orderRepository->save($order);
        
        // 2. 发布事件（异步更新其他服务）
        $this->eventBus->publish('order.created', [
            'order_id' => $order->getId(),
            'user_id' => $userId,
            'total' => $order->getTotal(),
        ]);
        
        return $order;
    }
}

// 用户服务监听事件，更新订单统计
namespace UserService;

class OrderEventListener
{
    public function handleOrderCreated(array $event): void
    {
        $userId = $event['user_id'];
        $total = $event['total'];
        
        // 更新用户订单统计（最终一致性）
        $this->userRepository->incrementOrderCount($userId);
        $this->userRepository->incrementOrderTotal($userId, $total);
    }
}
```

### Saga 模式（分布式事务）

```php
<?php
declare(strict_types=1);

namespace OrderService;

class OrderSaga
{
    private array $steps = [];
    
    public function createOrder(array $items, string $userId): Order
    {
        $sagaId = uniqid('saga_', true);
        
        try {
            // 步骤 1：预留库存
            $this->reserveInventory($sagaId, $items);
            $this->steps[] = ['action' => 'reserve_inventory', 'compensate' => 'release_inventory'];
            
            // 步骤 2：扣减用户余额
            $this->deductBalance($sagaId, $userId, $this->calculateTotal($items));
            $this->steps[] = ['action' => 'deduct_balance', 'compensate' => 'refund_balance'];
            
            // 步骤 3：创建订单
            $order = $this->createOrderRecord($userId, $items);
            $this->steps[] = ['action' => 'create_order', 'compensate' => 'cancel_order'];
            
            return $order;
        } catch (\Exception $e) {
            // 补偿所有已完成的步骤
            $this->compensate($sagaId);
            throw $e;
        }
    }
    
    private function compensate(string $sagaId): void
    {
        // 逆序执行补偿操作
        foreach (array_reverse($this->steps) as $step) {
            $this->executeCompensation($sagaId, $step['compensate']);
        }
    }
}
```

## 微服务挑战和解决方案

### 挑战 1：服务间通信复杂性

**问题**：服务间调用链路过长，网络延迟累积

**解决方案**：
- 使用异步消息队列
- 实现服务缓存
- 使用 API 网关聚合请求

### 挑战 2：数据一致性

**问题**：分布式环境下难以保证强一致性

**解决方案**：
- 接受最终一致性
- 使用 Saga 模式处理分布式事务
- 实现补偿机制

### 挑战 3：服务治理

**问题**：服务数量多，难以管理和监控

**解决方案**：
- 使用服务注册中心
- 实现统一日志和监控
- 使用 API 网关统一入口

### 挑战 4：测试复杂性

**问题**：服务间依赖复杂，难以测试

**解决方案**：
- 使用契约测试（Contract Testing）
- 实现服务 Mock
- 使用测试容器（Testcontainers）

## 何时使用微服务

### 适合场景

1. **大型团队**：多个团队独立开发
2. **复杂业务**：业务领域清晰，可拆分
3. **高并发**：需要独立扩展不同服务
4. **技术多样性**：不同服务需要不同技术栈

### 不适合场景

1. **小型项目**：团队小，业务简单
2. **初期项目**：业务边界不清晰
3. **强一致性要求**：需要强事务保证

## 练习

1. **设计微服务架构**
   - 选择一个业务系统（如电商、博客）
   - 设计服务拆分方案
   - 定义服务边界和接口

2. **实现服务间通信**
   - 实现 HTTP 同步通信
   - 实现消息队列异步通信
   - 对比两种方式的优缺点

3. **实现 API 网关**
   - 创建简单的 API 网关
   - 实现路由、认证、限流功能
   - 集成服务发现

4. **处理数据一致性**
   - 实现最终一致性方案
   - 实现 Saga 模式处理分布式事务
   - 实现补偿机制

5. **服务监控和治理**
   - 实现服务注册和发现
   - 实现统一日志收集
   - 实现服务健康检查

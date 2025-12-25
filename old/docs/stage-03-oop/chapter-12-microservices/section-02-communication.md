# 3.12.2 服务间通信

## 概述

微服务之间需要通过通信机制协作。理解同步和异步通信方式，以及 API 网关的作用，对于设计微服务系统至关重要。

## 同步通信：HTTP/REST

### REST API 调用

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

## 异步通信：消息队列

### 事件发布

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
class ApiGateway
{
    private array $routes = [];

    public function __construct()
    {
        $this->routes = [
            '/users' => 'http://user-service:8001',
            '/products' => 'http://product-service:8002',
            '/orders' => 'http://order-service:8003',
        ];
    }

    public function handle(Request $request): Response
    {
        $path = $request->getPath();
        $service = $this->routeToService($path);

        if ($service === null) {
            return new Response('Not Found', 404);
        }

        // 转发请求到后端服务
        return $this->forward($request, $service);
    }

    private function routeToService(string $path): ?string
    {
        foreach ($this->routes as $prefix => $service) {
            if (str_starts_with($path, $prefix)) {
                return $service;
            }
        }
        return null;
    }

    private function forward(Request $request, string $service): Response
    {
        $client = new Client();
        $response = $client->request(
            $request->getMethod(),
            $service . $request->getPath(),
            [
                'headers' => $request->getHeaders(),
                'body' => $request->getBody(),
            ]
        );

        return new Response(
            $response->getBody()->getContents(),
            $response->getStatusCode(),
            $response->getHeaders()
        );
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 服务客户端抽象
interface ServiceClient
{
    public function get(string $path): array;
    public function post(string $path, array $data): array;
}

class HttpServiceClient implements ServiceClient
{
    public function __construct(
        private string $baseUrl,
        private Client $httpClient
    ) {
    }

    public function get(string $path): array
    {
        $response = $this->httpClient->get($this->baseUrl . $path);
        return json_decode($response->getBody()->getContents(), true);
    }

    public function post(string $path, array $data): array
    {
        $response = $this->httpClient->post($this->baseUrl . $path, [
            'json' => $data
        ]);
        return json_decode($response->getBody()->getContents(), true);
    }
}

// 订单服务使用其他服务
class OrderService
{
    public function __construct(
        private ServiceClient $userService,
        private ServiceClient $productService
    ) {
    }

    public function createOrder(string $userId, array $items): Order
    {
        // 同步调用：验证用户
        $user = $this->userService->get("/users/{$userId}");

        // 同步调用：验证库存
        $this->productService->post('/products/validate-inventory', [
            'items' => $items
        ]);

        // 创建订单
        $order = new Order($userId, $items);
        return $order;
    }
}
```

## 注意事项

1. **同步 vs 异步**：根据需求选择合适的通信方式。

2. **超时处理**：同步调用需要设置合理的超时时间。

3. **错误处理**：妥善处理服务调用失败的情况。

4. **重试机制**：对于临时失败，实现重试机制。

5. **服务发现**：使用服务发现机制管理服务地址。

## 练习

1. 实现一个服务客户端，支持 HTTP 调用其他服务。

2. 创建一个订单服务，调用用户服务和商品服务。

3. 实现一个简单的事件发布和订阅系统。

4. 设计一个 API 网关，路由请求到不同的后端服务。

5. 创建一个支持服务发现的微服务通信系统。

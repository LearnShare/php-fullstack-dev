# 10.1.8 其他 PSR 标准

## PSR-6：缓存接口

### 概述

PSR-6 定义了缓存的标准接口，实现了不同缓存实现之间的互操作。

### 核心接口

- `CacheItemPoolInterface`：缓存项池接口
- `CacheItemInterface`：缓存项接口

### 使用示例

```php
<?php
use Psr\Cache\CacheItemPoolInterface;

class UserService
{
    public function __construct(
        private CacheItemPoolInterface $cache
    ) {}
    
    public function getUser(int $id): ?User
    {
        $item = $this->cache->getItem("user_{$id}");
        
        if ($item->isHit()) {
            return $item->get();
        }
        
        $user = $this->fetchUser($id);
        $item->set($user);
        $item->expiresAfter(3600);
        $this->cache->save($item);
        
        return $user;
    }
}
```

## PSR-13：超媒体链接

### 概述

PSR-13 定义了超媒体链接的标准接口，用于 RESTful API 中的链接处理。

### 核心接口

- `LinkInterface`：链接接口
- `LinkProviderInterface`：链接提供者接口

## PSR-14：事件分发器

### 概述

PSR-14 定义了事件分发器的标准接口。

### 核心接口

- `EventDispatcherInterface`：事件分发器接口
- `ListenerProviderInterface`：监听器提供者接口
- `StoppableEventInterface`：可停止事件接口

## PSR-15：HTTP 处理器

### 概述

PSR-15 定义了 HTTP 请求处理器的标准接口。

### 核心接口

- `RequestHandlerInterface`：请求处理器接口
- `MiddlewareInterface`：中间件接口

### 使用示例

```php
<?php
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

class AuthMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        if (!$this->isAuthenticated($request)) {
            return new Response(401);
        }
        
        return $handler->handle($request);
    }
}
```

## PSR-16：简单缓存

### 概述

PSR-16 定义了简单缓存的标准接口，是 PSR-6 的简化版本。

### 核心接口

- `SimpleCacheInterface`：简单缓存接口

### 使用示例

```php
<?php
use Psr\SimpleCache\CacheInterface;

class UserService
{
    public function __construct(
        private CacheInterface $cache
    ) {}
    
    public function getUser(int $id): ?User
    {
        $key = "user_{$id}";
        
        if ($this->cache->has($key)) {
            return $this->cache->get($key);
        }
        
        $user = $this->fetchUser($id);
        $this->cache->set($key, $user, 3600);
        
        return $user;
    }
}
```

## PSR-17：HTTP 工厂

### 概述

PSR-17 定义了 HTTP 消息工厂的标准接口。

### 核心接口

- `RequestFactoryInterface`：请求工厂接口
- `ResponseFactoryInterface`：响应工厂接口
- `ServerRequestFactoryInterface`：服务器请求工厂接口
- `StreamFactoryInterface`：流工厂接口
- `UriFactoryInterface`：URI 工厂接口
- `UploadedFileFactoryInterface`：上传文件工厂接口

## PSR-18：HTTP 客户端

### 概述

PSR-18 定义了 HTTP 客户端的标准接口。

### 核心接口

- `ClientInterface`：HTTP 客户端接口

### 使用示例

```php
<?php
use Psr\Http\Client\ClientInterface;
use Psr\Http\Message\RequestInterface;

class ApiClient
{
    public function __construct(
        private ClientInterface $client
    ) {}
    
    public function get(string $url): array
    {
        $request = $this->createRequest('GET', $url);
        $response = $this->client->sendRequest($request);
        
        return json_decode($response->getBody(), true);
    }
}
```

## PSR-19：PHPDoc 标签

### 概述

PSR-19 定义了 PHPDoc 标签的标准。

## PSR-20：时钟接口

### 概述

PSR-20 定义了时钟接口，用于获取当前时间。

### 核心接口

- `ClockInterface`：时钟接口

### 使用示例

```php
<?php
use Psr\Clock\ClockInterface;

class OrderService
{
    public function __construct(
        private ClockInterface $clock
    ) {}
    
    public function createOrder(): Order
    {
        $order = new Order();
        $order->createdAt = $this->clock->now();
        
        return $order;
    }
}
```

## PSR-21：策略模式

### 概述

PSR-21 定义了策略模式的标准接口。

## 总结

这些 PSR 标准共同构成了 PHP 生态系统的标准体系：

- **编码规范**：PSR-1、PSR-12
- **自动加载**：PSR-4
- **接口标准**：PSR-3、PSR-6、PSR-7、PSR-11、PSR-13、PSR-14、PSR-15、PSR-16、PSR-17、PSR-18、PSR-19、PSR-20、PSR-21

遵循这些标准，可以实现：

- 框架和库之间的互操作
- 代码的一致性和可维护性
- 更好的团队协作
- 推动 PHP 生态系统的发展

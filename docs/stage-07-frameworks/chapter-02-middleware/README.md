# 7.2 路由、中间件、Pipeline

## 目标

- 理解洋葱模型（Onion Model）的请求处理流程。
- 掌握中间件的设计与实现。
- 熟悉 Pipeline 模式的应用。
- 了解路由系统的设计原理。

## 洋葱模型

### 请求处理流程

```
请求
  ↓
中间件 1（前置）
  ↓
中间件 2（前置）
  ↓
中间件 3（前置）
  ↓
控制器/处理器
  ↓
中间件 3（后置）
  ↓
中间件 2（后置）
  ↓
中间件 1（后置）
  ↓
响应
```

## Pipeline 实现

```php
<?php
declare(strict_types=1);

class Pipeline
{
    private array $pipes = [];
    private mixed $passable;
    
    public function send(mixed $passable): self
    {
        $this->passable = $passable;
        return $this;
    }
    
    public function through(array $pipes): self
    {
        $this->pipes = $pipes;
        return $this;
    }
    
    public function then(callable $destination): mixed
    {
        $pipeline = array_reduce(
            array_reverse($this->pipes),
            $this->carry(),
            $this->prepareDestination($destination)
        );
        
        return $pipeline($this->passable);
    }
    
    private function carry(): callable
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                if (is_callable($pipe)) {
                    return $pipe($passable, $stack);
                }
                
                if (is_string($pipe) && class_exists($pipe)) {
                    $pipe = new $pipe();
                }
                
                return method_exists($pipe, 'handle')
                    ? $pipe->handle($passable, $stack)
                    : $pipe($passable, $stack);
            };
        };
    }
    
    private function prepareDestination(callable $destination): callable
    {
        return function ($passable) use ($destination) {
            return $destination($passable);
        };
    }
}

// 使用
$response = (new Pipeline())
    ->send($request)
    ->through([
        AuthenticateMiddleware::class,
        ValidateMiddleware::class,
        LoggingMiddleware::class,
    ])
    ->then(function ($request) {
        return handleRequest($request);
    });
```

## 中间件实现

```php
<?php
declare(strict_types=1);

interface Middleware
{
    public function handle(Request $request, callable $next): Response;
}

class AuthenticateMiddleware implements Middleware
{
    public function handle(Request $request, callable $next): Response
    {
        // 前置处理
        if (!$this->authenticate($request)) {
            return new Response('Unauthorized', 401);
        }
        
        // 调用下一个中间件
        $response = $next($request);
        
        // 后置处理
        // ...
        
        return $response;
    }
    
    private function authenticate(Request $request): bool
    {
        $token = $request->getHeader('Authorization');
        return $this->validateToken($token);
    }
}
```

## 路由系统

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];
    private array $middleware = [];
    
    public function get(string $path, callable|string $handler): Route
    {
        return $this->addRoute('GET', $path, $handler);
    }
    
    public function post(string $path, callable|string $handler): Route
    {
        return $this->addRoute('POST', $path, $handler);
    }
    
    private function addRoute(string $method, string $path, callable|string $handler): Route
    {
        $route = new Route($method, $path, $handler);
        $this->routes[] = $route;
        return $route;
    }
    
    public function dispatch(Request $request): Response
    {
        $route = $this->findRoute($request);
        
        if ($route === null) {
            return new Response('Not Found', 404);
        }
        
        $middleware = array_merge($this->middleware, $route->getMiddleware());
        
        return (new Pipeline())
            ->send($request)
            ->through($middleware)
            ->then(function ($request) use ($route) {
                return $route->handle($request);
            });
    }
}
```

## 练习

1. 实现一个完整的 Pipeline 系统，支持中间件的链式调用。

2. 创建多个中间件（认证、日志、缓存等），演示洋葱模型。

3. 设计一个路由系统，支持 RESTful 路由和参数绑定。

4. 实现中间件的条件执行，根据请求特征决定是否执行。

5. 创建一个路由缓存系统，提升路由匹配性能。

6. 设计一个中间件组系统，支持中间件的分组和复用。

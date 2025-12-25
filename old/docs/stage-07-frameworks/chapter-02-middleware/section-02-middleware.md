# 7.2.2 中间件

## 概述

中间件是处理 HTTP 请求的管道机制。本节介绍中间件概念、洋葱模型、中间件实现、中间件注册等内容。

## 中间件概念

### 什么是中间件

中间件是在请求处理前后执行的可重用代码。

```php
<?php
declare(strict_types=1);

interface MiddlewareInterface
{
    public function handle(Request $request, callable $next): Response;
}

class AuthMiddleware implements MiddlewareInterface
{
    public function handle(Request $request, callable $next): Response
    {
        // 请求前处理
        if (!$this->isAuthenticated($request)) {
            return new Response('Unauthorized', 401);
        }
        
        // 继续处理
        $response = $next($request);
        
        // 响应后处理
        return $response;
    }
    
    private function isAuthenticated(Request $request): bool
    {
        // 检查认证
        return $request->hasHeader('Authorization');
    }
}
```

## 洋葱模型

### 请求处理流程

```
Request → Middleware 1 → Middleware 2 → Handler → Middleware 2 → Middleware 1 → Response
```

### 实现洋葱模型

```php
<?php
declare(strict_types=1);

class MiddlewarePipeline
{
    private array $middleware = [];
    private callable $handler;
    
    public function __construct(callable $handler)
    {
        $this->handler = $handler;
    }
    
    public function add(MiddlewareInterface $middleware): self
    {
        $this->middleware[] = $middleware;
        return $this;
    }
    
    public function handle(Request $request): Response
    {
        $pipeline = array_reverse($this->middleware);
        $next = $this->handler;
        
        foreach ($pipeline as $middleware) {
            $next = function($request) use ($middleware, $next) {
                return $middleware->handle($request, $next);
            };
        }
        
        return $next($request);
    }
}
```

## 中间件实现

### 认证中间件

```php
<?php
declare(strict_types=1);

class AuthMiddleware implements MiddlewareInterface
{
    public function handle(Request $request, callable $next): Response
    {
        $token = $request->getHeader('Authorization');
        
        if (!$token || !$this->validateToken($token)) {
            return new Response('Unauthorized', 401);
        }
        
        $user = $this->getUserFromToken($token);
        $request->setUser($user);
        
        return $next($request);
    }
    
    private function validateToken(string $token): bool
    {
        // 验证 token
        return true;
    }
    
    private function getUserFromToken(string $token): User
    {
        // 从 token 获取用户
        return new User();
    }
}
```

### 日志中间件

```php
<?php
declare(strict_types=1);

class LoggingMiddleware implements MiddlewareInterface
{
    private Logger $logger;
    
    public function __construct(Logger $logger)
    {
        $this->logger = $logger;
    }
    
    public function handle(Request $request, callable $next): Response
    {
        $startTime = microtime(true);
        
        $this->logger->info('Request started', [
            'method' => $request->getMethod(),
            'path' => $request->getPath(),
        ]);
        
        $response = $next($request);
        
        $duration = microtime(true) - $startTime;
        
        $this->logger->info('Request completed', [
            'method' => $request->getMethod(),
            'path' => $request->getPath(),
            'status' => $response->getStatusCode(),
            'duration' => $duration,
        ]);
        
        return $response;
    }
}
```

### CORS 中间件

```php
<?php
declare(strict_types=1);

class CorsMiddleware implements MiddlewareInterface
{
    public function handle(Request $request, callable $next): Response
    {
        // 处理预检请求
        if ($request->getMethod() === 'OPTIONS') {
            return $this->handlePreflight($request);
        }
        
        $response = $next($request);
        
        // 添加 CORS 头
        $response->setHeader('Access-Control-Allow-Origin', '*');
        $response->setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
        $response->setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
        
        return $response;
    }
    
    private function handlePreflight(Request $request): Response
    {
        return new Response('', 204, [
            'Access-Control-Allow-Origin' => '*',
            'Access-Control-Allow-Methods' => 'GET, POST, PUT, DELETE',
            'Access-Control-Allow-Headers' => 'Content-Type, Authorization',
        ]);
    }
}
```

## 中间件注册

### 全局中间件

```php
<?php
declare(strict_types=1);

class Application
{
    private array $middleware = [];
    
    public function middleware(MiddlewareInterface $middleware): self
    {
        $this->middleware[] = $middleware;
        return $this;
    }
    
    public function handle(Request $request): Response
    {
        $pipeline = new MiddlewarePipeline(function($request) {
            return $this->dispatch($request);
        });
        
        foreach ($this->middleware as $middleware) {
            $pipeline->add($middleware);
        }
        
        return $pipeline->handle($request);
    }
}

// 使用
$app = new Application();
$app->middleware(new LoggingMiddleware($logger));
$app->middleware(new CorsMiddleware());
```

### 路由中间件

```php
<?php
declare(strict_types=1);

class Route
{
    private array $middleware = [];
    
    public function middleware(array|string $middleware): self
    {
        $this->middleware = array_merge(
            $this->middleware,
            is_array($middleware) ? $middleware : [$middleware]
        );
        return $this;
    }
    
    public function handle(Request $request): Response
    {
        $pipeline = new MiddlewarePipeline(function($request) {
            return $this->executeHandler($request);
        });
        
        foreach ($this->middleware as $middleware) {
            $pipeline->add($middleware);
        }
        
        return $pipeline->handle($request);
    }
}

// 使用
$router->get('/users', 'UserController@index')
    ->middleware(['auth', 'throttle']);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Application
{
    private array $globalMiddleware = [];
    private Router $router;
    
    public function __construct()
    {
        $this->router = new Router();
    }
    
    public function middleware(MiddlewareInterface $middleware): self
    {
        $this->globalMiddleware[] = $middleware;
        return $this;
    }
    
    public function handle(Request $request): Response
    {
        // 创建处理管道
        $pipeline = new MiddlewarePipeline(function($request) {
            return $this->router->dispatch($request);
        });
        
        // 添加全局中间件
        foreach ($this->globalMiddleware as $middleware) {
            $pipeline->add($middleware);
        }
        
        return $pipeline->handle($request);
    }
}

// 使用
$app = new Application();

// 注册全局中间件
$app->middleware(new LoggingMiddleware($logger));
$app->middleware(new CorsMiddleware());

// 注册路由
$app->router->get('/users', 'UserController@index')
    ->middleware(['auth']);

// 处理请求
$response = $app->handle($request);
```

## 最佳实践

1. **单一职责**：每个中间件只做一件事
2. **可重用**：中间件应该是可重用的
3. **顺序重要**：注意中间件的执行顺序
4. **错误处理**：中间件应该处理错误

## 注意事项

1. 中间件应该快速执行
2. 避免在中间件中执行耗时操作
3. 注意中间件的执行顺序
4. 合理使用中间件，不要过度使用

## 练习

1. 实现一个认证中间件，验证用户身份。

2. 创建一个日志中间件，记录请求和响应。

3. 实现 CORS 中间件，处理跨域请求。

4. 创建一个完整的中间件系统，支持全局和路由中间件。

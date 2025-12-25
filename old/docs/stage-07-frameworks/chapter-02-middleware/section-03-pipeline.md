# 7.2.3 Pipeline 模式

## 概述

Pipeline 模式是实现中间件和请求处理的核心模式。本节介绍 Pipeline 概念、Pipeline 实现、请求处理流程等内容。

## Pipeline 概念

### 什么是 Pipeline

Pipeline 是将多个处理步骤串联起来的模式，每个步骤都可以处理请求并传递给下一个步骤。

```
Request → Step 1 → Step 2 → Step 3 → Handler → Step 3 → Step 2 → Step 1 → Response
```

## Pipeline 实现

### 基础 Pipeline

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
        return function($stack, $pipe) {
            return function($passable) use ($stack, $pipe) {
                if (is_callable($pipe)) {
                    return $pipe($passable, $stack);
                }
                
                if (is_string($pipe)) {
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
        return function($passable) use ($destination) {
            return $destination($passable);
        };
    }
}

// 使用
$pipeline = new Pipeline();
$result = $pipeline->send($request)
    ->through([
        LoggingMiddleware::class,
        AuthMiddleware::class,
        CorsMiddleware::class,
    ])
    ->then(function($request) {
        return new Response('Hello');
    });
```

## 请求处理流程

### 完整 Pipeline 示例

```php
<?php
declare(strict_types=1);

class RequestPipeline
{
    private array $middleware = [];
    private callable $handler;
    
    public function __construct(callable $handler)
    {
        $this->handler = $handler;
    }
    
    public function pipe(MiddlewareInterface|callable $middleware): self
    {
        $this->middleware[] = $middleware;
        return $this;
    }
    
    public function process(Request $request): Response
    {
        $pipeline = $this->buildPipeline();
        return $pipeline($request);
    }
    
    private function buildPipeline(): callable
    {
        $pipeline = $this->handler;
        
        foreach (array_reverse($this->middleware) as $middleware) {
            $pipeline = function($request) use ($middleware, $pipeline) {
                if (is_callable($middleware)) {
                    return $middleware($request, $pipeline);
                }
                
                return $middleware->handle($request, $pipeline);
            };
        }
        
        return $pipeline;
    }
}
```

## Pipeline 与中间件

### 集成中间件

```php
<?php
declare(strict_types=1);

class Application
{
    private Pipeline $pipeline;
    private Router $router;
    
    public function __construct()
    {
        $this->router = new Router();
        $this->pipeline = new Pipeline();
    }
    
    public function middleware(MiddlewareInterface $middleware): self
    {
        $this->pipeline->pipe($middleware);
        return $this;
    }
    
    public function handle(Request $request): Response
    {
        return $this->pipeline->process($request, function($request) {
            return $this->router->dispatch($request);
        });
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Framework
{
    private Pipeline $pipeline;
    private Router $router;
    private Container $container;
    
    public function __construct(Container $container)
    {
        $this->container = $container;
        $this->pipeline = new Pipeline();
        $this->router = new Router();
    }
    
    public function middleware(string|MiddlewareInterface $middleware): self
    {
        if (is_string($middleware)) {
            $middleware = $this->container->make($middleware);
        }
        
        $this->pipeline->pipe($middleware);
        return $this;
    }
    
    public function get(string $path, callable|string $handler): Route
    {
        return $this->router->get($path, $handler);
    }
    
    public function handle(Request $request): Response
    {
        return $this->pipeline->process($request, function($request) {
            return $this->router->dispatch($request);
        });
    }
}

// 使用
$app = new Framework($container);

$app->middleware(LoggingMiddleware::class);
$app->middleware(CorsMiddleware::class);

$app->get('/users', function($request) {
    return new Response('User list');
});

$response = $app->handle($request);
```

## 最佳实践

1. **可组合**：Pipeline 应该支持灵活组合
2. **可扩展**：易于添加新的处理步骤
3. **性能**：避免不必要的开销
4. **清晰**：保持代码清晰易懂

## 注意事项

1. Pipeline 的执行顺序很重要
2. 注意错误处理和异常传播
3. 避免在 Pipeline 中执行耗时操作
4. 合理使用 Pipeline，不要过度设计

## 练习

1. 实现一个基础的 Pipeline 类，支持多个处理步骤。

2. 将 Pipeline 与中间件系统集成。

3. 创建一个支持条件执行的 Pipeline。

4. 实现 Pipeline 的异常处理机制。

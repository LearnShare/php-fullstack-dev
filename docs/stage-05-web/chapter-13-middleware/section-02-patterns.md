# 5.13.2 中间件模式

## 概述

中间件模式是处理请求响应的设计模式，通过 Pipeline 模式和装饰器模式实现。理解中间件模式的实现原理、掌握 Pipeline 模式和装饰器模式、了解 PSR-15 标准，对于设计和实现中间件系统至关重要。本节详细介绍中间件模式概述、Pipeline 模式、装饰器模式、中间件接口、模式实现等内容，帮助零基础学员理解中间件的设计原理。

中间件模式是现代 Web 框架的核心设计模式，通过 Pipeline 和装饰器模式实现了灵活的请求处理机制。理解这些设计模式对于设计和实现中间件系统至关重要。

**主要内容**：
- 中间件模式概述（设计模式概念、模式的优势、应用场景）
- Pipeline 模式（Pipeline 概念、请求传递、响应传递、实现方法）
- 装饰器模式（装饰器概念、功能增强、链式装饰、实现方法）
- 中间件接口（接口定义、标准接口、PSR-15 标准、接口实现）
- 模式实现（Pipeline 实现、装饰器实现、组合使用）
- 实际应用示例和最佳实践

## 特性

- **灵活组合**：灵活组合多个中间件
- **可扩展**：易于扩展新功能
- **标准化**：遵循 PSR-15 标准
- **高性能**：高效的执行机制
- **易于理解**：清晰的执行流程

## 中间件模式概述

### 设计模式概念

中间件模式结合了 Pipeline 模式和装饰器模式，实现了灵活的请求处理机制。

### 模式的优势

1. **解耦**：解耦请求处理逻辑
2. **可组合**：灵活组合中间件
3. **可扩展**：易于扩展新功能
4. **可测试**：易于单元测试
5. **标准化**：遵循标准接口

### 应用场景

1. **Web 框架**：Web 框架的请求处理
2. **API 网关**：API 网关的请求处理
3. **HTTP 客户端**：HTTP 客户端的请求处理
4. **消息队列**：消息队列的消息处理

## Pipeline 模式

### Pipeline 概念

Pipeline（管道）模式将多个处理步骤串联起来，数据依次通过每个步骤进行处理。

### 请求传递

**示例**：
```php
<?php
declare(strict_types=1);

class Pipeline
{
    private array $middlewares = [];

    public function pipe(callable $middleware): self
    {
        $this->middlewares[] = $middleware;
        return $this;
    }

    public function process($request)
    {
        $next = function($req) {
            // 最终处理器
            return $this->handleRequest($req);
        };

        // 反向遍历中间件，构建调用链
        foreach (array_reverse($this->middlewares) as $middleware) {
            $next = function($req) use ($middleware, $next) {
                return $middleware($req, $next);
            };
        }

        return $next($request);
    }

    private function handleRequest($request)
    {
        // 最终处理逻辑
        return new Response('OK', 200);
    }
}
```

### 响应传递

**示例**：
```php
<?php
declare(strict_types=1);

// Pipeline 中的响应传递是反向的
// middleware1 -> middleware2 -> handler -> middleware2 -> middleware1
```

### 实现方法

**完整示例**：
```php
<?php
declare(strict_types=1);

class MiddlewarePipeline
{
    private array $middlewares = [];
    private $handler;

    public function __construct(callable $handler)
    {
        $this->handler = $handler;
    }

    public function pipe(callable $middleware): self
    {
        $this->middlewares[] = $middleware;
        return $this;
    }

    public function process($request)
    {
        $next = $this->handler;

        // 反向构建调用链
        foreach (array_reverse($this->middlewares) as $middleware) {
            $next = function($req) use ($middleware, $next) {
                return $middleware($req, $next);
            };
        }

        return $next($request);
    }
}

// 使用
$pipeline = new MiddlewarePipeline(function($request) {
    return new Response('OK', 200);
});

$pipeline->pipe(function($request, $next) {
    echo "Middleware 1: Request\n";
    $response = $next($request);
    echo "Middleware 1: Response\n";
    return $response;
});

$pipeline->pipe(function($request, $next) {
    echo "Middleware 2: Request\n";
    $response = $next($request);
    echo "Middleware 2: Response\n";
    return $response;
});

$response = $pipeline->process(new Request());
```

## 装饰器模式

### 装饰器概念

装饰器模式动态地给对象添加功能，而不改变对象本身。

### 功能增强

**示例**：
```php
<?php
declare(strict_types=1);

interface HandlerInterface
{
    public function handle($request);
}

class BaseHandler implements HandlerInterface
{
    public function handle($request)
    {
        return new Response('OK', 200);
    }
}

class LoggingDecorator implements HandlerInterface
{
    public function __construct(
        private HandlerInterface $handler
    ) {}

    public function handle($request)
    {
        echo "Logging: Request received\n";
        $response = $this->handler->handle($request);
        echo "Logging: Response sent\n";
        return $response;
    }
}

class AuthDecorator implements HandlerInterface
{
    public function __construct(
        private HandlerInterface $handler
    ) {}

    public function handle($request)
    {
        if (!isAuthenticated()) {
            return new Response('Unauthorized', 401);
        }
        return $this->handler->handle($request);
    }
}

// 使用
$handler = new BaseHandler();
$handler = new LoggingDecorator($handler);
$handler = new AuthDecorator($handler);
$response = $handler->handle(new Request());
```

### 链式装饰

**示例**：
```php
<?php
declare(strict_types=1);

class DecoratorBuilder
{
    private HandlerInterface $handler;

    public function __construct(HandlerInterface $handler)
    {
        $this->handler = $handler;
    }

    public function withLogging(): self
    {
        $this->handler = new LoggingDecorator($this->handler);
        return $this;
    }

    public function withAuth(): self
    {
        $this->handler = new AuthDecorator($this->handler);
        return $this;
    }

    public function withCors(): self
    {
        $this->handler = new CorsDecorator($this->handler);
        return $this;
    }

    public function build(): HandlerInterface
    {
        return $this->handler;
    }
}

// 使用
$handler = (new DecoratorBuilder(new BaseHandler()))
    ->withLogging()
    ->withAuth()
    ->withCors()
    ->build();
```

### 实现方法

**示例**：
```php
<?php
declare(strict_types=1);

abstract class MiddlewareDecorator implements HandlerInterface
{
    public function __construct(
        protected HandlerInterface $handler
    ) {}

    public function handle($request)
    {
        return $this->process($request, function($req) {
            return $this->handler->handle($req);
        });
    }

    abstract protected function process($request, callable $next);
}

class LoggingMiddleware extends MiddlewareDecorator
{
    protected function process($request, callable $next)
    {
        $startTime = microtime(true);
        $response = $next($request);
        $duration = microtime(true) - $startTime;
        echo "Request took {$duration} seconds\n";
        return $response;
    }
}
```

## 中间件接口

### 接口定义

**示例**：
```php
<?php
declare(strict_types=1);

interface MiddlewareInterface
{
    public function process($request, callable $next);
}
```

### 标准接口

**PSR-15 标准接口**：
```php
<?php
declare(strict_types=1);

namespace Psr\Http\Server;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

interface MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface;
}

interface RequestHandlerInterface
{
    public function handle(ServerRequestInterface $request): ResponseInterface;
}
```

### PSR-15 标准

**PSR-15 实现示例**：
```php
<?php
declare(strict_types=1);

namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class AuthMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        if (!isAuthenticated()) {
            return new Response('Unauthorized', 401);
        }

        return $handler->handle($request);
    }
}
```

### 接口实现

**完整实现示例**：
```php
<?php
declare(strict_types=1);

namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class LoggingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $startTime = microtime(true);
        
        $response = $handler->handle($request);
        
        $duration = microtime(true) - $startTime;
        $this->log($request, $response, $duration);
        
        return $response;
    }

    private function log(
        ServerRequestInterface $request,
        ResponseInterface $response,
        float $duration
    ): void {
        error_log(sprintf(
            '%s %s %d %f',
            $request->getMethod(),
            $request->getUri()->getPath(),
            $response->getStatusCode(),
            $duration
        ));
    }
}
```

## 模式实现

### Pipeline 实现

**完整 Pipeline 实现**：
```php
<?php
declare(strict_types=1);

namespace App\Pipeline;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class MiddlewarePipeline implements RequestHandlerInterface
{
    private array $middlewares = [];
    private RequestHandlerInterface $fallbackHandler;

    public function __construct(RequestHandlerInterface $fallbackHandler)
    {
        $this->fallbackHandler = $fallbackHandler;
    }

    public function pipe(MiddlewareInterface $middleware): self
    {
        $this->middlewares[] = $middleware;
        return $this;
    }

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        $handler = $this->fallbackHandler;

        // 反向构建调用链
        foreach (array_reverse($this->middlewares) as $middleware) {
            $handler = new MiddlewareHandler($middleware, $handler);
        }

        return $handler->handle($request);
    }
}

class MiddlewareHandler implements RequestHandlerInterface
{
    public function __construct(
        private MiddlewareInterface $middleware,
        private RequestHandlerInterface $next
    ) {}

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        return $this->middleware->process($request, $this->next);
    }
}
```

### 装饰器实现

**装饰器实现示例**：
```php
<?php
declare(strict_types=1);

namespace App\Decorator;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;

abstract class MiddlewareDecorator implements RequestHandlerInterface
{
    public function __construct(
        protected RequestHandlerInterface $handler
    ) {}

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        return $this->process($request, function($req) {
            return $this->handler->handle($req);
        });
    }

    abstract protected function process(
        ServerRequestInterface $request,
        callable $next
    ): ResponseInterface;
}
```

### 组合使用

**示例**：
```php
<?php
declare(strict_types=1);

// Pipeline 和装饰器可以组合使用
$pipeline = new MiddlewarePipeline($fallbackHandler);

$pipeline
    ->pipe(new LoggingMiddleware())
    ->pipe(new AuthMiddleware())
    ->pipe(new CorsMiddleware());

$response = $pipeline->handle($request);
```

## 使用场景

### 框架设计

- Web 框架的请求处理
- 中间件系统设计
- 请求处理管道

### 请求处理

- HTTP 请求处理
- API 请求处理
- 消息处理

### 功能增强

- 添加横切功能
- 功能组合
- 动态增强

### 横切关注点

- 认证授权
- 日志记录
- 错误处理
- 性能监控

## 注意事项

### 执行顺序

- **重要**：中间件的执行顺序很重要
- **请求阶段**：按注册顺序执行
- **响应阶段**：按相反顺序执行

### 性能考虑

- **中间件数量**：避免过多中间件
- **执行效率**：优化中间件执行
- **缓存使用**：合理使用缓存

### 错误处理

- **异常传播**：正确处理异常
- **错误响应**：返回适当的错误响应
- **错误日志**：记录错误信息

### 接口标准化

- **PSR-15**：遵循 PSR-15 标准
- **接口一致性**：保持接口一致性
- **兼容性**：考虑兼容性

## 常见问题

### Pipeline 模式如何实现？

通过反向构建调用链实现：
```php
foreach (array_reverse($middlewares) as $middleware) {
    $next = function($req) use ($middleware, $next) {
        return $middleware($req, $next);
    };
}
```

### 装饰器模式如何应用？

通过装饰器类包装处理器：
```php
$handler = new LoggingDecorator(
    new AuthDecorator(
        new BaseHandler()
    )
);
```

### PSR-15 标准是什么？

PSR-15 是 PHP 标准建议，定义了 HTTP 服务器中间件的标准接口。

### 如何设计中间件接口？

遵循 PSR-15 标准，定义清晰的接口，保持接口一致性。

## 最佳实践

### 遵循 PSR-15 标准

- 使用 PSR-15 标准接口
- 保持接口一致性
- 提高兼容性

### 实现 Pipeline 模式

- 使用 Pipeline 模式组织中间件
- 优化执行效率
- 支持灵活组合

### 保持接口一致性

- 使用标准接口
- 保持接口设计一致
- 易于理解和维护

### 优化执行性能

- 避免过多中间件
- 优化中间件执行
- 合理使用缓存

## 相关章节

- **[5.13.1 中间件概述](section-01-overview.md)**：了解中间件概述的详细内容
- **[5.13.3 编写自定义中间件](section-03-custom.md)**：了解编写自定义中间件的详细内容
- **[5.13.4 中间件最佳实践](section-04-best-practices.md)**：了解中间件最佳实践的详细内容

## 练习任务

1. **实现 Pipeline 模式**
   - 创建 Pipeline 类
   - 实现中间件链
   - 测试 Pipeline 执行

2. **实现装饰器模式**
   - 创建装饰器类
   - 实现功能增强
   - 测试装饰器组合

3. **实现 PSR-15 中间件**
   - 实现 MiddlewareInterface
   - 实现 RequestHandlerInterface
   - 测试中间件执行

4. **实现中间件系统**
   - Pipeline 和装饰器组合
   - 中间件注册和管理
   - 中间件配置

5. **实现完整的中间件框架**
   - 标准接口实现
   - Pipeline 管理
   - 中间件测试和文档

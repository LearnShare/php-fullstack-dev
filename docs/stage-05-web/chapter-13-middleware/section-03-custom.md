# 5.13.3 编写自定义中间件

## 概述

编写自定义中间件是扩展应用功能的重要方式。理解中间件的编写方法、掌握请求和响应处理、实现中间件注册和配置，对于构建功能丰富的 Web 应用至关重要。本节详细介绍中间件编写概述、中间件结构、请求处理、响应处理、中间件注册、参数传递等内容，帮助零基础学员掌握中间件开发技术。

自定义中间件可以根据应用需求实现特定功能，如认证、日志、验证等。掌握中间件的编写方法对于扩展应用功能至关重要。

**主要内容**：
- 中间件编写概述（中间件结构、函数式中间件、类式中间件、接口实现）
- 中间件结构（基本结构、请求处理、响应处理、错误处理）
- 请求处理（请求修改、请求验证、请求日志、请求转换）
- 响应处理（响应修改、响应格式化、响应头设置、响应日志）
- 中间件注册（全局注册、路由注册、条件注册、优先级设置）
- 参数传递（配置参数、依赖注入、上下文传递）
- 实际应用示例和最佳实践

## 特性

- **灵活实现**：支持函数式和类式实现
- **可配置**：支持配置参数
- **可测试**：易于单元测试
- **可重用**：可在多个路由中重用
- **标准化**：遵循标准接口

## 中间件编写概述

### 中间件结构

**基本结构**：
```php
<?php
declare(strict_types=1);

function middleware($request, $next)
{
    // 请求处理
    $request = processRequest($request);
    
    // 继续处理
    $response = $next($request);
    
    // 响应处理
    $response = processResponse($response);
    
    return $response;
}
```

### 函数式中间件

**示例**：
```php
<?php
declare(strict_types=1);

function loggingMiddleware($request, $next)
{
    $startTime = microtime(true);
    
    $response = $next($request);
    
    $duration = microtime(true) - $startTime;
    logRequest($request, $response, $duration);
    
    return $response;
}
```

### 类式中间件

**示例**：
```php
<?php
declare(strict_types=1);

class AuthMiddleware
{
    public function handle($request, $next)
    {
        if (!isAuthenticated()) {
            return new Response('Unauthorized', 401);
        }
        
        return $next($request);
    }
}
```

### 接口实现

**示例**：
```php
<?php
declare(strict_types=1);

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

## 中间件结构

### 基本结构

**示例**：
```php
<?php
declare(strict_types=1);

class CustomMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        // 1. 请求预处理
        $request = $this->preProcess($request);
        
        // 2. 继续处理
        $response = $handler->handle($request);
        
        // 3. 响应后处理
        $response = $this->postProcess($response);
        
        return $response;
    }

    private function preProcess(ServerRequestInterface $request): ServerRequestInterface
    {
        // 请求预处理逻辑
        return $request;
    }

    private function postProcess(ResponseInterface $response): ResponseInterface
    {
        // 响应后处理逻辑
        return $response;
    }
}
```

### 请求处理

**示例**：
```php
<?php
declare(strict_types=1);

class RequestProcessingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        // 修改请求
        $request = $request->withAttribute('processed', true);
        
        // 验证请求
        if (!$this->validateRequest($request)) {
            return new Response('Bad Request', 400);
        }
        
        return $handler->handle($request);
    }

    private function validateRequest(ServerRequestInterface $request): bool
    {
        // 验证逻辑
        return true;
    }
}
```

### 响应处理

**示例**：
```php
<?php
declare(strict_types=1);

class ResponseProcessingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $response = $handler->handle($request);
        
        // 修改响应
        $response = $response->withHeader('X-Processed', 'true');
        
        // 格式化响应
        $response = $this->formatResponse($response);
        
        return $response;
    }

    private function formatResponse(ResponseInterface $response): ResponseInterface
    {
        // 格式化逻辑
        return $response;
    }
}
```

### 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorHandlingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        try {
            return $handler->handle($request);
        } catch (\Exception $e) {
            return $this->handleError($e);
        }
    }

    private function handleError(\Exception $e): ResponseInterface
    {
        error_log($e->getMessage());
        
        return new Response(
            json_encode(['error' => 'Internal Server Error']),
            500,
            ['Content-Type' => 'application/json']
        );
    }
}
```

## 请求处理

### 请求修改

**示例**：
```php
<?php
declare(strict_types=1);

class RequestModificationMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        // 添加请求属性
        $request = $request->withAttribute('user_id', getCurrentUserId());
        
        // 修改请求头
        $request = $request->withHeader('X-Client-Version', '1.0');
        
        return $handler->handle($request);
    }
}
```

### 请求验证

**示例**：
```php
<?php
declare(strict_types=1);

class ValidationMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        // 验证请求方法
        if (!in_array($request->getMethod(), ['GET', 'POST', 'PUT', 'DELETE'], true)) {
            return new Response('Method Not Allowed', 405);
        }
        
        // 验证请求参数
        if ($request->getMethod() === 'POST') {
            $body = json_decode($request->getBody()->getContents(), true);
            if (!$this->validateBody($body)) {
                return new Response('Bad Request', 400);
            }
        }
        
        return $handler->handle($request);
    }

    private function validateBody(array $body): bool
    {
        // 验证逻辑
        return isset($body['required_field']);
    }
}
```

### 请求日志

**示例**：
```php
<?php
declare(strict_types=1);

class RequestLoggingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $this->logRequest($request);
        
        $response = $handler->handle($request);
        
        $this->logResponse($request, $response);
        
        return $response;
    }

    private function logRequest(ServerRequestInterface $request): void
    {
        error_log(sprintf(
            'Request: %s %s',
            $request->getMethod(),
            $request->getUri()->getPath()
        ));
    }

    private function logResponse(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): void {
        error_log(sprintf(
            'Response: %s %s %d',
            $request->getMethod(),
            $request->getUri()->getPath(),
            $response->getStatusCode()
        ));
    }
}
```

### 请求转换

**示例**：
```php
<?php
declare(strict_types=1);

class RequestTransformMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        // 转换请求体
        $body = json_decode($request->getBody()->getContents(), true);
        $transformedBody = $this->transformBody($body);
        
        // 创建新请求
        $request = $request->withParsedBody($transformedBody);
        
        return $handler->handle($request);
    }

    private function transformBody(array $body): array
    {
        // 转换逻辑
        return $body;
    }
}
```

## 响应处理

### 响应修改

**示例**：
```php
<?php
declare(strict_types=1);

class ResponseModificationMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $response = $handler->handle($request);
        
        // 添加响应头
        $response = $response->withHeader('X-Processed-By', 'Middleware');
        
        // 修改响应体
        $body = $response->getBody()->getContents();
        $modifiedBody = $this->modifyBody($body);
        
        $response->getBody()->rewind();
        $response->getBody()->write($modifiedBody);
        
        return $response;
    }

    private function modifyBody(string $body): string
    {
        // 修改逻辑
        return $body;
    }
}
```

### 响应格式化

**示例**：
```php
<?php
declare(strict_types=1);

class ResponseFormattingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $response = $handler->handle($request);
        
        // 统一响应格式
        $body = json_decode($response->getBody()->getContents(), true);
        $formattedBody = [
            'success' => $response->getStatusCode() < 400,
            'data' => $body,
            'timestamp' => time(),
        ];
        
        $response->getBody()->rewind();
        $response->getBody()->write(json_encode($formattedBody));
        $response = $response->withHeader('Content-Type', 'application/json');
        
        return $response;
    }
}
```

### 响应头设置

**示例**：
```php
<?php
declare(strict_types=1);

class ResponseHeaderMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $response = $handler->handle($request);
        
        // 设置安全头
        $response = $response
            ->withHeader('X-Content-Type-Options', 'nosniff')
            ->withHeader('X-Frame-Options', 'DENY')
            ->withHeader('X-XSS-Protection', '1; mode=block');
        
        return $response;
    }
}
```

### 响应日志

**示例**：
```php
<?php
declare(strict_types=1);

class ResponseLoggingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $startTime = microtime(true);
        
        $response = $handler->handle($request);
        
        $duration = microtime(true) - $startTime;
        $this->logResponse($request, $response, $duration);
        
        return $response;
    }

    private function logResponse(
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

## 中间件注册

### 全局注册

**示例**：
```php
<?php
declare(strict_types=1);

$app = new Application();

// 全局注册中间件
$app->addMiddleware(new LoggingMiddleware());
$app->addMiddleware(new ErrorHandlingMiddleware());
$app->addMiddleware(new CorsMiddleware());
```

### 路由注册

**示例**：
```php
<?php
declare(strict_types=1);

$app = new Application();

// 路由级别注册中间件
$app->get('/users', 'UserController@index')
    ->middleware(new AuthMiddleware())
    ->middleware(new PermissionMiddleware('users.read'));
```

### 条件注册

**示例**：
```php
<?php
declare(strict_types=1);

$app = new Application();

// 条件注册中间件
$app->addMiddleware(new LoggingMiddleware(), function($request) {
    // 只在 API 路由使用
    return str_starts_with($request->getUri()->getPath(), '/api');
});
```

### 优先级设置

**示例**：
```php
<?php
declare(strict_types=1);

$app = new Application();

// 按优先级注册中间件
$app->addMiddleware(new ErrorHandlingMiddleware(), null, 100);  // 高优先级
$app->addMiddleware(new LoggingMiddleware(), null, 50);         // 中优先级
$app->addMiddleware(new CorsMiddleware(), null, 10);           // 低优先级
```

## 参数传递

### 配置参数

**示例**：
```php
<?php
declare(strict_types=1);

class ConfigurableMiddleware implements MiddlewareInterface
{
    public function __construct(
        private array $config
    ) {}

    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        if (!$this->config['enabled']) {
            return $handler->handle($request);
        }
        
        // 使用配置
        $response = $handler->handle($request);
        
        if ($this->config['add_header']) {
            $response = $response->withHeader('X-Custom', 'value');
        }
        
        return $response;
    }
}

// 使用
$middleware = new ConfigurableMiddleware([
    'enabled' => true,
    'add_header' => true,
]);
```

### 依赖注入

**示例**：
```php
<?php
declare(strict_types=1);

class AuthMiddleware implements MiddlewareInterface
{
    public function __construct(
        private AuthService $authService,
        private LoggerInterface $logger
    ) {}

    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        if (!$this->authService->isAuthenticated($request)) {
            $this->logger->warning('Unauthorized access attempt');
            return new Response('Unauthorized', 401);
        }
        
        return $handler->handle($request);
    }
}
```

### 上下文传递

**示例**：
```php
<?php
declare(strict_types=1);

class ContextMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        // 添加上下文
        $request = $request->withAttribute('context', [
            'request_id' => uniqid(),
            'timestamp' => time(),
        ]);
        
        return $handler->handle($request);
    }
}
```

## 使用场景

### 认证授权

- 用户认证
- 权限检查
- 会话管理

### 请求验证

- 数据验证
- 参数验证
- 格式验证

### 日志记录

- 请求日志
- 响应日志
- 性能日志

### 性能监控

- 请求时间监控
- 性能指标收集
- 性能分析

### 错误处理

- 统一错误处理
- 错误格式化
- 错误日志

## 注意事项

### 中间件职责

- **单一职责**：每个中间件只做一件事
- **避免复杂逻辑**：避免在中间件中实现复杂业务逻辑
- **保持简单**：保持中间件简单易懂

### 执行顺序

- **理解顺序**：理解中间件的执行顺序
- **合理组织**：合理组织中间件顺序
- **性能考虑**：考虑执行顺序对性能的影响

### 性能影响

- **避免过度处理**：避免在中间件中做过多处理
- **缓存使用**：合理使用缓存
- **异步处理**：考虑异步处理

### 错误处理

- **异常捕获**：在中间件中捕获异常
- **错误响应**：返回适当的错误响应
- **错误日志**：记录错误信息

## 常见问题

### 如何编写中间件？

实现 MiddlewareInterface 接口，实现 process 方法：
```php
class MyMiddleware implements MiddlewareInterface
{
    public function process($request, $handler) {
        // 处理逻辑
        return $handler->handle($request);
    }
}
```

### 如何注册中间件？

根据框架的不同，使用不同的注册方法：
- 全局注册：`$app->addMiddleware()`
- 路由注册：`$route->middleware()`

### 如何传递参数？

通过构造函数传递配置参数，或通过请求属性传递上下文。

### 如何控制执行流程？

通过提前返回响应或调用 `$handler->handle($request)` 控制执行流程。

## 最佳实践

### 单一职责

- 每个中间件只做一件事
- 避免在中间件中实现复杂业务逻辑
- 保持中间件简单

### 保持简单

- 避免复杂的处理逻辑
- 保持中间件可测试
- 易于理解和维护

### 可配置设计

- 支持配置参数
- 提供默认配置
- 易于定制

### 错误处理

- 在中间件中捕获异常
- 返回适当的错误响应
- 记录错误信息

## 相关章节

- **[5.13.1 中间件概述](section-01-overview.md)**：了解中间件概述的详细内容
- **[5.13.2 中间件模式](section-02-patterns.md)**：了解中间件模式的详细内容
- **[5.13.4 中间件最佳实践](section-04-best-practices.md)**：了解中间件最佳实践的详细内容

## 练习任务

1. **编写简单的中间件**
   - 实现日志中间件
   - 实现认证中间件
   - 实现 CORS 中间件

2. **实现请求处理中间件**
   - 请求验证中间件
   - 请求转换中间件
   - 请求日志中间件

3. **实现响应处理中间件**
   - 响应格式化中间件
   - 响应头设置中间件
   - 响应日志中间件

4. **实现可配置中间件**
   - 支持配置参数
   - 依赖注入
   - 上下文传递

5. **实现完整的中间件系统**
   - 中间件注册和管理
   - 中间件测试
   - 中间件文档

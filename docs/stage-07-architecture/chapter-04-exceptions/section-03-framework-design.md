# 7.4.3 框架级异常设计

## 概述

框架级异常设计是构建可维护应用的关键。一个完善的异常处理机制能够统一错误响应、提供友好的调试信息、记录详细的日志，同时保持代码的整洁。本节将详细介绍如何设计一个框架级的异常处理系统，包括异常处理机制、异常中间件、响应格式化等内容。

**主要内容**：
- 异常处理机制设计
- 全局异常处理器
- 异常中间件
- 异常响应格式化

## 异常处理机制设计

### 异常处理器接口

```php
<?php
declare(strict_types=1);

// 异常处理器接口
interface ExceptionHandlerInterface
{
    public function handle(Throwable $exception): Response;
}

// 异常映射
class ExceptionHandler
{
    private array $handlers = [];
    private ?ExceptionHandlerInterface $fallback = null;
    
    public function register(string $exceptionClass, callable $handler): self
    {
        $this->handlers[$exceptionClass] = $handler;
        return $this;
    }
    
    public function setFallback(callable $handler): self
    {
        $this->fallback = $handler;
        return $this;
    }
    
    public function handle(Throwable $exception): Response
    {
        // 精确匹配
        $class = get_class($exception);
        if (isset($this->handlers[$class])) {
            return ($this->handlers[$class])($exception);
        }
        
        // 父类匹配
        foreach ($this->handlers as $handlerClass => $handler) {
            if ($exception instanceof $handlerClass) {
                return $handler($exception);
            }
        }
        
        // 回退处理
        if ($this->fallback !== null) {
            return ($this->fallback)($exception);
        }
        
        // 默认处理
        return $this->defaultHandle($exception);
    }
    
    private function defaultHandle(Throwable $exception): Response
    {
        return Response::json([
            'error' => [
                'message' => $exception->getMessage(),
                'code' => 'INTERNAL_ERROR'
            ]
        ], 500);
    }
}
```

### 全局异常处理

```php
<?php
declare(strict_types=1);

// 设置全局异常处理
class ErrorHandler
{
    private ExceptionHandler $handler;
    private LoggerInterface $logger;
    private bool $debug;
    
    public function __construct(
        ExceptionHandler $handler,
        LoggerInterface $logger,
        bool $debug = false
    ) {
        $this->handler = $handler;
        $this->logger = $logger;
        $this->debug = $debug;
        
        $this->register();
    }
    
    private function register(): void
    {
        set_exception_handler([$this, 'handleException']);
        set_error_handler([$this, 'handleError']);
        register_shutdown_function([$this, 'handleShutdown']);
    }
    
    public function handleException(Throwable $exception): void
    {
        // 记录日志
        $this->logException($exception);
        
        // 处理异常
        $response = $this->handler->handle($exception);
        
        // 发送响应
        $response->send();
    }
    
    public function handleError(int $errno, string $errstr, string $errfile, int $errline): bool
    {
        if (!(error_reporting() & $errno)) {
            return false;
        }
        
        $exception = new ErrorException($errno, $errno, $errstr, $errfile, $errline);
        $this->handleException($exception);
        
        return true;
    }
    
    public function handleShutdown(): void
    {
        $error = error_get_last();
        if ($error !== null && in_array($error['type'], [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR])) {
            $exception = new ErrorException(
                E_ERROR,
                $error['type'],
                $error['message'],
                $error['file'],
                $error['line']
            );
            $this->handleException($exception);
        }
    }
    
    private function logException(Throwable $exception): void
    {
        $context = [
            'message' => $exception->getMessage(),
            'code' => $exception->getCode(),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString(),
        ];
        
        if ($exception instanceof AppException) {
            $context['error_code'] = $exception->getErrorCode();
            $context['context'] = $exception->getContext();
        }
        
        $this->logger->error('Exception occurred', $context);
    }
}
```

## 异常中间件

```php
<?php
declare(strict_types=1);

// 异常处理中间件
class ExceptionMiddleware
{
    private ExceptionHandler $handler;
    private LoggerInterface $logger;
    private bool $debug;
    
    public function __construct(
        ExceptionHandler $handler,
        LoggerInterface $logger,
        bool $debug = false
    ) {
        $this->handler = $handler;
        $this->logger = $logger;
        $this->debug = $debug;
    }
    
    public function __invoke(Request $request, RequestHandler $next): Response
    {
        try {
            return $next->handle($request);
        } catch (Throwable $exception) {
            return $this->processException($exception);
        }
    }
    
    private function processException(Throwable $exception): Response
    {
        // 记录日志
        $this->logException($exception);
        
        // 处理异常
        return $this->handler->handle($exception);
    }
    
    private function logException(Throwable $exception): void
    {
        $level = match(true) {
            $exception instanceof ValidationException => 'warning',
            $exception instanceof BusinessException => 'info',
            default => 'error'
        };
        
        $this->logger->$level('Exception', [
            'message' => $exception->getMessage(),
            'type' => get_class($exception),
            'trace' => $this->debug ? $exception->getTraceAsString() : null
        ]);
    }
}
```

## 异常响应格式化

### 响应格式设计

```php
<?php
declare(strict_types=1);

// 异常响应格式化
class ExceptionResponseFormatter
{
    private bool $debug;
    
    public function __construct(bool $debug = false)
    {
        $this->debug = $debug;
    }
    
    public function format(Throwable $exception): array
    {
        $response = [
            'success' => false,
            'error' => [
                'message' => $this->getMessage($exception),
                'code' => $this->getCode($exception)
            ]
        ];
        
        if ($this->debug) {
            $response['error']['details'] = $this->getDetails($exception);
        }
        
        if ($exception instanceof HasErrorsInterface) {
            $response['error']['errors'] = $exception->getErrors();
        }
        
        return $response;
    }
    
    private function getMessage(Throwable $exception): string
    {
        if ($exception instanceof I18nException) {
            return $exception->getTranslatedMessage();
        }
        
        return $exception->getMessage();
    }
    
    private function getCode(Throwable $exception): string
    {
        if ($exception instanceof HasErrorCodeInterface) {
            return $exception->getErrorCode();
        }
        
        return 'ERROR';
    }
    
    private function getDetails(Throwable $exception): array
    {
        return [
            'type' => get_class($exception),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString()
        ];
    }
}
```

### 处理器实现

```php
<?php
declare(strict_types=1);

// 具体的异常处理器实现

class ValidationExceptionHandler implements ExceptionHandlerInterface
{
    public function handle(Throwable $exception): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'VALIDATION_ERROR',
                'message' => 'Validation failed',
                'errors' => $exception->getErrors()
            ]
        ], 422);
    }
}

class NotFoundExceptionHandler implements ExceptionHandlerInterface
{
    public function handle(Throwable $exception): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'NOT_FOUND',
                'message' => $exception->getMessage()
            ]
        ], 404);
    }
}

class AuthenticationExceptionHandler implements ExceptionHandlerInterface
{
    public function handle(Throwable $exception): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'UNAUTHORIZED',
                'message' => 'Authentication required'
            ]
        ], 401);
    }
}

class AuthorizationExceptionHandler implements ExceptionHandlerInterface
{
    public function handle(Throwable $exception): Response
    {
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'FORBIDDEN',
                'message' => 'Access denied'
            ]
        ], 403);
    }
}

class GenericExceptionHandler implements ExceptionHandlerInterface
{
    private bool $debug;
    
    public function __construct(bool $debug = false)
    {
        $this->debug = $debug;
    }
    
    public function handle(Throwable $exception): Response
    {
        $response = [
            'success' => false,
            'error' => [
                'code' => 'INTERNAL_ERROR',
                'message' => $this->debug 
                    ? $exception->getMessage() 
                    : 'An internal error occurred'
            ]
        ];
        
        if ($this->debug) {
            $response['error']['details'] = [
                'type' => get_class($exception),
                'file' => $exception->getFile(),
                'line' => $exception->getLine()
            ];
        }
        
        return Response::json($response, 500);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 完整的异常处理系统

class Kernel
{
    private ExceptionHandler $exceptionHandler;
    private LoggerInterface $logger;
    private bool $debug;
    
    public function __construct(bool $debug = false)
    {
        $this->debug = $debug;
        $this->logger = new FileLogger('logs/app.log');
        $this->exceptionHandler = $this->createExceptionHandler();
    }
    
    private function createExceptionHandler(): ExceptionHandler
    {
        $formatter = new ExceptionResponseFormatter($this->debug);
        
        $handler = new ExceptionHandler();
        
        // 注册具体处理器
        $handler->register(ValidationException::class, 
            fn($e) => new Response(json_encode([
                'success' => false,
                'error' => [
                    'code' => 'VALIDATION_ERROR',
                    'message' => 'Validation failed',
                    'errors' => $e->getErrors()
                ]
            ]), 422, ['Content-Type' => 'application/json'])
        );
        
        $handler->register(ResourceNotFoundException::class,
            fn($e) => new Response(json_encode([
                'success' => false,
                'error' => [
                    'code' => 'NOT_FOUND',
                    'message' => $e->getMessage()
                ]
            ]), 404, ['Content-Type' => 'application/json'])
        );
        
        $handler->register(AuthenticationException::class,
            fn($e) => new Response(json_encode([
                'success' => false,
                'error' => [
                    'code' => 'UNAUTHORIZED',
                    'message' => 'Authentication required'
                ]
            ]), 401, ['Content-Type' => 'application/json'])
        );
        
        $handler->register(AuthorizationException::class,
            fn($e) => new Response(json_encode([
                'success' => false,
                'error' => [
                    'code' => 'FORBIDDEN',
                    'message' => 'Access denied'
                ]
            ]), 403, ['Content-Type' => 'application/json'])
        );
        
        // 设置回退处理器
        $handler->setFallback(new GenericExceptionHandler($this->debug));
        
        return $handler;
    }
    
    public function run(): void
    {
        $errorHandler = new ErrorHandler(
            $this->exceptionHandler,
            $this->logger,
            $this->debug
        );
    }
}

// 使用
$kernel = new Kernel(debug: true);
$kernel->run();
```

## 练习任务

1. **设计异常处理系统**：为一个 Web 应用设计完整的异常处理系统

2. **实现全局处理**：实现一个全局异常处理器

3. **创建异常中间件**：创建一个异常处理中间件

4. **格式化响应**：设计一个异常响应格式化系统

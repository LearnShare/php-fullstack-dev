# 2.17 错误与异常处理

## 目标

- 理解 PHP 错误类型、错误报告机制、日志记录方法。
- 熟悉异常体系（`Exception`、`Error`、`Throwable`）及自定义异常。
- 构建标准的错误处理流程（捕获、记录、反馈）。

## 错误级别

- **致命错误** (`E_ERROR`)：脚本终止，如调用未定义函数。
- **警告** (`E_WARNING`)：输出警告，脚本继续。
- **通知** (`E_NOTICE`)：轻微问题，如未定义变量。
- **可捕获错误** (`E_RECOVERABLE_ERROR`)：可被 `try/catch` 捕获。

设置错误报告：

```php
error_reporting(E_ALL);
ini_set('display_errors', '0');
ini_set('log_errors', '1');
```

## 异常体系

- 所有可捕获异常实现 `Throwable` 接口。
- `Exception`：传统异常；`Error`：执行时错误（如类型错误）。
- 自定义异常：继承 `Exception` 或其子类。

```php
class InvalidOrderException extends RuntimeException {}

function placeOrder(array $payload): void
{
    if (!isset($payload['items'])) {
        throw new InvalidOrderException('Items required');
    }
}
```

## try/catch/finally

```php
try {
    $result = $service->process($data);
} catch (InvalidArgumentException $e) {
    logger()->warning('invalid_payload', ['message' => $e->getMessage()]);
    throw $e;
} finally {
    $connection->close();
}
```

- `finally` 块总会执行，适合释放资源。
- 捕获顺序：先子类再父类，否则子类异常会被父类捕获。

## 多层异常处理

- 应用层：转换异常为用户友好信息。
- 领域层：抛出语义化异常（如 `DomainException`、`ValidationException`）。
- 基础设施层：记录日志并抛出更通用的异常。

## 错误处理器

- `set_error_handler(callable $handler)`：将错误转换为异常/日志。
- 示例：把警告转为 `ErrorException` 便于捕获。

```php
set_error_handler(function (int $severity, string $message, string $file, int $line) {
    throw new ErrorException($message, 0, $severity, $file, $line);
});
```

## 日志记录

- 使用 `PSR-3` 日志接口：`$logger->error($message, $context);`
- `Monolog` 常见处理器：StreamHandler、DailyRotatingFileHandler、SlackHandler。
- 添加 `request_id`、`user_id`、`trace_id` 等上下文信息。

## 全局异常处理

- CLI 脚本：顶层 `try/catch`，友好输出错误信息。
- Web 应用：框架通常提供异常中间件，返回 JSON 或 HTML 错误页。
- `register_shutdown_function` 捕获致命错误。

```php
register_shutdown_function(function () {
    $error = error_get_last();
    if ($error && $error['type'] === E_ERROR) {
        logger()->critical('fatal', $error);
    }
});
```

## 异常 vs 返回值

- 异常适用于异常流程（输入错误、外部服务失败）。
- 正常流程使用返回值，避免滥用异常控制逻辑。

## 错误处理最佳实践

### 1. 分层错误处理

```php
<?php
declare(strict_types=1);

/**
 * 错误处理最佳实践示例
 * 
 * 分层处理：
 * 1. 领域层：抛出语义化异常
 * 2. 应用层：转换异常为用户友好信息
 * 3. 基础设施层：记录日志并处理
 */

// 领域层：抛出语义化异常
namespace App\Domain;

class OrderService
{
    public function createOrder(array $items, int $userId): Order
    {
        // 业务规则验证
        if (empty($items)) {
            throw new InvalidOrderException('Order must contain at least one item');
        }
        
        if ($userId <= 0) {
            throw new InvalidArgumentException('Invalid user ID');
        }
        
        // 创建订单逻辑
        return new Order($items, $userId);
    }
}

// 应用层：转换异常
namespace App\Application;

use App\Domain\OrderService;
use App\Domain\InvalidOrderException;

class OrderApplicationService
{
    public function __construct(
        private OrderService $orderService,
        private LoggerInterface $logger
    ) {}
    
    public function createOrder(array $data): array
    {
        try {
            $order = $this->orderService->createOrder(
                $data['items'],
                $data['user_id']
            );
            
            return [
                'success' => true,
                'order_id' => $order->getId(),
            ];
            
        } catch (InvalidOrderException $e) {
            // 业务异常：返回用户友好的错误信息
            $this->logger->warning('Order creation failed', [
                'error' => $e->getMessage(),
                'user_id' => $data['user_id'] ?? null,
            ]);
            
            return [
                'success' => false,
                'error' => '订单创建失败：' . $e->getMessage(),
            ];
            
        } catch (InvalidArgumentException $e) {
            // 参数错误：记录并返回错误
            $this->logger->warning('Invalid order data', [
                'error' => $e->getMessage(),
                'data' => $data,
            ]);
            
            return [
                'success' => false,
                'error' => '请求参数错误',
            ];
            
        } catch (\Exception $e) {
            // 未知错误：记录详细日志，返回通用错误信息
            $this->logger->error('Unexpected error in order creation', [
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString(),
                'data' => $data,
            ]);
            
            return [
                'success' => false,
                'error' => '系统错误，请稍后重试',
            ];
        }
    }
}
```

### 2. 错误恢复策略

```php
<?php
declare(strict_types=1);

/**
 * 错误恢复策略示例
 * 
 * 对于可恢复的错误，实现重试机制
 */
class RetryableService
{
    private LoggerInterface $logger;
    
    public function callExternalApi(string $url, int $maxRetries = 3): array
    {
        $attempts = 0;
        $lastException = null;
        
        while ($attempts < $maxRetries) {
            try {
                // 调用外部 API
                $response = $this->httpClient->get($url);
                return json_decode($response->getBody()->getContents(), true);
                
            } catch (NetworkException $e) {
                // 网络错误：可以重试
                $attempts++;
                $lastException = $e;
                
                $this->logger->warning('API call failed, retrying', [
                    'attempt' => $attempts,
                    'max_retries' => $maxRetries,
                    'error' => $e->getMessage(),
                ]);
                
                // 指数退避：等待时间逐渐增加
                $waitTime = pow(2, $attempts) * 100000; // 微秒
                usleep($waitTime);
                
            } catch (ClientException $e) {
                // 客户端错误（4xx）：不应该重试
                $this->logger->error('API client error', [
                    'status' => $e->getResponse()->getStatusCode(),
                    'error' => $e->getMessage(),
                ]);
                throw $e;
                
            } catch (\Exception $e) {
                // 其他错误：记录并抛出
                $this->logger->error('Unexpected API error', [
                    'error' => $e->getMessage(),
                ]);
                throw $e;
            }
        }
        
        // 所有重试都失败
        throw new ServiceUnavailableException(
            "API call failed after {$maxRetries} attempts",
            0,
            $lastException
        );
    }
}
```

### 3. 错误上下文记录

```php
<?php
declare(strict_types=1);

/**
 * 错误上下文记录示例
 * 
 * 记录足够的上下文信息，便于问题排查
 */
class ContextualErrorHandler
{
    private LoggerInterface $logger;
    
    public function processPayment(array $paymentData): void
    {
        // 收集上下文信息
        $context = [
            'user_id' => $paymentData['user_id'] ?? null,
            'order_id' => $paymentData['order_id'] ?? null,
            'amount' => $paymentData['amount'] ?? null,
            'payment_method' => $paymentData['method'] ?? null,
            'request_id' => $_SERVER['REQUEST_ID'] ?? null,
            'ip' => $_SERVER['REMOTE_ADDR'] ?? null,
        ];
        
        try {
            // 处理支付逻辑
            $this->paymentGateway->charge($paymentData);
            
            $this->logger->info('Payment processed successfully', $context);
            
        } catch (PaymentException $e) {
            // 记录支付错误，包含完整上下文
            $this->logger->error('Payment processing failed', array_merge($context, [
                'error' => $e->getMessage(),
                'error_code' => $e->getCode(),
                'gateway_response' => $e->getGatewayResponse(),
            ]));
            
            throw $e;
            
        } catch (\Exception $e) {
            // 记录未知错误，包含堆栈跟踪
            $this->logger->critical('Unexpected payment error', array_merge($context, [
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
            ]));
            
            throw $e;
        }
    }
}
```

### 4. 全局异常处理器

```php
<?php
declare(strict_types=1);

/**
 * 全局异常处理器
 * 
 * 统一处理所有未捕获的异常
 */
class GlobalExceptionHandler
{
    private LoggerInterface $logger;
    private bool $isProduction;
    
    public function __construct(LoggerInterface $logger, bool $isProduction = false)
    {
        $this->logger = $logger;
        $this->isProduction = $isProduction;
    }
    
    public function register(): void
    {
        // 注册异常处理器
        set_exception_handler([$this, 'handleException']);
        
        // 注册错误处理器
        set_error_handler([$this, 'handleError']);
        
        // 注册关闭函数（捕获致命错误）
        register_shutdown_function([$this, 'handleShutdown']);
    }
    
    public function handleException(\Throwable $e): void
    {
        // 记录异常
        $this->logger->critical('Uncaught exception', [
            'message' => $e->getMessage(),
            'code' => $e->getCode(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'trace' => $e->getTraceAsString(),
        ]);
        
        // 返回响应
        if ($this->isProduction) {
            // 生产环境：返回通用错误信息
            http_response_code(500);
            header('Content-Type: application/json');
            echo json_encode([
                'error' => 'Internal server error',
            ]);
        } else {
            // 开发环境：返回详细错误信息
            http_response_code(500);
            header('Content-Type: application/json');
            echo json_encode([
                'error' => $e->getMessage(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
                'trace' => $e->getTrace(),
            ], JSON_PRETTY_PRINT);
        }
    }
    
    public function handleError(
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        // 将错误转换为异常
        if (!(error_reporting() & $severity)) {
            return false; // 错误被抑制
        }
        
        throw new \ErrorException($message, 0, $severity, $file, $line);
    }
    
    public function handleShutdown(): void
    {
        $error = error_get_last();
        
        if ($error !== null && in_array($error['type'], [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR])) {
            // 致命错误
            $this->logger->emergency('Fatal error', [
                'message' => $error['message'],
                'file' => $error['file'],
                'line' => $error['line'],
            ]);
        }
    }
}

// 使用
$handler = new GlobalExceptionHandler($logger, $isProduction);
$handler->register();
```

## 练习

1. 实现 `SafeFileReader`，当文件不存在/不可读时抛出自定义异常，并记录日志。

2. 使用 `set_error_handler` 将所有警告转为 `ErrorException`，验证在 `try/catch` 中捕获。

3. 编写一个全局异常处理器，将未捕获异常统一转换为标准 JSON 响应。

4. **实现分层错误处理**：领域层抛出语义化异常，应用层转换为用户友好信息。

5. **实现错误恢复策略**：为可恢复的错误（如网络错误）实现重试机制。

6. **实现错误上下文记录**：记录足够的上下文信息，便于问题排查。

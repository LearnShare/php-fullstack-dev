# 7.4.4 异常处理最佳实践

## 概述

异常处理是软件开发中的重要环节，良好的异常处理能够提升用户体验、便于问题排查、保证系统稳定性。本节将详细介绍异常处理的最佳实践，包括异常设计原则、处理策略、日志记录和监控等方面。

**主要内容**：
- 异常设计最佳实践
- 异常处理策略
- 异常日志记录
- 异常监控和告警

## 异常设计最佳实践

### 1. 使用具体异常类型

```php
<?php
declare(strict_types=1);

// 不好的做法 - 使用通用异常
function getUser(int $id): User
{
    $user = $this->repository->find($id);
    
    if ($user === null) {
        throw new Exception("User not found"); // 太笼统
    }
    
    return $user;
}

// 好的做法 - 使用具体异常
function getUser(int $id): User
{
    $user = $this->repository->find($id);
    
    if ($user === null) {
        throw new UserNotFoundException($id); // 具体明确
    }
    
    return $user;
}
```

### 2. 包含足够的上下文

```php
<?php
declare(strict_types=1);

// 好的异常包含上下文信息
class ResourceNotFoundException extends Exception
{
    private string $resourceType;
    private mixed $resourceId;
    private array $searchCriteria;
    
    public function __construct(
        string $resourceType,
        mixed $resourceId,
        array $searchCriteria = []
    ) {
        $this->resourceType = $resourceType;
        $this->resourceId = $resourceId;
        $this->searchCriteria = $searchCriteria;
        
        $message = "{$resourceType} not found";
        if (!empty($searchCriteria)) {
            $message .= " with criteria: " . json_encode($searchCriteria);
        }
        
        parent::__construct($message);
    }
    
    public function getResourceType(): string { return $this->resourceType; }
    public function getResourceId(): mixed { return $this->resourceId; }
    public function getSearchCriteria(): array { return $this->searchCriteria; }
}
```

### 3. 异常层次合理

```php
<?php
declare(strict_types=1);

// 合理的异常层次
abstract class AppException extends Exception
{
    abstract public function getErrorCode(): string;
    abstract public function getStatusCode(): int;
}

// 业务异常
abstract class BusinessException extends AppException
{
    public function getStatusCode(): int { return 400; }
}

// 验证异常
class ValidationException extends BusinessException
{
    public function getErrorCode(): string { return 'VALIDATION_ERROR'; }
    public function getStatusCode(): int { return 422; }
}

// 资源异常
class ResourceNotFoundException extends BusinessException
{
    public function getErrorCode(): string { return 'NOT_FOUND'; }
    public function getStatusCode(): int { return 404; }
}

// 系统异常
abstract class SystemException extends AppException
{
    public function getStatusCode(): int { return 500; }
}
```

## 异常处理策略

### 1. 捕获具体异常

```php
<?php
declare(strict_types=1);

// 好的做法 - 从具体到通用
try {
    $user = $userService->getUser($id);
} catch (UserNotFoundException $e) {
    // 处理用户不存在
    return Response::json([
        'error' => [
            'code' => 'USER_NOT_FOUND',
            'message' => $e->getMessage()
        ]
    ], 404);
    
} catch (ValidationException $e) {
    // 处理验证错误
    return Response::json([
        'error' => [
            'code' => 'VALIDATION_ERROR',
            'message' => $e->getMessage(),
            'errors' => $e->getErrors()
        ]
    ], 422);
    
} catch (BusinessException $e) {
    // 处理其他业务异常
    return Response::json([
        'error' => [
            'code' => $e->getErrorCode(),
            'message' => $e->getMessage()
        ]
    ], $e->getStatusCode());
    
} catch (Throwable $e) {
    // 处理系统异常
    $this->logger->error('Unexpected error', ['exception' => $e]);
    return Response::json([
        'error' => [
            'code' => 'INTERNAL_ERROR',
            'message' => 'An unexpected error occurred'
        ]
    ], 500);
}
```

### 2. 不要过度捕获

```php
<?php
declare(strict_types=1);

// 不好的做法 - 捕获所有异常
try {
    $result = $this->process($data);
} catch (Exception $e) {
    // 吞掉了所有异常
    return null;
}

// 好的做法 - 只捕获需要处理的异常
try {
    $result = $this->process($data);
} catch (ValidationException $e) {
    // 只处理验证异常，其他异常向上传播
    throw $e;
}
```

### 3. 异常转换

```php
<?php
declare(strict_types=1);

// 在边界转换异常
class UserController
{
    private UserService $userService;
    
    public function getUser(int $id): Response
    {
        try {
            $user = $this->userService->getUser($id);
            return Response::json(['data' => $user->toArray()]);
            
        } catch (UserNotFoundException $e) {
            return Response::json([
                'error' => [
                    'code' => 'USER_NOT_FOUND',
                    'message' => $e->getMessage()
                ]
            ], 404);
        }
    }
}
```

## 异常日志记录

### 1. 分级记录

```php
<?php
declare(strict_types=1);

// 根据异常类型选择日志级别

try {
    $this->orderService->process($order);
} catch (ValidationException $e) {
    // 验证异常 - warning
    $this->logger->warning('Validation failed', [
        'errors' => $e->getErrors(),
        'order_id' => $order->getId()
    ]);
    throw $e;
    
} catch (BusinessException $e) {
    // 业务异常 - info
    $this->logger->info('Business rule violation', [
        'exception' => $e->getMessage(),
        'order_id' => $order->getId()
    ]);
    throw $e;
    
} catch (DatabaseException $e) {
    // 数据库异常 - error
    $this->logger->error('Database error', [
        'exception' => $e->getMessage(),
        'query' => $e->getQuery(),
        'order_id' => $order->getId()
    ]);
    throw $e;
    
} catch (Throwable $e) {
    // 系统异常 - critical
    $this->logger->critical('Unexpected error', [
        'exception' => $e->getMessage(),
        'trace' => $e->getTraceAsString(),
        'order_id' => $order->getId()
    ]);
    throw $e;
}
```

### 2. 记录上下文

```php
<?php
declare(strict_types=1);

// 记录丰富的上下文信息
class ExceptionLogger
{
    public function log(Throwable $exception, array $context = []): void
    {
        $data = [
            'message' => $exception->getMessage(),
            'type' => get_class($exception),
            'code' => $exception->getCode(),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'timestamp' => date('Y-m-d H:i:s'),
            'context' => $context
        ];
        
        // 添加请求信息
        if (isset($_SERVER['REQUEST_URI'])) {
            $data['request'] = [
                'uri' => $_SERVER['REQUEST_URI'],
                'method' => $_SERVER['REQUEST_METHOD'] ?? 'CLI',
                'ip' => $_SERVER['REMOTE_ADDR'] ?? null,
                'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? null
            ];
        }
        
        // 添加用户信息
        if (isset($_SESSION['user_id'])) {
            $data['user'] = [
                'id' => $_SESSION['user_id']
            ];
        }
        
        // 记录完整堆栈
        $data['stack_trace'] = $exception->getTraceAsString();
        
        $this->write($data);
    }
}
```

## 异常监控和告警

### 1. 异常统计

```php
<?php
declare(strict_types=1);

// 异常统计
class ExceptionMonitor
{
    private Metrics $metrics;
    private AlertService $alerts;
    
    public function __construct(Metrics $metrics, AlertService $alerts)
    {
        $this->metrics = $metrics;
        $this->alerts = $alerts;
    }
    
    public function record(Throwable $exception): void
    {
        $type = get_class($exception);
        
        // 记录异常计数
        $this->metrics->increment("exceptions.{$type}");
        
        // 记录异常类型分布
        $this->metrics->increment('exceptions.total');
        
        // 检查是否需要告警
        $this->checkAlert($type);
    }
    
    private function checkAlert(string $type): void
    {
        $count = $this->metrics->get("exceptions.{$type}.count");
        
        if ($count > $this->getThreshold($type)) {
            $this->alerts->send("Too many {$type} exceptions: {$count}");
        }
    }
    
    private function getThreshold(string $type): int
    {
        // 不同异常类型不同阈值
        return match($type) {
            ValidationException::class => 100,
            DatabaseException::class => 10,
            default => 50
        };
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 完整的异常处理最佳实践示例

class OrderService
{
    private OrderRepository $repository;
    private PaymentGateway $payment;
    private InventoryService $inventory;
    private ExceptionLogger $logger;
    private ExceptionMonitor $monitor;
    
    public function __construct(
        OrderRepository $repository,
        PaymentGateway $payment,
        InventoryService $inventory,
        ExceptionLogger $logger,
        ExceptionMonitor $monitor
    ) {
        $this->repository = $repository;
        $this->payment = $payment;
        $this->inventory = $inventory;
        $this->logger = $logger;
        $this->monitor = $monitor;
    }
    
    public function createOrder(array $data): Order
    {
        try {
            // 验证库存
            $this->validateInventory($data['items']);
            
            // 创建订单
            $order = $this->repository->create($data);
            
            // 处理支付
            $this->processPayment($order, $data['payment']);
            
            return $order;
            
        } catch (InsufficientStockException $e) {
            $this->logger->warning('Insufficient stock', [
                'items' => $e->getOutOfStockItems(),
                'order_data' => $data
            ]);
            $this->monitor->record($e);
            throw $e;
            
        } catch (PaymentFailedException $e) {
            $this->logger->info('Payment failed', [
                'order_id' => $e->getOrderId(),
                'reason' => $e->getReason()
            ]);
            $this->monitor->record($e);
            throw $e;
            
        } catch (DatabaseException $e) {
            $this->logger->error('Database error creating order', [
                'exception' => $e,
                'order_data' => $data
            ]);
            $this->monitor->record($e);
            throw new OrderProcessingException(
                'Failed to create order',
                previous: $e
            );
        }
    }
    
    private function validateInventory(array $items): void
    {
        $outOfStock = [];
        
        foreach ($items as $item) {
            if (!$this->inventory->check($item['product_id'], $item['quantity'])) {
                $outOfStock[] = $item['product_id'];
            }
        }
        
        if (!empty($outOfStock)) {
            throw new InsufficientStockException($outOfStock);
        }
    }
}
```

## 练习任务

1. **审查异常处理**：审查一个现有项目的异常处理代码，识别问题

2. **改进异常处理**：为一个服务类添加异常处理最佳实践

3. **实现日志系统**：实现一个包含上下文信息的异常日志系统

4. **设计监控**：设计一个异常监控系统，包括统计和告警

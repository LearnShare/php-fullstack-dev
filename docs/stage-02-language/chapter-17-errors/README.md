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

## 练习

1. 实现 `SafeFileReader`，当文件不存在/不可读时抛出自定义异常，并记录日志。
2. 使用 `set_error_handler` 将所有警告转为 `ErrorException`，验证在 `try/catch` 中捕获。
3. 编写一个全局异常处理器，将未捕获异常统一转换为标准 JSON 响应。

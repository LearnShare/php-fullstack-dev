# 4.4.2 API 请求处理

## 概述

API 请求处理需要支持多种 Content-Type，提供统一的解析接口和错误处理。本节详细介绍不同 Content-Type 的处理方式、请求体解析封装、错误处理，以及 API 响应格式。

## 不同 Content-Type 处理

### Content-Type 检测

```php
<?php
declare(strict_types=1);

function getContentType(): string
{
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    
    // 移除参数（如 charset）
    if (($pos = strpos($contentType, ';')) !== false) {
        $contentType = substr($contentType, 0, $pos);
    }
    
    return trim($contentType);
}
```

### 多格式支持

```php
<?php
declare(strict_types=1);

class RequestParser
{
    public function parse(): array
    {
        $contentType = $this->getContentType();
        $rawInput = file_get_contents('php://input');

        return match ($contentType) {
            'application/json' => $this->parseJson($rawInput),
            'application/x-www-form-urlencoded' => $this->parseForm($rawInput),
            'multipart/form-data' => $_POST,  // 已自动解析
            default => throw new InvalidArgumentException("Unsupported Content-Type: {$contentType}"),
        };
    }

    private function parseJson(string $input): array
    {
        if (function_exists('json_validate') && !json_validate($input)) {
            throw new InvalidArgumentException('Invalid JSON');
        }
        
        $data = json_decode($input, true, 10);
        if (json_last_error() !== JSON_ERROR_NONE) {
            throw new InvalidArgumentException('JSON decode error');
        }
        
        return $data;
    }

    private function parseForm(string $input): array
    {
        parse_str($input, $data);
        return $data;
    }

    private function getContentType(): string
    {
        $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
        if (($pos = strpos($contentType, ';')) !== false) {
            $contentType = substr($contentType, 0, $pos);
        }
        return trim($contentType);
    }
}
```

## 请求体解析封装

### 统一接口

```php
<?php
declare(strict_types=1);

class ApiRequest
{
    private array $data = [];

    public function __construct()
    {
        $this->parse();
    }

    private function parse(): void
    {
        $method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
        
        if ($method === 'GET') {
            $this->data = $_GET;
        } elseif ($method === 'POST') {
            $contentType = $this->getContentType();
            
            if ($contentType === 'application/json') {
                $this->data = $this->parseJson();
            } else {
                $this->data = $_POST;
            }
        }
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return $this->data[$key] ?? $default;
    }

    public function all(): array
    {
        return $this->data;
    }

    private function parseJson(): array
    {
        $input = file_get_contents('php://input');
        if (function_exists('json_validate') && !json_validate($input)) {
            return [];
        }
        return json_decode($input, true) ?? [];
    }

    private function getContentType(): string
    {
        $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
        if (($pos = strpos($contentType, ';')) !== false) {
            $contentType = substr($contentType, 0, $pos);
        }
        return trim($contentType);
    }
}
```

## 错误处理

### 统一错误响应

```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data = null, string $message = 'Success'): void
    {
        header('Content-Type: application/json');
        http_response_code(200);
        echo json_encode([
            'success' => true,
            'message' => $message,
            'data' => $data,
        ], JSON_UNESCAPED_UNICODE);
        exit;
    }

    public static function error(string $message, int $code = 400, mixed $errors = null): void
    {
        header('Content-Type: application/json');
        http_response_code($code);
        echo json_encode([
            'success' => false,
            'message' => $message,
            'errors' => $errors,
        ], JSON_UNESCAPED_UNICODE);
        exit;
    }

    public static function validationError(array $errors): void
    {
        self::error('Validation failed', 422, $errors);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

try {
    $request = new ApiRequest();
    $data = $request->all();
    
    // 验证数据
    $validator = new InputValidator();
    $validation = $validator->validate($data, [
        'name' => ['required' => true, 'type' => 'string'],
        'email' => ['required' => true, 'type' => 'email'],
    ]);
    
    if (!$validation['valid']) {
        ApiResponse::validationError($validation['errors']);
    }
    
    // 处理业务逻辑
    $result = processData($data);
    
    ApiResponse::success($result, 'Data processed successfully');
    
} catch (InvalidArgumentException $e) {
    ApiResponse::error($e->getMessage(), 400);
} catch (Exception $e) {
    ApiResponse::error('Internal server error', 500);
}
```

## 注意事项

1. **Content-Type 检测**：正确识别请求的 Content-Type
2. **错误处理**：提供统一的错误响应格式
3. **安全性**：验证和清理所有输入数据
4. **性能**：合理使用 `json_validate()` 提升性能

## 练习

1. 实现一个支持多种 Content-Type 的请求解析类。

2. 创建一个统一的 API 响应处理类，支持成功和错误响应。

3. 实现请求验证和错误处理的完整流程。

4. 编写一个 API 中间件，自动解析请求并处理错误。

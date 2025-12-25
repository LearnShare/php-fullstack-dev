# 4.4.1 JSON 请求解析

## 概述

处理 JSON API 请求是现代 Web 开发的核心技能。本节详细介绍为什么 `$_POST` 在 JSON 请求时为空、如何使用 `php://input` 读取原始请求体、`json_validate()` 函数的使用，以及 JSON 解码的安全处理。

## 为什么 $_POST 是空的

### Content-Type 的作用

- `$_POST` 仅在 `Content-Type: application/x-www-form-urlencoded` 或 `multipart/form-data` 时自动填充
- 当 `Content-Type: application/json` 时，数据不会自动填充到 `$_POST`
- 需要手动读取原始请求体

### 请求类型对比

| Content-Type | 数据位置 | 自动填充 |
| :--- | :--- | :--- |
| `application/x-www-form-urlencoded` | `$_POST` | 是 |
| `multipart/form-data` | `$_POST`、`$_FILES` | 是 |
| `application/json` | `php://input` | 否 |
| `text/xml` | `php://input` | 否 |

## 读取原始请求体

### php://input 流

- `php://input` 是一个只读流，允许读取原始 POST 数据
- 只能读取一次，读取后流指针会移动
- 比 `$HTTP_RAW_POST_DATA` 更灵活（已废弃）

### 基础用法

```php
<?php
declare(strict_types=1);

// 读取原始请求体
$rawInput = file_get_contents('php://input');

// 解析 JSON
$data = json_decode($rawInput, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    throw new InvalidArgumentException('Invalid JSON: ' . json_last_error_msg());
}
```

### json_validate() 函数（PHP 8.3+）

```php
<?php
declare(strict_types=1);

$rawInput = file_get_contents('php://input');

// PHP 8.3+ 推荐方式：先验证再解码
if (function_exists('json_validate') && json_validate($rawInput)) {
    $data = json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR);
} else {
    // PHP 8.2 及以下：直接解码并检查错误
    $data = json_decode($rawInput, true);
    if (json_last_error() !== JSON_ERROR_NONE) {
        throw new InvalidArgumentException('Invalid JSON');
    }
}
```

## JSON 解码安全处理

### 深度限制

```php
<?php
declare(strict_types=1);

// 限制 JSON 深度，防止栈溢出攻击
$data = json_decode($rawInput, true, 10);  // 最大深度 10
```

### 大小限制

```php
<?php
declare(strict_types=1);

// 限制请求体大小
$maxSize = 1024 * 1024;  // 1MB
$rawInput = file_get_contents('php://input');

if (strlen($rawInput) > $maxSize) {
    throw new InvalidArgumentException('Request body too large');
}
```

### 错误处理

```php
<?php
declare(strict_types=1);

function parseJsonRequest(): array
{
    $rawInput = file_get_contents('php://input');
    
    if (empty($rawInput)) {
        throw new InvalidArgumentException('Empty request body');
    }

    // PHP 8.3+
    if (function_exists('json_validate')) {
        if (!json_validate($rawInput)) {
            throw new InvalidArgumentException('Invalid JSON format');
        }
        return json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR);
    }

    // PHP 8.2 及以下
    $data = json_decode($rawInput, true, 10);
    if (json_last_error() !== JSON_ERROR_NONE) {
        throw new InvalidArgumentException('JSON decode error: ' . json_last_error_msg());
    }

    return $data;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class JsonRequestParser
{
    private const MAX_SIZE = 1024 * 1024;  // 1MB
    private const MAX_DEPTH = 10;

    public function parse(): array
    {
        $rawInput = $this->readInput();
        $this->validateSize($rawInput);
        
        if (function_exists('json_validate')) {
            if (!json_validate($rawInput)) {
                throw new InvalidArgumentException('Invalid JSON');
            }
            return json_decode($rawInput, true, self::MAX_DEPTH, JSON_THROW_ON_ERROR);
        }

        $data = json_decode($rawInput, true, self::MAX_DEPTH);
        if (json_last_error() !== JSON_ERROR_NONE) {
            throw new InvalidArgumentException('JSON error: ' . json_last_error_msg());
        }

        return $data;
    }

    private function readInput(): string
    {
        $input = file_get_contents('php://input');
        if ($input === false) {
            throw new RuntimeException('Failed to read input');
        }
        return $input;
    }

    private function validateSize(string $input): void
    {
        if (strlen($input) > self::MAX_SIZE) {
            throw new InvalidArgumentException('Request body too large');
        }
    }
}
```

## 注意事项

1. **只能读取一次**：`php://input` 只能读取一次，需要缓存内容
2. **大小限制**：设置合理的请求体大小限制
3. **深度限制**：限制 JSON 深度，防止栈溢出
4. **错误处理**：提供清晰的错误信息

## 练习

1. 实现一个 JSON 请求解析类，包含验证、解码、错误处理。

2. 创建一个 API 请求处理函数，能够安全地解析 JSON 请求体。

3. 实现请求体大小和 JSON 深度的限制功能。

4. 编写一个函数，支持多种 Content-Type 的请求体解析。

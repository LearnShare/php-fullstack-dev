# 4.4 请求体解析：处理 JSON / API 请求

## 目标

- 理解为什么 AJAX / API 传 JSON 时 `$_POST` 是空的。
- 掌握使用 `php://input` 读取原始请求体。
- 熟悉 JSON 序列化与反序列化的安全处理。
- 能够正确处理各种 Content-Type 的 API 请求。

## 为什么 $_POST 是空的

### Content-Type 的作用

- `$_POST` 仅在 `Content-Type: application/x-www-form-urlencoded` 或 `multipart/form-data` 时自动填充。
- 当 `Content-Type: application/json` 时，数据不会自动填充到 `$_POST`。
- 需要手动读取原始请求体。

### 请求类型对比

| Content-Type                          | 数据位置           | 自动填充 |
| :------------------------------------- | :----------------- | :------- |
| `application/x-www-form-urlencoded`   | `$_POST`           | 是       |
| `multipart/form-data`                  | `$_POST`、`$_FILES` | 是       |
| `application/json`                     | `php://input`      | 否       |
| `text/xml`                             | `php://input`      | 否       |

## 读取原始请求体

### php://input 流

- `php://input` 是一个只读流，允许读取原始 POST 数据。
- 只能读取一次，读取后流指针会移动。
- 比 `$HTTP_RAW_POST_DATA` 更灵活（已废弃）。

### json_validate() 函数（PHP 8.3+）

- **语法**：`json_validate(string $json, int $depth = 512, int $flags = 0): bool`
- **说明**：验证 JSON 字符串是否有效，无需解码，性能更好。
- **返回值**：`true` 表示 JSON 有效，`false` 表示无效。

```php
<?php
declare(strict_types=1);

// PHP 8.3+ 推荐方式：先验证再解码
if (function_exists('json_validate') && json_validate($json)) {
    $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
} else {
    // PHP 8.2 及以下：直接解码并检查错误
    $data = json_decode($json, true);
    if (json_last_error() !== JSON_ERROR_NONE) {
        throw new InvalidArgumentException('Invalid JSON');
    }
}
```

### 性能对比

```php
<?php
declare(strict_types=1);

// 方式一：直接解码（PHP 8.2 及以下）
function decodeJsonLegacy(string $json): array
{
    $data = json_decode($json, true);
    if (json_last_error() !== JSON_ERROR_NONE) {
        throw new InvalidArgumentException('Invalid JSON');
    }
    return $data;
}

// 方式二：先验证再解码（PHP 8.3+，性能更好）
function decodeJsonModern(string $json): array
{
    if (!json_validate($json)) {
        throw new InvalidArgumentException('Invalid JSON');
    }
    return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
}

// 性能优势：json_validate() 只验证不解码，比 json_decode() 更快
// 特别适合需要多次验证但只解码一次的场景
```

```php
<?php
declare(strict_types=1);

// 读取原始请求体
$rawInput = file_get_contents('php://input');

// 对于 JSON 请求
$data = json_decode($rawInput, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid JSON']);
    exit;
}

// 使用数据
$name = $data['name'] ?? '';
$email = $data['email'] ?? '';
```

### 完整示例

```php
<?php
declare(strict_types=1);

/**
 * 获取 JSON 输入数据
 * 
 * @return array 解析后的 JSON 数据，如果请求体为空则返回空数组
 * @throws InvalidArgumentException 当 Content-Type 不是 application/json 或 JSON 无效时
 */
function getJsonInput(): array
{
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    
    // 检查 Content-Type
    if (strpos($contentType, 'application/json') === false) {
        throw new InvalidArgumentException('Content-Type must be application/json');
    }
    
    // 读取原始输入
    $rawInput = file_get_contents('php://input');
    
    if (empty($rawInput)) {
        return [];
    }
    
    // PHP 8.3+ 使用 json_validate() 提升性能
    if (function_exists('json_validate') && !json_validate($rawInput)) {
        throw new InvalidArgumentException('Invalid JSON');
    }
    
    // 解析 JSON
    $data = json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR);
    
    return $data ?? [];
}

// 使用
try {
    $data = getJsonInput();
    $name = $data['name'] ?? '';
    $email = $data['email'] ?? '';
    
    // 处理数据
    echo json_encode(['success' => true, 'data' => $data]);
} catch (InvalidArgumentException $e) {
    http_response_code(400);
    echo json_encode(['error' => $e->getMessage()]);
}
```

## JSON 安全处理

### json_decode 参数

- **语法**：`json_decode(string $json, ?bool $assoc = null, int $depth = 512, int $flags = 0): mixed`
- **参数说明**：
  - `$json`：要解码的 JSON 字符串。
  - `$assoc`：为 `true` 时返回数组，`false` 时返回对象。
  - `$depth`：最大嵌套深度，防止深度嵌套攻击。
  - `$flags`：解码选项（如 `JSON_THROW_ON_ERROR`）。

```php
<?php
declare(strict_types=1);

// 安全解码
$data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);

// 或使用 try-catch
try {
    $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    // 处理 JSON 错误
    error_log('JSON decode error: ' . $e->getMessage());
    throw new InvalidArgumentException('Invalid JSON');
}
```

### 深度限制

```php
<?php
// 限制嵌套深度，防止攻击
$maxDepth = 10;
$data = json_decode($json, true, $maxDepth, JSON_THROW_ON_ERROR);
```

### 大小限制

```php
<?php
declare(strict_types=1);

function getJsonInput(int $maxSize = 1048576): array // 默认 1MB
{
    $contentLength = (int) ($_SERVER['CONTENT_LENGTH'] ?? 0);
    
    if ($contentLength > $maxSize) {
        throw new InvalidArgumentException('Request body too large');
    }
    
    $rawInput = file_get_contents('php://input');
    
    if (strlen($rawInput) > $maxSize) {
        throw new InvalidArgumentException('Request body too large');
    }
    
    return json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR) ?? [];
}
```

## 请求处理类

### 完整的请求处理示例

```php
<?php
declare(strict_types=1);

class JsonRequest
{
    private array $data = [];
    private bool $parsed = false;

    public function __construct()
    {
        $this->parse();
    }

    private function parse(): void
    {
        if ($this->parsed) {
            return;
        }

        $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
        
        // 只处理 JSON 请求
        if (strpos($contentType, 'application/json') === false) {
            return;
        }

        $rawInput = file_get_contents('php://input');
        
        if (empty($rawInput)) {
            $this->data = [];
            $this->parsed = true;
            return;
        }

        // 检查大小
        $maxSize = 1024 * 1024; // 1MB
        if (strlen($rawInput) > $maxSize) {
            throw new RuntimeException('Request body too large');
        }

        // 解析 JSON
        $this->data = json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR) ?? [];
        $this->parsed = true;
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return $this->data[$key] ?? $default;
    }

    public function all(): array
    {
        return $this->data;
    }

    public function has(string $key): bool
    {
        return isset($this->data[$key]);
    }

    public function validate(array $rules): array
    {
        $errors = [];
        
        foreach ($rules as $key => $rule) {
            $value = $this->get($key);
            
            if (strpos($rule, 'required') !== false && empty($value)) {
                $errors[$key] = "{$key} is required";
            }
            
            if (strpos($rule, 'email') !== false && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
                $errors[$key] = "{$key} must be a valid email";
            }
            
            // 更多验证规则...
        }
        
        return $errors;
    }
}

// 使用
$request = new JsonRequest();
$name = $request->get('name');
$email = $request->get('email');

$errors = $request->validate([
    'name' => 'required',
    'email' => 'required|email',
]);

if (!empty($errors)) {
    http_response_code(422);
    echo json_encode(['errors' => $errors]);
    exit;
}
```

## 处理不同 Content-Type

### 统一请求处理

```php
<?php
declare(strict_types=1);

class Request
{
    public static function getBody(): array
    {
        $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
        
        if (strpos($contentType, 'application/json') !== false) {
            return self::getJsonBody();
        }
        
        if (strpos($contentType, 'application/x-www-form-urlencoded') !== false) {
            return $_POST;
        }
        
        if (strpos($contentType, 'multipart/form-data') !== false) {
            return $_POST;
        }
        
        return [];
    }

    private static function getJsonBody(): array
    {
        $rawInput = file_get_contents('php://input');
        
        if (empty($rawInput)) {
            return [];
        }
        
        try {
            return json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR) ?? [];
        } catch (JsonException $e) {
            throw new InvalidArgumentException('Invalid JSON: ' . $e->getMessage());
        }
    }
}

// 使用
$data = Request::getBody();
```

## JSON 序列化安全

### json_encode 选项

```php
<?php
// 安全编码
$data = [
    'name' => 'Alice',
    'email' => 'alice@example.com',
];

// 使用安全选项
$json = json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);

// 避免使用 JSON_HEX_TAG 等选项，除非有特殊需求
```

### 防止 JSON 注入

```php
<?php
declare(strict_types=1);

function safeJsonEncode(mixed $data): string
{
    $json = json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    
    if (json_last_error() !== JSON_ERROR_NONE) {
        throw new RuntimeException('JSON encode error: ' . json_last_error_msg());
    }
    
    return $json;
}

// 使用
$response = [
    'success' => true,
    'data' => $userData,
];

header('Content-Type: application/json; charset=UTF-8');
echo safeJsonEncode($response);
```

## API 响应格式

### 标准 JSON 响应

```php
<?php
declare(strict_types=1);

class JsonResponse
{
    public static function success(mixed $data = null, int $code = 200): void
    {
        http_response_code($code);
        header('Content-Type: application/json; charset=UTF-8');
        
        echo json_encode([
            'success' => true,
            'data' => $data,
        ], JSON_UNESCAPED_UNICODE);
    }

    public static function error(string $message, int $code = 400, array $errors = []): void
    {
        http_response_code($code);
        header('Content-Type: application/json; charset=UTF-8');
        
        $response = [
            'success' => false,
            'error' => $message,
        ];
        
        if (!empty($errors)) {
            $response['errors'] = $errors;
        }
        
        echo json_encode($response, JSON_UNESCAPED_UNICODE);
    }
}

// 使用
try {
    $data = processRequest();
    JsonResponse::success($data);
} catch (Exception $e) {
    JsonResponse::error($e->getMessage(), 500);
}
```

## 实际应用示例

### RESTful API 端点

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

header('Content-Type: application/json; charset=UTF-8');

$method = $_SERVER['REQUEST_METHOD'];
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// 路由处理
if ($method === 'POST' && $path === '/api/users') {
    try {
        // 读取 JSON 请求体
        $rawInput = file_get_contents('php://input');
        $data = json_decode($rawInput, true, 512, JSON_THROW_ON_ERROR);
        
        // 验证数据
        if (empty($data['name']) || empty($data['email'])) {
            http_response_code(422);
            echo json_encode(['error' => 'Name and email are required']);
            exit;
        }
        
        // 处理业务逻辑
        $user = createUser($data['name'], $data['email']);
        
        // 返回响应
        http_response_code(201);
        echo json_encode([
            'success' => true,
            'data' => $user,
        ], JSON_UNESCAPED_UNICODE);
        
    } catch (JsonException $e) {
        http_response_code(400);
        echo json_encode(['error' => 'Invalid JSON']);
    } catch (Exception $e) {
        http_response_code(500);
        echo json_encode(['error' => 'Internal server error']);
    }
}
```

## 最佳实践

### 1. 始终检查 Content-Type

```php
$contentType = $_SERVER['CONTENT_TYPE'] ?? '';
if (strpos($contentType, 'application/json') === false) {
    http_response_code(415);
    exit('Unsupported Media Type');
}
```

### 2. 限制请求体大小

```php
$maxSize = 1024 * 1024; // 1MB
$contentLength = (int) ($_SERVER['CONTENT_LENGTH'] ?? 0);
if ($contentLength > $maxSize) {
    http_response_code(413);
    exit('Payload Too Large');
}
```

### 3. 使用 JSON_THROW_ON_ERROR

```php
// 推荐：抛出异常
$data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);

// 不推荐：手动检查错误
$data = json_decode($json, true);
if (json_last_error() !== JSON_ERROR_NONE) {
    // 处理错误
}
```

### 4. 统一错误处理

```php
try {
    $data = json_decode($input, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    return JsonResponse::error('Invalid JSON format', 400);
}
```

## 练习

1. 创建一个 `JsonRequest` 类，封装 JSON 请求的读取、解析和验证功能。

2. 实现一个 API 端点，接收 JSON 请求，验证数据，并返回标准 JSON 响应。

3. 编写一个请求体大小限制中间件，拒绝超过指定大小的请求。

4. 创建一个统一的请求处理函数，能够根据 Content-Type 自动选择解析方式。

5. 实现一个 JSON 响应格式化类，提供 success、error、validation 等方法。

6. 设计一个 API 请求验证器，支持必填字段、类型验证、格式验证等规则。

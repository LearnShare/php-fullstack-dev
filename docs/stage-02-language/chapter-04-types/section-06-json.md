# 2.4.6 JSON 与数组互转

## 概述

JSON（JavaScript Object Notation）是数据交换的常用格式，PHP 提供了 `json_encode()` 和 `json_decode()` 函数进行 JSON 编码和解码。本节详细介绍这两个函数的语法、常用选项标志、错误处理、实际应用场景。

掌握 JSON 编码和解码是 Web 开发和 API 开发的基础。JSON 格式简洁、易读，广泛用于数据交换、配置文件、日志记录等场景。

## 特性

- **编码**：将 PHP 值（数组、对象等）转换为 JSON 字符串
- **解码**：将 JSON 字符串转换为 PHP 值（数组、对象）
- **选项控制**：通过选项标志控制编码/解码行为
- **错误处理**：提供错误检测和处理机制
- **UTF-8 支持**：完整支持 UTF-8 编码

## 语法/定义

### json_encode() - JSON 编码

**语法**：`json_encode(mixed $value, int $flags = 0, int $depth = 512): string|false`

**参数**：
- `$value`：要编码的值（任意类型，通常是数组或对象）
- `$flags`：可选，选项标志的位掩码，默认为 0
- `$depth`：可选，最大深度，默认为 512

**返回值**：
- 成功：返回 JSON 字符串（类型为 `string`）
- 失败：返回 `false`

**常用选项标志**：
- `JSON_PRETTY_PRINT`：格式化输出，提高可读性
- `JSON_UNESCAPED_UNICODE`：不转义 Unicode 字符
- `JSON_UNESCAPED_SLASHES`：不转义斜杠
- `JSON_NUMERIC_CHECK`：将数字字符串转换为数字
- `JSON_THROW_ON_ERROR`：错误时抛出异常（PHP 7.3+）

### json_decode() - JSON 解码

**语法**：`json_decode(string $json, ?bool $associative = null, int $depth = 512, int $flags = 0): mixed`

**参数**：
- `$json`：要解码的 JSON 字符串（类型为 `string`）
- `$associative`：可选，如果为 `true`，返回关联数组；如果为 `false`，返回对象；如果为 `null`，根据 JSON 内容决定
- `$depth`：可选，最大深度，默认为 512
- `$flags`：可选，选项标志的位掩码，默认为 0

**返回值**：
- 成功：返回解码后的 PHP 值（数组或对象）
- 失败：返回 `null`（如果使用 `JSON_THROW_ON_ERROR` 标志会抛出异常）

**常用选项标志**：
- `JSON_OBJECT_AS_ARRAY`：将对象解码为关联数组
- `JSON_BIGINT_AS_STRING`：将大整数解码为字符串
- `JSON_INVALID_UTF8_IGNORE`：忽略无效的 UTF-8 字符
- `JSON_THROW_ON_ERROR`：错误时抛出异常（PHP 7.3+）

### 错误检测函数

**json_last_error()**：`json_last_error(): int`

**返回值**：返回最后一次 JSON 操作的错误代码

**错误代码常量**：
- `JSON_ERROR_NONE`：没有错误
- `JSON_ERROR_DEPTH`：达到最大深度
- `JSON_ERROR_STATE_MISMATCH`：无效或格式错误的 JSON
- `JSON_ERROR_CTRL_CHAR`：控制字符错误
- `JSON_ERROR_SYNTAX`：语法错误
- `JSON_ERROR_UTF8`：UTF-8 字符错误
- `JSON_ERROR_RECURSION`：递归错误
- `JSON_ERROR_INF_OR_NAN`：INF 或 NAN 值
- `JSON_ERROR_UNSUPPORTED_TYPE`：不支持的类型

**json_last_error_msg()**：`json_last_error_msg(): string`

**返回值**：返回最后一次 JSON 操作的错误消息（类型为 `string`）

## 基本用法

### 示例 1：基本编码和解码

```php
<?php
declare(strict_types=1);

// 编码数组为 JSON
$data = [
    'name' => 'Alice',
    'age' => 25,
    'hobbies' => ['reading', 'coding'],
];

$json = json_encode($data);
echo "JSON: {$json}\n";

// 解码 JSON 为数组
$decoded = json_decode($json, true);
print_r($decoded);

// 解码为对象
$decodedObj = json_decode($json, false);
echo "Name: {$decodedObj->name}\n";
echo "Age: {$decodedObj->age}\n";
```

**执行**：

```bash
php basic-json.php
```

**输出**：

```
JSON: {"name":"Alice","age":25,"hobbies":["reading","coding"]}
Array
(
    [name] => Alice
    [age] => 25
    [hobbies] => Array
        (
            [0] => reading
            [1] => coding
        )
)
Name: Alice
Age: 25
```

### 示例 2：选项标志使用

```php
<?php
declare(strict_types=1);

$data = [
    'name' => '张三',
    'age' => 25,
    'url' => 'https://example.com',
];

// 基本编码
$json1 = json_encode($data);
echo "Basic: {$json1}\n";

// 格式化输出
$json2 = json_encode($data, JSON_PRETTY_PRINT);
echo "Pretty: {$json2}\n";

// 不转义 Unicode
$json3 = json_encode($data, JSON_UNESCAPED_UNICODE);
echo "Unicode: {$json3}\n";

// 组合选项
$json4 = json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
echo "Combined: {$json4}\n";
```

**执行**：

```bash
php flags-example.php
```

**输出**：

```
Basic: {"name":"\u5f20\u4e09","age":25,"url":"https:\/\/example.com"}
Pretty: {
    "name": "\u5f20\u4e09",
    "age": 25,
    "url": "https:\/\/example.com"
}
Unicode: {"name":"张三","age":25,"url":"https:\/\/example.com"}
Combined: {
    "name": "张三",
    "age": 25,
    "url": "https://example.com"
}
```

### 示例 3：错误处理

```php
<?php
declare(strict_types=1);

// 编码错误处理
$resource = fopen('test.txt', 'r');
$json = json_encode($resource);
if ($json === false) {
    $error = json_last_error_msg();
    echo "Encode error: {$error}\n";
}
fclose($resource);

// 解码错误处理
$invalidJson = '{"name": "Alice", "age": }';  // 无效 JSON
$decoded = json_decode($invalidJson, true);
if ($decoded === null) {
    $error = json_last_error();
    $errorMsg = json_last_error_msg();
    echo "Decode error code: {$error}\n";
    echo "Decode error message: {$errorMsg}\n";
}

// 使用 JSON_THROW_ON_ERROR（PHP 7.3+）
try {
    $json = json_encode($resource, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    echo "Exception: " . $e->getMessage() . "\n";
}
```

### 示例 4：实际应用场景

```php
<?php
declare(strict_types=1);

// API 响应
function apiResponse(array $data, int $statusCode = 200): string
{
    $response = [
        'status' => $statusCode === 200 ? 'success' : 'error',
        'data' => $data,
        'timestamp' => time(),
    ];
    
    return json_encode($response, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
}

$data = ['users' => [['id' => 1, 'name' => 'Alice']]];
echo apiResponse($data) . "\n";

// 配置文件读取
function loadConfig(string $filename): array
{
    $json = file_get_contents($filename);
    if ($json === false) {
        throw new RuntimeException("Failed to read config file");
    }
    
    $config = json_decode($json, true);
    if ($config === null) {
        $error = json_last_error_msg();
        throw new RuntimeException("Invalid JSON: {$error}");
    }
    
    return $config;
}
```

## 完整代码示例

### 示例 1：完整的 JSON 工具类

```php
<?php
declare(strict_types=1);

class JsonHelper
{
    /**
     * 编码为 JSON（带错误处理）
     */
    public static function encode(mixed $value, int $flags = 0): string
    {
        $flags |= JSON_UNESCAPED_UNICODE;  // 默认不转义 Unicode
        
        $json = json_encode($value, $flags);
        if ($json === false) {
            $error = json_last_error_msg();
            throw new RuntimeException("JSON encode error: {$error}");
        }
        
        return $json;
    }
    
    /**
     * 解码 JSON（带错误处理）
     */
    public static function decode(string $json, bool $associative = true): mixed
    {
        $decoded = json_decode($json, $associative);
        if ($decoded === null && json_last_error() !== JSON_ERROR_NONE) {
            $error = json_last_error_msg();
            throw new RuntimeException("JSON decode error: {$error}");
        }
        
        return $decoded;
    }
    
    /**
     * 格式化 JSON
     */
    public static function prettyEncode(mixed $value): string
    {
        return self::encode($value, JSON_PRETTY_PRINT);
    }
}

// 使用
$data = ['name' => 'Alice', 'age' => 25];
$json = JsonHelper::prettyEncode($data);
echo $json . "\n";

$decoded = JsonHelper::decode($json);
print_r($decoded);
```

### 示例 2：API 数据交换

```php
<?php
declare(strict_types=1);

// 发送 JSON 响应
function sendJsonResponse(array $data, int $statusCode = 200): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json; charset=UTF-8');
    
    $response = [
        'status' => $statusCode === 200 ? 'success' : 'error',
        'data' => $data,
    ];
    
    echo json_encode($response, JSON_UNESCAPED_UNICODE);
    exit;
}

// 接收 JSON 请求
function getJsonRequest(): array
{
    $input = file_get_contents('php://input');
    if ($input === false) {
        throw new RuntimeException("Failed to read request body");
    }
    
    $data = json_decode($input, true);
    if ($data === null) {
        $error = json_last_error_msg();
        throw new RuntimeException("Invalid JSON: {$error}");
    }
    
    return $data;
}

// 使用示例（Web 环境）
// sendJsonResponse(['message' => 'Hello, World!']);
// $requestData = getJsonRequest();
```

### 示例 3：配置文件管理

```php
<?php
declare(strict_types=1);

class ConfigManager
{
    public static function load(string $filename): array
    {
        if (!file_exists($filename)) {
            throw new RuntimeException("Config file not found: {$filename}");
        }
        
        $json = file_get_contents($filename);
        if ($json === false) {
            throw new RuntimeException("Failed to read config file");
        }
        
        $config = json_decode($json, true);
        if ($config === null) {
            $error = json_last_error_msg();
            throw new RuntimeException("Invalid JSON in config: {$error}");
        }
        
        return $config;
    }
    
    public static function save(string $filename, array $config): void
    {
        $json = json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
        if ($json === false) {
            $error = json_last_error_msg();
            throw new RuntimeException("Failed to encode config: {$error}");
        }
        
        $result = file_put_contents($filename, $json);
        if ($result === false) {
            throw new RuntimeException("Failed to write config file");
        }
    }
}

// 使用
$config = ConfigManager::load('config.json');
$config['new_key'] = 'new_value';
ConfigManager::save('config.json', $config);
```

## 使用场景

### API 数据交换

- **RESTful API**：发送和接收 JSON 数据
- **前后端通信**：前端和后端之间的数据交换
- **微服务通信**：服务之间的数据交换

### 配置文件

- **应用配置**：存储应用配置信息
- **用户设置**：存储用户设置和偏好
- **缓存数据**：存储缓存数据

### 数据持久化

- **数据存储**：将数据保存为 JSON 格式
- **日志记录**：结构化日志记录
- **数据备份**：数据备份和恢复

### 数据转换

- **数组转换**：数组和 JSON 之间的转换
- **对象序列化**：对象序列化为 JSON
- **数据格式化**：格式化输出数据

## 注意事项

### 编码问题

- **UTF-8 编码**：确保数据使用 UTF-8 编码
- **Unicode 转义**：使用 `JSON_UNESCAPED_UNICODE` 避免 Unicode 转义
- **字符编码**：注意字符编码的一致性

### 深度限制

- **默认深度**：默认最大深度为 512
- **深度限制**：超过深度限制会导致编码/解码失败
- **调整深度**：根据数据调整 `$depth` 参数

### 错误处理

- **检查返回值**：始终检查 `json_encode()` 和 `json_decode()` 的返回值
- **使用 JSON_THROW_ON_ERROR**：PHP 7.3+ 使用 `JSON_THROW_ON_ERROR` 标志
- **错误消息**：使用 `json_last_error_msg()` 获取详细错误信息

### 类型转换

- **数组 vs 对象**：使用 `$associative` 参数控制返回类型
- **数字字符串**：注意数字字符串和数字的区别
- **null 值**：JSON 中的 `null` 解码为 PHP 的 `null`

## 常见问题

### 问题 1：JSON 编码失败

**症状**：`json_encode()` 返回 `false`

**原因**：包含无法编码的值（如资源、循环引用等）

**错误示例**：

```php
<?php
declare(strict_types=1);

$resource = fopen('test.txt', 'r');
$data = ['file' => $resource];
$json = json_encode($data);  // false
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：移除无法编码的值
$data = ['name' => 'Alice'];
$json = json_encode($data);

// 方法2：转换为可编码的值
$resource = fopen('test.txt', 'r');
$data = ['file' => 'resource_id_' . (int) $resource];
$json = json_encode($data);

// 方法3：使用 JSON_THROW_ON_ERROR
try {
    $json = json_encode($data, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### 问题 2：JSON 解码失败

**症状**：`json_decode()` 返回 `null`

**原因**：JSON 格式错误或无效

**错误示例**：

```php
<?php
declare(strict_types=1);

$invalidJson = '{"name": "Alice", "age": }';  // 无效 JSON
$decoded = json_decode($invalidJson, true);   // null
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$json = '{"name": "Alice", "age": 25}';
$decoded = json_decode($json, true);

if ($decoded === null) {
    $error = json_last_error();
    if ($error !== JSON_ERROR_NONE) {
        $errorMsg = json_last_error_msg();
        echo "Decode error: {$errorMsg}\n";
    }
} else {
    print_r($decoded);
}

// 使用 JSON_THROW_ON_ERROR
try {
    $decoded = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### 问题 3：Unicode 字符转义

**症状**：中文字符被转义为 `\uXXXX` 格式

**原因**：未使用 `JSON_UNESCAPED_UNICODE` 标志

**解决方法**：

```php
<?php
declare(strict_types=1);

$data = ['name' => '张三'];

// 默认会转义
$json1 = json_encode($data);
echo $json1 . "\n";  // {"name":"\u5f20\u4e09"}

// 不转义 Unicode
$json2 = json_encode($data, JSON_UNESCAPED_UNICODE);
echo $json2 . "\n";  // {"name":"张三"}
```

## 最佳实践

### 编码

- **使用选项标志**：根据需求使用合适的选项标志
- **格式化输出**：开发环境使用 `JSON_PRETTY_PRINT` 提高可读性
- **Unicode 处理**：使用 `JSON_UNESCAPED_UNICODE` 处理中文
- **错误处理**：始终检查返回值和处理错误

### 解码

- **指定返回类型**：明确指定 `$associative` 参数
- **错误处理**：检查返回值和使用 `json_last_error()`
- **使用 JSON_THROW_ON_ERROR**：PHP 7.3+ 使用异常处理错误
- **验证数据**：解码后验证数据结构和内容

### 错误处理

- **检查返回值**：始终检查 `json_encode()` 和 `json_decode()` 的返回值
- **使用异常**：PHP 7.3+ 使用 `JSON_THROW_ON_ERROR` 标志
- **错误消息**：使用 `json_last_error_msg()` 获取详细错误信息
- **日志记录**：记录 JSON 操作错误，便于调试

### 性能优化

- **缓存结果**：如果可能，缓存 JSON 编码结果
- **避免重复编码**：避免在循环中重复编码相同数据
- **深度控制**：根据实际数据调整 `$depth` 参数

## 对比分析

### 数组 vs 对象返回

| 特性 | 关联数组 | 对象 |
|:-----|:---------|:------|
| 访问方式 | `$arr['key']` | `$obj->key` |
| 类型安全 | 弱 | 中 |
| 性能 | 略好 | 略差 |
| 推荐场景 | 数据处理 | 面向对象 |

**选择建议**：
- **数据处理**：使用关联数组
- **面向对象**：使用对象
- **API 响应**：通常使用关联数组

### JSON vs 其他格式

| 特性 | JSON | XML | YAML |
|:-----|:-----|:-----|:------|
| 可读性 | 高 | 中 | 高 |
| 解析性能 | 好 | 差 | 中 |
| 数据大小 | 小 | 大 | 中 |
| 支持度 | 高 | 高 | 中 |

**选择建议**：
- **API 数据交换**：使用 JSON
- **配置文件**：使用 JSON 或 YAML
- **文档格式**：使用 XML 或 Markdown

## 相关章节

- **2.4.2 复合类型**：了解数组和对象
- **2.8 数组完整指南**：详细了解数组操作
- **阶段五：Web/API 开发**：详细了解 API 开发
- **2.2.1 输出 API**：了解输出函数

## 练习任务

1. **基本编码解码练习**：
   - 练习将数组编码为 JSON
   - 练习将 JSON 解码为数组和对象
   - 理解不同返回类型的区别
   - 测试各种数据类型的编码

2. **选项标志练习**：
   - 练习使用 `JSON_PRETTY_PRINT` 格式化输出
   - 练习使用 `JSON_UNESCAPED_UNICODE` 处理中文
   - 练习组合多个选项标志
   - 理解各选项的作用

3. **错误处理练习**：
   - 练习检测编码错误
   - 练习检测解码错误
   - 使用 `JSON_THROW_ON_ERROR` 处理错误
   - 实现完整的错误处理机制

4. **实际应用练习**：
   - 实现一个 JSON 工具类
   - 实现 API 响应函数
   - 实现配置文件管理
   - 测试不同场景下的 JSON 操作

5. **综合练习**：
   - 创建一个完整的 JSON 处理工具
   - 实现编码、解码、格式化等功能
   - 实现完整的错误处理
   - 进行代码审查，确保代码质量

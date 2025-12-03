# 2.4.6 JSON 与数组互转

## 概述

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式。PHP 提供了 `json_encode()` 和 `json_decode()` 函数用于在 PHP 数据结构和 JSON 字符串之间进行转换。

## json_encode() - 编码为 JSON

### 基本语法

**语法**：`json_encode(mixed $value, int $flags = 0, int $depth = 512): string|false`

**参数**：
- `$value`：要编码的值（通常是数组或对象）
- `$flags`：可选，JSON 编码选项（位掩码）
- `$depth`：可选，最大嵌套深度

**返回值**：成功返回 JSON 字符串，失败返回 `false`。

### 基本示例

```php
<?php
declare(strict_types=1);

$data = [
    'name' => 'Alice',
    'age' => 25,
    'email' => 'alice@example.com'
];

$json = json_encode($data);
echo $json . "\n";
// 输出：{"name":"Alice","age":25,"email":"alice@example.com"}
```

### 常用选项标志

#### JSON_UNESCAPED_UNICODE

不转义 Unicode 字符：

```php
<?php
declare(strict_types=1);

$data = ['name' => '张三', 'city' => '北京'];
$json1 = json_encode($data);
echo $json1 . "\n";
// 输出：{"name":"\u5f20\u4e09","city":"\u5317\u4eac"}

$json2 = json_encode($data, JSON_UNESCAPED_UNICODE);
echo $json2 . "\n";
// 输出：{"name":"张三","city":"北京"}
```

#### JSON_PRETTY_PRINT

格式化输出，便于阅读：

```php
<?php
declare(strict_types=1);

$data = ['name' => 'Alice', 'age' => 25];
$json = json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
echo $json . "\n";
// 输出：
// {
//     "name": "Alice",
//     "age": 25
// }
```

#### JSON_NUMERIC_CHECK

将数字字符串转换为数字：

```php
<?php
declare(strict_types=1);

$data = ['id' => '123', 'count' => '456'];
$json = json_encode($data, JSON_NUMERIC_CHECK);
echo $json . "\n";
// 输出：{"id":123,"count":456}
```

#### 组合使用多个选项

```php
<?php
declare(strict_types=1);

$data = [
    'name' => '张三',
    'age' => 25,
    'tags' => ['php', 'mysql']
];

$json = json_encode(
    $data,
    JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT
);
echo $json . "\n";
```

### 处理编码错误

```php
<?php
declare(strict_types=1);

$data = ['name' => 'Alice'];

$json = json_encode($data);
if ($json === false) {
    $error = json_last_error_msg();
    echo "JSON encoding failed: {$error}\n";
} else {
    echo $json . "\n";
}
```

## json_decode() - 解码 JSON

### 基本语法

**语法**：`json_decode(string $json, ?bool $associative = null, int $depth = 512, int $flags = 0): mixed`

**参数**：
- `$json`：要解码的 JSON 字符串
- `$associative`：如果为 `true`，返回关联数组；如果为 `false`，返回对象；如果为 `null`，根据 JSON 内容决定
- `$depth`：可选，最大嵌套深度
- `$flags`：可选，JSON 解码选项

**返回值**：成功返回解码后的值，失败返回 `null`。

### 基本示例

```php
<?php
declare(strict_types=1);

$json = '{"name":"Alice","age":25,"email":"alice@example.com"}';

// 返回关联数组
$array = json_decode($json, true);
print_r($array);
// 输出：
// Array
// (
//     [name] => Alice
//     [age] => 25
//     [email] => alice@example.com
// )

// 返回对象
$object = json_decode($json, false);
var_dump($object);
// 输出：object(stdClass)#1 (3) { ... }
```

### 处理解码错误

#### 使用 JSON_THROW_ON_ERROR（PHP 7.3+）

```php
<?php
declare(strict_types=1);

$json = '{"name":"Alice"}';

try {
    $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
    print_r($data);
} catch (JsonException $e) {
    echo "JSON decode error: " . $e->getMessage() . "\n";
}
```

#### 传统错误处理

```php
<?php
declare(strict_types=1);

$json = 'invalid json';

$data = json_decode($json, true);
if (json_last_error() !== JSON_ERROR_NONE) {
    $error = json_last_error_msg();
    echo "JSON decode error: {$error}\n";
}
```

### JSON 错误代码

| 常量              | 值   | 说明                     |
| :---------------- | :--- | :----------------------- |
| `JSON_ERROR_NONE` | 0    | 没有错误                 |
| `JSON_ERROR_DEPTH` | 1    | 超出最大嵌套深度         |
| `JSON_ERROR_STATE_MISMATCH` | 2 | 无效或格式错误的 JSON |
| `JSON_ERROR_CTRL_CHAR` | 3 | 控制字符错误           |
| `JSON_ERROR_SYNTAX` | 4 | 语法错误               |
| `JSON_ERROR_UTF8` | 5 | 编码错误               |

## 完整示例

### 示例 1：API 响应处理

```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data, string $message = 'Success'): string
    {
        $response = [
            'success' => true,
            'message' => $message,
            'data' => $data,
            'timestamp' => time()
        ];
        
        return json_encode($response, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
    }
    
    public static function error(string $message, int $code = 400): string
    {
        $response = [
            'success' => false,
            'message' => $message,
            'code' => $code,
            'timestamp' => time()
        ];
        
        return json_encode($response, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
    }
    
    public static function parse(string $json): array
    {
        try {
            return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        } catch (JsonException $e) {
            throw new RuntimeException("Invalid JSON: " . $e->getMessage());
        }
    }
}

// 使用示例
$response = ApiResponse::success(['user' => ['name' => 'Alice', 'age' => 25]]);
echo $response . "\n";

$parsed = ApiResponse::parse($response);
print_r($parsed);
```

### 示例 2：配置文件处理

```php
<?php
declare(strict_types=1);

class ConfigManager
{
    public static function loadFromJson(string $filename): array
    {
        if (!file_exists($filename)) {
            throw new RuntimeException("Config file not found: {$filename}");
        }
        
        $content = file_get_contents($filename);
        if ($content === false) {
            throw new RuntimeException("Cannot read config file: {$filename}");
        }
        
        try {
            return json_decode($content, true, 512, JSON_THROW_ON_ERROR);
        } catch (JsonException $e) {
            throw new RuntimeException("Invalid JSON in config file: " . $e->getMessage());
        }
    }
    
    public static function saveToJson(array $config, string $filename): void
    {
        $json = json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
        if ($json === false) {
            throw new RuntimeException("JSON encoding failed: " . json_last_error_msg());
        }
        
        if (file_put_contents($filename, $json) === false) {
            throw new RuntimeException("Cannot write config file: {$filename}");
        }
    }
}

// 使用示例
$config = [
    'app_name' => 'My Application',
    'version' => '1.0.0',
    'database' => [
        'host' => 'localhost',
        'port' => 3306,
        'name' => 'myapp'
    ]
];

ConfigManager::saveToJson($config, 'config.json');
$loaded = ConfigManager::loadFromJson('config.json');
print_r($loaded);
```

### 示例 3：处理嵌套数据结构

```php
<?php
declare(strict_types=1);

$complexData = [
    'users' => [
        [
            'id' => 1,
            'name' => 'Alice',
            'roles' => ['admin', 'user'],
            'metadata' => [
                'created_at' => '2024-01-01',
                'last_login' => '2024-01-15'
            ]
        ],
        [
            'id' => 2,
            'name' => 'Bob',
            'roles' => ['user'],
            'metadata' => [
                'created_at' => '2024-01-02',
                'last_login' => null
            ]
        ]
    ],
    'pagination' => [
        'page' => 1,
        'per_page' => 10,
        'total' => 2
    ]
];

$json = json_encode($complexData, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
echo $json . "\n";

// 解码
$decoded = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
echo "Total users: " . count($decoded['users']) . "\n";
```

## 注意事项

1. **编码错误处理**：始终检查 `json_encode()` 的返回值，失败时返回 `false`。

2. **解码错误处理**：使用 `JSON_THROW_ON_ERROR` 标志或检查 `json_last_error()`。

3. **Unicode 处理**：处理中文等 Unicode 字符时，使用 `JSON_UNESCAPED_UNICODE` 标志。

4. **深度限制**：注意 `$depth` 参数，默认 512 层嵌套。

5. **数据类型**：JSON 不支持 PHP 的所有数据类型（如资源、闭包等），这些类型会被忽略或转换为 null。

6. **大整数**：JavaScript 的 Number 类型精度有限，大整数可能丢失精度。

## 练习

1. 创建一个 `JsonHelper` 类，封装 JSON 编码和解码操作，提供错误处理和常用选项。

2. 编写一个函数 `validateJson(string $json): bool`，验证 JSON 字符串是否有效。

3. 实现一个函数 `jsonPrettyPrint(string $json): string`，格式化 JSON 字符串（如果已经是格式化的，则重新格式化）。

4. 创建一个 API 响应类，使用 JSON 格式返回标准化的 API 响应。

5. 编写一个函数，将 PHP 对象转换为 JSON，并处理循环引用的情况。

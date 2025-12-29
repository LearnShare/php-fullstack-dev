# 5.5.1 JSON 请求解析

## 概述

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，广泛用于 Web API 和前后端数据交互。JSON 请求解析是 API 开发的基础，理解 JSON 格式、掌握 JSON 解析方法、正确处理 JSON 数据，对于构建现代 Web 应用至关重要。本节详细介绍 JSON 格式的特点、`json_decode()` 函数的使用、请求体读取、错误处理、数据验证等内容，帮助零基础学员掌握 JSON 数据处理。

在 Web 开发中，客户端通常通过 HTTP POST 请求发送 JSON 格式的数据到服务器。PHP 需要读取请求体、解析 JSON 数据、验证数据格式、处理解析错误。理解这些过程对于构建健壮的 API 接口至关重要。

**主要内容**：
- JSON 格式概述（语法规则、数据类型、与 PHP 的对应关系）
- `json_decode()` 函数的语法、参数和返回值
- 请求体读取（`php://input` 流）
- JSON 解析选项（关联数组模式、深度限制、大数字处理）
- 错误处理（`json_last_error()`、错误类型判断）
- 数据验证和类型检查
- 实际应用示例和最佳实践

## 特性

- **轻量级**：JSON 格式简洁，易于阅读和编写
- **语言无关**：JSON 是语言无关的数据格式
- **类型支持**：支持字符串、数字、布尔值、数组、对象、null
- **自动解析**：PHP 提供 `json_decode()` 自动解析 JSON
- **错误处理**：提供详细的错误信息

## JSON 格式概述

### JSON 语法规则

JSON 数据必须遵循以下规则：

1. **数据格式**：键值对格式
2. **键名**：必须用双引号包裹
3. **值类型**：字符串、数字、布尔值、数组、对象、null
4. **分隔符**：使用逗号分隔元素
5. **结构**：使用花括号 `{}` 表示对象，方括号 `[]` 表示数组

### JSON 数据类型

| JSON 类型 | PHP 类型 | 示例 |
|:----------|:---------|:-----|
| `string` | `string` | `"hello"` |
| `number` | `int` 或 `float` | `123`、`45.67` |
| `boolean` | `bool` | `true`、`false` |
| `array` | `array` | `[1, 2, 3]` |
| `object` | `object` 或 `array` | `{"key": "value"}` |
| `null` | `null` | `null` |

### JSON 示例

**示例 1：简单对象**
```json
{
  "name": "John",
  "age": 30,
  "city": "New York"
}
```

**示例 2：数组**
```json
[
  {"id": 1, "name": "Alice"},
  {"id": 2, "name": "Bob"}
]
```

**示例 3：嵌套结构**
```json
{
  "user": {
    "id": 123,
    "name": "John",
    "email": "john@example.com"
  },
  "tags": ["php", "web", "development"]
}
```

## json_decode() 函数

### 语法

**语法**：`json_decode(string $json, ?bool $associative = null, int $depth = 512, int $flags = 0): mixed`

### 参数

- `$json`：要解码的 JSON 字符串
- `$associative`：可选，如果为 `true`，返回关联数组；如果为 `false`，返回对象；如果为 `null`（默认），根据 JSON 类型决定
- `$depth`：可选，最大嵌套深度（默认 512）
- `$flags`：可选，解码选项（`JSON_BIGINT_AS_STRING`、`JSON_INVALID_UTF8_IGNORE` 等）

### 返回值

- 成功：返回解码后的 PHP 值（数组、对象、标量）
- 失败：返回 `null`（需要检查 `json_last_error()`）

### 基本用法

**示例 1：基本解码**
```php
<?php
declare(strict_types=1);

$json = '{"name": "John", "age": 30}';
$data = json_decode($json, true);

print_r($data);
```

**输出**：
```
Array
(
    [name] => John
    [age] => 30
)
```

**示例 2：对象模式**
```php
<?php
declare(strict_types=1);

$json = '{"name": "John", "age": 30}';
$data = json_decode($json, false);  // 返回对象

echo $data->name;  // John
echo $data->age;   // 30
```

**示例 3：关联数组模式（推荐）**
```php
<?php
declare(strict_types=1);

$json = '{"name": "John", "age": 30}';
$data = json_decode($json, true);  // 返回关联数组

echo $data['name'];  // John
echo $data['age'];   // 30
```

### 关联数组模式

**推荐使用关联数组模式**（`$associative = true`），原因：
- 更符合 PHP 习惯
- 更容易处理数据
- 避免对象属性访问问题

**示例**：
```php
<?php
declare(strict_types=1);

$json = '{"user": {"name": "John", "age": 30}}';
$data = json_decode($json, true);

// 使用数组访问
echo $data['user']['name'];  // John
```

### 深度限制

**示例**：
```php
<?php
declare(strict_types=1);

$json = '{"level1": {"level2": {"level3": "value"}}}';

// 默认深度（512）
$data1 = json_decode($json, true);

// 限制深度为 2
$data2 = json_decode($json, true, 2);
// 如果 JSON 深度超过 2，返回 null
```

### 大数字处理

**问题**：JavaScript 中的大整数在 PHP 中可能丢失精度。

**解决方案**：使用 `JSON_BIGINT_AS_STRING` 标志。

**示例**：
```php
<?php
declare(strict_types=1);

$json = '{"id": 9007199254740991}';  // JavaScript Number.MAX_SAFE_INTEGER

// 默认处理（可能丢失精度）
$data1 = json_decode($json, true);
var_dump($data1['id']);  // float(9.007199254741E+15)

// 作为字符串处理
$data2 = json_decode($json, true, 512, JSON_BIGINT_AS_STRING);
var_dump($data2['id']);  // string(16) "9007199254740991"
```

## 请求体读取

### php://input 流

`php://input` 是一个只读流，用于读取原始请求体数据。

**特点**：
- 只读流
- 只能读取一次
- 不包含 HTTP 头
- 适用于 POST、PUT、PATCH 等请求

### 读取请求体

**示例 1：基本读取**
```php
<?php
declare(strict_types=1);

// 读取请求体
$json = file_get_contents('php://input');

// 解析 JSON
$data = json_decode($json, true);

if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
    throw new InvalidArgumentException('Invalid JSON');
}

print_r($data);
```

**示例 2：安全读取**
```php
<?php
declare(strict_types=1);

function readJsonRequest(): array
{
    // 检查请求方法
    if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
        throw new RuntimeException('Only POST requests are allowed');
    }

    // 检查 Content-Type
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    if (!str_contains($contentType, 'application/json')) {
        throw new RuntimeException('Content-Type must be application/json');
    }

    // 读取请求体
    $json = file_get_contents('php://input');
    
    if ($json === false || $json === '') {
        throw new RuntimeException('Empty request body');
    }

    // 解析 JSON
    $data = json_decode($json, true);
    
    if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
        throw new InvalidArgumentException('Invalid JSON: ' . json_last_error_msg());
    }

    return $data;
}

// 使用
try {
    $data = readJsonRequest();
    print_r($data);
} catch (Exception $e) {
    http_response_code(400);
    echo json_encode(['error' => $e->getMessage()]);
}
```

### 大小限制

**注意**：`php://input` 的读取受 `post_max_size` 配置限制。

**检查配置**：
```php
<?php
declare(strict_types=1);

$maxSize = ini_get('post_max_size');
echo "最大 POST 大小: {$maxSize}\n";
```

## JSON 解析选项

### 常用标志

| 标志 | 说明 |
|:-----|:-----|
| `JSON_BIGINT_AS_STRING` | 将大整数作为字符串返回 |
| `JSON_INVALID_UTF8_IGNORE` | 忽略无效的 UTF-8 字符 |
| `JSON_INVALID_UTF8_SUBSTITUTE` | 用替换字符替换无效的 UTF-8 字符 |
| `JSON_OBJECT_AS_ARRAY` | 将对象解码为关联数组 |

### 组合使用

**示例**：
```php
<?php
declare(strict_types=1);

$json = '{"id": 9007199254740991, "name": "John"}';

$flags = JSON_BIGINT_AS_STRING | JSON_OBJECT_AS_ARRAY;
$data = json_decode($json, true, 512, $flags);

var_dump($data);
```

## 错误处理

### json_last_error() 函数

**语法**：`json_last_error(): int`

**返回值**：返回最后一次 JSON 操作的错误代码。

### 错误常量

| 常量 | 值 | 说明 |
|:-----|:---|:-----|
| `JSON_ERROR_NONE` | 0 | 没有错误 |
| `JSON_ERROR_DEPTH` | 1 | 达到了最大堆栈深度 |
| `JSON_ERROR_STATE_MISMATCH` | 2 | 无效或异常的 JSON |
| `JSON_ERROR_CTRL_CHAR` | 3 | 控制字符错误 |
| `JSON_ERROR_SYNTAX` | 4 | 语法错误 |
| `JSON_ERROR_UTF8` | 5 | 无效的 UTF-8 字符 |
| `JSON_ERROR_RECURSION` | 6 | 递归引用 |
| `JSON_ERROR_INF_OR_NAN` | 7 | 包含 INF 或 NAN |
| `JSON_ERROR_UNSUPPORTED_TYPE` | 8 | 不支持的类型 |

### json_last_error_msg() 函数

**语法**：`json_last_error_msg(): string`

**返回值**：返回最后一次 JSON 操作的错误消息。

### 错误处理示例

**示例 1：基本错误检查**
```php
<?php
declare(strict_types=1);

$json = '{"name": "John", "age": }';  // 无效 JSON
$data = json_decode($json, true);

if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
    $error = json_last_error_msg();
    echo "JSON 解析错误: {$error}\n";
}
```

**示例 2：详细错误处理**
```php
<?php
declare(strict_types=1);

function decodeJsonSafe(string $json): array
{
    $data = json_decode($json, true);
    
    if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
        $errorCode = json_last_error();
        $errorMsg = json_last_error_msg();
        
        throw new InvalidArgumentException(
            "JSON 解析失败 (错误代码: {$errorCode}): {$errorMsg}"
        );
    }
    
    return $data;
}

// 使用
try {
    $json = '{"name": "John"}';
    $data = decodeJsonSafe($json);
    print_r($data);
} catch (InvalidArgumentException $e) {
    echo "错误: {$e->getMessage()}\n";
}
```

## 数据验证

### 类型验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateJsonData(array $data, array $schema): bool
{
    foreach ($schema as $key => $type) {
        if (!isset($data[$key])) {
            return false;  // 缺少必需字段
        }
        
        $value = $data[$key];
        $expectedType = match ($type) {
            'string' => 'string',
            'int' => 'integer',
            'float' => 'double',
            'bool' => 'boolean',
            'array' => 'array',
            default => $type,
        };
        
        if (gettype($value) !== $expectedType) {
            return false;  // 类型不匹配
        }
    }
    
    return true;
}

// 使用
$data = ['name' => 'John', 'age' => 30];
$schema = ['name' => 'string', 'age' => 'int'];

if (validateJsonData($data, $schema)) {
    echo "数据验证通过\n";
} else {
    echo "数据验证失败\n";
}
```

### 完整验证示例

**示例**：
```php
<?php
declare(strict_types=1);

class JsonRequestValidator
{
    public static function validate(array $data, array $rules): array
    {
        $errors = [];

        foreach ($rules as $field => $rule) {
            $value = $data[$field] ?? null;

            // 检查必需字段
            if (($rule['required'] ?? false) && $value === null) {
                $errors[$field] = "字段 {$field} 是必需的";
                continue;
            }

            // 如果字段为空且不是必需的，跳过验证
            if ($value === null) {
                continue;
            }

            // 类型验证
            if (isset($rule['type'])) {
                $expectedType = $rule['type'];
                $actualType = gettype($value);
                
                if ($expectedType === 'int' && $actualType !== 'integer') {
                    $errors[$field] = "字段 {$field} 必须是整数";
                } elseif ($expectedType === 'string' && $actualType !== 'string') {
                    $errors[$field] = "字段 {$field} 必须是字符串";
                } elseif ($expectedType === 'array' && $actualType !== 'array') {
                    $errors[$field] = "字段 {$field} 必须是数组";
                }
            }

            // 范围验证
            if (isset($rule['min']) && $value < $rule['min']) {
                $errors[$field] = "字段 {$field} 不能小于 {$rule['min']}";
            }

            if (isset($rule['max']) && $value > $rule['max']) {
                $errors[$field] = "字段 {$field} 不能大于 {$rule['max']}";
            }
        }

        return $errors;
    }
}

// 使用
$data = ['name' => 'John', 'age' => 30];
$rules = [
    'name' => ['type' => 'string', 'required' => true],
    'age' => ['type' => 'int', 'required' => true, 'min' => 0, 'max' => 120],
];

$errors = JsonRequestValidator::validate($data, $rules);
if (empty($errors)) {
    echo "验证通过\n";
} else {
    print_r($errors);
}
```

## 使用场景

### API 请求处理

- 处理 POST API 请求
- 解析 JSON 请求体
- 验证请求数据

### 数据交换

- 前后端数据交互
- 微服务通信
- 第三方 API 集成

### 配置读取

- 读取 JSON 配置文件
- 解析配置数据
- 验证配置格式

### 数据存储

- 存储 JSON 数据
- 读取 JSON 数据
- 数据序列化

## 注意事项

### JSON 格式验证

- **始终验证**：解析后检查 `json_last_error()`
- **格式检查**：验证 JSON 格式是否正确
- **类型检查**：验证数据类型是否符合预期

### 深度限制

- **默认深度**：默认最大深度为 512
- **深度限制**：根据需求设置合适的深度
- **性能考虑**：过深的嵌套可能影响性能

### 内存使用

- **大 JSON**：大 JSON 文件可能占用大量内存
- **流式处理**：考虑使用流式处理大 JSON
- **大小限制**：注意 `post_max_size` 配置

### 错误处理

- **检查返回值**：检查 `json_decode()` 的返回值
- **错误信息**：使用 `json_last_error_msg()` 获取错误信息
- **异常处理**：使用 try-catch 处理异常

## 常见问题

### 如何读取 JSON 请求体？

使用 `file_get_contents('php://input')` 读取请求体：

```php
<?php
declare(strict_types=1);

$json = file_get_contents('php://input');
$data = json_decode($json, true);
```

### json_decode() 的参数含义？

- **第一个参数**：JSON 字符串
- **第二个参数**：`true` 返回关联数组，`false` 返回对象
- **第三个参数**：最大嵌套深度
- **第四个参数**：解码选项标志

### 如何检查 JSON 解析错误？

使用 `json_last_error()` 检查错误：

```php
<?php
declare(strict_types=1);

$data = json_decode($json, true);
if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
    $error = json_last_error_msg();
    echo "错误: {$error}\n";
}
```

### 如何处理大 JSON 数据？

1. **增加内存限制**：`ini_set('memory_limit', '256M')`
2. **流式处理**：使用流式 JSON 解析器
3. **分块处理**：将大 JSON 分块处理

## 最佳实践

### 始终验证 JSON 格式

- 检查 `json_last_error()`
- 验证数据格式
- 提供清晰的错误信息

### 使用关联数组模式

- 使用 `json_decode($json, true)` 返回关联数组
- 更符合 PHP 习惯
- 更容易处理数据

### 检查解析错误

- 始终检查 `json_last_error()`
- 使用 `json_last_error_msg()` 获取错误信息
- 提供用户友好的错误消息

### 限制 JSON 深度和大小

- 设置合理的深度限制
- 限制 JSON 大小
- 防止恶意请求

## 相关章节

- **[5.5.2 API 请求处理](section-02-api-requests.md)**：了解 API 请求处理的详细内容
- **[5.3.1 $_GET、$_POST 与 $_REQUEST](../chapter-03-superglobals/section-01-get-post-request.md)**：了解其他请求数据获取方法

## 练习任务

1. **实现 JSON 请求解析函数**
   - 读取请求体
   - 解析 JSON
   - 验证数据格式

2. **实现 JSON 验证器**
   - 验证数据类型
   - 验证必需字段
   - 验证数据范围

3. **实现错误处理**
   - 检查 JSON 解析错误
   - 提供清晰的错误信息
   - 返回适当的 HTTP 状态码

4. **处理大 JSON 数据**
   - 实现流式处理
   - 处理内存限制
   - 优化性能

5. **实现 JSON API 端点**
   - 创建 API 端点
   - 处理 JSON 请求
   - 返回 JSON 响应

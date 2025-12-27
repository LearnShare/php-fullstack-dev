# 4.6.4 序列化与反序列化

## 概述

序列化是将数据结构或对象转换为可存储或传输格式的过程，反序列化是将序列化的数据还原为原始数据结构或对象的过程。在实际应用中，我们经常需要序列化数据以便存储到文件、数据库或通过网络传输，然后在需要时反序列化恢复数据。

PHP 提供了 `serialize()` 和 `unserialize()` 函数来实现序列化和反序列化，同时也支持 JSON 序列化。理解序列化的概念、使用方法，以及安全注意事项，对于开发安全的 PHP 应用至关重要。

**主要内容**：
- 序列化概述（什么是序列化、为什么需要序列化）
- `serialize()` 函数（序列化数据）
- `unserialize()` 函数（反序列化数据）
- JSON 序列化（`json_encode()`、`json_decode()`）
- 序列化安全注意事项（对象注入攻击）
- 实际应用场景和最佳实践

## 特性

- **数据持久化**：可以将数据存储到文件或数据库
- **数据传输**：可以将数据通过网络传输
- **对象恢复**：可以恢复对象的状态
- **格式多样**：支持多种序列化格式

## 语法/定义

### serialize() 函数

**语法**：`serialize(mixed $value): string`

**参数**：
- `$value`：要序列化的值（可以是任何类型）

**返回值**：返回序列化后的字符串。

### unserialize() 函数

**语法**：`unserialize(string $data, array $options = []): mixed`

**参数**：
- `$data`：序列化的字符串
- `$options`：可选，反序列化选项

**返回值**：返回反序列化后的值，失败返回 `false`。

### json_encode() 函数

**语法**：`json_encode(mixed $value, int $flags = 0, int $depth = 512): string|false`

**参数**：
- `$value`：要编码的值
- `$flags`：可选，编码选项
- `$depth`：可选，最大深度

**返回值**：成功返回 JSON 字符串，失败返回 `false`。

### json_decode() 函数

**语法**：`json_decode(string $json, ?bool $assoc = null, int $depth = 512, int $flags = 0): mixed`

**参数**：
- `$json`：JSON 字符串
- `$assoc`：可选，如果为 `true` 返回关联数组
- `$depth`：可选，最大深度
- `$flags`：可选，解码选项

**返回值**：成功返回解码后的值，失败返回 `null`。

## 基本用法

### 示例 1：基本序列化和反序列化

```php
<?php
declare(strict_types=1);

// 序列化数组
$data = ['name' => 'Alice', 'age' => 25, 'city' => 'Beijing'];
$serialized = serialize($data);
echo "序列化: {$serialized}\n";

// 反序列化
$unserialized = unserialize($serialized);
print_r($unserialized);

// 序列化对象
class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {
    }
}

$user = new User('Bob', 30);
$serialized = serialize($user);
echo "序列化对象: {$serialized}\n";

$unserialized = unserialize($serialized);
echo "姓名: {$unserialized->name}, 年龄: {$unserialized->age}\n";
```

**说明**：
- `serialize()` 将数据转换为字符串
- `unserialize()` 将字符串还原为数据
- 支持数组、对象等复杂数据结构

### 示例 2：JSON 序列化

```php
<?php
declare(strict_types=1);

// JSON 序列化（推荐用于数据交换）
$data = ['name' => 'Alice', 'age' => 25, 'city' => 'Beijing'];
$json = json_encode($data);
echo "JSON: {$json}\n";

// JSON 反序列化
$decoded = json_decode($json, true);  // true 返回关联数组
print_r($decoded);

// 对象序列化
class Product
{
    public function __construct(
        public string $name,
        public float $price
    ) {
    }
}

$product = new Product('Laptop', 999.99);
$json = json_encode($product);
echo "JSON: {$json}\n";

$decoded = json_decode($json);
echo "名称: {$decoded->name}, 价格: {$decoded->price}\n";
```

**说明**：
- JSON 序列化是跨语言的标准格式
- `json_encode()` 将数据转换为 JSON 字符串
- `json_decode()` 将 JSON 字符串还原为数据

### 示例 3：序列化到文件

```php
<?php
declare(strict_types=1);

class DataStore
{
    public static function save(mixed $data, string $filename): void
    {
        $serialized = serialize($data);
        file_put_contents($filename, $serialized);
    }
    
    public static function load(string $filename): mixed
    {
        if (!file_exists($filename)) {
            return null;
        }
        
        $serialized = file_get_contents($filename);
        return unserialize($serialized);
    }
}

// 使用
$data = ['users' => ['Alice', 'Bob', 'Charlie']];
DataStore::save($data, 'data.dat');

$loaded = DataStore::load('data.dat');
print_r($loaded);
```

**说明**：
- 序列化数据可以保存到文件
- 可以从文件加载并反序列化数据

### 示例 4：JSON 序列化选项

```php
<?php
declare(strict_types=1);

$data = ['name' => 'Alice', 'age' => 25];

// 基本 JSON 编码
$json1 = json_encode($data);
echo "基本: {$json1}\n";

// 美化输出
$json2 = json_encode($data, JSON_PRETTY_PRINT);
echo "美化:\n{$json2}\n";

// Unicode 转义
$json3 = json_encode(['name' => '张三'], JSON_UNESCAPED_UNICODE);
echo "Unicode: {$json3}\n";

// JSON 解码选项
$json = '{"name":"Alice","age":25}';
$decoded1 = json_decode($json, false);  // 返回对象
$decoded2 = json_decode($json, true);   // 返回数组

echo "对象: {$decoded1->name}\n";
echo "数组: {$decoded2['name']}\n";
```

**说明**：
- JSON 编码支持多种选项
- `JSON_PRETTY_PRINT` 美化输出
- `JSON_UNESCAPED_UNICODE` 不转义 Unicode

### 示例 5：序列化安全注意事项

```php
<?php
declare(strict_types=1);

// ⚠️ 安全警告：不要反序列化不可信的数据
// unserialize() 可能导致对象注入攻击

class SafeUnserialize
{
    public static function safeUnserialize(string $data, array $allowedClasses = []): mixed
    {
        // 使用 allowed_classes 选项限制允许的类
        $options = ['allowed_classes' => $allowedClasses];
        return unserialize($data, $options);
    }
}

// 只允许特定类
$serialized = serialize(new stdClass());
$unserialized = SafeUnserialize::safeUnserialize($serialized, [stdClass::class]);

// 或者不允许任何类（只允许基本类型）
$unserialized = SafeUnserialize::safeUnserialize($serialized, []);
```

**说明**：
- `unserialize()` 可能导致对象注入攻击
- 使用 `allowed_classes` 选项限制允许的类
- 不要反序列化不可信的数据

### 示例 6：JSON vs Serialize 对比

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {
    }
}

$user = new User('Alice', 25);

// PHP Serialize（PHP 专用）
$serialized = serialize($user);
echo "Serialize: {$serialized}\n";
// 输出：O:4:"User":2:{s:4:"name";s:5:"Alice";s:3:"age";i:25;}

// JSON（跨语言）
$json = json_encode($user);
echo "JSON: {$json}\n";
// 输出：{"name":"Alice","age":25}

// JSON 不能保留方法，只保留属性
// Serialize 可以保留完整对象（包括方法）
```

**说明**：
- PHP Serialize：PHP 专用，保留完整对象
- JSON：跨语言，只保留数据（不保留方法）

## 使用场景

### 场景 1：数据持久化

将数据序列化后存储到文件或数据库。

**示例**：

```php
<?php
declare(strict_types=1);

// 存储配置
$config = ['db_host' => 'localhost', 'db_name' => 'myapp'];
$serialized = serialize($config);
file_put_contents('config.dat', $serialized);

// 读取配置
$serialized = file_get_contents('config.dat');
$config = unserialize($serialized);
```

### 场景 2：数据传输

使用 JSON 序列化在客户端和服务器之间传输数据。

**示例**：

```php
<?php
declare(strict_types=1);

// 服务器端：返回 JSON
$data = ['users' => ['Alice', 'Bob']];
header('Content-Type: application/json');
echo json_encode($data);

// 客户端：接收 JSON
$json = file_get_contents('php://input');
$data = json_decode($json, true);
```

## 注意事项

### 序列化安全

不要反序列化不可信的数据，可能导致对象注入攻击。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 危险：反序列化用户输入
// $data = unserialize($_POST['data']);  // 危险！

// ✅ 安全：使用 allowed_classes 限制
$data = unserialize($_POST['data'], ['allowed_classes' => []]);

// ✅ 更安全：使用 JSON（不支持对象注入）
$data = json_decode($_POST['data'], true);
```

### JSON vs Serialize

根据场景选择合适的格式：
- JSON：跨语言、数据交换、API
- Serialize：PHP 内部、需要保留对象方法

**示例**：

```php
<?php
declare(strict_types=1);

// JSON：用于 API 数据交换
$json = json_encode($data);

// Serialize：用于 PHP 内部数据存储
$serialized = serialize($data);
```

## 常见问题

### 问题 1：serialize() 和 json_encode() 的区别？

**回答**：
- `serialize()`：PHP 专用格式，保留完整对象（包括方法）
- `json_encode()`：JSON 格式，跨语言，只保留数据（不保留方法）

**示例**：

```php
<?php
declare(strict_types=1);

$data = ['name' => 'Alice'];

// Serialize（PHP 专用）
$serialized = serialize($data);
// 输出：a:1:{s:4:"name";s:5:"Alice";}

// JSON（跨语言）
$json = json_encode($data);
// 输出：{"name":"Alice"}
```

### 问题 2：序列化安全吗？

**回答**：`unserialize()` 可能导致对象注入攻击，不要反序列化不可信的数据。使用 `allowed_classes` 选项或使用 JSON。

**示例**：见"序列化安全注意事项"部分

### 问题 3：什么时候使用 JSON，什么时候使用 Serialize？

**回答**：
- JSON：跨语言数据交换、API、配置文件
- Serialize：PHP 内部数据存储、需要保留对象方法

**示例**：见"JSON vs Serialize 对比"部分

### 问题 4：JSON 能序列化对象吗？

**回答**：JSON 可以序列化对象的属性，但不能保留方法。对象会被转换为包含属性的对象或关联数组。

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {
    }
    
    public function greet(): string
    {
        return "Hello, {$this->name}";
    }
}

$user = new User('Alice', 25);
$json = json_encode($user);
// 输出：{"name":"Alice","age":25}
// 方法 greet() 不会被序列化
```

## 最佳实践

### 1. 优先使用 JSON

对于数据交换和 API，优先使用 JSON 而不是 Serialize。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：使用 JSON
$json = json_encode($data);

// ⚠️ 不推荐：使用 Serialize（除非需要 PHP 专用特性）
// $serialized = serialize($data);
```

### 2. 安全反序列化

不要反序列化不可信的数据，使用 `allowed_classes` 选项限制。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 安全：限制允许的类
$data = unserialize($input, ['allowed_classes' => [AllowedClass::class]]);

// ✅ 更安全：使用 JSON
$data = json_decode($input, true);
```

### 3. 错误处理

检查序列化和反序列化的返回值，处理错误情况。

**示例**：

```php
<?php
declare(strict_types=1);

$json = json_encode($data);
if ($json === false) {
    throw new RuntimeException('JSON encoding failed: ' . json_last_error_msg());
}

$decoded = json_decode($json, true);
if ($decoded === null && json_last_error() !== JSON_ERROR_NONE) {
    throw new RuntimeException('JSON decoding failed: ' . json_last_error_msg());
}
```

### 4. 使用类型检查

反序列化后检查数据类型，确保数据符合预期。

**示例**：

```php
<?php
declare(strict_types=1);

$decoded = json_decode($json, true);
if (!is_array($decoded)) {
    throw new InvalidArgumentException('Expected array');
}
```

## 对比分析

### serialize() vs json_encode()

| 特性         | serialize()                   | json_encode()                 |
|:-------------|:------------------------------|:------------------------------|
| **格式**     | PHP 专用格式                  | JSON（标准格式）              |
| **跨语言**   | ❌ 只支持 PHP                 | ✅ 支持所有语言               |
| **对象方法** | ✅ 保留方法                   | ❌ 不保留方法                 |
| **可读性**   | ⚠️ 可读性较差                 | ✅ 可读性好                   |
| **安全性**   | ⚠️ 可能有安全问题             | ✅ 相对安全                   |
| **使用场景** | PHP 内部数据存储              | 数据交换、API                 |

### 对象注入风险

| 方法           | 对象注入风险 | 安全性                         |
|:---------------|:-------------|:-------------------------------|
| **unserialize()** | ⚠️ 高风险    | 可能导致对象注入攻击           |
| **json_decode()** | ✅ 低风险    | 不支持对象，相对安全           |

## 练习任务

1. **序列化工具类**：创建一个工具类，封装安全的序列化和反序列化功能。

2. **JSON API 工具**：实现一个工具，使用 JSON 进行数据交换。

3. **数据持久化工具**：创建一个工具，使用序列化持久化数据。

4. **序列化安全工具**：编写一个工具，安全地处理序列化数据。

5. **序列化格式对比工具**：实现一个工具，对比不同序列化格式的特点。

## 相关章节

- **[4.6.5 数据持久化](section-05-data-persistence.md)**：了解数据持久化的相关内容
- **[4.7.1 错误处理机制](../chapter-07-errors/section-01-error-handling.md)**：了解错误处理的相关内容

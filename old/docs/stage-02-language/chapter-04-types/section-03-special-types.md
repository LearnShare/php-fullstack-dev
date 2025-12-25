# 2.4.3 特殊类型

## 概述

特殊类型包括 `null` 和 `resource`。`null` 表示"无值"，`resource` 表示外部资源（如文件句柄、数据库连接等）。PHP 8.0+ 逐步将资源类型对象化。

## null 类型

### 基本概念

`null` 表示变量没有值或未初始化。它是 PHP 中唯一可能为 `null` 的类型。

### 语法

```php
<?php
declare(strict_types=1);

$var = null;
$var = NULL;  // 大小写不敏感，但推荐小写
```

### 类型检测

#### `is_null()`

**语法**：`is_null(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果值为 `null`，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_null(null));    // bool(true)
var_dump(is_null(0));       // bool(false)
var_dump(is_null(""));      // bool(false)
var_dump(is_null([]));      // bool(false)
```

### null 与 isset()、empty()

```php
<?php
declare(strict_types=1);

$var = null;

var_dump(isset($var));  // bool(false) - null 被视为未设置
var_dump(empty($var));  // bool(true) - null 被视为空值

// 未定义的变量
var_dump(isset($undefined));  // bool(false)
var_dump(is_null($undefined)); // 会产生警告
```

### 空合并运算符（??）

PHP 7.0+ 引入了空合并运算符，用于提供默认值：

```php
<?php
declare(strict_types=1);

$name = $input['name'] ?? 'Unknown';
$age = $user->age ?? 0;
$email = $_GET['email'] ?? null;
```

### 空合并赋值运算符（??=）

PHP 7.4+ 引入了空合并赋值运算符：

```php
<?php
declare(strict_types=1);

$config['debug'] ??= false;
$user['name'] ??= 'Guest';
```

### 完整示例

```php
<?php
declare(strict_types=1);

function getUserName(?array $user): string
{
    return $user['name'] ?? 'Guest';
}

function processValue(mixed $value): string
{
    if ($value === null) {
        return 'No value provided';
    }
    
    return (string) $value;
}

// 使用示例
$user1 = ['name' => 'Alice'];
$user2 = null;

echo getUserName($user1) . "\n";  // Alice
echo getUserName($user2) . "\n";   // Guest

echo processValue(null) . "\n";    // No value provided
echo processValue(42) . "\n";      // 42
```

## resource 类型

### 基本概念

资源类型是对外部资源的引用，如文件句柄、数据库连接、图像资源等。PHP 8.0+ 逐步将资源类型对象化。

### 常见资源类型

#### 文件资源

```php
<?php
declare(strict_types=1);

// 打开文件
$file = fopen('data.txt', 'r');
var_dump(is_resource($file));  // bool(true)
var_dump(get_resource_type($file));  // stream

// 读取文件
$content = fread($file, 1024);

// 关闭资源
fclose($file);
```

#### 数据库连接资源

```php
<?php
declare(strict_types=1);

// MySQLi 连接（PHP 8.0+ 返回对象，不再是资源）
$mysqli = new mysqli('localhost', 'user', 'password', 'database');

// 旧版本 MySQL 扩展（已废弃）
// $link = mysql_connect('localhost', 'user', 'password');
```

#### cURL 资源

```php
<?php
declare(strict_types=1);

// PHP 7.4- 返回资源
// $ch = curl_init();

// PHP 8.0+ 返回 CurlHandle 对象
$ch = curl_init();
var_dump($ch instanceof CurlHandle);  // bool(true)
```

### 类型检测

#### `is_resource()`

**语法**：`is_resource(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是资源类型，返回 `true`；否则返回 `false`。注意：PHP 8.0+ 中，许多资源已对象化，此函数可能返回 `false`。

```php
<?php
declare(strict_types=1);

$file = fopen('php://memory', 'r+');

// PHP 7.4-
// var_dump(is_resource($file));  // bool(true)

// PHP 8.0+
var_dump(is_resource($file));  // bool(false) - 已对象化
var_dump($file instanceof \GdImage);  // 检查具体类型
```

#### `get_resource_type()`

**语法**：`get_resource_type(resource $resource): string`

**参数**：
- `$resource`：要检查的资源

**返回值**：返回资源类型名称，如果不是资源则返回空字符串。

```php
<?php
declare(strict_types=1);

$file = fopen('php://memory', 'r+');
echo get_resource_type($file) . "\n";  // stream
```

### 资源管理

#### 自动释放

PHP 会在脚本结束时自动释放所有资源，但最佳实践是显式释放：

```php
<?php
declare(strict_types=1);

function readFileContent(string $filename): string
{
    $file = fopen($filename, 'r');
    if ($file === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        $content = fread($file, filesize($filename));
        return $content;
    } finally {
        fclose($file);  // 确保资源被释放
    }
}
```

#### 使用 try-finally

```php
<?php
declare(strict_types=1);

$file = fopen('data.txt', 'r');
try {
    // 使用资源
    $content = fread($file, 1024);
} finally {
    fclose($file);  // 无论是否发生异常，都会执行
}
```

### PHP 8.0+ 资源对象化

PHP 8.0+ 将许多资源类型转换为对象：

| 旧资源类型 | PHP 8.0+ 对象类型 |
| :--------- | :---------------- |
| `gd`       | `GdImage`         |
| `curl`     | `CurlHandle`      |
| `xml`      | `XMLParser`       |
| `openssl`  | `OpenSSLAsymmetricKey` / `OpenSSLCertificate` |

```php
<?php
declare(strict_types=1);

// PHP 8.0+ 图像资源
$image = imagecreate(100, 100);
var_dump($image instanceof \GdImage);  // bool(true)

// PHP 8.0+ cURL 资源
$ch = curl_init();
var_dump($ch instanceof \CurlHandle);  // bool(true)
```

### 完整示例

```php
<?php
declare(strict_types=1);

class FileHandler
{
    private $file;
    
    public function __construct(string $filename, string $mode = 'r')
    {
        $this->file = fopen($filename, $mode);
        if ($this->file === false) {
            throw new RuntimeException("Cannot open file: {$filename}");
        }
    }
    
    public function read(int $length): string|false
    {
        return fread($this->file, $length);
    }
    
    public function write(string $data): int|false
    {
        return fwrite($this->file, $data);
    }
    
    public function close(): void
    {
        if (is_resource($this->file) || $this->file !== null) {
            fclose($this->file);
            $this->file = null;
        }
    }
    
    public function __destruct()
    {
        $this->close();
    }
}

// 使用示例
try {
    $handler = new FileHandler('data.txt', 'r');
    $content = $handler->read(1024);
    echo $content;
} finally {
    $handler->close();
}
```

## 注意事项

1. **null 检查**：使用 `=== null` 或 `is_null()` 检查 null，不要使用 `== null`（可能与其他值相等）。

2. **资源释放**：始终显式释放资源，使用 try-finally 确保资源被正确释放。

3. **PHP 8.0+ 变化**：注意资源类型已对象化，使用 `instanceof` 检查类型而不是 `is_resource()`。

4. **空合并运算符**：优先使用 `??` 提供默认值，而不是 `isset()` + 三元运算符。

## 练习

1. 编写一个函数 `safeGet(array $array, string $key, mixed $default = null): mixed`，安全地获取数组值，支持嵌套键（如 "user.name"）。

2. 创建一个 `ResourceManager` 类，管理多个资源，确保在析构时正确释放所有资源。

3. 实现一个函数 `parseConfig(string $filename): array`，读取配置文件，使用 try-finally 确保文件资源被正确释放。

4. 编写一个函数，检查值是否为 null 或空，并返回适当的默认值。

5. 创建一个资源包装类，使用 `__destruct()` 方法自动释放资源。

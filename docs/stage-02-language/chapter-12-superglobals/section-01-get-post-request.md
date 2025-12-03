# 2.12.1 $_GET、$_POST 与 $_REQUEST

## 概述

`$_GET`、`$_POST` 和 `$_REQUEST` 是 PHP 的超级全局变量，用于接收来自 HTTP 请求的数据。理解它们的区别和安全使用方法对于 Web 开发至关重要。

## $_GET - 查询字符串参数

### 基本概念

`$_GET` 用于接收通过 URL 查询字符串传递的参数（GET 请求）。

### 基本用法

```php
<?php
declare(strict_types=1);

// URL: http://example.com/page.php?id=1&name=Alice
$id = $_GET['id'] ?? null;
$name = $_GET['name'] ?? null;

echo "ID: {$id}, Name: {$name}\n";
```

### 检查键是否存在

```php
<?php
declare(strict_types=1);

if (isset($_GET['id'])) {
    $id = $_GET['id'];
} else {
    $id = null;
}

// 或使用 null 合并运算符
$id = $_GET['id'] ?? null;
```

## $_POST - 表单提交数据

### 基本概念

`$_POST` 用于接收通过 HTTP POST 方法提交的表单数据。

### 基本用法

```php
<?php
declare(strict_types=1);

// 处理表单提交
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = $_POST['email'] ?? '';
    
    // 处理数据
    echo "Name: {$name}, Email: {$email}\n";
}
```

### 表单示例

```html
<!-- form.html -->
<form method="POST" action="process.php">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

```php
<?php
// process.php
declare(strict_types=1);

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = $_POST['email'] ?? '';
    // 处理数据
}
```

## $_REQUEST - 不推荐使用

### 基本概念

`$_REQUEST` 包含 `$_GET`、`$_POST` 和 `$_COOKIE` 的合并数据。

### 为什么不推荐

1. **来源不明确**：无法确定数据来自 GET、POST 还是 COOKIE
2. **安全风险**：可能包含不可信的数据源
3. **性能影响**：需要合并多个数组

### 替代方案

```php
<?php
declare(strict_types=1);

// 不推荐
$value = $_REQUEST['key'] ?? null;

// 推荐：明确指定来源
$value = $_POST['key'] ?? $_GET['key'] ?? null;
```

## filter_input() - 安全读取

### 基本语法

**语法**：`filter_input(int $type, string $var_name, int $filter = FILTER_DEFAULT, array|int $options = 0): mixed`

**参数**：
- `$type`：输入类型（`INPUT_GET`、`INPUT_POST`、`INPUT_COOKIE` 等）
- `$var_name`：变量名
- `$filter`：过滤器类型
- `$options`：过滤器选项

### 基本用法

```php
<?php
declare(strict_types=1);

// 安全读取 GET 参数
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
if ($id === false || $id === null) {
    $id = 0;  // 默认值
}

// 安全读取 POST 参数
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
if ($email === false) {
    echo "Invalid email\n";
}
```

### 常用过滤器

| 过滤器                    | 说明           |
| :------------------------ | :------------- |
| `FILTER_VALIDATE_INT`     | 验证整数       |
| `FILTER_VALIDATE_FLOAT`   | 验证浮点数     |
| `FILTER_VALIDATE_EMAIL`   | 验证邮箱       |
| `FILTER_VALIDATE_URL`     | 验证 URL       |
| `FILTER_SANITIZE_STRING`  | 清理字符串     |
| `FILTER_SANITIZE_NUMBER_INT` | 清理为整数 |

### 示例

```php
<?php
declare(strict_types=1);

// 验证整数
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
if ($id === false || $id === null) {
    throw new InvalidArgumentException('Invalid ID');
}

// 验证邮箱
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
if ($email === false) {
    throw new InvalidArgumentException('Invalid email');
}

// 清理字符串
$name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING);
```

## filter_input_array() - 批量处理

### 语法

**语法**：`filter_input_array(int $type, array|int $options = FILTER_DEFAULT, bool $add_empty = true): array|false|null`

### 基本用法

```php
<?php
declare(strict_types=1);

$filters = [
    'id' => FILTER_VALIDATE_INT,
    'email' => FILTER_VALIDATE_EMAIL,
    'name' => FILTER_SANITIZE_STRING
];

$data = filter_input_array(INPUT_POST, $filters);
if ($data === false || $data === null) {
    throw new RuntimeException('Invalid input');
}

// $data 包含过滤后的数据
```

## 完整示例

```php
<?php
declare(strict_types=1);

class RequestHandler
{
    public static function getInt(string $key, int $default = 0): int
    {
        $value = filter_input(INPUT_GET, $key, FILTER_VALIDATE_INT);
        return $value !== false && $value !== null ? $value : $default;
    }
    
    public static function getString(string $key, string $default = ''): string
    {
        $value = filter_input(INPUT_GET, $key, FILTER_SANITIZE_STRING);
        return $value !== false && $value !== null ? $value : $default;
    }
    
    public static function postEmail(string $key): ?string
    {
        $value = filter_input(INPUT_POST, $key, FILTER_VALIDATE_EMAIL);
        return $value !== false ? $value : null;
    }
    
    public static function validateForm(array $rules): array
    {
        $data = [];
        $errors = [];
        
        foreach ($rules as $field => $filter) {
            $value = filter_input(INPUT_POST, $field, $filter);
            if ($value === false || $value === null) {
                $errors[$field] = "Invalid {$field}";
            } else {
                $data[$field] = $value;
            }
        }
        
        return ['data' => $data, 'errors' => $errors];
    }
}

// 使用示例
$id = RequestHandler::getInt('id', 0);
$name = RequestHandler::getString('name', 'Guest');
$email = RequestHandler::postEmail('email');

$rules = [
    'name' => FILTER_SANITIZE_STRING,
    'email' => FILTER_VALIDATE_EMAIL,
    'age' => FILTER_VALIDATE_INT
];
$result = RequestHandler::validateForm($rules);
```

## 安全建议

1. **始终验证输入**：使用 `filter_input()` 验证所有用户输入。

2. **明确数据来源**：不要使用 `$_REQUEST`，明确使用 `$_GET` 或 `$_POST`。

3. **类型检查**：使用适当的过滤器验证数据类型。

4. **默认值**：为可能缺失的参数提供默认值。

5. **错误处理**：检查 `filter_input()` 的返回值，处理验证失败的情况。

## 注意事项

1. **GET vs POST**：GET 用于获取数据，POST 用于提交数据。

2. **数据大小**：GET 参数有长度限制，POST 数据可以更大。

3. **安全性**：POST 数据不会出现在 URL 中，相对更安全。

4. **缓存**：GET 请求可能被缓存，POST 请求不会被缓存。

5. **幂等性**：GET 请求应该是幂等的，POST 请求可以有副作用。

## 练习

1. 创建一个请求处理类，安全地读取和验证 GET 和 POST 参数。

2. 编写一个函数，使用 `filter_input_array()` 批量验证表单数据。

3. 实现一个表单验证器，检查必填字段和数据类型。

4. 创建一个函数，比较直接访问 `$_GET` 和使用 `filter_input()` 的区别。

5. 编写一个函数，安全地处理嵌套的表单数据。

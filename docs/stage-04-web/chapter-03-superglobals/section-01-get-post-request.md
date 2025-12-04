# 4.3.1 $_GET、$_POST 与 $_REQUEST

## 概述

`$_GET` 和 `$_POST` 是处理 Web 输入最常用的超全局变量。本节详细介绍查询参数处理、表单数据处理、数组参数、`$_REQUEST` 的安全隐患，以及输入过滤与验证的最佳实践。

## $_GET - 查询参数

### 基础用法

`$_GET` 包含 URL 查询字符串中的参数。

```php
<?php
// URL: http://example.com/users.php?id=1&page=2

$userId = $_GET['id'] ?? null;
$page = $_GET['page'] ?? 1;

echo "User ID: {$userId}\n";
echo "Page: {$page}\n";
```

### 安全处理

```php
<?php
declare(strict_types=1);

// ❌ 不安全的做法
$id = $_GET['id']; // 可能未定义，可能包含恶意数据

// ✅ 安全的做法
function getInt(string $key, int $default = 0): int
{
    if (!isset($_GET[$key])) {
        return $default;
    }
    return (int) filter_var($_GET[$key], FILTER_VALIDATE_INT, [
        'options' => ['default' => $default, 'min_range' => 1]
    ]);
}

function getString(string $key, string $default = ''): string
{
    if (!isset($_GET[$key])) {
        return $default;
    }
    return htmlspecialchars(trim($_GET[$key]), ENT_QUOTES, 'UTF-8');
}

$userId = getInt('id');
$search = getString('q');
```

### 数组参数处理

```php
<?php
// URL: http://example.com/search.php?tags[]=php&tags[]=mysql

$tags = $_GET['tags'] ?? [];

// 验证数组
if (!is_array($tags)) {
    $tags = [];
}

// 过滤和验证每个元素
$tags = array_filter(
    array_map('trim', $tags),
    fn($tag) => !empty($tag) && strlen($tag) <= 50
);
```

## $_POST - 表单数据

### 基础用法

`$_POST` 包含通过 POST 方法提交的表单数据。

```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = $_POST['email'] ?? '';
    
    echo "Name: {$name}\n";
    echo "Email: {$email}\n";
}
```

### 表单处理示例

```php
<?php
declare(strict_types=1);

function validateEmail(string $email): bool
{
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}

function sanitizeString(string $input): string
{
    return htmlspecialchars(trim($input), ENT_QUOTES, 'UTF-8');
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $errors = [];
    
    $name = sanitizeString($_POST['name'] ?? '');
    $email = sanitizeString($_POST['email'] ?? '');
    $age = (int) ($_POST['age'] ?? 0);
    
    // 验证
    if (empty($name)) {
        $errors[] = '姓名不能为空';
    }
    
    if (empty($email) || !validateEmail($email)) {
        $errors[] = '邮箱格式不正确';
    }
    
    if ($age < 18 || $age > 120) {
        $errors[] = '年龄必须在18-120之间';
    }
    
    if (empty($errors)) {
        // 保存数据
        echo "数据验证通过\n";
    } else {
        foreach ($errors as $error) {
            echo "错误: {$error}\n";
        }
    }
}
```

### 使用 filter_input

```php
<?php
declare(strict_types=1);

// 使用 filter_input 更安全
$name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING);
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
$age = filter_input(INPUT_POST, 'age', FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 18, 'max_range' => 120]
]);

if ($email === false) {
    echo "邮箱格式错误\n";
}
```

## $_REQUEST - 为什么不建议使用

### 安全隐患

- `$_REQUEST` 合并了 `$_GET`、`$_POST`、`$_COOKIE` 的数据
- 无法确定数据来源，存在安全风险
- 可能被 Cookie 覆盖 GET/POST 数据

```php
<?php
// ❌ 危险示例
$id = $_REQUEST['id']; // 不知道来自 GET、POST 还是 COOKIE

// 攻击场景：
// 1. 攻击者设置恶意 Cookie: id=malicious
// 2. 正常请求: GET /users.php?id=1
// 3. $_REQUEST['id'] 可能来自 Cookie，而非 GET 参数
```

### 推荐做法

```php
<?php
// ✅ 明确指定数据来源
$id = $_GET['id'] ?? $_POST['id'] ?? null;

// 或使用 filter_input 指定输入源
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT)
   ?? filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);
```

## 输入过滤与验证

### filter_var 函数

```php
<?php
declare(strict_types=1);

// 验证邮箱
$email = filter_var($_POST['email'] ?? '', FILTER_VALIDATE_EMAIL);
if ($email === false) {
    echo "邮箱格式错误\n";
}

// 验证整数
$age = filter_var($_POST['age'] ?? '', FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 18, 'max_range' => 120]
]);

// 验证 URL
$url = filter_var($_GET['url'] ?? '', FILTER_VALIDATE_URL);
```

### 自定义验证类

```php
<?php
declare(strict_types=1);

class InputValidator
{
    public static function validateEmail(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }

    public static function validateInt(string $value, ?int $min = null, ?int $max = null): ?int
    {
        $options = [];
        if ($min !== null) {
            $options['min_range'] = $min;
        }
        if ($max !== null) {
            $options['max_range'] = $max;
        }
        
        $result = filter_var($value, FILTER_VALIDATE_INT, ['options' => $options]);
        return $result !== false ? $result : null;
    }

    public static function sanitizeString(string $input): string
    {
        return htmlspecialchars(trim($input), ENT_QUOTES, 'UTF-8');
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class RequestHandler
{
    public function getInt(string $key, int $default = 0): int
    {
        return filter_input(INPUT_GET, $key, FILTER_VALIDATE_INT) ?: $default;
    }

    public function postString(string $key, string $default = ''): string
    {
        $value = filter_input(INPUT_POST, $key, FILTER_SANITIZE_STRING);
        return $value !== null ? trim($value) : $default;
    }

    public function validateForm(array $rules): array
    {
        $data = [];
        $errors = [];

        foreach ($rules as $field => $rule) {
            $value = $_POST[$field] ?? null;
            
            if (($rule['required'] ?? false) && empty($value)) {
                $errors[$field] = $rule['message'] ?? "{$field} 是必填项";
                continue;
            }

            if (!empty($value)) {
                switch ($rule['type'] ?? 'string') {
                    case 'email':
                        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                            $errors[$field] = "{$field} 格式不正确";
                        }
                        break;
                    case 'int':
                        $value = filter_var($value, FILTER_VALIDATE_INT);
                        if ($value === false) {
                            $errors[$field] = "{$field} 必须是整数";
                        }
                        break;
                }
            }

            $data[$field] = $value;
        }

        return ['data' => $data, 'errors' => $errors];
    }
}
```

## 注意事项

1. **始终验证输入**：不要信任任何用户输入
2. **明确数据来源**：避免使用 `$_REQUEST`
3. **类型转换**：使用 `filter_var` 进行安全的类型转换
4. **转义输出**：输出时转义特殊字符

## 练习

1. 创建一个输入验证类，包含验证邮箱、整数、字符串长度等方法。

2. 实现一个表单处理函数，验证并清理用户输入，返回验证结果和错误信息。

3. 实现一个请求参数解析器，能够安全地获取 GET、POST 参数，并自动进行类型转换和验证。

4. 设计一个输入过滤器，能够根据规则自动过滤和验证表单数据。

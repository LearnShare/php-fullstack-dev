# 4.3 超全局变量：Web 输入的核心

## 目标

- 深入理解 `$_GET`、`$_POST`、`$_SERVER`、`$_COOKIE` 等超全局变量的使用。
- 掌握表单数据处理、查询参数解析、Header 信息获取。
- 理解 `$_REQUEST` 的安全隐患，避免使用。
- 掌握输入过滤与验证，确保数据安全。

## $_GET - 查询参数

### 基础用法

- `$_GET` 包含 URL 查询字符串中的参数。
- 通过 `?key=value&key2=value2` 格式传递。

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

// 不安全的做法
$id = $_GET['id']; // 可能未定义，可能包含恶意数据

// 安全的做法
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
// URL: http://example.com/search.php?tags[]=php&tags[]=mysql&tags[]=laravel

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

foreach ($tags as $tag) {
    echo htmlspecialchars($tag, ENT_QUOTES, 'UTF-8') . "\n";
}
```

## $_POST - 表单数据

### 基础用法

- `$_POST` 包含通过 POST 方法提交的表单数据。
- 仅当 `Content-Type: application/x-www-form-urlencoded` 或 `multipart/form-data` 时可用。

```php
<?php
// 表单提交
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = $_POST['email'] ?? '';
    
    // 处理数据
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
    
    // 获取并验证数据
    $name = sanitizeString($_POST['name'] ?? '');
    $email = sanitizeString($_POST['email'] ?? '');
    $age = (int) ($_POST['age'] ?? 0);
    
    // 验证
    if (empty($name)) {
        $errors[] = '姓名不能为空';
    } elseif (strlen($name) > 100) {
        $errors[] = '姓名长度不能超过100个字符';
    }
    
    if (empty($email)) {
        $errors[] = '邮箱不能为空';
    } elseif (!validateEmail($email)) {
        $errors[] = '邮箱格式不正确';
    }
    
    if ($age < 18 || $age > 120) {
        $errors[] = '年龄必须在18-120之间';
    }
    
    // 处理结果
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

if ($age === false) {
    echo "年龄无效\n";
}
```

## $_SERVER - 服务器信息

### 常用变量

| 变量名              | 说明                           | 示例                           |
| :------------------ | :----------------------------- | :----------------------------- |
| `REQUEST_METHOD`    | HTTP 请求方法                  | `GET`、`POST`、`PUT`、`DELETE` |
| `REQUEST_URI`       | 请求的 URI                     | `/users/1?page=2`              |
| `SCRIPT_NAME`       | 当前脚本路径                   | `/index.php`                   |
| `QUERY_STRING`      | 查询字符串                     | `id=1&page=2`                  |
| `HTTP_HOST`         | 主机名                         | `example.com`                  |
| `HTTP_USER_AGENT`   | 用户代理                       | `Mozilla/5.0...`               |
| `REMOTE_ADDR`       | 客户端 IP 地址                 | `192.168.1.1`                  |
| `SERVER_NAME`       | 服务器名称                     | `example.com`                  |
| `SERVER_PORT`       | 服务器端口                     | `80`、`443`                    |
| `HTTPS`             | 是否 HTTPS                     | `on` 或空                      |
| `CONTENT_TYPE`      | 请求内容类型                   | `application/json`              |
| `CONTENT_LENGTH`    | 请求体长度                     | `1024`                         |

### 实际应用

```php
<?php
declare(strict_types=1);

function getRequestMethod(): string
{
    return $_SERVER['REQUEST_METHOD'] ?? 'GET';
}

function getRequestUri(): string
{
    return $_SERVER['REQUEST_URI'] ?? '/';
}

function getBaseUrl(): string
{
    $protocol = (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off') ? 'https' : 'http';
    $host = $_SERVER['HTTP_HOST'] ?? 'localhost';
    return "{$protocol}://{$host}";
}

function getClientIp(): string
{
    // 考虑代理情况
    if (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $ips = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        return trim($ips[0]);
    }
    
    return $_SERVER['REMOTE_ADDR'] ?? '0.0.0.0';
}

function isAjaxRequest(): bool
{
    return isset($_SERVER['HTTP_X_REQUESTED_WITH']) 
        && strtolower($_SERVER['HTTP_X_REQUESTED_WITH']) === 'xmlhttprequest';
}

// 使用示例
echo "Method: " . getRequestMethod() . "\n";
echo "URI: " . getRequestUri() . "\n";
echo "Base URL: " . getBaseUrl() . "\n";
echo "Client IP: " . getClientIp() . "\n";
echo "Is AJAX: " . (isAjaxRequest() ? 'Yes' : 'No') . "\n";
```

### 获取所有 Headers

```php
<?php
function getAllHeaders(): array
{
    if (function_exists('getallheaders')) {
        return getallheaders();
    }
    
    // 手动构建（当 getallheaders 不可用时）
    $headers = [];
    foreach ($_SERVER as $key => $value) {
        if (strpos($key, 'HTTP_') === 0) {
            $header = str_replace(' ', '-', ucwords(strtolower(str_replace('_', ' ', substr($key, 5)))));
            $headers[$header] = $value;
        }
    }
    return $headers;
}

$headers = getAllHeaders();
print_r($headers);
```

## $_COOKIE - Cookie 数据

### 基础用法

```php
<?php
// 读取 Cookie
$userId = $_COOKIE['user_id'] ?? null;
$theme = $_COOKIE['theme'] ?? 'light';

// 设置 Cookie
setcookie('user_id', '123', time() + 3600, '/', '', true, true);
// 参数：名称、值、过期时间、路径、域名、secure、httponly

// 删除 Cookie
setcookie('user_id', '', time() - 3600, '/');
```

### 安全设置

```php
<?php
declare(strict_types=1);

function setSecureCookie(
    string $name,
    string $value,
    int $expires = 3600,
    string $path = '/',
    ?string $domain = null
): bool {
    return setcookie(
        $name,
        $value,
        time() + $expires,
        $path,
        $domain,
        true,  // secure: 仅 HTTPS
        true   // httponly: 防止 JavaScript 访问
    );
}

// 使用
setSecureCookie('session_id', 'abc123', 7200);
```

## $_REQUEST - 为什么不建议使用

### 安全隐患

- `$_REQUEST` 合并了 `$_GET`、`$_POST`、`$_COOKIE` 的数据。
- 无法确定数据来源，存在安全风险。
- 可能被 Cookie 覆盖 GET/POST 数据。

```php
<?php
// 危险示例
$id = $_REQUEST['id']; // 不知道来自 GET、POST 还是 COOKIE

// 攻击场景：
// 1. 攻击者设置恶意 Cookie: id=malicious
// 2. 正常请求: GET /users.php?id=1
// 3. $_REQUEST['id'] 可能来自 Cookie，而非 GET 参数
```

### 推荐做法

```php
<?php
// 明确指定数据来源
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
if ($age === false) {
    echo "年龄无效\n";
}

// 验证 URL
$url = filter_var($_GET['url'] ?? '', FILTER_VALIDATE_URL);
if ($url === false) {
    echo "URL 格式错误\n";
}

// 清理字符串
$name = filter_var($_POST['name'] ?? '', FILTER_SANITIZE_STRING);
```

### 自定义验证函数

```php
<?php
declare(strict_types=1);

class InputValidator
{
    public static function validateEmail(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }

    public static function validateInt(
        string $value,
        ?int $min = null,
        ?int $max = null
    ): ?int {
        $options = [];
        if ($min !== null) {
            $options['min_range'] = $min;
        }
        if ($max !== null) {
            $options['max_range'] = $max;
        }
        
        $result = filter_var($value, FILTER_VALIDATE_INT, [
            'options' => $options
        ]);
        
        return $result !== false ? $result : null;
    }

    public static function sanitizeString(string $input): string
    {
        return htmlspecialchars(trim($input), ENT_QUOTES, 'UTF-8');
    }

    public static function validateRequired(mixed $value): bool
    {
        if (is_string($value)) {
            return trim($value) !== '';
        }
        return $value !== null && $value !== '';
    }
}

// 使用
$email = InputValidator::validateEmail($_POST['email'] ?? '');
$age = InputValidator::validateInt($_POST['age'] ?? '', 18, 120);
$name = InputValidator::sanitizeString($_POST['name'] ?? '');
```

## 类型转换与编码

### 类型转换

```php
<?php
declare(strict_types=1);

// 所有超全局变量都是字符串或数组
$id = $_GET['id']; // string

// 需要类型转换
$id = (int) $_GET['id'];
$price = (float) $_POST['price'];
$isActive = (bool) $_POST['is_active'];

// 使用 filter_var 更安全
$id = filter_var($_GET['id'], FILTER_VALIDATE_INT);
$price = filter_var($_POST['price'], FILTER_VALIDATE_FLOAT);
```

### 编码问题

```php
<?php
// 确保使用 UTF-8 编码
mb_internal_encoding('UTF-8');

// 处理多字节字符串
$name = mb_convert_encoding($_POST['name'] ?? '', 'UTF-8', 'auto');
$name = mb_trim($name); // 如果定义了 mb_trim 函数

// 验证字符长度（考虑多字节）
if (mb_strlen($name) > 100) {
    echo "名称过长\n";
}
```

## 最佳实践

### 1. 始终验证输入

```php
// 不推荐
$id = $_GET['id'];

// 推荐
$id = filter_var($_GET['id'] ?? '', FILTER_VALIDATE_INT);
if ($id === false) {
    throw new InvalidArgumentException('Invalid ID');
}
```

### 2. 明确数据来源

```php
// 不推荐
$data = $_REQUEST['key'];

// 推荐
$data = $_GET['key'] ?? $_POST['key'] ?? null;
```

### 3. 转义输出

```php
// 不推荐
echo $_GET['name'];

// 推荐
echo htmlspecialchars($_GET['name'] ?? '', ENT_QUOTES, 'UTF-8');
```

### 4. 使用类型提示

```php
function processUser(int $userId, string $name): void
{
    // 函数内部无需再次验证类型
}
```

## 练习

1. 创建一个输入验证类，包含验证邮箱、整数、字符串长度等方法。

2. 实现一个表单处理函数，验证并清理用户输入，返回验证结果和错误信息。

3. 编写一个函数获取客户端真实 IP 地址，考虑代理和负载均衡的情况。

4. 创建一个安全的 Cookie 设置函数，包含所有安全选项（secure、httponly、samesite）。

5. 实现一个请求参数解析器，能够安全地获取 GET、POST 参数，并自动进行类型转换和验证。

6. 设计一个输入过滤器，能够根据规则自动过滤和验证表单数据，返回清理后的数据或错误信息。

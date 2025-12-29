# 5.3.1 $_GET、$_POST 与 $_REQUEST

## 概述

`$_GET`、`$_POST` 和 `$_REQUEST` 是 PHP 中获取 HTTP 请求数据的核心超全局变量。理解这些变量的区别、使用方法和安全注意事项，对于正确处理 Web 输入至关重要。本节详细介绍这些超全局变量的使用方法、数据获取和验证、安全注意事项等内容，帮助零基础学员掌握 Web 输入处理。

在 Web 开发中，客户端通过 HTTP 请求向服务器发送数据。PHP 提供了超全局变量来访问这些数据：`$_GET` 用于获取 URL 参数（GET 请求），`$_POST` 用于获取请求体数据（POST 请求），`$_REQUEST` 包含了 `$_GET`、`$_POST` 和 `$_COOKIE` 的合并数据。理解这些变量的区别和正确使用方法，对于构建安全的 Web 应用至关重要。

**主要内容**：
- 超全局变量的概念和特点
- `$_GET` 超全局变量的使用方法和应用场景
- `$_POST` 超全局变量的使用方法和应用场景
- `$_REQUEST` 超全局变量的结构、优先级问题和安全风险
- 数据获取方法（直接访问、`filter_input()` 函数）
- 数据验证和过滤
- 数组参数的处理
- 安全注意事项和最佳实践

## 特性

- **超全局变量**：在任何作用域都可以访问，无需使用 `global` 关键字
- **自动填充**：PHP 自动填充这些变量，无需手动解析
- **数组结构**：这些变量都是关联数组，键是参数名，值是参数值
- **类型灵活**：值可以是字符串、数组等类型
- **安全性**：需要手动验证和过滤，防止安全漏洞

## 超全局变量概述

### 什么是超全局变量

超全局变量（Superglobal）是 PHP 中特殊的全局变量，可以在任何作用域（函数、类方法等）中直接访问，无需使用 `global` 关键字。

**超全局变量列表**：
- `$_GET`：GET 请求参数
- `$_POST`：POST 请求参数
- `$_REQUEST`：GET、POST、COOKIE 的合并
- `$_SERVER`：服务器和请求信息
- `$_SESSION`：会话数据
- `$_COOKIE`：Cookie 数据
- `$_FILES`：上传文件数据
- `$_ENV`：环境变量
- `$GLOBALS`：所有全局变量

### 超全局变量的特点

1. **自动填充**：PHP 自动解析 HTTP 请求并填充这些变量
2. **全局访问**：在任何作用域都可以直接访问
3. **数组结构**：都是关联数组
4. **类型灵活**：值可以是字符串、数组等

## $_GET 超全局变量

### 概述

`$_GET` 用于获取 URL 查询字符串（Query String）中的参数，通常用于 GET 请求。

### 语法

**访问方式**：`$_GET['key']` 或 `$_GET['key'] ?? default`

### 数据来源

`$_GET` 的数据来自 URL 查询字符串：

```
http://example.com/page.php?id=123&name=John
```

在这个 URL 中：
- `id=123` 对应 `$_GET['id'] = '123'`
- `name=John` 对应 `$_GET['name'] = 'John'`

### 基本用法

**示例 1：获取单个参数**
```php
<?php
declare(strict_types=1);

// 获取 id 参数
$id = $_GET['id'] ?? null;

if ($id !== null) {
    echo "ID: {$id}\n";
} else {
    echo "ID 参数不存在\n";
}
```

**访问 URL**：`http://example.com/page.php?id=123`

**输出**：
```
ID: 123
```

**示例 2：获取多个参数**
```php
<?php
declare(strict_types=1);

$id = $_GET['id'] ?? null;
$name = $_GET['name'] ?? '未知';
$page = (int) ($_GET['page'] ?? 1);

echo "ID: {$id}\n";
echo "姓名: {$name}\n";
echo "页码: {$page}\n";
```

**访问 URL**：`http://example.com/page.php?id=123&name=John&page=2`

**输出**：
```
ID: 123
姓名: John
页码: 2
```

### 数组参数

URL 可以传递数组参数。

**示例**：
```php
<?php
declare(strict_types=1);

// URL: page.php?tags[]=php&tags[]=web&tags[]=development
$tags = $_GET['tags'] ?? [];

if (is_array($tags)) {
    foreach ($tags as $tag) {
        echo "标签: {$tag}\n";
    }
}
```

**访问 URL**：`http://example.com/page.php?tags[]=php&tags[]=web&tags[]=development`

**输出**：
```
标签: php
标签: web
标签: development
```

### 使用场景

- **分页**：传递页码参数
- **搜索**：传递搜索关键词
- **筛选**：传递筛选条件
- **详情页**：传递资源 ID

## $_POST 超全局变量

### 概述

`$_POST` 用于获取 POST 请求体中的数据，通常用于表单提交和 API 请求。

### 语法

**访问方式**：`$_POST['key']` 或 `$_POST['key'] ?? default`

### 数据来源

`$_POST` 的数据来自 HTTP 请求体，Content-Type 通常是：
- `application/x-www-form-urlencoded`：表单数据
- `multipart/form-data`：文件上传

### 基本用法

**示例 1：处理表单提交**
```php
<?php
declare(strict_types=1);

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = $_POST['email'] ?? '';
    
    echo "姓名: {$name}\n";
    echo "邮箱: {$email}\n";
}
```

**HTML 表单**：
```html
<form method="POST" action="process.php">
    <input type="text" name="name" placeholder="姓名">
    <input type="email" name="email" placeholder="邮箱">
    <button type="submit">提交</button>
</form>
```

**示例 2：处理 JSON 请求**

注意：`$_POST` 只能获取 `application/x-www-form-urlencoded` 和 `multipart/form-data` 格式的数据。对于 `application/json` 格式，需要使用 `php://input`。

```php
<?php
declare(strict_types=1);

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    
    if (str_contains($contentType, 'application/json')) {
        // JSON 请求，使用 php://input
        $rawInput = file_get_contents('php://input');
        $data = json_decode($rawInput, true);
    } else {
        // 表单请求，使用 $_POST
        $data = $_POST;
    }
    
    print_r($data);
}
```

### 数组参数

POST 请求也可以传递数组参数。

**示例**：
```php
<?php
declare(strict_types=1);

// 表单字段：hobbies[]=reading&hobbies[]=coding
$hobbies = $_POST['hobbies'] ?? [];

if (is_array($hobbies)) {
    foreach ($hobbies as $hobby) {
        echo "爱好: {$hobby}\n";
    }
}
```

### 使用场景

- **表单提交**：处理用户提交的表单数据
- **API 请求**：处理 POST API 请求
- **数据创建**：创建新资源
- **数据更新**：更新现有资源

## $_REQUEST 超全局变量

### 概述

`$_REQUEST` 是一个合并数组，包含了 `$_GET`、`$_POST` 和 `$_COOKIE` 的数据。

### 语法

**访问方式**：`$_REQUEST['key']` 或 `$_REQUEST['key'] ?? default`

### 数据结构

`$_REQUEST` 的结构：
```php
$_REQUEST = array_merge($_COOKIE, $_POST, $_GET);
```

**注意**：合并顺序取决于 `php.ini` 中的 `request_order` 配置。

### 优先级问题

`$_REQUEST` 的优先级取决于 `request_order` 配置：

**默认配置**（`request_order = "GP"`）：
- 优先级：`$_POST` > `$_GET`
- `$_COOKIE` 不包含在 `$_REQUEST` 中

**包含 COOKIE**（`request_order = "GPC"`）：
- 优先级：`$_POST` > `$_GET` > `$_COOKIE`

**示例**：
```php
<?php
declare(strict_types=1);

// 假设 URL: page.php?name=John
// POST 数据: name=Jane
// Cookie: name=Bob

// $_REQUEST 的值取决于 request_order 配置
// 默认情况下：$_REQUEST['name'] = 'Jane'（POST 优先）
```

### 安全风险

**问题**：
1. **数据来源不明确**：无法确定数据来自 GET、POST 还是 COOKIE
2. **安全风险**：COOKIE 数据可能被恶意修改
3. **不符合 RESTful 原则**：GET 和 POST 应该有不同的用途

**示例**（安全风险）：
```php
<?php
declare(strict_types=1);

// 危险：使用 $_REQUEST，无法确定数据来源
$userId = $_REQUEST['user_id'];

// 如果 user_id 来自 COOKIE，可能被客户端修改
// 导致权限提升等安全问题
```

### 使用建议

**不推荐使用 `$_REQUEST`**，原因：
1. 数据来源不明确
2. 存在安全风险
3. 不符合最佳实践

**推荐做法**：
- 明确使用 `$_GET` 或 `$_POST`
- 根据请求方法选择对应的超全局变量
- 使用 `$_SERVER['REQUEST_METHOD']` 判断请求方法

## 数据获取方法

### 方法一：直接访问

直接使用数组访问方式获取数据。

**示例**：
```php
<?php
declare(strict_types=1);

// 直接访问
$id = $_GET['id'] ?? null;
$name = $_POST['name'] ?? '';

// 检查键是否存在
if (isset($_GET['id'])) {
    $id = $_GET['id'];
}
```

**优点**：
- 简单直接
- 性能好

**缺点**：
- 需要手动验证
- 容易忘记验证

### 方法二：使用 filter_input() 函数

`filter_input()` 函数可以获取并验证输入数据。

### filter_input() 函数

**语法**：`filter_input(int $type, string $var_name, int $filter = FILTER_DEFAULT, array|int $options = 0): mixed`

**参数**：
- `$type`：输入类型（`INPUT_GET`、`INPUT_POST`、`INPUT_COOKIE`、`INPUT_SERVER`、`INPUT_ENV`）
- `$var_name`：变量名
- `$filter`：过滤器类型（可选）
- `$options`：过滤器选项（可选）

**返回值**：成功返回过滤后的值，失败返回 `null` 或 `false`。

**示例 1：获取并验证整数**
```php
<?php
declare(strict_types=1);

// 获取并验证整数
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null) {
    echo "ID 无效\n";
} else {
    echo "ID: {$id}\n";
}
```

**示例 2：获取并验证邮箱**
```php
<?php
declare(strict_types=1);

// 获取并验证邮箱
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);

if ($email === false || $email === null) {
    echo "邮箱格式无效\n";
} else {
    echo "邮箱: {$email}\n";
}
```

**示例 3：获取并验证 URL**
```php
<?php
declare(strict_types=1);

// 获取并验证 URL
$url = filter_input(INPUT_GET, 'url', FILTER_VALIDATE_URL);

if ($url === false || $url === null) {
    echo "URL 格式无效\n";
} else {
    echo "URL: {$url}\n";
}
```

### 常用过滤器

| 过滤器 | 说明 | 示例 |
|:-------|:-----|:-----|
| `FILTER_VALIDATE_INT` | 验证整数 | `filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT)` |
| `FILTER_VALIDATE_EMAIL` | 验证邮箱 | `filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL)` |
| `FILTER_VALIDATE_URL` | 验证 URL | `filter_input(INPUT_GET, 'url', FILTER_VALIDATE_URL)` |
| `FILTER_SANITIZE_STRING` | 清理字符串 | `filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING)` |
| `FILTER_SANITIZE_EMAIL` | 清理邮箱 | `filter_input(INPUT_POST, 'email', FILTER_SANITIZE_EMAIL)` |

## 数据验证和过滤

### 验证方法

#### 1. 类型验证

**示例**：
```php
<?php
declare(strict_types=1);

// 验证整数
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
if ($id === false || $id === null) {
    die('ID 必须是整数');
}

// 验证浮点数
$price = filter_input(INPUT_POST, 'price', FILTER_VALIDATE_FLOAT);
if ($price === false || $price === null) {
    die('价格必须是数字');
}
```

#### 2. 格式验证

**示例**：
```php
<?php
declare(strict_types=1);

// 验证邮箱格式
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
if ($email === false || $email === null) {
    die('邮箱格式无效');
}

// 验证 URL 格式
$url = filter_input(INPUT_GET, 'url', FILTER_VALIDATE_URL);
if ($url === false || $url === null) {
    die('URL 格式无效');
}
```

#### 3. 范围验证

**示例**：
```php
<?php
declare(strict_types=1);

// 验证整数范围
$age = filter_input(INPUT_POST, 'age', FILTER_VALIDATE_INT, [
    'options' => [
        'min_range' => 1,
        'max_range' => 120,
    ],
]);

if ($age === false || $age === null) {
    die('年龄必须在 1-120 之间');
}
```

### 过滤方法

#### 1. 字符串清理

**示例**：
```php
<?php
declare(strict_types=1);

// 清理字符串（移除标签）
$name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING);
// 注意：FILTER_SANITIZE_STRING 在 PHP 8.1+ 已废弃，使用 FILTER_SANITIZE_FULL_SPECIAL_CHARS

// PHP 8.1+ 推荐使用
$name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_FULL_SPECIAL_CHARS);
```

#### 2. 邮箱清理

**示例**：
```php
<?php
declare(strict_types=1);

// 清理邮箱
$email = filter_input(INPUT_POST, 'email', FILTER_SANITIZE_EMAIL);
```

#### 3. URL 清理

**示例**：
```php
<?php
declare(strict_types=1);

// 清理 URL
$url = filter_input(INPUT_GET, 'url', FILTER_SANITIZE_URL);
```

### 完整验证示例

```php
<?php
declare(strict_types=1);

class InputValidator
{
    public static function validateGetInt(string $key, ?int $default = null): ?int
    {
        $value = filter_input(INPUT_GET, $key, FILTER_VALIDATE_INT);
        return $value !== false && $value !== null ? $value : $default;
    }

    public static function validatePostString(string $key, string $default = ''): string
    {
        $value = filter_input(INPUT_POST, $key, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
        return $value !== false && $value !== null ? $value : $default;
    }

    public static function validatePostEmail(string $key): ?string
    {
        $value = filter_input(INPUT_POST, $key, FILTER_VALIDATE_EMAIL);
        return $value !== false && $value !== null ? $value : null;
    }
}

// 使用
$id = InputValidator::validateGetInt('id');
$name = InputValidator::validatePostString('name');
$email = InputValidator::validatePostEmail('email');
```

## 数组参数处理

### GET 数组参数

**URL 格式**：`page.php?tags[]=php&tags[]=web&tags[]=development`

**处理方式**：
```php
<?php
declare(strict_types=1);

$tags = $_GET['tags'] ?? [];

if (is_array($tags)) {
    foreach ($tags as $tag) {
        // 验证和清理每个标签
        $cleanTag = filter_var($tag, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
        echo "标签: {$cleanTag}\n";
    }
}
```

### POST 数组参数

**表单字段**：
```html
<input type="checkbox" name="hobbies[]" value="reading">
<input type="checkbox" name="hobbies[]" value="coding">
<input type="checkbox" name="hobbies[]" value="sports">
```

**处理方式**：
```php
<?php
declare(strict_types=1);

$hobbies = $_POST['hobbies'] ?? [];

if (is_array($hobbies)) {
    $cleanHobbies = [];
    foreach ($hobbies as $hobby) {
        $cleanHobbies[] = filter_var($hobby, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
    }
    print_r($cleanHobbies);
}
```

### 多维数组参数

**URL 格式**：`page.php?user[name]=John&user[email]=john@example.com`

**处理方式**：
```php
<?php
declare(strict_types=1);

$user = $_GET['user'] ?? [];

if (is_array($user)) {
    $name = filter_var($user['name'] ?? '', FILTER_SANITIZE_FULL_SPECIAL_CHARS);
    $email = filter_var($user['email'] ?? '', FILTER_VALIDATE_EMAIL);
    
    echo "姓名: {$name}\n";
    echo "邮箱: {$email}\n";
}
```

## 使用场景

### 表单数据处理

- 处理用户提交的表单数据
- 验证表单输入
- 清理和过滤数据

### URL 参数获取

- 获取分页参数
- 获取搜索关键词
- 获取筛选条件

### API 请求处理

- 处理 GET API 请求
- 处理 POST API 请求
- 验证 API 参数

### 数据验证

- 验证用户输入
- 过滤恶意内容
- 确保数据安全

## 注意事项

### 数据验证和过滤

- **始终验证输入**：验证所有用户输入
- **使用 filter_input()**：使用 `filter_input()` 函数验证和过滤
- **提供默认值**：使用 `??` 运算符提供默认值
- **类型转换**：根据需要进行类型转换

### XSS 防护

- **转义输出**：输出时使用 `htmlspecialchars()` 转义
- **验证输入**：验证和过滤所有输入
- **使用模板引擎**：模板引擎默认自动转义

### SQL 注入防护

- **使用预处理语句**：使用 PDO 或 mysqli 的预处理语句
- **不要直接拼接 SQL**：避免将用户输入直接拼接到 SQL 语句中
- **验证数据类型**：验证数值类型参数

### 避免使用 $_REQUEST

- **不推荐使用**：`$_REQUEST` 数据来源不明确，存在安全风险
- **明确使用**：根据请求方法明确使用 `$_GET` 或 `$_POST`
- **判断请求方法**：使用 `$_SERVER['REQUEST_METHOD']` 判断请求方法

### 数组参数处理

- **检查类型**：使用 `is_array()` 检查是否为数组
- **遍历验证**：对数组中的每个元素进行验证
- **防止类型混淆**：确保参数类型符合预期

## 常见问题

### $_GET 和 $_POST 的区别？

| 特性 | $_GET | $_POST |
|:-----|:------|:-------|
| 数据位置 | URL 查询字符串 | HTTP 请求体 |
| 数据大小 | 受 URL 长度限制 | 无限制（受服务器配置限制） |
| 安全性 | 数据在 URL 中可见 | 数据在请求体中，相对安全 |
| 缓存 | 可缓存 | 不可缓存 |
| 用途 | 获取资源 | 创建/更新资源 |
| 幂等性 | 幂等 | 非幂等 |

### 为什么避免使用 $_REQUEST？

1. **数据来源不明确**：无法确定数据来自 GET、POST 还是 COOKIE
2. **安全风险**：COOKIE 数据可能被恶意修改
3. **不符合 RESTful 原则**：GET 和 POST 应该有不同的用途
4. **调试困难**：难以追踪数据来源

### 如何安全地获取数据？

1. **使用 filter_input()**：使用 `filter_input()` 函数验证和过滤
2. **提供默认值**：使用 `??` 运算符提供默认值
3. **验证类型**：验证数据类型和格式
4. **转义输出**：输出时转义特殊字符

### 如何处理数组参数？

1. **检查类型**：使用 `is_array()` 检查是否为数组
2. **遍历验证**：对数组中的每个元素进行验证
3. **使用 filter_var_array()**：使用 `filter_var_array()` 批量验证数组

**示例**：
```php
<?php
declare(strict_types=1);

$tags = $_GET['tags'] ?? [];

if (is_array($tags)) {
    $cleanTags = filter_var_array($tags, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
    print_r($cleanTags);
}
```

## 最佳实践

### 明确使用 $_GET 或 $_POST

- 根据请求方法选择对应的超全局变量
- 使用 `$_SERVER['REQUEST_METHOD']` 判断请求方法
- 避免使用 `$_REQUEST`

### 始终验证和过滤输入

- 使用 `filter_input()` 函数验证和过滤
- 验证数据类型和格式
- 提供默认值

### 使用 filter_input() 函数

- 使用 `filter_input()` 获取和验证数据
- 选择合适的过滤器
- 处理验证失败的情况

### 提供默认值

- 使用 `??` 运算符提供默认值
- 避免使用未定义的数组键
- 使用 `isset()` 检查键是否存在

### 转义输出

- 输出时使用 `htmlspecialchars()` 转义
- 使用模板引擎自动转义
- 防止 XSS 攻击

## 相关章节

- **[5.3.2 $_SERVER、$_SESSION 与 $_COOKIE](section-02-server-session-cookie.md)**：了解其他超全局变量的使用方法
- **[5.3.3 $_FILES 与安全处理](section-03-files-security.md)**：了解文件上传的处理方法
- **[5.5 请求体解析：处理 JSON / API 请求](../chapter-05-json-requests/readme.md)**：了解 JSON 请求的处理方法

## 练习任务

1. **处理 GET 请求参数**
   - 创建一个页面，接收并显示 GET 参数
   - 验证参数类型和格式
   - 处理缺失参数的情况

2. **处理 POST 表单提交**
   - 创建一个表单页面
   - 处理表单提交数据
   - 验证和清理表单数据

3. **实现数据验证函数**
   - 创建 `validateGetInt()` 函数
   - 创建 `validatePostEmail()` 函数
   - 创建 `validatePostString()` 函数

4. **处理数组参数**
   - 处理多选复选框数据
   - 处理多维数组参数
   - 验证数组中的每个元素

5. **实现安全的输入处理**
   - 使用 `filter_input()` 验证所有输入
   - 转义所有输出
   - 测试 XSS 和 SQL 注入防护

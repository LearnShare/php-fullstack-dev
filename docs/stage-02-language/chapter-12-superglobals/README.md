# 2.12 超级全局变量

## 目标

- 熟悉 PHP 内置的超级全局变量及其生命周期。
- 能安全地从 `$_GET`、`$_POST`、`$_COOKIE`、`$_FILES`、`$_SERVER` 等读取数据。
- 掌握表单上传、Session、请求信息与安全过滤策略。

## 列表总览

| 变量        | 说明                               |
| :---------- | :--------------------------------- |
| `$_GET`     | 查询字符串参数                     |
| `$_POST`    | 表单提交（`application/x-www-form-urlencoded` 或 `multipart/form-data`） |
| `$_REQUEST` | `$_GET + $_POST + $_COOKIE`（不建议使用） |
| `$_SERVER`  | 请求元数据（方法、URI、Header）    |
| `$_COOKIE`  | 浏览器 Cookie                      |
| `$_SESSION` | Session 数据                       |
| `$_FILES`   | 上传文件信息                       |
| `$_ENV`     | 环境变量                           |
| `GLOBALS`   | 所有全局变量的数组                 |

## `$_GET` 与 `$_POST`

- 使用 `filter_input(INPUT_GET, 'key')` 安全读取。
- 对数字参数使用 `FILTER_VALIDATE_INT`。
- 对字符串参数 `FILTER_SANITIZE_SPECIAL_CHARS`。

```php
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT) ?? 1;
$token = filter_input(INPUT_POST, 'csrf_token', FILTER_DEFAULT);
```

## `$_SERVER`

常用键：

| 键名              | 说明               |
| :---------------- | :----------------- |
| `REQUEST_METHOD`  | HTTP 方法          |
| `REQUEST_URI`     | 路径与查询字符串   |
| `HTTP_HOST`       | host 头            |
| `HTTP_USER_AGENT` | 客户端 UA          |
| `REMOTE_ADDR`     | 客户端 IP          |

示例：

```php
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    exit('Method Not Allowed');
}
```

## `$_COOKIE` 与 `$_SESSION`

- 设置 Cookie：`setcookie('name', 'value', $expire, '/', '', true, true);`
- Session：
  ```php
  session_start();
  $_SESSION['user_id'] = 1;
  session_regenerate_id(true);
  ```
- 避免将敏感数据放在 Cookie；Session 可存于 Redis/Memcached。

## `$_FILES`

- 结构示例：
  ```php
  $_FILES['avatar'] = [
      'name' => 'me.png',
      'type' => 'image/png',
      'tmp_name' => '/tmp/php123.tmp',
      'error' => 0,
      'size' => 123456,
  ];
  ```
- 检查：
  - `UPLOAD_ERR_OK` 等错误码。
  - MIME 类型：`finfo_file()` 或 `mime_content_type()`.
  - 大小：`$_FILES['avatar']['size']`。
  - 使用 `move_uploaded_file()` 移动到安全目录。

## 安全建议

1. **不要信任客户端数据**：所有输入都需过滤/验证。
2. **CSRF 防护**：表单中包含 token，并验证 `$_POST['csrf_token']`。
3. **XSS 预防**：输出前使用 `htmlspecialchars`。
4. **SQL 注入**：使用 PDO 预处理，不直接拼接 `$_GET`/`$_POST` 值。
5. **上传文件**：限制文件类型、大小、存储路径，避免可执行权限。

## 练习

1. 编写 `request(string $key, mixed $default = null)` 助手函数，从 `$_GET` 或 `$_POST` 读取数据并自动过滤。
2. 实现一个上传接口，验证文件大小、MIME 类型，并生成随机文件名保存。
3. 使用 Session 构建一个简易的闪存消息（Flash Message）系统：`flash('key', 'value')` 设置，`flash('key')` 读取后删除。

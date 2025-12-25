# 2.2.1 输出 API

## 目标

- 掌握 `echo`、`print`、`printf`、`sprintf` 的语义差异与使用场景。
- 理解输出缓冲机制（`ob_start`、`ob_get_clean`）及其在模板渲染、邮件生成中的应用。
- 区分 CLI 与 Web 环境下的输出行为差异。
- 构建“开发输出 vs 生产日志”的思维，避免把调试信息泄漏到用户界面。

## 基础输出函数

### `echo`

#### 语法

```php
echo expression [, expression ... ]
```

#### 参数说明

- `expression`：任意数量的字符串、变量或表达式。
- 支持多个参数，用逗号分隔。
- 不会自动换行，需要手动添加 `\n`（CLI）或 `<br>`（HTML）。

#### 返回值

- 无返回值（`void`）。

#### 使用场景

- 输出 HTML 片段、模板数据。
- 快速调试输出。
- CLI 脚本的标准输出。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 基础用法
echo "Hello World\n";

// 多个参数
$name = "张三";
$age = 25;
echo "姓名：", $name, "，年龄：", $age, "\n";
// 姓名：张三，年龄：25

// 输出变量
$price = 99.99;
echo "价格：$price\n";  // 双引号内变量会被解析
echo '价格：$price\n';  // 单引号内变量不会被解析，输出：价格：$price\n

// 输出表达式结果
echo "总和：", (10 + 20), "\n";  // 总和：30

// HTML 输出
echo "<div class='user'>";
echo "<h1>", htmlspecialchars($name), "</h1>";
echo "</div>";

// 数组输出（需要转换）
$items = ['apple', 'banana', 'orange'];
echo implode(', ', $items), "\n";  // apple, banana, orange
```

#### 性能特点

- `echo` 是语言构造（language construct），不是函数，执行速度最快。
- 多个参数时性能优于字符串连接。

```php
// 推荐：使用多个参数
echo "User: ", $name, " Age: ", $age, "\n";

// 不推荐：字符串连接（性能较差）
echo "User: " . $name . " Age: " . $age . "\n";
```

### `print`

#### 语法

```php
print expression
```

#### 参数说明

- `expression`：仅接受一个表达式。
- 不能像 `echo` 那样传递多个参数。

#### 返回值

- 始终返回 `1`（成功），因此可用于条件表达式。

#### 使用场景

- 需要在三元运算符、`if` 语句中嵌入输出时。
- 需要将输出作为表达式的一部分时。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 基础用法
print "Hello World\n";

// 在条件表达式中使用
if (print("Ready? ")) {
    echo "Go!\n";
}
// 输出：Ready? Go!

// 三元运算符中使用
$status = print("Processing... ") ? "OK" : "Failed";
echo $status, "\n";
// 输出：Processing... OK

// 与 echo 的区别
echo "A", "B", "C\n";  // 可以多个参数
// print "A", "B", "C\n";  // 错误：print 只能一个参数

// 返回值示例
$result = print("Test\n");
var_dump($result);  // int(1)
```

#### 性能特点

- `print` 也是语言构造，但性能略低于 `echo`（因为需要返回 1）。
- 实际开发中，除非需要返回值，否则优先使用 `echo`。

### `printf`

#### 语法

```php
printf(string $format, mixed ...$values): int
```

#### 参数说明

- `$format`：格式化字符串，包含占位符（如 `%s`、`%d`、`%.2f`）。
- `...$values`：可变参数，依次填充占位符。

#### 返回值

- 返回输出的字节数（`int`）。

#### 使用场景

- 直接输出带格式的文本（金额、时间、编号）。
- 需要精确控制输出格式时。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 基础格式化
$name = "张三";
$age = 25;
printf("姓名：%s，年龄：%d 岁\n", $name, $age);
// 姓名：张三，年龄：25 岁

// 金额格式化
$price = 199.99;
printf("价格：%.2f CNY\n", $price);
// 价格：199.99 CNY

// 订单号格式化（左侧补零）
$orderId = 7;
printf("订单号：%05d\n", $orderId);
// 订单号：00007

// 获取输出字节数
$bytes = printf("Hello %s\n", "World");
echo "输出字节数：$bytes\n";
// Hello World
// 输出字节数：12

// 复杂格式
$userId = 12345;
$balance = 999.99;
printf("用户ID：%08d | 余额：%+.2f CNY\n", $userId, $balance);
// 用户ID：00012345 | 余额：+999.99 CNY
```

#### 与 `echo` 的对比

```php
// 使用 echo（需要手动格式化）
$price = 99.99;
echo "价格：" . number_format($price, 2) . " CNY\n";

// 使用 printf（更简洁）
printf("价格：%.2f CNY\n", $price);
```

### `sprintf`

#### 语法

```php
sprintf(string $format, mixed ...$values): string
```

#### 参数说明

- 与 `printf` 完全相同，但结果以字符串返回，不直接输出。

#### 返回值

- 返回格式化后的字符串（`string`）。

#### 使用场景

- 需要拼接、存储、记录日志时。
- 生成文件名、URL、SQL 语句等。
- 需要多次使用格式化结果时。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 基础用法
$message = sprintf("用户 %s 登录成功\n", "张三");
echo $message;

// 日志记录
$logLine = sprintf(
    "[%s] %s: %s\n",
    date('Y-m-d H:i:s'),
    "INFO",
    "订单创建成功"
);
file_put_contents('app.log', $logLine, FILE_APPEND);

// 生成文件名
$userId = 42;
$fileIndex = 7;
$filename = sprintf("user_%05d_file_%03d.jpg", $userId, $fileIndex);
echo $filename, "\n";  // user_00042_file_007.jpg

// 拼接 SQL 查询（注意：实际应使用预处理语句）
$table = "users";
$id = 123;
$query = sprintf("SELECT * FROM %s WHERE id = %d", $table, $id);
// 注意：此示例仅用于演示，实际应使用 PDO 预处理语句防止 SQL 注入

// 构建 URL
$baseUrl = "https://api.example.com";
$endpoint = "users";
$userId = 42;
$url = sprintf("%s/%s/%d", $baseUrl, $endpoint, $userId);
echo $url, "\n";  // https://api.example.com/users/42

// 邮件模板
$userName = "张三";
$orderId = 12345;
$amount = 199.99;
$emailBody = sprintf(
    "尊敬的 %s，\n您的订单 %05d 已创建，金额：%.2f CNY。\n",
    $userName,
    $orderId,
    $amount
);
// mail($user->email, "订单确认", $emailBody);
```

#### 与 `printf` 的对比

```php
// printf：直接输出
printf("价格：%.2f\n", 99.99);

// sprintf：返回字符串
$priceStr = sprintf("价格：%.2f", 99.99);
echo $priceStr, "\n";

// sprintf 的优势：可以复用
$template = sprintf("用户：%s，余额：%.2f", $name, $balance);
echo $template, "\n";
file_put_contents('user.txt', $template);
logger()->info($template);
```

## 输出缓冲机制

### `ob_start`

#### 语法

```php
ob_start(?callable $callback = null, int $chunk_size = 0, int $flags = PHP_OUTPUT_HANDLER_STDFLAGS): bool
```

#### 参数说明

- `$callback`：可选的回调函数，用于处理缓冲区内容（如压缩、过滤）。
- `$chunk_size`：缓冲区大小（字节），0 表示无限制。
- `$flags`：输出处理标志，通常使用默认值 `PHP_OUTPUT_HANDLER_STDFLAGS`。

#### 返回值

- 成功返回 `true`，失败返回 `false`。

#### 使用场景

- 捕获模板输出、生成邮件正文。
- 测试渲染结果。
- 压缩输出内容。
- 延迟输出（在设置 HTTP 头之前）。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 基础用法：捕获输出
ob_start();
echo "这是被缓冲的内容\n";
echo "这些内容不会立即输出\n";
$content = ob_get_clean();
echo "捕获的内容：\n", $content;

// 模板渲染
function renderTemplate(string $template, array $data): string
{
    ob_start();
    extract($data);
    include $template;
    return ob_get_clean();
}

// 使用示例
$data = ['name' => '张三', 'age' => 25];
$html = renderTemplate('user.php', $data);

// 邮件生成
ob_start();
include __DIR__ . '/email-template.php';
$emailBody = ob_get_clean();
mail($user->email, 'Welcome', $emailBody);

// 压缩输出
ob_start(function ($buffer) {
    return gzencode($buffer);
});
echo "大量内容...";
ob_end_flush();  // 输出压缩后的内容

// 过滤输出（移除调试信息）
ob_start(function ($buffer) {
    // 移除所有 var_dump 输出
    return preg_replace('/.*var_dump.*\n/', '', $buffer);
});
echo "正常内容\n";
var_dump("调试信息");  // 这行会被过滤
echo "更多内容\n";
ob_end_flush();

// 嵌套缓冲
ob_start();
echo "外层内容\n";
ob_start();
echo "内层内容\n";
$inner = ob_get_clean();
echo "内层：", $inner, "\n";
$outer = ob_get_clean();
echo "外层：", $outer, "\n";
```

### `ob_get_clean`

#### 语法

```php
ob_get_clean(): string|false
```

#### 参数说明

- 无参数。

#### 返回值

- 返回当前缓冲区内容（`string`），并清空该缓冲。
- 如果缓冲区不存在，返回 `false`。

#### 使用场景

- 与 `ob_start()` 搭配，把输出保存为字符串。
- 获取模板渲染结果。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 基础用法
ob_start();
echo "第一行\n";
echo "第二行\n";
$content = ob_get_clean();
echo "捕获的内容：\n", $content;

// 模板渲染
ob_start();
?>
<div class="user">
    <h1><?= htmlspecialchars($name) ?></h1>
    <p>年龄：<?= $age ?></p>
</div>
<?php
$html = ob_get_clean();
echo $html;

// 错误处理
if (ob_get_level() > 0) {
    $content = ob_get_clean();
    if ($content !== false) {
        echo "成功捕获：", $content, "\n";
    }
} else {
    echo "没有活动的缓冲区\n";
}
```

### 其他输出缓冲函数

#### `ob_get_contents()`

- 获取缓冲区内容，但不清空缓冲区。

```php
ob_start();
echo "内容\n";
$content = ob_get_contents();  // 获取但不清空
echo "再次获取：", ob_get_contents(), "\n";  // 仍然可以获取
ob_end_clean();  // 清空并关闭
```

#### `ob_end_flush()`

- 输出缓冲区内容并关闭缓冲区。

```php
ob_start();
echo "内容\n";
ob_end_flush();  // 输出并关闭
```

#### `ob_end_clean()`

- 清空缓冲区内容并关闭缓冲区（不输出）。

```php
ob_start();
echo "这不会被输出\n";
ob_end_clean();  // 清空并关闭，不输出
```

#### `ob_get_level()`

- 返回当前嵌套缓冲区的层级数。

```php
echo ob_get_level(), "\n";  // 0（没有缓冲区）
ob_start();
echo ob_get_level(), "\n";  // 1
ob_start();
echo ob_get_level(), "\n";  // 2
```

## CLI 与 Web 环境差异

### CLI 环境

```php
<?php
declare(strict_types=1);

// CLI 中直接输出到终端
echo "Hello\n";  // 输出到标准输出
fwrite(STDERR, "Error\n");  // 输出到标准错误

// 换行符使用 \n
echo "第一行\n第二行\n";

// 颜色输出（ANSI 转义码）
echo "\033[31m红色文字\033[0m\n";
echo "\033[32m绿色文字\033[0m\n";
```

### Web 环境

```php
<?php
declare(strict_types=1);

// Web 中输出到 HTTP 响应
echo "Hello";  // 输出到响应体

// 换行符在 HTML 中需要 <br> 或保留格式
echo "第一行<br>第二行\n";
echo "<pre>第一行\n第二行</pre>";

// 注意：在输出 HTML 前不能有任何输出（包括空格、BOM）
// 否则无法设置 HTTP 头
header('Content-Type: text/html; charset=UTF-8');
echo "<html>...</html>";
```

### 环境检测

```php
<?php
declare(strict_types=1);

function isCli(): bool
{
    return php_sapi_name() === 'cli' || php_sapi_name() === 'phpdbg';
}

// 根据环境选择输出方式
if (isCli()) {
    echo "CLI 模式\n";
} else {
    echo "<p>Web 模式</p>";
}
```

## 实际应用示例

### 示例 1：模板渲染系统

```php
<?php
declare(strict_types=1);

class TemplateRenderer
{
    public function render(string $template, array $data = []): string
    {
        ob_start();
        extract($data);
        include $template;
        return ob_get_clean();
    }
}

// 使用
$renderer = new TemplateRenderer();
$html = $renderer->render('user.php', [
    'name' => '张三',
    'age' => 25
]);
echo $html;
```

### 示例 2：日志格式化

```php
<?php
declare(strict_types=1);

class Logger
{
    public function log(string $level, string $message, array $context = []): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $contextStr = json_encode($context, JSON_UNESCAPED_UNICODE);
        
        $logLine = sprintf(
            "[%s] %-8s | %s | %s\n",
            $timestamp,
            $level,
            $message,
            $contextStr
        );
        
        file_put_contents('app.log', $logLine, FILE_APPEND);
    }
}

$logger = new Logger();
$logger->log('INFO', '用户登录', ['user_id' => 123]);
```

### 示例 3：响应构建器

```php
<?php
declare(strict_types=1);

class ResponseBuilder
{
    private array $headers = [];
    private string $body = '';
    
    public function header(string $name, string $value): self
    {
        $this->headers[$name] = $value;
        return $this;
    }
    
    public function body(string $content): self
    {
        $this->body = $content;
        return $this;
    }
    
    public function send(): void
    {
        foreach ($this->headers as $name => $value) {
            header("$name: $value");
        }
        echo $this->body;
    }
}

// 使用
$response = new ResponseBuilder();
$response
    ->header('Content-Type', 'application/json')
    ->body(json_encode(['status' => 'success']))
    ->send();
```

## 相关章节

本节主要介绍用于输出的函数（`echo`、`print`、`printf`、`sprintf`）。如需了解其他相关内容，请参考：

- **[2.2.3 调试函数和策略](section-03-debugging.md)**：详细介绍 `var_dump()`、`print_r()`、`var_export()`、`debug_backtrace()` 等调试函数的用法、参数说明、使用场景和完整示例。这些函数用于检测和输出变量及数据结构，在开发调试中非常有用。

- **[2.4.4 类型检测](../chapter-04-types/section-04-type-detection.md)**：详细介绍 `gettype()`、`is_int()`、`is_string()`、`is_array()` 等类型检测函数的用法、参数说明、使用场景和最佳实践。这些函数用于检测变量的类型，是编写健壮代码的重要工具。

- **[2.15 isset、empty 与 Null 合并运算](../chapter-15-null-system/readme.md)**：详细介绍 `isset()`、`empty()`、`is_null()` 和空合并运算符（`??`）的用法、参数说明、使用场景和最佳实践。在输出变量之前，通常需要先检查变量是否存在或为空，这些函数和运算符是处理这种情况的重要工具。

## 常见错误与注意事项

1. **输出顺序问题**：在 Web 环境中，必须在输出任何内容之前设置 HTTP 头。

```php
// 错误示例
echo "内容";
header('Content-Type: text/html');  // 错误：已有输出

// 正确示例
header('Content-Type: text/html');
echo "内容";
```

2. **BOM 问题**：文件开头的 BOM 会导致无法设置 HTTP 头。

```php
// 确保文件是 UTF-8 无 BOM 编码
// 在编辑器中检查并移除 BOM
```

3. **输出缓冲嵌套**：注意缓冲区的嵌套层级。

```php
ob_start();
echo "外层\n";
ob_start();
echo "内层\n";
$inner = ob_get_clean();  // 只关闭内层
$outer = ob_get_clean();  // 关闭外层
```

## 练习

1. 编写一个函数 `formatUserInfo($name, $age, $email)`，使用 `sprintf` 格式化用户信息并返回字符串。

2. 实现一个简单的模板引擎，使用输出缓冲捕获 `include` 的模板输出。

3. 创建一个日志类，使用 `sprintf` 格式化日志行，并支持不同日志级别（INFO、WARNING、ERROR）。

4. 编写一个函数，检测当前运行环境（CLI 或 Web），并根据环境选择不同的输出方式。

5. 使用输出缓冲实现一个内容压缩功能，将输出内容进行 gzip 压缩。

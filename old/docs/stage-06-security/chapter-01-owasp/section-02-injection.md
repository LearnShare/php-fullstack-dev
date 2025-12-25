# 6.1.2 注入攻击防护

## 概述

注入攻击是最常见的安全风险之一。本节详细介绍 SQL 注入、XSS 攻击、命令注入、文件包含等注入攻击，以及相应的防护方法。

## SQL 注入

### 什么是 SQL 注入

通过构造恶意 SQL 语句，绕过应用程序的安全检查。

### SQL 注入示例

```php
<?php
// 危险示例
$id = $_GET['id'];
$sql = "SELECT * FROM users WHERE id = {$id}";
// 如果 id = "1 OR 1=1"，会返回所有用户

// 安全示例：使用预处理语句
$id = $_GET['id'];
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
```

### 防护措施

#### 1. 使用 PDO 预处理

```php
<?php
declare(strict_types=1);

// 始终使用预处理语句
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ? AND status = ?");
$stmt->execute([$email, $status]);
```

#### 2. 使用 ORM

```php
<?php
// Eloquent ORM 自动使用预处理
$user = User::where('email', $email)
    ->where('status', $status)
    ->first();
```

#### 3. 输入验证

```php
<?php
declare(strict_types=1);

function validateInput(mixed $input, string $type): mixed
{
    return match ($type) {
        'int' => filter_var($input, FILTER_VALIDATE_INT),
        'email' => filter_var($input, FILTER_VALIDATE_EMAIL),
        'string' => filter_var($input, FILTER_SANITIZE_STRING),
        default => throw new InvalidArgumentException("Unknown type: {$type}"),
    };
}
```

## XSS 攻击

### 什么是 XSS

- **XSS**：Cross-Site Scripting，跨站脚本攻击
- 攻击者在网页中注入恶意脚本
- 脚本在用户浏览器中执行，窃取信息或执行操作

### XSS 类型

#### 1. 反射型 XSS

```php
<?php
// 危险示例
$name = $_GET['name'];
echo "Hello, {$name}";  // 如果 name 是 <script>alert('XSS')</script>

// 安全示例
$name = htmlspecialchars($_GET['name'] ?? '', ENT_QUOTES, 'UTF-8');
echo "Hello, {$name}";
```

#### 2. 存储型 XSS

```php
<?php
// 危险示例
$comment = $_POST['comment'];
$stmt = $pdo->prepare("INSERT INTO comments (content) VALUES (?)");
$stmt->execute([$comment]);

// 安全示例
$comment = htmlspecialchars($_POST['comment'] ?? '', ENT_QUOTES, 'UTF-8');
$stmt = $pdo->prepare("INSERT INTO comments (content) VALUES (?)");
$stmt->execute([$comment]);
```

### XSS 防护

#### 1. 输出转义

```php
<?php
declare(strict_types=1);

function e(string $string): string
{
    return htmlspecialchars($string, ENT_QUOTES | ENT_HTML5, 'UTF-8');
}

// 使用
echo e($userInput);
```

#### 2. HTML Purifier

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use HTMLPurifier;
use HTMLPurifier_Config;

$config = HTMLPurifier_Config::createDefault();
$purifier = new HTMLPurifier($config);

// 清理 HTML，保留安全标签
$clean = $purifier->purify($userInput);
```

#### 3. Content Security Policy (CSP)

```php
<?php
// 设置 CSP 头
header("Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'");
```

## 命令注入

### 防护措施

```php
<?php
declare(strict_types=1);

// 危险示例
$command = "ls " . $_GET['dir'];
exec($command);

// 安全示例
$dir = escapeshellarg($_GET['dir'] ?? '');
$command = "ls " . $dir;
exec($command);
```

## 文件包含

### 防护措施

```php
<?php
declare(strict_types=1);

// 危险示例
include $_GET['file'];

// 安全示例
$allowedFiles = ['header.php', 'footer.php'];
$file = $_GET['file'] ?? '';
if (!in_array($file, $allowedFiles, true)) {
    throw new InvalidArgumentException('Invalid file');
}
include $file;
```

## 完整示例

```php
<?php
declare(strict_types=1);

class InjectionProtection
{
    public function sanitizeInput(string $input, string $type): string
    {
        return match ($type) {
            'html' => htmlspecialchars($input, ENT_QUOTES, 'UTF-8'),
            'sql' => addslashes($input),  // 但最好使用预处理
            'shell' => escapeshellarg($input),
            default => $input,
        };
    }

    public function validateInput(mixed $input, string $type): bool
    {
        return match ($type) {
            'int' => filter_var($input, FILTER_VALIDATE_INT) !== false,
            'email' => filter_var($input, FILTER_VALIDATE_EMAIL) !== false,
            'url' => filter_var($input, FILTER_VALIDATE_URL) !== false,
            default => false,
        };
    }
}
```

## 注意事项

1. **预处理语句**：始终使用预处理语句防止 SQL 注入
2. **输出转义**：所有用户输入都要转义
3. **输入验证**：验证所有用户输入
4. **最小权限**：使用最小权限原则

## 练习

1. 实现完整的注入攻击防护系统。

2. 创建一个输入验证和清理工具。

3. 编写 XSS 防护中间件。

4. 实现 SQL 注入检测工具。

# 6.1 OWASP Top 10（2025）风险模型与防御

## 目标

- 理解 OWASP Top 10 安全风险。
- 掌握 XSS（跨站脚本攻击）的防护方法。
- 了解 CSRF（跨站请求伪造）的防御策略。
- 熟悉 SQL 注入的防护（PDO 预处理 + ORM）。
- 了解其他常见安全风险及防护措施。

## OWASP Top 10 概览

### 2025 版 Top 10

1. **A01:2021 – Broken Access Control**（访问控制失效）
2. **A02:2021 – Cryptographic Failures**（加密失败）
3. **A03:2021 – Injection**（注入攻击）
4. **A04:2021 – Insecure Design**（不安全设计）
5. **A05:2021 – Security Misconfiguration**（安全配置错误）
6. **A06:2021 – Vulnerable Components**（易受攻击的组件）
7. **A07:2021 – Authentication Failures**（身份验证失败）
8. **A08:2021 – Software and Data Integrity**（软件和数据完整性）
9. **A09:2021 – Security Logging Failures**（安全日志记录失败）
10. **A10:2021 – Server-Side Request Forgery**（服务端请求伪造）

## XSS（跨站脚本攻击）

### 什么是 XSS

- **XSS**：Cross-Site Scripting，跨站脚本攻击。
- 攻击者在网页中注入恶意脚本。
- 脚本在用户浏览器中执行，窃取信息或执行操作。

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
// 存储的恶意脚本会在其他用户查看时执行

// 安全示例
$comment = htmlspecialchars($_POST['comment'] ?? '', ENT_QUOTES, 'UTF-8');
$stmt = $pdo->prepare("INSERT INTO comments (content) VALUES (?)");
$stmt->execute([$comment]);
```

#### 3. DOM-Based XSS

```javascript
// 危险示例
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('output').innerHTML = name;  // 直接插入 HTML

// 安全示例
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('output').textContent = name;  // 使用 textContent
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

## CSRF（跨站请求伪造）

### 什么是 CSRF

- **CSRF**：Cross-Site Request Forgery，跨站请求伪造。
- 攻击者诱导用户执行非预期的操作。

### CSRF 攻击示例

```html
<!-- 恶意网站 -->
<img src="https://bank.com/transfer?to=attacker&amount=1000" />
```

### CSRF 防护

#### 1. CSRF Token

```php
<?php
declare(strict_types=1);

session_start();

function generateCsrfToken(): string
{
    if (!isset($_SESSION['csrf_token'])) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }
    return $_SESSION['csrf_token'];
}

function validateCsrfToken(string $token): bool
{
    return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
}

// 在表单中
$token = generateCsrfToken();
echo "<input type='hidden' name='csrf_token' value='{$token}'>";

// 验证
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $token = $_POST['csrf_token'] ?? '';
    if (!validateCsrfToken($token)) {
        http_response_code(403);
        die('Invalid CSRF token');
    }
}
```

#### 2. SameSite Cookie

```php
<?php
// 设置 SameSite Cookie
setcookie('session_id', $sessionId, [
    'samesite' => 'Strict',  // 或 'Lax'
    'secure' => true,
    'httponly' => true,
]);
```

#### 3. 双提交 Cookie

```php
<?php
declare(strict_types=1);

// 设置 Cookie
$csrfToken = bin2hex(random_bytes(32));
setcookie('csrf_token', $csrfToken, [
    'samesite' => 'Strict',
    'secure' => true,
    'httponly' => false,  // 允许 JavaScript 访问
]);

// 在表单中
echo "<input type='hidden' name='csrf_token' value='{$csrfToken}'>";

// 验证
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $cookieToken = $_COOKIE['csrf_token'] ?? '';
    $formToken = $_POST['csrf_token'] ?? '';
    
    if (empty($cookieToken) || !hash_equals($cookieToken, $formToken)) {
        http_response_code(403);
        die('Invalid CSRF token');
    }
}
```

## SQL 注入

### 什么是 SQL 注入

- 通过构造恶意 SQL 语句，绕过应用程序的安全检查。

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

## 其他安全风险

### Broken Authentication

```php
<?php
declare(strict_types=1);

// 1. 使用强密码哈希
$hash = password_hash($password, PASSWORD_ARGON2ID);

// 2. 实现登录限制
class LoginLimiter
{
    private Redis $redis;
    
    public function checkLimit(string $ip, string $username): bool
    {
        $key = "login_attempts:{$ip}:{$username}";
        $attempts = (int) $this->redis->get($key);
        
        if ($attempts >= 5) {
            return false;  // 超过限制
        }
        
        return true;
    }
    
    public function recordAttempt(string $ip, string $username): void
    {
        $key = "login_attempts:{$ip}:{$username}";
        $this->redis->incr($key);
        $this->redis->expire($key, 900);  // 15 分钟
    }
}
```

### Security Misconfiguration

```php
<?php
// 1. 关闭错误显示
ini_set('display_errors', '0');
ini_set('log_errors', '1');

// 2. 隐藏服务器信息
header_remove('X-Powered-By');

// 3. 限制文件上传
ini_set('upload_max_filesize', '10M');
ini_set('post_max_size', '10M');
```

### SSRF（服务端请求伪造）

```php
<?php
declare(strict_types=1);

function safeFetchUrl(string $url): string
{
    // 验证 URL
    $parsed = parse_url($url);
    
    // 禁止内网地址
    $forbiddenHosts = ['127.0.0.1', 'localhost', '0.0.0.0', '10.0.0.0/8', '172.16.0.0/12', '192.168.0.0/16'];
    
    $host = $parsed['host'] ?? '';
    foreach ($forbiddenHosts as $forbidden) {
        if (strpos($host, $forbidden) === 0) {
            throw new InvalidArgumentException('Forbidden host');
        }
    }
    
    // 只允许 HTTP/HTTPS
    $scheme = $parsed['scheme'] ?? '';
    if (!in_array($scheme, ['http', 'https'], true)) {
        throw new InvalidArgumentException('Invalid scheme');
    }
    
    // 使用 cURL 并限制重定向
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, false);  // 禁止重定向
    curl_setopt($ch, CURLOPT_TIMEOUT, 5);
    
    $result = curl_exec($ch);
    curl_close($ch);
    
    return $result;
}
```

## 安全最佳实践

### 1. 输入验证

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
        $int = filter_var($value, FILTER_VALIDATE_INT);
        if ($int === false) {
            return null;
        }
        
        if ($min !== null && $int < $min) {
            return null;
        }
        
        if ($max !== null && $int > $max) {
            return null;
        }
        
        return $int;
    }
    
    public static function sanitizeString(string $input): string
    {
        return htmlspecialchars(trim($input), ENT_QUOTES, 'UTF-8');
    }
}
```

### 2. 输出转义

```php
<?php
// 始终转义输出
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');

// 在模板中
<?= htmlspecialchars($variable, ENT_QUOTES, 'UTF-8') ?>
```

### 3. 使用 HTTPS

```php
<?php
// 强制 HTTPS
if (!isset($_SERVER['HTTPS']) || $_SERVER['HTTPS'] !== 'on') {
    header('Location: https://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'], true, 301);
    exit;
}
```

### 4. 安全配置

```php
<?php
// php.ini 安全配置
ini_set('display_errors', '0');
ini_set('expose_php', '0');
ini_set('allow_url_fopen', '0');  // 禁止远程文件包含
ini_set('allow_url_include', '0');
```

## 安全检测工具

### 静态分析工具

#### Yama
- **说明**：基于精确操作码的数据流分析工具，用于检测 PHP 应用程序中的漏洞
- **用途**：检测 SQL 注入、XSS、代码注入等安全漏洞
- **特点**：精确的操作码分析，减少误报

#### PHUZZ
- **说明**：覆盖引导的模糊测试工具，用于发现 PHP Web 应用程序中的漏洞
- **用途**：自动化安全测试，发现未知漏洞
- **特点**：基于覆盖率引导，提高测试效率

### Composer 安全审计

```bash
# 检查依赖包的安全漏洞（Composer 2.4+）
composer audit

# JSON 格式输出
composer audit --format=json

# 在 CI/CD 中集成
# .github/workflows/security.yml
- name: Security Audit
  run: composer audit
```

### 集成到 CI/CD

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install
      - run: composer audit
      # 可以集成其他安全扫描工具
```

## 练习

1. 实现一个 CSRF 防护中间件，自动生成和验证 Token。

2. 创建一个 XSS 防护函数库，包含各种转义和清理方法。

3. 编写一个输入验证类，支持多种数据类型的验证和清理。

4. 实现一个 SSRF 防护函数，安全地获取远程 URL 内容。

5. 设计一个安全配置检查工具，验证应用的安全配置。

6. 创建一个安全审计日志系统，记录所有安全相关事件。

7. 集成 `composer audit` 到 CI/CD 流程，自动检测依赖漏洞。

8. 配置安全扫描工具（如 Yama、PHUZZ），定期检查应用安全。

# 5.11.2 请求签名与验证

## 概述

请求签名用于验证请求的完整性和真实性，防止请求被篡改或伪造。理解请求签名的概念、掌握签名算法、实现签名生成和验证，对于构建安全的 API 接口、支付接口、第三方集成等场景至关重要。本节详细介绍请求签名的概念、签名算法（HMAC-SHA256）、签名生成、签名验证、时间戳验证、重放攻击防护等内容，帮助零基础学员掌握请求签名技术。

请求签名是 API 安全的重要机制，通过签名可以验证请求是否来自合法的客户端，以及请求内容是否被篡改。理解签名的工作原理对于实现安全的 API 接口至关重要。

**主要内容**：
- 请求签名概念（什么是请求签名、签名的作用、签名算法选择）
- 签名算法（HMAC-SHA256、HMAC-SHA1、RSA 签名）
- 签名生成（请求数据排序、签名字符串构建、HMAC 签名、签名添加）
- 签名验证（签名提取、签名重新计算、签名比较、验证失败处理）
- 时间戳验证（时间戳添加、时间戳验证、时间窗口）
- 重放攻击防护（Nonce 机制、时间戳验证、请求唯一性）
- 实际应用示例和最佳实践

## 特性

- **完整性验证**：验证请求内容未被篡改
- **真实性验证**：验证请求来源的真实性
- **防重放**：防止请求被重复使用
- **标准算法**：使用标准签名算法
- **易于实现**：实现相对简单

## 请求签名概念

### 什么是请求签名

请求签名是对请求数据进行加密哈希处理，生成签名值，用于验证请求的完整性和真实性。

### 签名的作用

1. **完整性验证**：验证请求内容未被篡改
2. **真实性验证**：验证请求来自合法客户端
3. **防重放攻击**：结合时间戳和 Nonce 防止重放
4. **API 安全**：保护 API 接口安全

### 签名算法选择

**常用算法**：
- **HMAC-SHA256**：推荐，安全性高
- **HMAC-SHA1**：已不推荐，安全性较低
- **RSA**：非对称加密，适合公钥场景

## 签名算法

### HMAC-SHA256

**HMAC（Hash-based Message Authentication Code）**：基于哈希的消息认证码。

**特点**：
- 使用共享密钥
- 安全性高
- 性能好
- 易于实现

**示例**：
```php
<?php
declare(strict_types=1);

$data = 'request_data';
$secretKey = 'your-secret-key';
$signature = hash_hmac('sha256', $data, $secretKey);
```

### HMAC-SHA1

**示例**：
```php
<?php
declare(strict_types=1);

$signature = hash_hmac('sha1', $data, $secretKey);
```

**注意**：SHA1 已不推荐，建议使用 SHA256。

### RSA 签名

**示例**：
```php
<?php
declare(strict_types=1);

// 使用私钥签名
openssl_sign($data, $signature, $privateKey, OPENSSL_ALGO_SHA256);

// 使用公钥验证
$isValid = openssl_verify($data, $signature, $publicKey, OPENSSL_ALGO_SHA256) === 1;
```

## 签名生成

### 请求数据排序

**示例**：
```php
<?php
declare(strict_types=1);

function buildSignString(array $params): string
{
    // 移除签名参数
    unset($params['signature']);
    
    // 按键名排序
    ksort($params);
    
    // 构建签名字符串
    return http_build_query($params);
}
```

### 签名字符串构建

**示例**：
```php
<?php
declare(strict_types=1);

function buildSignString(array $params, string $method, string $path): string
{
    // 移除签名参数
    unset($params['signature']);
    unset($params['timestamp']);
    
    // 按键名排序
    ksort($params);
    
    // 构建签名字符串
    $parts = [
        $method,
        $path,
        http_build_query($params),
    ];
    
    return implode("\n", $parts);
}
```

### HMAC 签名

**示例**：
```php
<?php
declare(strict_types=1);

class RequestSigner
{
    private string $secretKey;

    public function __construct(string $secretKey)
    {
        $this->secretKey = $secretKey;
    }

    public function sign(array $params, string $method = 'POST', string $path = '/'): string
    {
        // 添加时间戳
        $params['timestamp'] = time();
        
        // 构建签名字符串
        $signString = $this->buildSignString($params, $method, $path);
        
        // 生成签名
        $signature = hash_hmac('sha256', $signString, $this->secretKey);
        
        return $signature;
    }

    private function buildSignString(array $params, string $method, string $path): string
    {
        unset($params['signature']);
        ksort($params);
        
        $parts = [
            $method,
            $path,
            http_build_query($params),
        ];
        
        return implode("\n", $parts);
    }
}
```

### 签名添加

**示例**：
```php
<?php
declare(strict_types=1);

// 生成签名并添加到请求
$signer = new RequestSigner($secretKey);
$params = ['key1' => 'value1', 'key2' => 'value2'];
$signature = $signer->sign($params, 'POST', '/api/endpoint');
$params['signature'] = $signature;

// 发送请求
$ch = curl_init('https://api.example.com/endpoint');
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => http_build_query($params),
    CURLOPT_HTTPHEADER => [
        'X-Signature: ' . $signature,
        'X-Timestamp: ' . $params['timestamp'],
    ],
]);
```

## 签名验证

### 签名提取

**示例**：
```php
<?php
declare(strict_types=1);

function extractSignature(): ?string
{
    // 从 Header 获取
    $signature = $_SERVER['HTTP_X_SIGNATURE'] ?? null;
    if ($signature !== null) {
        return $signature;
    }
    
    // 从请求参数获取
    return $_POST['signature'] ?? $_GET['signature'] ?? null;
}
```

### 签名重新计算

**示例**：
```php
<?php
declare(strict_types=1);

class RequestVerifier
{
    private string $secretKey;

    public function __construct(string $secretKey)
    {
        $this->secretKey = $secretKey;
    }

    public function verify(array $params, string $method = 'POST', string $path = '/'): bool
    {
        // 提取签名
        $receivedSignature = $params['signature'] ?? null;
        if ($receivedSignature === null) {
            return false;
        }
        
        // 重新计算签名
        $expectedSignature = $this->calculateSignature($params, $method, $path);
        
        // 安全比较签名
        return hash_equals($expectedSignature, $receivedSignature);
    }

    private function calculateSignature(array $params, string $method, string $path): string
    {
        $signString = $this->buildSignString($params, $method, $path);
        return hash_hmac('sha256', $signString, $this->secretKey);
    }

    private function buildSignString(array $params, string $method, string $path): string
    {
        unset($params['signature']);
        ksort($params);
        
        $parts = [
            $method,
            $path,
            http_build_query($params),
        ];
        
        return implode("\n", $parts);
    }
}
```

### 签名比较

**重要**：使用 `hash_equals()` 进行安全比较，防止时序攻击。

**示例**：
```php
<?php
declare(strict_types=1);

// 安全比较（推荐）
$isValid = hash_equals($expectedSignature, $receivedSignature);

// 不安全比较（不推荐）
$isValid = $expectedSignature === $receivedSignature;  // 可能受到时序攻击
```

### 验证失败处理

**示例**：
```php
<?php
declare(strict_types=1);

function verifyRequest(): void
{
    $verifier = new RequestVerifier($secretKey);
    
    $params = array_merge($_GET, $_POST);
    $method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
    $path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);
    
    if (!$verifier->verify($params, $method, $path)) {
        http_response_code(401);
        echo json_encode(['error' => 'Invalid signature']);
        exit;
    }
}
```

## 时间戳验证

### 时间戳添加

**示例**：
```php
<?php
declare(strict_types=1);

function signRequest(array $params): array
{
    // 添加时间戳
    $params['timestamp'] = time();
    
    // 生成签名
    $signature = generateSignature($params);
    $params['signature'] = $signature;
    
    return $params;
}
```

### 时间戳验证

**示例**：
```php
<?php
declare(strict_types=1);

function verifyTimestamp(array $params, int $window = 300): bool
{
    $timestamp = $params['timestamp'] ?? 0;
    $now = time();
    
    // 检查时间戳是否在有效窗口内（默认 5 分钟）
    if (abs($now - $timestamp) > $window) {
        return false;  // 时间戳过期或无效
    }
    
    return true;
}
```

### 时间窗口

**示例**：
```php
<?php
declare(strict_types=1);

class TimestampVerifier
{
    private int $window;

    public function __construct(int $window = 300)
    {
        $this->window = $window;  // 默认 5 分钟
    }

    public function verify(array $params): bool
    {
        $timestamp = $params['timestamp'] ?? 0;
        $now = time();
        
        // 检查时间戳是否在有效窗口内
        if (abs($now - $timestamp) > $this->window) {
            return false;
        }
        
        // 检查时间戳是否在未来（时钟偏差）
        if ($timestamp > $now + 60) {
            return false;  // 允许 1 分钟的时钟偏差
        }
        
        return true;
    }
}
```

## 重放攻击防护

### Nonce 机制

**Nonce（Number used once）**：一次性随机数，用于防止重放攻击。

**示例**：
```php
<?php
declare(strict_types=1);

class NonceManager
{
    private Redis $redis;

    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }

    public function generateNonce(): string
    {
        return bin2hex(random_bytes(16));
    }

    public function verifyNonce(string $nonce, int $ttl = 300): bool
    {
        $key = "nonce:{$nonce}";
        
        // 检查 Nonce 是否已使用
        if ($this->redis->exists($key)) {
            return false;  // Nonce 已使用
        }
        
        // 标记 Nonce 已使用
        $this->redis->setex($key, $ttl, '1');
        
        return true;
    }
}
```

### 时间戳验证

**示例**：
```php
<?php
declare(strict_types=1);

// 结合时间戳和 Nonce
function verifyRequest(array $params): bool
{
    // 1. 验证时间戳
    $timestampVerifier = new TimestampVerifier(300);
    if (!$timestampVerifier->verify($params)) {
        return false;
    }
    
    // 2. 验证 Nonce
    $nonceManager = new NonceManager($redis);
    $nonce = $params['nonce'] ?? '';
    if (!$nonceManager->verifyNonce($nonce)) {
        return false;
    }
    
    // 3. 验证签名
    $verifier = new RequestVerifier($secretKey);
    if (!$verifier->verify($params)) {
        return false;
    }
    
    return true;
}
```

### 请求唯一性

**示例**：
```php
<?php
declare(strict_types=1);

class RequestIdManager
{
    private Redis $redis;

    public function verifyRequestId(string $requestId, int $ttl = 300): bool
    {
        $key = "request_id:{$requestId}";
        
        // 检查请求 ID 是否已使用
        if ($this->redis->exists($key)) {
            return false;  // 请求 ID 已使用
        }
        
        // 标记请求 ID 已使用
        $this->redis->setex($key, $ttl, '1');
        
        return true;
    }
}

// 使用
function generateRequestId(): string
{
    return bin2hex(random_bytes(16)) . '_' . time();
}
```

## 使用场景

### API 安全

- API 接口签名验证
- 第三方 API 调用
- API 密钥管理

### 支付接口

- 支付请求签名
- 支付回调验证
- 支付安全

### 第三方集成

- 第三方服务集成
- Webhook 验证
- 回调验证

### 敏感操作

- 敏感操作签名
- 重要操作验证
- 操作审计

## 注意事项

### 密钥安全

- **安全存储**：安全存储密钥
- **密钥轮换**：定期轮换密钥
- **密钥分发**：安全分发密钥

### 时间戳验证

- **时间窗口**：设置合理的时间窗口
- **时钟同步**：考虑时钟偏差
- **时区处理**：正确处理时区

### 签名算法

- **使用 HMAC-SHA256**：推荐使用 HMAC-SHA256
- **避免弱算法**：避免使用 MD5、SHA1
- **算法一致性**：客户端和服务器使用相同算法

### 重放攻击

- **Nonce 机制**：使用 Nonce 防止重放
- **时间戳验证**：验证时间戳
- **请求唯一性**：确保请求唯一性

## 常见问题

### 如何生成请求签名？

1. 排序请求参数
2. 构建签名字符串
3. 使用 HMAC 生成签名
4. 添加签名到请求

### 如何验证签名？

1. 提取签名
2. 重新计算签名
3. 安全比较签名
4. 验证时间戳和 Nonce

### 如何防止重放攻击？

1. **时间戳验证**：验证时间戳在有效窗口内
2. **Nonce 机制**：使用 Nonce 确保请求唯一性
3. **请求 ID**：使用请求 ID 标记请求

### 签名算法的选择？

- **HMAC-SHA256**：推荐，安全性高
- **HMAC-SHA1**：不推荐，安全性较低
- **RSA**：适合公钥场景

## 最佳实践

### 使用 HMAC 算法

- 使用 HMAC-SHA256
- 使用共享密钥
- 安全存储密钥

### 验证时间戳

- 设置合理的时间窗口
- 验证时间戳有效性
- 考虑时钟偏差

### 安全存储密钥

- 使用环境变量存储密钥
- 使用密钥管理服务
- 定期轮换密钥

### 实现重放攻击防护

- 使用 Nonce 机制
- 验证时间戳
- 确保请求唯一性

## 相关章节

- **[5.11.1 Rate Limiting](section-01-rate-limiting.md)**：了解 Rate Limiting 的详细内容
- **[5.11.3 安全最佳实践](section-03-security-best-practices.md)**：了解安全最佳实践的详细内容

## 练习任务

1. **实现请求签名类**
   - 签名生成
   - 签名验证
   - 时间戳验证

2. **实现重放攻击防护**
   - Nonce 机制
   - 时间戳验证
   - 请求唯一性

3. **实现完整的签名验证系统**
   - 签名生成和验证
   - 时间戳和 Nonce 验证
   - 错误处理

4. **实现密钥管理**
   - 密钥存储
   - 密钥轮换
   - 密钥分发

5. **实现完整的 API 安全系统**
   - 请求签名
   - Rate Limiting
   - 安全配置

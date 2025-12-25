# 4.10.2 请求签名与验证

## 概述

请求签名是验证请求完整性和防止篡改的重要机制。本节详细介绍请求签名原理、HMAC 签名、时间戳验证、签名验证中间件，以及完整实现。

## 请求签名原理

### 签名流程

```
1. 客户端构造请求
   ↓
2. 生成签名（HMAC）
   ↓
3. 将签名添加到请求头
   ↓
4. 服务器验证签名
   ↓
5. 允许/拒绝请求
```

### 签名组成

```
签名 = HMAC(请求方法 + 请求路径 + 查询参数 + 请求体 + 时间戳, 密钥)
```

## HMAC 签名

### 客户端签名

```php
<?php
declare(strict_types=1);

class RequestSigner
{
    private string $secret;

    public function __construct(string $secret)
    {
        $this->secret = $secret;
    }

    public function sign(string $method, string $path, string $body, int $timestamp): string
    {
        $message = "{$method}\n{$path}\n{$body}\n{$timestamp}";
        return hash_hmac('sha256', $message, $this->secret);
    }

    public function generateHeaders(string $method, string $path, string $body): array
    {
        $timestamp = time();
        $signature = $this->sign($method, $path, $body, $timestamp);
        
        return [
            'X-Timestamp' => $timestamp,
            'X-Signature' => $signature,
        ];
    }
}
```

### 服务器验证

```php
<?php
declare(strict_types=1);

class RequestVerifier
{
    private string $secret;
    private int $timestampTolerance = 300;  // 5 分钟

    public function verify(array $request): bool
    {
        $method = $request['REQUEST_METHOD'];
        $path = parse_url($request['REQUEST_URI'], PHP_URL_PATH);
        $body = file_get_contents('php://input');
        $timestamp = (int) ($request['HTTP_X_TIMESTAMP'] ?? 0);
        $signature = $request['HTTP_X_SIGNATURE'] ?? '';

        // 验证时间戳
        if (abs(time() - $timestamp) > $this->timestampTolerance) {
            return false;
        }

        // 验证签名
        $expectedSignature = hash_hmac('sha256', "{$method}\n{$path}\n{$body}\n{$timestamp}", $this->secret);
        
        return hash_equals($expectedSignature, $signature);
    }
}
```

## 时间戳验证

### 防重放攻击

```php
<?php
declare(strict_types=1);

class TimestampValidator
{
    private \Redis $redis;
    private int $tolerance = 300;  // 5 分钟

    public function validate(int $timestamp, string $nonce): bool
    {
        // 检查时间戳
        if (abs(time() - $timestamp) > $this->tolerance) {
            return false;
        }

        // 检查 nonce 是否已使用（防重放）
        $nonceKey = "nonce:{$nonce}";
        if ($this->redis->exists($nonceKey)) {
            return false;
        }

        // 记录 nonce
        $this->redis->setex($nonceKey, $this->tolerance * 2, '1');
        
        return true;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SignatureMiddleware
{
    private RequestVerifier $verifier;

    public function handle(): void
    {
        if (!$this->verifier->verify($_SERVER)) {
            http_response_code(401);
            echo json_encode(['error' => 'Invalid signature']);
            exit;
        }
    }
}
```

## 注意事项

1. **密钥安全**：妥善保管签名密钥
2. **时间戳验证**：防止重放攻击
3. **Nonce 机制**：使用 nonce 防止重复请求
4. **HTTPS**：生产环境必须使用 HTTPS

## 练习

1. 实现请求签名生成和验证功能。

2. 创建一个签名验证中间件，自动验证请求签名。

3. 实现时间戳和 nonce 验证，防止重放攻击。

4. 设计一个请求签名系统，支持多种签名算法。

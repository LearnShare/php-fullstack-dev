# 4.10.3 安全最佳实践

## 概述

API 安全需要多层次的防护。本节详细介绍幂等性机制、防重放攻击、请求验证、安全配置，以及完整的安全实践。

## 幂等性机制

### 实现幂等性

```php
<?php
declare(strict_types=1);

class IdempotencyMiddleware
{
    private \Redis $redis;

    public function handle(string $idempotencyKey): mixed
    {
        // 检查是否已处理
        $cached = $this->redis->get("idempotency:{$idempotencyKey}");
        if ($cached !== false) {
            return json_decode($cached, true);
        }

        // 处理请求
        $result = $this->processRequest();

        // 缓存结果（24 小时）
        $this->redis->setex("idempotency:{$idempotencyKey}", 86400, json_encode($result));

        return $result;
    }
}
```

## 防重放攻击

### Nonce 机制

```php
<?php
declare(strict_types=1);

class ReplayProtection
{
    private \Redis $redis;

    public function validate(string $nonce, int $timestamp): bool
    {
        // 检查时间戳
        if (abs(time() - $timestamp) > 300) {
            return false;
        }

        // 检查 nonce 是否已使用
        $key = "nonce:{$nonce}";
        if ($this->redis->exists($key)) {
            return false;
        }

        // 记录 nonce
        $this->redis->setex($key, 600, '1');
        
        return true;
    }
}
```

## 请求验证

### 完整验证流程

```php
<?php
declare(strict_types=1);

class SecurityMiddleware
{
    public function handle(): void
    {
        // 1. 验证签名
        if (!$this->verifySignature()) {
            $this->reject('Invalid signature');
        }

        // 2. 验证时间戳
        if (!$this->verifyTimestamp()) {
            $this->reject('Invalid timestamp');
        }

        // 3. 验证 nonce
        if (!$this->verifyNonce()) {
            $this->reject('Replay attack detected');
        }

        // 4. 限流检查
        if (!$this->checkRateLimit()) {
            $this->reject('Rate limit exceeded');
        }
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ApiSecurity
{
    public function validateRequest(): bool
    {
        return $this->verifySignature()
            && $this->verifyTimestamp()
            && $this->verifyNonce()
            && $this->checkRateLimit();
    }
}
```

## 注意事项

1. **多层防护**：使用多种安全机制
2. **日志记录**：记录安全事件
3. **监控告警**：监控异常请求
4. **定期审查**：定期审查安全配置

## 练习

1. 实现幂等性机制，防止重复操作。

2. 创建防重放攻击系统，使用 nonce 和时间戳。

3. 实现完整的安全中间件，包含所有验证步骤。

4. 设计一个安全监控系统，记录和告警安全事件。

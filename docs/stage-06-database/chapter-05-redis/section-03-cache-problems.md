# 6.5.3 缓存问题与解决方案

## 概述

在使用缓存系统时，可能会遇到各种问题，如缓存穿透、缓存击穿、缓存雪崩等。这些问题如果不及时解决，可能导致数据库压力过大、系统性能下降甚至系统崩溃。理解这些问题的原因和解决方案对于构建稳定可靠的缓存系统至关重要。

本节详细介绍常见的缓存问题（缓存穿透、缓存击穿、缓存雪崩、缓存一致性等）及其解决方案，帮助零基础学员识别和解决缓存问题。

**主要内容**：
- 缓存穿透问题与解决方案
- 缓存击穿问题与解决方案
- 缓存雪崩问题与解决方案
- 缓存一致性问题与解决方案
- 缓存预热和刷新
- 完整示例和最佳实践

---

## 特性

- **问题识别**：识别常见的缓存问题
- **解决方案**：提供有效的解决方案
- **性能优化**：优化缓存性能
- **稳定性提升**：提高系统稳定性
- **最佳实践**：提供最佳实践指导

---

## 缓存穿透

### 什么是缓存穿透

缓存穿透是指查询一个不存在的数据，缓存和数据库都没有，导致每次查询都要访问数据库。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 查询不存在的用户 ID
$user = $cache->get("user:99999", function () use ($pdo) {
    // 缓存未命中，查询数据库
    $stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
    $stmt->execute(['id' => 99999]);
    $user = $stmt->fetch();
    // 数据库也没有，返回 null
    return $user;
});

// 如果大量请求查询不存在的 ID，会导致数据库压力过大
```

**影响**：

- 数据库压力过大
- 系统性能下降
- 可能被恶意攻击利用

### 解决方案

**1. 布隆过滤器（Bloom Filter）**

使用布隆过滤器判断数据是否存在。

**示例**：

```php
<?php
declare(strict_types=1);

class CacheService
{
    private Redis $redis;
    private PDO $pdo;
    
    public function getUser(int $id): ?array
    {
        $cacheKey = "user:{$id}";
        
        // 1. 使用布隆过滤器判断
        if (!$this->bloomFilter->exists("users", $id)) {
            return null;  // 不存在，直接返回
        }
        
        // 2. 查缓存
        $cached = $this->redis->get($cacheKey);
        if ($cached !== false) {
            return unserialize($cached);
        }
        
        // 3. 查数据库
        $user = $this->getUserFromDb($id);
        
        if ($user) {
            $this->redis->setex($cacheKey, 3600, serialize($user));
        } else {
            // 4. 缓存空值，防止穿透
            $this->redis->setex($cacheKey, 60, serialize(null));
        }
        
        return $user;
    }
}
```

**2. 缓存空值**

对于不存在的数据，缓存空值，设置较短的过期时间。

**示例**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        // 如果是空值标记，直接返回 null
        if ($cached === 'NULL') {
            return null;
        }
        return unserialize($cached);
    }
    
    $user = $this->getUserFromDb($id);
    
    if ($user) {
        $this->redis->setex($cacheKey, 3600, serialize($user));
    } else {
        // 缓存空值，设置较短的过期时间
        $this->redis->setex($cacheKey, 60, 'NULL');
    }
    
    return $user;
}
```

**3. 参数验证**

在应用层进行参数验证，过滤无效请求。

**示例**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    // 参数验证
    if ($id <= 0 || $id > 1000000) {
        return null;  // 无效 ID，直接返回
    }
    
    // 继续查询...
}
```

---

## 缓存击穿

### 什么是缓存击穿

缓存击穿是指热点数据缓存过期，大量请求同时访问数据库。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 热点数据缓存过期
// 大量请求同时查询数据库
for ($i = 0; $i < 1000; $i++) {
    $user = $cache->get("user:1", function () use ($pdo) {
        // 缓存过期，所有请求都查询数据库
        return $this->getUserFromDb(1);
    });
}
```

**影响**：

- 数据库压力瞬间增大
- 系统性能下降
- 可能导致数据库崩溃

### 解决方案

**1. 互斥锁（Mutex Lock）**

使用互斥锁，只允许一个请求查询数据库。

**示例**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    $lockKey = "lock:user:{$id}";
    
    // 1. 查缓存
    $cached = $this->redis->get($cacheKey);
    if ($cached !== false) {
        return unserialize($cached);
    }
    
    // 2. 尝试获取锁
    $locked = $this->redis->set($lockKey, '1', ['nx', 'ex' => 10]);
    
    if ($locked) {
        try {
            // 3. 获取锁成功，查询数据库
            $user = $this->getUserFromDb($id);
            
            if ($user) {
                $this->redis->setex($cacheKey, 3600, serialize($user));
            }
            
            return $user;
        } finally {
            // 4. 释放锁
            $this->redis->del($lockKey);
        }
    } else {
        // 5. 获取锁失败，等待并重试
        usleep(100000);  // 等待 100ms
        return $this->getUser($id);  // 重试
    }
}
```

**2. 永不过期 + 异步更新**

热点数据设置永不过期，后台异步更新。

**示例**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        $data = unserialize($cached);
        
        // 检查是否需要更新
        if (time() - $data['_updated_at'] > 3600) {
            // 异步更新
            $this->updateCacheAsync($id);
        }
        
        return $data['user'];
    }
    
    // 缓存未命中，查询数据库
    $user = $this->getUserFromDb($id);
    
    if ($user) {
        $this->redis->set($cacheKey, serialize([
            'user' => $user,
            '_updated_at' => time()
        ]));
    }
    
    return $user;
}
```

**3. 多级缓存**

使用多级缓存，减少数据库访问。

**示例**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    // 1. 查本地缓存
    $localCache = $this->localCache->get("user:{$id}");
    if ($localCache !== null) {
        return $localCache;
    }
    
    // 2. 查 Redis 缓存
    $redisCache = $this->redis->get("user:{$id}");
    if ($redisCache !== false) {
        $user = unserialize($redisCache);
        $this->localCache->set("user:{$id}", $user, 60);
        return $user;
    }
    
    // 3. 查数据库
    $user = $this->getUserFromDb($id);
    
    if ($user) {
        $this->redis->setex("user:{$id}", 3600, serialize($user));
        $this->localCache->set("user:{$id}", $user, 60);
    }
    
    return $user;
}
```

---

## 缓存雪崩

### 什么是缓存雪崩

缓存雪崩是指大量缓存同时过期，导致大量请求同时访问数据库。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 大量缓存同时过期
// 所有请求同时查询数据库
for ($i = 1; $i <= 1000; $i++) {
    $cache->set("user:{$i}", $user, 3600);  // 同时设置 3600 秒过期
}

// 3600 秒后，所有缓存同时过期
// 大量请求同时查询数据库
```

**影响**：

- 数据库压力瞬间增大
- 系统性能急剧下降
- 可能导致系统崩溃

### 解决方案

**1. 随机过期时间**

为缓存设置随机的过期时间，避免同时过期。

**示例**：

```php
<?php
declare(strict_types=1);

public function setCache(string $key, mixed $value, int $baseTtl = 3600): void
{
    // 基础过期时间 + 随机时间（0-600秒）
    $ttl = $baseTtl + rand(0, 600);
    $this->redis->setex($key, $ttl, serialize($value));
}
```

**2. 缓存预热**

在缓存过期前提前刷新缓存。

**示例**：

```php
<?php
declare(strict_types=1);

class CacheRefresh
{
    private Redis $redis;
    private PDO $pdo;
    
    public function refreshCache(int $id): void
    {
        $cacheKey = "user:{$id}";
        $ttl = $this->redis->ttl($cacheKey);
        
        // 如果剩余时间少于 10%，提前刷新
        if ($ttl > 0 && $ttl < 360) {
            $user = $this->getUserFromDb($id);
            if ($user) {
                $this->redis->setex($cacheKey, 3600, serialize($user));
            }
        }
    }
    
    public function refreshAll(): void
    {
        // 定期刷新所有热点数据
        $hotUserIds = [1, 2, 3, 4, 5];
        
        foreach ($hotUserIds as $id) {
            $this->refreshCache($id);
        }
    }
}
```

**3. 多级缓存**

使用多级缓存，减少缓存雪崩的影响。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用本地缓存 + Redis 缓存
// 即使 Redis 缓存失效，本地缓存仍然有效
```

**4. 限流和降级**

在缓存失效时，使用限流和降级策略。

**示例**：

```php
<?php
declare(strict_types=1);

class CacheService
{
    private int $requestCount = 0;
    private int $lastResetTime = 0;
    
    public function getUser(int $id): ?array
    {
        // 限流：每秒最多 100 个请求
        $currentTime = time();
        if ($currentTime !== $this->lastResetTime) {
            $this->requestCount = 0;
            $this->lastResetTime = $currentTime;
        }
        
        if ($this->requestCount >= 100) {
            // 降级：返回默认值或错误
            return null;
        }
        
        $this->requestCount++;
        
        // 继续查询...
    }
}
```

---

## 缓存一致性

### 缓存一致性问题

缓存一致性问题是指缓存数据与数据库数据不一致。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 场景 1：更新数据库，忘记更新缓存
$pdo->prepare('UPDATE users SET name = :name WHERE id = :id')->execute(['id' => 1, 'name' => 'Jane']);
// 缓存仍然是旧数据

// 场景 2：更新缓存，数据库更新失败
$redis->set("user:1", serialize(['name' => 'Jane']));
$pdo->prepare('UPDATE users SET name = :name WHERE id = :id')->execute(['id' => 1, 'name' => 'Jane']);
// 如果数据库更新失败，缓存和数据库不一致
```

### 解决方案

**1. 更新时删除缓存**

更新数据时删除缓存，下次读取时重新加载。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 1. 更新数据库
    $result = $this->updateUserInDb($id, $data);
    
    if ($result) {
        // 2. 删除缓存
        $this->redis->del("user:{$id}");
    }
    
    return $result;
}
```

**2. 更新时更新缓存**

更新数据时同时更新缓存。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 1. 更新数据库
    $result = $this->updateUserInDb($id, $data);
    
    if ($result) {
        // 2. 更新缓存
        $user = $this->getUserFromDb($id);
        $this->redis->setex("user:{$id}", 3600, serialize($user));
    }
    
    return $result;
}
```

**3. 延迟双删**

更新数据时先删除缓存，更新数据库，延迟一段时间后再删除缓存。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 1. 先删除缓存
    $this->redis->del("user:{$id}");
    
    // 2. 更新数据库
    $result = $this->updateUserInDb($id, $data);
    
    if ($result) {
        // 3. 延迟删除缓存（异步）
        $this->scheduleDelayedDelete("user:{$id}", 1);
    }
    
    return $result;
}
```

**4. 使用消息队列**

使用消息队列异步更新缓存。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 1. 更新数据库
    $result = $this->updateUserInDb($id, $data);
    
    if ($result) {
        // 2. 发送消息到队列
        $this->queue->push('cache:invalidate', [
            'key' => "user:{$id}"
        ]);
    }
    
    return $result;
}

// 消费者处理消息
public function handleCacheInvalidate(string $key): void
{
    $this->redis->del($key);
}
```

---

## 完整示例

### 缓存服务（解决所有问题）

```php
<?php
declare(strict_types=1);

class CacheService
{
    private Redis $redis;
    private PDO $pdo;
    
    public function getUser(int $id): ?array
    {
        $cacheKey = "user:{$id}";
        $lockKey = "lock:user:{$id}";
        
        // 1. 参数验证（防止缓存穿透）
        if ($id <= 0) {
            return null;
        }
        
        // 2. 查缓存
        $cached = $this->redis->get($cacheKey);
        if ($cached !== false) {
            if ($cached === 'NULL') {
                return null;  // 空值标记
            }
            return unserialize($cached);
        }
        
        // 3. 尝试获取锁（防止缓存击穿）
        $locked = $this->redis->set($lockKey, '1', ['nx', 'ex' => 10]);
        
        if ($locked) {
            try {
                // 4. 查询数据库
                $user = $this->getUserFromDb($id);
                
                if ($user) {
                    // 5. 写入缓存（随机过期时间，防止缓存雪崩）
                    $ttl = 3600 + rand(0, 600);
                    $this->redis->setex($cacheKey, $ttl, serialize($user));
                } else {
                    // 6. 缓存空值（防止缓存穿透）
                    $this->redis->setex($cacheKey, 60, 'NULL');
                }
                
                return $user;
            } finally {
                // 7. 释放锁
                $this->redis->del($lockKey);
            }
        } else {
            // 8. 获取锁失败，等待并重试
            usleep(100000);
            return $this->getUser($id);
        }
    }
    
    public function updateUser(int $id, array $data): bool
    {
        // 1. 先删除缓存（保证一致性）
        $this->redis->del("user:{$id}");
        
        // 2. 更新数据库
        $result = $this->updateUserInDb($id, $data);
        
        if ($result) {
            // 3. 延迟删除缓存（延迟双删）
            $this->scheduleDelayedDelete("user:{$id}", 1);
        }
        
        return $result;
    }
    
    private function scheduleDelayedDelete(string $key, int $delay): void
    {
        // 使用延迟任务删除缓存
        $this->redis->setex("delayed:delete:{$key}", $delay, '1');
    }
}
```

---

## 使用场景

### 高并发场景

在高并发场景下，缓存问题更容易出现，需要特别注意。

### 热点数据

热点数据更容易出现缓存击穿和缓存雪崩。

### 数据一致性要求高

数据一致性要求高的场景，需要特别注意缓存一致性。

---

## 注意事项

### 缓存穿透

防止缓存穿透，使用布隆过滤器或缓存空值。

### 缓存击穿

防止缓存击穿，使用互斥锁或永不过期策略。

### 缓存雪崩

防止缓存雪崩，使用随机过期时间或缓存预热。

### 缓存一致性

维护缓存一致性，使用合适的更新策略。

---

## 常见问题

### 什么是缓存穿透？

查询不存在的数据，缓存和数据库都没有，导致每次查询都要访问数据库。

### 什么是缓存击穿？

热点数据缓存过期，大量请求同时访问数据库。

### 什么是缓存雪崩？

大量缓存同时过期，导致大量请求同时访问数据库。

### 如何保证缓存一致性？

- 更新时删除缓存
- 更新时更新缓存
- 使用延迟双删
- 使用消息队列

---

## 最佳实践

### 防止缓存穿透

- 使用布隆过滤器
- 缓存空值
- 参数验证

### 防止缓存击穿

- 使用互斥锁
- 永不过期 + 异步更新
- 多级缓存

### 防止缓存雪崩

- 随机过期时间
- 缓存预热
- 多级缓存
- 限流和降级

### 保证缓存一致性

- 更新时删除缓存
- 更新时更新缓存
- 使用延迟双删
- 使用消息队列

---

## 练习任务

1. **缓存穿透解决**
   - 实现布隆过滤器
   - 实现空值缓存
   - 测试穿透防护
   - 验证效果

2. **缓存击穿解决**
   - 实现互斥锁
   - 实现永不过期策略
   - 测试击穿防护
   - 验证效果

3. **缓存雪崩解决**
   - 实现随机过期时间
   - 实现缓存预热
   - 测试雪崩防护
   - 验证效果

4. **缓存一致性**
   - 实现更新时删除缓存
   - 实现延迟双删
   - 测试一致性
   - 验证效果

5. **综合应用**
   - 创建一个完整的缓存服务
   - 解决所有缓存问题
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.5.1 Redis 基础](section-01-redis-basics.md)
- [6.5.2 缓存策略](section-02-cache-strategies.md)
- [6.5.4 高级特性](section-04-advanced-features.md)

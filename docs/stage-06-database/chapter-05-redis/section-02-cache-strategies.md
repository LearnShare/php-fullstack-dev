# 6.5.2 缓存策略

## 概述

缓存策略是使用缓存系统的关键，合理的缓存策略可以显著提高应用性能，减少数据库负载。不同的业务场景需要不同的缓存策略，理解各种缓存策略的特点和适用场景对于优化应用性能至关重要。

本节详细介绍各种缓存策略，包括缓存模式（Cache-Aside、Read-Through、Write-Through、Write-Behind）、缓存更新策略、缓存失效策略、缓存预热等，帮助零基础学员掌握缓存策略的设计和实现。

**主要内容**：
- 缓存策略概述
- 缓存模式（Cache-Aside、Read-Through、Write-Through、Write-Behind）
- 缓存更新策略
- 缓存失效策略
- 缓存预热和刷新
- 完整示例和最佳实践

---

## 特性

- **性能优化**：通过缓存减少数据库访问
- **负载降低**：降低数据库负载
- **响应加速**：提高应用响应速度
- **策略灵活**：支持多种缓存策略
- **可配置**：支持灵活的配置和调整

---

## 缓存策略概述

### 什么是缓存策略

缓存策略是指在使用缓存系统时的数据读写策略，包括何时读取缓存、何时更新缓存、何时失效缓存等。

**缓存策略的目标**：

- **提高性能**：减少数据库访问，提高响应速度
- **保证一致性**：保证缓存数据与数据库数据的一致性
- **降低负载**：降低数据库负载
- **优化成本**：在性能和成本之间找到平衡

### 缓存策略的重要性

合理的缓存策略可以：

- **显著提高性能**：减少数据库查询，提高响应速度
- **降低系统负载**：减少数据库压力
- **提高用户体验**：快速响应，提升用户体验
- **降低成本**：减少数据库资源消耗

---

## 缓存模式

### Cache-Aside（旁路缓存）

Cache-Aside 是最常用的缓存模式，应用负责维护缓存。

**工作流程**：

1. **读取**：先查缓存，缓存未命中则查数据库，然后写入缓存
2. **写入**：直接写入数据库，然后删除缓存

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    private PDO $pdo;
    private Redis $redis;
    
    public function getUser(int $id): ?array
    {
        // 1. 先查缓存
        $cacheKey = "user:{$id}";
        $cached = $this->redis->get($cacheKey);
        
        if ($cached !== false) {
            return unserialize($cached);
        }
        
        // 2. 缓存未命中，查数据库
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($user) {
            // 3. 写入缓存
            $this->redis->setex($cacheKey, 3600, serialize($user));
        }
        
        return $user;
    }
    
    public function updateUser(int $id, array $data): bool
    {
        // 1. 更新数据库
        $stmt = $this->pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
        $result = $stmt->execute(['id' => $id, 'name' => $data['name']]);
        
        if ($result) {
            // 2. 删除缓存
            $this->redis->del("user:{$id}");
        }
        
        return $result;
    }
}
```

**特点**：

- **简单**：实现简单，易于理解
- **灵活**：应用完全控制缓存
- **一致性**：需要手动维护一致性

### Read-Through（读穿透）

Read-Through 模式中，缓存系统负责从数据库加载数据。

**工作流程**：

1. **读取**：应用只查缓存，缓存未命中时缓存系统自动从数据库加载
2. **写入**：直接写入数据库，然后删除缓存

**示例**：

```php
<?php
declare(strict_types=1);

class CacheService
{
    private PDO $pdo;
    private Redis $redis;
    
    public function get(string $key, callable $loader, int $ttl = 3600): mixed
    {
        // 1. 先查缓存
        $cached = $this->redis->get($key);
        
        if ($cached !== false) {
            return unserialize($cached);
        }
        
        // 2. 缓存未命中，从数据库加载
        $value = $loader();
        
        if ($value !== null) {
            // 3. 写入缓存
            $this->redis->setex($key, $ttl, serialize($value));
        }
        
        return $value;
    }
}

// 使用
$cache = new CacheService($pdo, $redis);

$user = $cache->get("user:1", function () use ($pdo) {
    $stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
    $stmt->execute(['id' => 1]);
    return $stmt->fetch(PDO::FETCH_ASSOC);
}, 3600);
```

### Write-Through（写穿透）

Write-Through 模式中，写入操作同时更新缓存和数据库。

**工作流程**：

1. **写入**：同时写入缓存和数据库
2. **读取**：只查缓存

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    private PDO $pdo;
    private Redis $redis;
    
    public function updateUser(int $id, array $data): bool
    {
        // 1. 更新数据库
        $stmt = $this->pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
        $result = $stmt->execute(['id' => $id, 'name' => $data['name']]);
        
        if ($result) {
            // 2. 更新缓存
            $user = $this->getUserFromDb($id);
            $this->redis->setex("user:{$id}", 3600, serialize($user));
        }
        
        return $result;
    }
}
```

### Write-Behind（写回）

Write-Behind 模式中，写入操作先更新缓存，然后异步写入数据库。

**工作流程**：

1. **写入**：先更新缓存，立即返回，异步写入数据库
2. **读取**：只查缓存

**特点**：

- **高性能**：写入操作立即返回
- **风险**：数据可能丢失
- **复杂**：实现复杂

---

## 缓存更新策略

### 更新时删除缓存

更新数据时删除缓存，下次读取时重新加载。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 更新数据库
    $stmt = $this->pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
    $result = $stmt->execute(['id' => $id, 'name' => $data['name']]);
    
    if ($result) {
        // 删除缓存
        $this->redis->del("user:{$id}");
    }
    
    return $result;
}
```

### 更新时更新缓存

更新数据时同时更新缓存。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 更新数据库
    $stmt = $this->pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
    $result = $stmt->execute(['id' => $id, 'name' => $data['name']]);
    
    if ($result) {
        // 更新缓存
        $user = $this->getUserFromDb($id);
        $this->redis->setex("user:{$id}", 3600, serialize($user));
    }
    
    return $result;
}
```

### 延迟双删

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
    $stmt = $this->pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
    $result = $stmt->execute(['id' => $id, 'name' => $data['name']]);
    
    if ($result) {
        // 3. 延迟删除缓存（异步）
        $this->redis->setex("user:{$id}:delete", 1, '1');
    }
    
    return $result;
}
```

---

## 缓存失效策略

### 定时过期

设置固定的过期时间。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置 1 小时过期
$this->redis->setex("user:{$id}", 3600, serialize($user));
```

### 滑动过期

每次访问时更新过期时间。

**示例**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        // 更新过期时间
        $this->redis->expire($cacheKey, 3600);
        return unserialize($cached);
    }
    
    // 从数据库加载...
}
```

### 主动失效

根据业务逻辑主动失效缓存。

**示例**：

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): bool
{
    // 更新数据库
    $result = $this->updateUserInDb($id, $data);
    
    if ($result) {
        // 主动失效相关缓存
        $this->redis->del("user:{$id}");
        $this->redis->del("user:{$id}:posts");
        $this->redis->del("users:list");
    }
    
    return $result;
}
```

---

## 缓存预热

### 什么是缓存预热

缓存预热是在应用启动或低峰期预先加载热点数据到缓存。

**目的**：

- 提高启动后的性能
- 避免冷启动问题
- 提前加载热点数据

**示例**：

```php
<?php
declare(strict_types=1);

class CacheWarmup
{
    private PDO $pdo;
    private Redis $redis;
    
    public function warmup(): void
    {
        // 预热热点用户
        $stmt = $this->pdo->query('SELECT * FROM users WHERE status = "active" LIMIT 100');
        $users = $stmt->fetchAll(PDO::FETCH_ASSOC);
        
        foreach ($users as $user) {
            $this->redis->setex(
                "user:{$user['id']}",
                3600,
                serialize($user)
            );
        }
        
        // 预热热门文章
        $stmt = $this->pdo->query('SELECT * FROM posts WHERE status = "published" ORDER BY views DESC LIMIT 50');
        $posts = $stmt->fetchAll(PDO::FETCH_ASSOC);
        
        foreach ($posts as $post) {
            $this->redis->setex(
                "post:{$post['id']}",
                3600,
                serialize($post)
            );
        }
    }
}
```

---

## 完整示例

### 缓存服务实现

```php
<?php
declare(strict_types=1);

class CacheService
{
    private Redis $redis;
    private PDO $pdo;
    
    public function __construct(Redis $redis, PDO $pdo)
    {
        $this->redis = $redis;
        $this->pdo = $pdo;
    }
    
    // Cache-Aside 模式
    public function get(string $key, callable $loader, int $ttl = 3600): mixed
    {
        // 1. 先查缓存
        $cached = $this->redis->get($key);
        
        if ($cached !== false) {
            return unserialize($cached);
        }
        
        // 2. 缓存未命中，从数据库加载
        $value = $loader();
        
        if ($value !== null) {
            // 3. 写入缓存
            $this->redis->setex($key, $ttl, serialize($value));
        }
        
        return $value;
    }
    
    // 更新时删除缓存
    public function invalidate(string $key): void
    {
        $this->redis->del($key);
    }
    
    // 更新时更新缓存
    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $this->redis->setex($key, $ttl, serialize($value));
    }
    
    // 批量失效
    public function invalidatePattern(string $pattern): void
    {
        $keys = $this->redis->keys($pattern);
        if (!empty($keys)) {
            $this->redis->del(...$keys);
        }
    }
}

// 使用
$cache = new CacheService($redis, $pdo);

// 读取（Cache-Aside）
$user = $cache->get("user:1", function () use ($pdo) {
    $stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
    $stmt->execute(['id' => 1]);
    return $stmt->fetch(PDO::FETCH_ASSOC);
}, 3600);

// 更新时删除缓存
$cache->invalidate("user:1");
```

---

## 使用场景

### 读多写少

Cache-Aside 模式适合读多写少的场景。

### 写多读少

Write-Through 模式适合写多读少的场景。

### 高一致性要求

Write-Through 模式适合高一致性要求的场景。

---

## 注意事项

### 缓存一致性

注意维护缓存与数据库的一致性。

### 缓存穿透

防止缓存穿透（查询不存在的数据）。

### 缓存雪崩

防止缓存雪崩（大量缓存同时失效）。

---

## 常见问题

### 如何选择缓存策略？

根据业务场景选择：

- **读多写少**：Cache-Aside
- **写多读少**：Write-Through
- **高一致性**：Write-Through
- **高性能写入**：Write-Behind

### 如何保证缓存一致性？

- 更新时删除缓存
- 更新时更新缓存
- 使用延迟双删

---

## 最佳实践

### 选择合适的策略

根据业务场景选择合适的缓存策略。

### 维护一致性

注意维护缓存与数据库的一致性。

### 防止缓存问题

防止缓存穿透、缓存雪崩等问题。

---

## 练习任务

1. **实现缓存策略**
   - 实现 Cache-Aside 模式
   - 实现 Write-Through 模式
   - 测试不同策略
   - 对比性能差异

2. **缓存更新策略**
   - 实现更新时删除缓存
   - 实现更新时更新缓存
   - 实现延迟双删
   - 测试一致性

3. **缓存失效策略**
   - 实现定时过期
   - 实现滑动过期
   - 实现主动失效
   - 测试失效效果

4. **缓存预热**
   - 实现缓存预热
   - 测试预热效果
   - 优化预热策略
   - 编写预热脚本

5. **综合应用**
   - 创建一个完整的缓存系统
   - 实现多种缓存策略
   - 优化缓存性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.5.1 Redis 基础](section-01-redis-basics.md)
- [6.5.3 缓存问题与解决方案](section-03-cache-problems.md)
- [6.5.4 高级特性](section-04-advanced-features.md)

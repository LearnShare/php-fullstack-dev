# 5.7 Redis 缓存策略与性能优化

## 目标

- 理解缓存的作用与使用场景。
- 掌握常见的缓存模式（Cache-Aside、Write-Through、Write-Back）。
- 了解缓存一致性问题及解决方案。
- 掌握缓存穿透、击穿、雪崩的处理方法。
- 熟悉 Redis 分布式锁的实现。

## Redis 基础

### 连接 Redis

```php
<?php
declare(strict_types=1);

// 使用 phpredis 扩展
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->auth('password');  // 如果需要密码

// 使用 Predis 库
require __DIR__ . '/vendor/autoload.php';
use Predis\Client;

$redis = new Client([
    'scheme' => 'tcp',
    'host' => '127.0.0.1',
    'port' => 6379,
    'password' => 'password',
]);
```

### 基本操作

```php
<?php
declare(strict_types=1);

// 字符串操作
$redis->set('key', 'value');
$redis->set('key', 'value', 3600);  // 设置过期时间（秒）
$value = $redis->get('key');
$redis->del('key');

// 哈希操作
$redis->hset('user:1', 'name', 'Alice');
$redis->hset('user:1', 'email', 'alice@example.com');
$user = $redis->hgetall('user:1');

// 列表操作
$redis->lpush('list', 'item1');
$redis->rpush('list', 'item2');
$item = $redis->lpop('list');

// 集合操作
$redis->sadd('set', 'member1');
$redis->sadd('set', 'member2');
$members = $redis->smembers('set');

// 有序集合操作
$redis->zadd('sorted_set', 100, 'member1');
$redis->zadd('sorted_set', 200, 'member2');
$top = $redis->zrevrange('sorted_set', 0, 9);
```

## 缓存模式

### Cache-Aside（旁路缓存）

- 应用程序负责缓存管理。
- 先查缓存，缓存未命中则查数据库并写入缓存。

```php
<?php
declare(strict_types=1);

class UserService
{
    private Redis $redis;
    private PDO $db;
    
    public function __construct(Redis $redis, PDO $db)
    {
        $this->redis = $redis;
        $this->db = $db;
    }
    
    public function getUser(int $id): ?array
    {
        // 1. 先查缓存
        $cacheKey = "user:{$id}";
        $cached = $this->redis->get($cacheKey);
        
        if ($cached !== false) {
            return json_decode($cached, true);
        }
        
        // 2. 缓存未命中，查数据库
        $stmt = $this->db->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $user = $stmt->fetch();
        
        if ($user === false) {
            return null;
        }
        
        // 3. 写入缓存
        $this->redis->setex($cacheKey, 3600, json_encode($user));
        
        return $user;
    }
    
    public function updateUser(int $id, array $data): void
    {
        // 1. 更新数据库
        $stmt = $this->db->prepare("UPDATE users SET name = ?, email = ? WHERE id = ?");
        $stmt->execute([$data['name'], $data['email'], $id]);
        
        // 2. 删除缓存
        $this->redis->del("user:{$id}");
    }
}
```

### Write-Through（写穿透）

- 同时更新缓存和数据库。

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): void
{
    // 1. 更新数据库
    $stmt = $this->db->prepare("UPDATE users SET name = ?, email = ? WHERE id = ?");
    $stmt->execute([$data['name'], $data['email'], $id]);
    
    // 2. 更新缓存
    $user = $this->getUserFromDb($id);
    $this->redis->setex("user:{$id}", 3600, json_encode($user));
}
```

### Write-Back（写回）

- 先更新缓存，异步更新数据库。

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): void
{
    // 1. 更新缓存
    $user = array_merge($this->getUser($id), $data);
    $this->redis->setex("user:{$id}", 3600, json_encode($user));
    $this->redis->sadd('dirty_keys', "user:{$id}");  // 标记为脏数据
    
    // 2. 异步更新数据库（后台任务）
    // ...
}
```

## 缓存一致性

### 问题场景

- 缓存与数据库数据不一致。
- 多级缓存之间的不一致。

### 解决方案

#### 1. 删除缓存策略

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): void
{
    // 1. 更新数据库
    $this->updateUserInDb($id, $data);
    
    // 2. 删除缓存（让下次查询时重新加载）
    $this->redis->del("user:{$id}");
}
```

#### 2. 双删策略

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): void
{
    // 1. 删除缓存
    $this->redis->del("user:{$id}");
    
    // 2. 更新数据库
    $this->updateUserInDb($id, $data);
    
    // 3. 延迟删除（处理并发问题）
    sleep(1);
    $this->redis->del("user:{$id}");
}
```

#### 3. 使用消息队列

```php
<?php
declare(strict_types=1);

public function updateUser(int $id, array $data): void
{
    // 1. 更新数据库
    $this->updateUserInDb($id, $data);
    
    // 2. 发送缓存失效消息
    $this->queue->push('cache:invalidate', ['key' => "user:{$id}"]);
}
```

## 缓存问题处理

### 缓存穿透

**问题**：查询不存在的数据，每次都查数据库。

**解决方案**：

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        // 检查是否是空值标记
        if ($cached === 'NULL') {
            return null;
        }
        return json_decode($cached, true);
    }
    
    // 查数据库
    $user = $this->getUserFromDb($id);
    
    if ($user === null) {
        // 缓存空值，防止穿透（设置较短过期时间）
        $this->redis->setex($cacheKey, 60, 'NULL');
        return null;
    }
    
    $this->redis->setex($cacheKey, 3600, json_encode($user));
    return $user;
}
```

### 缓存击穿

**问题**：热点数据过期，大量请求同时访问数据库。

**解决方案**：

```php
<?php
declare(strict_types=1);

/**
 * 获取用户信息（防止缓存击穿）
 * 
 * 缓存击穿问题：热点数据过期时，大量请求同时访问数据库
 * 解决方案：使用分布式锁，确保只有一个请求查询数据库
 * 
 * @param int $id 用户 ID
 * @return array|null 用户信息，不存在返回 null
 */
public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    
    // 第一步：尝试从缓存获取
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        // 缓存命中，直接返回
        return json_decode($cached, true);
    }
    
    // 第二步：缓存未命中，尝试获取分布式锁
    // 使用分布式锁防止多个请求同时查询数据库
    $lockKey = "lock:user:{$id}";
    $lock = $this->acquireLock($lockKey, 10);  // 10 秒超时
    
    if ($lock) {
        try {
            // 第三步：双重检查（Double-Check）
            // 在获取锁后再次检查缓存，可能其他请求已经更新了缓存
            $cached = $this->redis->get($cacheKey);
            if ($cached !== false) {
                return json_decode($cached, true);
            }
            
            // 第四步：查询数据库（只有获取到锁的请求才会执行）
            $user = $this->getUserFromDb($id);
            
            if ($user !== null) {
                $this->redis->setex($cacheKey, 3600, json_encode($user));
            }
            
            return $user;
        } finally {
            $this->releaseLock($lockKey, $lock);
        }
    } else {
        // 获取锁失败，等待后重试
        usleep(100000);  // 100ms
        return $this->getUser($id);
    }
}
```

### 缓存雪崩

**问题**：大量缓存同时过期，导致数据库压力骤增。

**解决方案**：

```php
<?php
declare(strict_types=1);

/**
 * 获取用户信息（防止缓存雪崩）
 * 
 * 缓存雪崩问题：大量缓存同时过期，导致数据库压力骤增
 * 解决方案：为每个缓存设置随机的过期时间，避免同时过期
 * 
 * @param int $id 用户 ID
 * @return array|null 用户信息，不存在返回 null
 */
public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    
    // 尝试从缓存获取
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        return json_decode($cached, true);
    }
    
    // 缓存未命中，查询数据库
    $user = $this->getUserFromDb($id);
    
    if ($user !== null) {
        // 设置随机过期时间，避免缓存雪崩
        // 基础过期时间 3600 秒 + 随机 0-600 秒 = 3600-4200 秒
        // 这样不同用户的缓存会在不同时间过期，分散数据库压力
        $ttl = 3600 + mt_rand(0, 600);
        $this->redis->setex($cacheKey, $ttl, json_encode($user));
    }
    
    return $user;
}
```

## 分布式锁

### SetNX 实现

```php
<?php
declare(strict_types=1);

/**
 * 分布式锁实现
 * 
 * 使用 Redis SETNX 实现分布式锁，确保在分布式环境下只有一个进程可以执行关键操作
 * 
 * 原理：
 * 1. 使用 SETNX（SET if Not eXists）原子操作
 * 2. 设置锁值和过期时间，防止死锁
 * 3. 释放锁时验证锁值，防止误删其他进程的锁
 */
class DistributedLock
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    /**
     * 获取分布式锁
     * 
     * @param string $key 锁的键名
     * @param int $timeout 锁的超时时间（秒），防止死锁
     * @return string|null 锁的值（用于释放锁时验证），获取失败返回 null
     */
    public function acquire(string $key, int $timeout = 10): ?string
    {
        $lockKey = "lock:{$key}";
        
        // 生成唯一的锁值，用于释放锁时验证
        // 防止误删其他进程获取的锁
        $lockValue = uniqid('', true);
        
        // 使用 SET 命令的 NX（不存在才设置）和 EX（设置过期时间）选项
        // 这是一个原子操作，确保只有一个进程能成功设置
        // NX: 只有当键不存在时才设置
        // EX: 设置过期时间（秒），防止进程崩溃导致死锁
        if ($this->redis->set($lockKey, $lockValue, ['nx', 'ex' => $timeout])) {
            return $lockValue;  // 成功获取锁，返回锁值
        }
        
        // 获取锁失败，说明其他进程已持有锁
        return null;
    }
    
    /**
     * 释放分布式锁
     * 
     * 使用 Lua 脚本保证原子性，确保只有锁的持有者才能释放锁
     * 
     * @param string $key 锁的键名
     * @param string $lockValue 锁的值（获取锁时返回的值）
     * @return bool 是否成功释放锁
     */
    public function release(string $key, string $lockValue): bool
    {
        $lockKey = "lock:{$key}";
        
        // 使用 Lua 脚本保证原子性
        // 脚本逻辑：
        // 1. 获取锁的当前值
        // 2. 如果当前值等于传入的锁值，说明是锁的持有者，可以删除
        // 3. 否则返回 0，不删除（可能是锁已过期或被其他进程获取）
        $script = "
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end
        ";
        
        // 执行 Lua 脚本
        // KEYS[1] = $lockKey, ARGV[1] = $lockValue
        // 最后一个参数 1 表示 KEYS 数组的长度
        return $this->redis->eval($script, [$lockKey, $lockValue], 1) === 1;
    }
}

// 使用
$lock = new DistributedLock($redis);
$lockValue = $lock->acquire('resource:1', 10);

if ($lockValue !== null) {
    try {
        // 执行临界区代码
        // ...
    } finally {
        $lock->release('resource:1', $lockValue);
    }
}
```

### Redlock 算法

```php
<?php
declare(strict_types=1);

class RedLock
{
    private array $redisInstances;
    
    public function __construct(array $redisInstances)
    {
        $this->redisInstances = $redisInstances;
    }
    
    public function lock(string $resource, int $ttl): bool
    {
        $lockValue = uniqid();
        $quorum = (int) (count($this->redisInstances) / 2) + 1;
        $acquired = 0;
        
        foreach ($this->redisInstances as $redis) {
            if ($redis->set("lock:{$resource}", $lockValue, ['nx', 'ex' => $ttl])) {
                $acquired++;
            }
        }
        
        return $acquired >= $quorum;
    }
}
```

## 缓存最佳实践

### 1. 设置合理的过期时间

```php
// 根据数据更新频率设置
$ttl = 3600;  // 1 小时
$ttl = 86400;  // 1 天
$ttl = 600;  // 10 分钟
```

### 2. 使用缓存预热

```php
<?php
declare(strict_types=1);

// 系统启动时预热热点数据
function warmupCache(Redis $redis, PDO $db): void
{
    $stmt = $db->query("SELECT * FROM products WHERE is_hot = 1");
    $products = $stmt->fetchAll();
    
    foreach ($products as $product) {
        $redis->setex(
            "product:{$product['id']}",
            3600,
            json_encode($product)
        );
    }
}
```

### 3. 监控缓存命中率

```php
<?php
declare(strict_types=1);

class CacheStats
{
    private Redis $redis;
    private int $hits = 0;
    private int $misses = 0;
    
    public function recordHit(): void
    {
        $this->hits++;
        $this->redis->incr('cache:hits');
    }
    
    public function recordMiss(): void
    {
        $this->misses++;
        $this->redis->incr('cache:misses');
    }
    
    public function getHitRate(): float
    {
        $total = $this->hits + $this->misses;
        return $total > 0 ? $this->hits / $total : 0;
    }
}
```

## 练习

1. 实现一个缓存管理器，支持 Cache-Aside 模式，包含缓存穿透防护。

2. 创建一个分布式锁类，使用 Redis SetNX 实现，支持锁续期。

3. 设计一个缓存一致性方案，处理数据库更新时的缓存失效。

4. 实现一个缓存预热系统，在系统启动时加载热点数据。

5. 编写一个缓存监控工具，统计缓存命中率和性能指标。

6. 创建一个多级缓存系统，使用本地缓存和 Redis 缓存。

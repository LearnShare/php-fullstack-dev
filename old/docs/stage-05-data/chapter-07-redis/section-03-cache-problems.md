# 5.7.3 缓存问题与解决方案

## 概述

缓存使用中会遇到穿透、击穿、雪崩等问题。本节详细介绍这些问题及其解决方案，包括缓存空值、分布式锁、随机过期时间等技术。

## 缓存穿透

### 问题

查询不存在的数据，每次都查数据库。

### 解决方案

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

## 缓存击穿

### 问题

热点数据过期，大量请求同时访问数据库。

### 解决方案

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    
    // 第一步：尝试从缓存获取
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        return json_decode($cached, true);
    }
    
    // 第二步：缓存未命中，尝试获取分布式锁
    $lockKey = "lock:user:{$id}";
    $lock = $this->acquireLock($lockKey, 10);
    
    if ($lock) {
        try {
            // 第三步：双重检查
            $cached = $this->redis->get($cacheKey);
            if ($cached !== false) {
                return json_decode($cached, true);
            }
            
            // 第四步：查询数据库
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
        usleep(100000);
        return $this->getUser($id);
    }
}
```

## 缓存雪崩

### 问题

大量缓存同时过期，导致数据库压力骤增。

### 解决方案

```php
<?php
declare(strict_types=1);

public function getUser(int $id): ?array
{
    $cacheKey = "user:{$id}";
    
    $cached = $this->redis->get($cacheKey);
    
    if ($cached !== false) {
        return json_decode($cached, true);
    }
    
    $user = $this->getUserFromDb($id);
    
    if ($user !== null) {
        // 设置随机过期时间，避免缓存雪崩
        $ttl = 3600 + mt_rand(0, 600);
        $this->redis->setex($cacheKey, $ttl, json_encode($user));
    }
    
    return $user;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CacheService
{
    private Redis $redis;
    private PDO $db;

    public function getUser(int $id): ?array
    {
        $cacheKey = "user:{$id}";
        
        // 1. 查缓存
        $cached = $this->redis->get($cacheKey);
        if ($cached !== false) {
            if ($cached === 'NULL') {
                return null;  // 空值标记
            }
            return json_decode($cached, true);
        }
        
        // 2. 获取分布式锁
        $lockKey = "lock:user:{$id}";
        $lock = $this->acquireLock($lockKey, 10);
        
        if (!$lock) {
            usleep(100000);
            return $this->getUser($id);
        }
        
        try {
            // 3. 双重检查
            $cached = $this->redis->get($cacheKey);
            if ($cached !== false) {
                return $cached === 'NULL' ? null : json_decode($cached, true);
            }
            
            // 4. 查数据库
            $user = $this->getUserFromDb($id);
            
            // 5. 写入缓存（随机过期时间）
            $ttl = $user !== null ? 3600 + mt_rand(0, 600) : 60;
            $value = $user !== null ? json_encode($user) : 'NULL';
            $this->redis->setex($cacheKey, $ttl, $value);
            
            return $user;
        } finally {
            $this->releaseLock($lockKey, $lock);
        }
    }
}
```

## 注意事项

1. **空值缓存**：缓存空值防止穿透
2. **分布式锁**：使用锁防止击穿
3. **随机过期**：设置随机过期时间防止雪崩
4. **双重检查**：获取锁后再次检查缓存

## 练习

1. 实现完整的缓存问题解决方案。

2. 创建一个缓存服务类，处理所有缓存问题。

3. 编写缓存问题测试，验证解决方案。

4. 实现缓存监控，监控缓存命中率。

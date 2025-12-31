# 6.5.1 Redis 基础

## 概述

Redis（Remote Dictionary Server）是一个开源的内存数据结构存储系统，可以用作数据库、缓存和消息中间件。Redis 支持多种数据结构（字符串、列表、集合、有序集合、哈希等），提供了高性能的数据操作能力。

本节详细介绍 Redis 的基础知识，包括 Redis 的概念、特性、数据结构、PHP 客户端使用、基本操作等，帮助零基础学员掌握 Redis 的使用方法。

**主要内容**：
- Redis 概述和特性
- Redis 数据结构（字符串、列表、集合、有序集合、哈希）
- PHP Redis 客户端使用
- 基本操作（SET、GET、DEL 等）
- 连接管理和配置
- 完整示例和最佳实践

---

## 特性

- **内存存储**：数据存储在内存中，读写速度快
- **数据结构丰富**：支持多种数据结构
- **持久化**：支持数据持久化到磁盘
- **高性能**：读写性能极高
- **原子操作**：所有操作都是原子性的

---

## Redis 概述

### 什么是 Redis

Redis 是一个开源的内存数据结构存储系统，可以用作数据库、缓存和消息中间件。

**Redis 的特点**：

- **内存存储**：数据存储在内存中
- **数据结构丰富**：支持多种数据结构
- **高性能**：读写性能极高
- **持久化**：支持数据持久化
- **原子操作**：所有操作都是原子性的

### Redis 的用途

Redis 的主要用途包括：

1. **缓存**：作为缓存系统，提高应用性能
2. **会话存储**：存储用户会话数据
3. **消息队列**：作为消息中间件
4. **计数器**：实现计数器和排行榜
5. **实时分析**：实时数据分析和统计

### Redis 的优势

Redis 的优势包括：

- **高性能**：内存存储，读写速度快
- **数据结构丰富**：支持多种数据结构
- **原子操作**：所有操作都是原子性的
- **持久化**：支持数据持久化
- **分布式**：支持主从复制和集群

---

## Redis 数据结构

### 字符串（String）

字符串是 Redis 最基本的数据类型，可以存储字符串、整数或浮点数。

**基本操作**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// SET：设置值
$redis->set('name', 'John');

// GET：获取值
$name = $redis->get('name');

// SETEX：设置值并指定过期时间（秒）
$redis->setex('token', 3600, 'abc123');

// SETNX：只在键不存在时设置
$redis->setnx('lock', 'locked');

// INCR：递增
$redis->incr('counter');

// DECR：递减
$redis->decr('counter');

// INCRBY：增加指定值
$redis->incrBy('counter', 10);

// APPEND：追加字符串
$redis->append('name', ' Doe');
```

### 列表（List）

列表是有序的字符串列表，支持从两端操作。

**基本操作**：

```php
<?php
declare(strict_types=1);

// LPUSH：从左边推入
$redis->lPush('list', 'item1');
$redis->lPush('list', 'item2');

// RPUSH：从右边推入
$redis->rPush('list', 'item3');

// LPOP：从左边弹出
$item = $redis->lPop('list');

// RPOP：从右边弹出
$item = $redis->rPop('list');

// LRANGE：获取列表范围
$items = $redis->lRange('list', 0, -1);

// LLEN：获取列表长度
$length = $redis->lLen('list');
```

### 集合（Set）

集合是无序的字符串集合，不允许重复元素。

**基本操作**：

```php
<?php
declare(strict_types=1);

// SADD：添加元素
$redis->sAdd('set', 'item1');
$redis->sAdd('set', 'item2');

// SMEMBERS：获取所有元素
$members = $redis->sMembers('set');

// SISMEMBER：检查元素是否存在
$exists = $redis->sIsMember('set', 'item1');

// SREM：删除元素
$redis->sRem('set', 'item1');

// SCARD：获取集合大小
$size = $redis->sCard('set');

// SINTER：求交集
$intersection = $redis->sInter('set1', 'set2');

// SUNION：求并集
$union = $redis->sUnion('set1', 'set2');
```

### 有序集合（Sorted Set）

有序集合是有序的字符串集合，每个元素关联一个分数。

**基本操作**：

```php
<?php
declare(strict_types=1);

// ZADD：添加元素
$redis->zAdd('sorted_set', 100, 'item1');
$redis->zAdd('sorted_set', 200, 'item2');

// ZRANGE：获取范围
$items = $redis->zRange('sorted_set', 0, -1);

// ZREVRANGE：反向获取范围
$items = $redis->zRevRange('sorted_set', 0, -1);

// ZSCORE：获取分数
$score = $redis->zScore('sorted_set', 'item1');

// ZRANK：获取排名
$rank = $redis->zRank('sorted_set', 'item1');

// ZCARD：获取集合大小
$size = $redis->zCard('sorted_set');
```

### 哈希（Hash）

哈希是字段和值的映射表。

**基本操作**：

```php
<?php
declare(strict_types=1);

// HSET：设置字段值
$redis->hSet('user:1', 'name', 'John');
$redis->hSet('user:1', 'email', 'john@example.com');

// HGET：获取字段值
$name = $redis->hGet('user:1', 'name');

// HMSET：设置多个字段值
$redis->hMSet('user:1', [
    'name' => 'John',
    'email' => 'john@example.com',
    'age' => 30
]);

// HMGET：获取多个字段值
$values = $redis->hMGet('user:1', ['name', 'email']);

// HGETALL：获取所有字段值
$all = $redis->hGetAll('user:1');

// HDEL：删除字段
$redis->hDel('user:1', 'age');

// HKEYS：获取所有字段名
$keys = $redis->hKeys('user:1');
```

---

## PHP Redis 客户端

### 安装 Redis 扩展

安装 PHP Redis 扩展。

**方法 1：使用 PECL**

```bash
pecl install redis
```

**方法 2：使用 Composer**

```bash
composer require predis/predis
```

### 连接 Redis

使用 Redis 扩展连接 Redis。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 Redis 扩展
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->auth('password');  // 如果设置了密码

// 方法 2：使用 Predis
use Predis\Client;

$redis = new Client([
    'scheme' => 'tcp',
    'host' => '127.0.0.1',
    'port' => 6379,
    'password' => 'password'
]);
```

### 基本操作

使用 Redis 客户端进行基本操作。

**示例**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 字符串操作
$redis->set('key', 'value');
$value = $redis->get('key');

// 列表操作
$redis->lPush('list', 'item');
$item = $redis->lPop('list');

// 集合操作
$redis->sAdd('set', 'item');
$members = $redis->sMembers('set');

// 哈希操作
$redis->hSet('hash', 'field', 'value');
$value = $redis->hGet('hash', 'field');

// 删除键
$redis->del('key');

// 检查键是否存在
$exists = $redis->exists('key');

// 设置过期时间
$redis->expire('key', 3600);
```

---

## 连接管理

### 连接配置

配置 Redis 连接参数。

**示例**：

```php
<?php
declare(strict_types=1);

class RedisConnection
{
    private static ?Redis $instance = null;
    
    public static function getInstance(): Redis
    {
        if (self::$instance === null) {
            self::$instance = new Redis();
            self::$instance->connect(
                $_ENV['REDIS_HOST'] ?? '127.0.0.1',
                (int)($_ENV['REDIS_PORT'] ?? 6379)
            );
            
            if ($password = $_ENV['REDIS_PASSWORD'] ?? null) {
                self::$instance->auth($password);
            }
        }
        
        return self::$instance;
    }
}

// 使用
$redis = RedisConnection::getInstance();
```

### 连接池

虽然 PHP Redis 扩展本身不支持连接池，但可以使用单例模式管理连接。

**示例**：

```php
<?php
declare(strict_types=1);

class RedisPool
{
    private static array $connections = [];
    
    public static function getConnection(string $name = 'default'): Redis
    {
        if (!isset(self::$connections[$name])) {
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);
            self::$connections[$name] = $redis;
        }
        
        return self::$connections[$name];
    }
}
```

---

## 完整示例

### Redis 缓存示例

```php
<?php
declare(strict_types=1);

class CacheService
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function get(string $key): mixed
    {
        $value = $this->redis->get($key);
        return $value !== false ? unserialize($value) : null;
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): bool
    {
        return $this->redis->setex($key, $ttl, serialize($value));
    }
    
    public function delete(string $key): bool
    {
        return $this->redis->del($key) > 0;
    }
    
    public function remember(string $key, int $ttl, callable $callback): mixed
    {
        $value = $this->get($key);
        
        if ($value !== null) {
            return $value;
        }
        
        $value = $callback();
        $this->set($key, $value, $ttl);
        
        return $value;
    }
}

// 使用
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$cache = new CacheService($redis);

// 缓存数据
$cache->set('user:1', ['name' => 'John', 'email' => 'john@example.com'], 3600);

// 获取缓存
$user = $cache->get('user:1');

// 记住模式
$user = $cache->remember('user:1', 3600, function () {
    return ['name' => 'John', 'email' => 'john@example.com'];
});
```

---

## 使用场景

### 缓存

Redis 作为缓存系统，提高应用性能。

### 会话存储

Redis 存储用户会话数据。

### 计数器

Redis 实现计数器和排行榜。

---

## 注意事项

### 内存管理

Redis 是内存数据库，需要注意内存使用。

### 持久化

根据需求配置持久化策略。

### 连接管理

合理管理 Redis 连接，避免连接泄漏。

---

## 常见问题

### 如何连接 Redis？

使用 Redis 扩展或 Predis 客户端连接。

### Redis 支持哪些数据结构？

支持字符串、列表、集合、有序集合、哈希等。

### 如何设置过期时间？

使用 `expire` 或 `setex` 方法设置过期时间。

---

## 最佳实践

### 合理使用数据结构

根据需求选择合适的数据结构。

### 设置过期时间

为缓存数据设置合理的过期时间。

### 连接管理

使用单例模式管理 Redis 连接。

---

## 练习任务

1. **Redis 基础操作**
   - 连接 Redis
   - 实现字符串操作
   - 实现列表操作
   - 测试基本功能

2. **数据结构操作**
   - 实现集合操作
   - 实现有序集合操作
   - 实现哈希操作
   - 测试各种数据结构

3. **缓存实现**
   - 实现缓存服务
   - 实现缓存读写
   - 实现缓存过期
   - 测试缓存功能

4. **性能测试**
   - 测试 Redis 性能
   - 对比数据库查询
   - 分析性能差异
   - 编写性能报告

5. **综合应用**
   - 创建一个完整的缓存系统
   - 实现各种缓存策略
   - 优化缓存性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.5.2 缓存策略](section-02-cache-strategies.md)
- [6.5.3 缓存问题与解决方案](section-03-cache-problems.md)
- [6.5.4 高级特性](section-04-advanced-features.md)
- [6.2 PDO 入门与高安全模式](../chapter-02-pdo/readme.md)

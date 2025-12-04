# 5.7.1 Redis 基础

## 概述

Redis 是一个高性能的内存数据库，广泛用于缓存、消息队列等场景。本节详细介绍 Redis 连接、基本操作、数据类型，以及 Redis 命令的使用。

## Redis 连接

### 使用 phpredis 扩展

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->auth('password');  // 如果需要密码
```

### 使用 Predis 库

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';
use Predis\Client;

$redis = new Client([
    'scheme' => 'tcp',
    'host' => '127.0.0.1',
    'port' => 6379,
    'password' => 'password',
]);
```

## 基本操作

### 字符串操作

```php
<?php
declare(strict_types=1);

$redis->set('key', 'value');
$redis->set('key', 'value', 3600);  // 设置过期时间（秒）
$value = $redis->get('key');
$redis->del('key');
```

### 哈希操作

```php
<?php
declare(strict_types=1);

$redis->hset('user:1', 'name', 'Alice');
$redis->hset('user:1', 'email', 'alice@example.com');
$user = $redis->hgetall('user:1');
```

### 列表操作

```php
<?php
declare(strict_types=1);

$redis->lpush('list', 'item1');
$redis->rpush('list', 'item2');
$item = $redis->lpop('list');
```

### 集合操作

```php
<?php
declare(strict_types=1);

$redis->sadd('set', 'member1');
$redis->sadd('set', 'member2');
$members = $redis->smembers('set');
```

### 有序集合操作

```php
<?php
declare(strict_types=1);

$redis->zadd('sorted_set', 100, 'member1');
$redis->zadd('sorted_set', 200, 'member2');
$top = $redis->zrevrange('sorted_set', 0, 9);
```

## 数据类型

| 类型 | 说明 | 使用场景 |
| :--- | :--- | :--- |
| String | 字符串 | 简单键值对 |
| Hash | 哈希表 | 对象存储 |
| List | 列表 | 队列、栈 |
| Set | 集合 | 去重、交集并集 |
| Sorted Set | 有序集合 | 排行榜 |

## 完整示例

```php
<?php
declare(strict_types=1);

class RedisService
{
    private Redis $redis;

    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }

    public function set(string $key, mixed $value, int $ttl = 0): bool
    {
        $data = json_encode($value);
        if ($ttl > 0) {
            return $this->redis->setex($key, $ttl, $data);
        }
        return $this->redis->set($key, $data);
    }

    public function get(string $key): mixed
    {
        $data = $this->redis->get($key);
        return $data !== false ? json_decode($data, true) : null;
    }
}
```

## 注意事项

1. **连接管理**：合理管理 Redis 连接
2. **序列化**：复杂数据需要序列化
3. **过期时间**：合理设置过期时间
4. **错误处理**：处理连接失败等错误

## 练习

1. 创建一个 Redis 服务类，封装常用操作。

2. 实现 Redis 连接池管理。

3. 编写一个 Redis 数据序列化工具。

4. 实现 Redis 错误处理和重连机制。

# 5.7.2 缓存策略

## 概述

缓存策略决定了缓存和数据库的同步方式。本节详细介绍 Cache-Aside、Write-Through、Write-Back 等缓存模式，以及缓存更新策略。

## Cache-Aside（旁路缓存）

### 原理

- 应用程序负责缓存管理
- 先查缓存，缓存未命中则查数据库并写入缓存

### 实现

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

## Write-Through（写穿透）

### 原理

- 同时更新缓存和数据库
- 保证缓存和数据库的一致性

### 实现

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

## Write-Back（写回）

### 原理

- 先更新缓存，异步更新数据库
- 提升写入性能

### 实现

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

## 缓存更新策略

### 删除缓存策略

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

### 双删策略

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

## 完整示例

```php
<?php
declare(strict_types=1);

class CacheStrategy
{
    private Redis $redis;
    private PDO $db;

    public function cacheAsideGet(int $id): ?array
    {
        $cacheKey = "user:{$id}";
        $cached = $this->redis->get($cacheKey);
        
        if ($cached !== false) {
            return json_decode($cached, true);
        }
        
        $user = $this->getUserFromDb($id);
        if ($user !== null) {
            $this->redis->setex($cacheKey, 3600, json_encode($user));
        }
        
        return $user;
    }

    public function writeThroughUpdate(int $id, array $data): void
    {
        $this->updateUserInDb($id, $data);
        $user = $this->getUserFromDb($id);
        $this->redis->setex("user:{$id}", 3600, json_encode($user));
    }
}
```

## 注意事项

1. **策略选择**：根据业务场景选择合适的缓存策略
2. **一致性**：保证缓存和数据库的一致性
3. **性能考虑**：平衡一致性和性能
4. **错误处理**：处理缓存和数据库操作失败

## 练习

1. 实现 Cache-Aside 缓存策略。

2. 实现 Write-Through 缓存策略。

3. 实现 Write-Back 缓存策略。

4. 对比不同缓存策略的性能和一致性。

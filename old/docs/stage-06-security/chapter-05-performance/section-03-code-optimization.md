# 6.5.3 代码优化技巧

## 概述

代码优化是提升应用性能的重要手段。本节介绍算法优化、数据结构选择、缓存使用、数据库优化等代码层面的优化技巧。

## 算法优化

### 1. 选择合适的数据结构

```php
<?php
declare(strict_types=1);

// 不推荐：使用数组查找
function findUser(array $users, int $id): ?array
{
    foreach ($users as $user) {
        if ($user['id'] === $id) {
            return $user;
        }
    }
    return null;
}

// 推荐：使用关联数组（哈希表）
function findUser(array $users, int $id): ?array
{
    return $users[$id] ?? null;
}
```

### 2. 避免嵌套循环

```php
<?php
declare(strict_types=1);

// 不推荐：O(n²) 复杂度
function findCommon(array $array1, array $array2): array
{
    $common = [];
    foreach ($array1 as $item1) {
        foreach ($array2 as $item2) {
            if ($item1 === $item2) {
                $common[] = $item1;
            }
        }
    }
    return $common;
}

// 推荐：O(n) 复杂度
function findCommon(array $array1, array $array2): array
{
    $set = array_flip($array2);
    $common = [];
    foreach ($array1 as $item) {
        if (isset($set[$item])) {
            $common[] = $item;
        }
    }
    return $common;
}
```

### 3. 使用内置函数

```php
<?php
declare(strict_types=1);

// 不推荐：手动实现
function arraySum(array $array): int
{
    $sum = 0;
    foreach ($array as $value) {
        $sum += $value;
    }
    return $sum;
}

// 推荐：使用内置函数
$sum = array_sum($array);
```

## 数据结构选择

### 1. 数组 vs 对象

```php
<?php
declare(strict_types=1);

// 数组：快速访问，内存占用小
$user = ['id' => 1, 'name' => 'John'];

// 对象：类型安全，可扩展
class User
{
    public function __construct(
        public int $id,
        public string $name,
    ) {}
}
```

### 2. SplFixedArray

```php
<?php
declare(strict_types=1);

// 固定大小数组，性能更好
$array = new SplFixedArray(1000);
for ($i = 0; $i < 1000; $i++) {
    $array[$i] = $i;
}
```

## 缓存使用

### 1. 内存缓存

```php
<?php
declare(strict_types=1);

class MemoryCache
{
    private static array $cache = [];
    
    public static function get(string $key, callable $callback): mixed
    {
        if (!isset(self::$cache[$key])) {
            self::$cache[$key] = $callback();
        }
        return self::$cache[$key];
    }
    
    public static function clear(): void
    {
        self::$cache = [];
    }
}

// 使用
$data = MemoryCache::get('expensive_operation', function() {
    return expensiveOperation();
});
```

### 2. 结果缓存

```php
<?php
declare(strict_types=1);

function cachedFunction(string $key, callable $callback, int $ttl = 3600): mixed
{
    $cacheFile = "/tmp/cache_{$key}.php";
    
    if (file_exists($cacheFile) && (time() - filemtime($cacheFile)) < $ttl) {
        return unserialize(file_get_contents($cacheFile));
    }
    
    $result = $callback();
    file_put_contents($cacheFile, serialize($result));
    
    return $result;
}
```

## 数据库优化

### 1. 批量操作

```php
<?php
declare(strict_types=1);

// 不推荐：逐条插入
foreach ($users as $user) {
    $stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
    $stmt->execute([$user['name'], $user['email']]);
}

// 推荐：批量插入
$stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
$pdo->beginTransaction();
foreach ($users as $user) {
    $stmt->execute([$user['name'], $user['email']]);
}
$pdo->commit();
```

### 2. 预编译语句复用

```php
<?php
declare(strict_types=1);

class UserRepository
{
    private ?PDOStatement $findByIdStmt = null;
    
    public function findById(int $id): ?array
    {
        if ($this->findByIdStmt === null) {
            $this->findByIdStmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        }
        
        $this->findByIdStmt->execute([$id]);
        return $this->findByIdStmt->fetch() ?: null;
    }
}
```

### 3. 索引优化

```sql
-- 创建索引
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_created_at ON users(created_at);

-- 复合索引
CREATE INDEX idx_user_status ON users(user_id, status);
```

## 字符串优化

### 1. 字符串拼接

```php
<?php
declare(strict_types=1);

// 不推荐：多次拼接
$result = '';
foreach ($items as $item) {
    $result .= $item . ',';
}

// 推荐：使用 implode
$result = implode(',', $items);
```

### 2. 正则表达式优化

```php
<?php
declare(strict_types=1);

// 预编译正则表达式
$pattern = '/^[a-z0-9]+$/i';
$compiled = preg_match($pattern, $string);

// 使用非捕获组
$pattern = '/(?:prefix_)(\d+)/'; // 非捕获组性能更好
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 优化前
class UserServiceOld
{
    public function getUsersWithOrders(): array
    {
        $users = $this->getAllUsers();
        $result = [];
        
        foreach ($users as $user) {
            $orders = $this->getOrdersByUserId($user['id']); // N+1 问题
            $user['orders'] = $orders;
            $result[] = $user;
        }
        
        return $result;
    }
}

// 优化后
class UserService
{
    public function getUsersWithOrders(): array
    {
        // 1. 批量获取用户
        $users = $this->getAllUsers();
        $userIds = array_column($users, 'id');
        
        // 2. 批量获取订单
        $orders = $this->getOrdersByUserIds($userIds);
        
        // 3. 使用关联数组快速查找
        $ordersByUserId = [];
        foreach ($orders as $order) {
            $ordersByUserId[$order['user_id']][] = $order;
        }
        
        // 4. 合并数据
        foreach ($users as &$user) {
            $user['orders'] = $ordersByUserId[$user['id']] ?? [];
        }
        
        return $users;
    }
    
    private function getOrdersByUserIds(array $userIds): array
    {
        $placeholders = implode(',', array_fill(0, count($userIds), '?'));
        $stmt = $this->pdo->prepare("SELECT * FROM orders WHERE user_id IN ({$placeholders})");
        $stmt->execute($userIds);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}
```

## 最佳实践

1. **选择合适算法**：根据数据规模选择算法
2. **使用缓存**：缓存计算结果
3. **批量操作**：减少数据库查询次数
4. **预编译语句**：复用预编译语句
5. **索引优化**：为常用查询字段创建索引

## 注意事项

1. 不要过度优化，保持代码可读性
2. 先测量性能，再优化
3. 考虑内存使用，避免内存溢出
4. 定期审查和优化代码

## 练习

1. 优化一个存在 N+1 查询问题的函数。

2. 实现一个结果缓存机制，缓存函数执行结果。

3. 优化一个字符串处理函数，使用更高效的方法。

4. 创建一个批量操作工具类，支持批量插入和更新。

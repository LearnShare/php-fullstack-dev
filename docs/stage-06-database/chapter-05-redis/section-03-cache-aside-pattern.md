# 6.5.3 旁路缓存模式 (Cache-Aside Pattern)

## 概述

在将 Redis 用作数据库缓存时，我们需要一套清晰的逻辑来规定应用程序应该如何与缓存和数据库进行交互。在多种缓存策略中，**旁路缓存模式 (Cache-Aside Pattern)** 是最常用、最直观，也是最推荐的一种。

这个模式的核心思想是：**应用程序代码需要同时维护缓存和数据库，缓存本身不与数据库直接通信**。因为应用程序的逻辑“位于”缓存的“旁边”，所以被称为“旁路”缓存。

该模式的实现主要分为两部分：**读取数据逻辑**和**写入/更新数据逻辑**。

## 读取逻辑 (Read Path) 

读取数据的逻辑遵循“**先读缓存，后读数据库**”的原则。

**算法步骤**：
1.  应用程序收到一个读请求（例如，获取 ID 为 101 的用户信息）。
2.  应用程序根据一个确定的键（如 `user:101`）去查询 **Redis 缓存**。
3.  **缓存命中 (Cache Hit)**：
    -   如果在缓存中找到了数据，则直接将数据返回给客户端。
    -   查询结束。
4.  **缓存未命中 (Cache Miss)**：
    -   如果在缓存中没有找到数据，则应用程序再去查询**数据库**。
    -   数据库查询结束后：
        -   **如果数据库中有数据**：应用程序将这份数据**存入 Redis 缓存**（并设置一个合理的**过期时间 TTL**），然后将数据返回给客户端。
        -   **如果数据库中也没有数据**：应用程序直接向客户端返回空结果。

**图示**：
![Cache-Aside Read Path](https://i.imgur.com/8Yv2LqJ.png)

**PHP + Predis 示例**：
```php
<?php
// ... $redis 和 $pdo 客户端实例

/**
 * 使用旁路缓存模式获取用户信息
 * @param int $userId
 * @return array|null
 */
function getUserById(int $userId): ?array
{
    global $redis, $pdo;
    
    $cacheKey = "user:{$userId}";

    // 1. 先从 Redis 读取
    $cachedUser = $redis->get($cacheKey);

    if ($cachedUser) {
        // 3. 缓存命中
        echo "缓存命中！\n";
        return json_decode($cachedUser, true);
    }

    // 4. 缓存未命中
    echo "缓存未命中，查询数据库...\n";
    
    // 4a. 查询数据库
    $stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");
    $stmt->execute([':id' => $userId]);
    $user = $stmt->fetch(PDO::FETCH_ASSOC);

    if ($user) {
        // 4b. 将数据存入缓存，并设置 1 小时 (3600秒) 的过期时间
        $redis->setex($cacheKey, 3600, json_encode($user));
    }
    
    // 4c. 返回从数据库获取的数据
    return $user ?: null;
}

// --- 使用 ---
$user = getUserById(1);
print_r($user);

// 再次调用，这次应该会命中缓存
$user = getUserById(1);
print_r($user);
```

## 写入/更新逻辑 (Write Path) 与缓存失效

当数据发生变化时（通过 `UPDATE` 或 `DELETE` 操作），存储在缓存中的旧数据就变得“不一致”了。我们必须有一种机制来处理这些“脏”缓存，这个过程称为**缓存失效 (Cache Invalidation)**。

在旁路缓存模式中，最安全、最简单的策略是：**在更新数据时，直接删除缓存**。

**为什么是删除缓存，而不是更新缓存？**
1.  **懒加载 (Lazy Loading)**：删除缓存后，我们并不需要立即去查询数据库并更新缓存。而是把这个“更新缓存”的动作，推迟到下一次读请求发生时“懒惰”地执行。这可以节省不必要的计算，因为被更新的数据可能在很长一段时间内都不会被再次读取。
2.  **避免复杂性与竞态条件**：如果两个写操作几乎同时发生，去“更新”缓存可能会导致缓存中存入一个中间的、错误的状态。而“删除”缓存则是一个简单、幂等的操作，不会有这个问题。

**算法步骤**：
1.  应用程序收到一个写请求（`UPDATE` 或 `DELETE`）。
2.  **先更新数据库**。
3.  **再删除 Redis 缓存中对应的键**。

**PHP + Predis 示例**:
```php
<?php
// ... $redis 和 $pdo 客户端实例

/**
 * 更新用户邮箱，并使缓存失效
 * @param int $userId
 * @param string $newEmail
 * @return bool
 */
function updateUserEmail(int $userId, string $newEmail): bool
{
    global $redis, $pdo;
    
    // 1. 先更新数据库
    $stmt = $pdo->prepare("UPDATE users SET email = :email WHERE id = :id");
    $success = $stmt->execute([':email' => $newEmail, ':id' => $userId]);

    if ($success) {
        // 2. 再删除缓存
        $cacheKey = "user:{$userId}";
        $redis->del($cacheKey);
        echo "数据库已更新，缓存 '{$cacheKey}' 已被删除。\n";
    }

    return $success;
}

// --- 使用 ---
updateUserEmail(1, 'new.alice@example.com');
```

**思考：先删缓存还是先更新数据库？**
答案是：**先更新数据库，再删缓存**。
-   如果**先删缓存**，再更新数据库：
    1.  线程 A 删除了缓存。
    2.  在线程 A 更新数据库**之前**，一个读请求（线程 B）到来。
    3.  线程 B 发现缓存未命中，去查询数据库，读到的是**旧数据**。
    4.  线程 B 将这个**旧数据**又写回了缓存。
    5.  线程 A 完成了数据库的更新。
    6.  结果：数据库是新的，但缓存中却是旧数据，数据不一致将持续到缓存过期。
-   而**先更新数据库**，再删缓存，即使在极端的并发情况下（写操作与一个读操作交错），不一致的时间窗口也极其短暂，相对更安全。

## 必须设置过期时间 (TTL)

为每一个写入缓存的键都设置一个**过期时间 (Time-To-Live, TTL)** 是一个至关重要的最佳实践。

-   **兜底策略**：它可以作为缓存失效逻辑的“兜底”。如果因为某些 Bug 或异常导致更新数据库后删除缓存失败，这个 TTL 可以保证脏数据不会永久地留在缓存中，它最长只会在缓存中存活一个 TTL 的时间。
-   **内存管理**：它可以自动清除那些不经常被访问的“冷”数据，防止 Redis 内存无限增长。

在 Predis 中，可以使用 `setex` 命令或在 `set` 命令中加 `EX` 选项来设置 TTL。
```php
// 设置一个键，10 分钟后过期
$redis->setex('mykey', 600, 'myvalue');

// 与上面等价
$redis->set('mykey', 'myvalue', 'EX', 600);
```

---
## 练习任务

1.  **缓存文章内容**：
    为你的博客系统编写一个 `getPostContent(int $postId)` 函数。该函数应实现完整的旁路缓存读取逻辑：先尝试从 Redis 读取 `post:{id}:content`，如果不存在，则从数据库查询，然后将结果存入 Redis 并设置 10 分钟的过期时间。

2.  **实现缓存失效**：
    编写一个 `updatePostContent(int $postId, string $newContent)` 函数。该函数在更新数据库中文章内容的同时，必须删除 Redis 中对应的缓存键。

3.  **思考缓存策略**：
    你的网站首页需要显示一个“最新发布的 5 篇文章”的列表。这个列表数据非常适合被缓存。当一个管理员发布了一篇新文章后，你应该如何使这个列表的缓存失效？是“更新”缓存还是“删除”缓存更好？为什么？

```
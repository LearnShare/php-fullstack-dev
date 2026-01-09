# 6.5.2 Redis 核心数据类型

## 概述

Redis 的强大之处并不仅仅在于其速度，更在于其“值 (Value)”支持多种高级数据结构。与只能存储简单字符串的 Memcached 不同，Redis 在服务器端实现了这些数据结构，使得我们可以通过命令直接对其进行高效的操作，从而极大地简化了应用端的代码逻辑。

本节将详细介绍 Redis 最核心的五种数据类型：
1.  **String (字符串)**
2.  **Hash (哈希)**
3.  **List (列表)**
4.  **Set (集合)**
5.  **Sorted Set (有序集合 / ZSET)**

## 1. String (字符串)

-   **定义**：这是 Redis 最基本的数据类型。一个键 (Key) 对应一个字符串值。这个字符串可以是任何形式的数据，如文本、数字、或者序列化后的对象（如 JSON）。
-   **PHP 类比**：一个简单的变量，`$key = "some value";`。
-   **核心命令**：`SET`, `GET`, `INCR`/`DECR`, `INCRBY`/`DECRBY`, `MGET` (批量获取), `SETEX` (设置带过期时间的值)。
-   **主要用途**：
    -   **缓存**：缓存页面 HTML、JSON 格式的 API 响应、序列化后的 PHP 对象等。
    -   **计数器**：利用 `INCR` 命令的原子性，可以非常方便地实现网站访问量、文章点赞数等计数功能。
-   **Predis 示例**：
    ```php
    <?php
    // ... $redis 客户端实例
    
    // 缓存一个用户信息（JSON 编码）
    $user = ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'];
    $redis->set('user:1', json_encode($user));
    
    // 获取并解码
    $cachedUserJson = $redis->get('user:1');
    $cachedUser = json_decode($cachedUserJson, true);
    print_r($cachedUser);
    
    // 实现一个文章阅读计数器
    $page_views = $redis->incr('post:101:views');
    echo "文章 101 的阅读量是: {$page_views}";
    ```

## 2. Hash (哈希)

-   **定义**：一个键 (Key) 对应一个包含多个“字段 (field)”和“值 (value)”的映射表。它非常适合用来存储结构化的对象。
-   **PHP 类比**：一个关联数组，`$user = ['name' => 'Bob', 'age' => 30];`。
-   **核心命令**：`HSET`, `HGET`, `HMGET` (批量获取字段), `HGETALL`, `HINCRBY`, `HDEL`。
-   **主要用途**：
    -   **存储对象**：相比于将对象序列化成 JSON 字符串再存入 String 类型，使用 Hash 可以让你独立地获取或修改对象的某个字段，而无需读取和重写整个对象，效率更高。
-   **Predis 示例**：
    ```php
    <?php
    // ... $redis 客户端实例
    
    // 存储一个用户对象的多个字段
    $redis->hset('user:2', 'name', 'Bob');
    $redis->hset('user:2', 'age', 30);
    
    // 或者使用 hmset 批量设置
    $redis->hmset('user:2', [
        'email' => 'bob@example.com',
        'city' => 'New York'
    ]);
    
    // 获取单个字段
    $name = $redis->hget('user:2', 'name');
    echo "用户 2 的姓名: {$name}\n"; // 输出: Bob
    
    // 获取所有字段
    $user_info = $redis->hgetall('user:2');
    print_r($user_info);
    // 输出: ['name' => 'Bob', 'age' => '30', 'email' => 'bob@example.com', 'city' => 'New York']
    ```

## 3. List (列表)

-   **定义**：一个键 (Key) 对应一个**有序**的字符串元素列表。它是一个双向链表，支持在列表的头部 (Left) 和尾部 (Right) 进行快速的推入 (Push) 和弹出 (Pop) 操作。
-   **PHP 类比**：一个简单的索引数组，`$logs = ['error message 1', 'error message 2'];`。
-   **核心命令**：`LPUSH`/`RPUSH`, `LPOP`/`RPOP`, `LLEN` (获取长度), `LRANGE` (获取指定范围的元素)。
-   **主要用途**：
    -   **消息队列/任务队列**：一个生产者通过 `LPUSH` 向队列中添加任务，一个或多个消费者通过 `RPOP`（或阻塞版的 `BRPOP`）来获取并处理任务。
    -   **时间线 (Timeline)**：存储用户的最新动态、最新发布的文章列表等。
-   **Predis 示例**：
    ```php
    <?php
    // ... $redis 客户端实例
    
    // 模拟任务入队
    $redis->lpush('tasks:email_queue', '{"to":"user1@a.com","subject":"Welcome"}');
    $redis->lpush('tasks:email_queue', '{"to":"user2@b.com","subject":"Password Reset"}');
    
    // 模拟消费者处理任务
    $taskJson = $redis->rpop('tasks:email_queue');
    if ($taskJson) {
        $task = json_decode($taskJson, true);
        echo "正在处理发送给 {$task['to']} 的邮件...\n";
    }
    
    // 查看队列中剩余的元素
    $remaining_tasks = $redis->lrange('tasks:email_queue', 0, -1);
    print_r($remaining_tasks);
    ```

## 4. Set (集合)

-   **定义**：一个键 (Key) 对应一个**无序**且**元素唯一**的字符串集合。
-   **PHP 类比**：一个值唯一的数组，`array_unique(['tagA', 'tagB', 'tagA'])` 的结果。
-   **核心命令**：`SADD`, `SREM` (移除元素), `SMEMBERS` (获取所有成员), `SISMEMBER` (判断成员是否存在), `SINTER` (交集), `SUNION` (并集), `SDIFF` (差集)。
-   **主要用途**：
    -   **标签系统**：为一个文章或商品添加标签，天然去重。
    -   **共同好友/共同关注**：使用 `SINTER` 可以极快地计算出两个用户的共同好友。
    -   **抽奖系统**：存储所有参与抽奖的用户 ID，保证唯一性。
-   **Predis 示例**：
    ```php
    <?php
    // ... $redis 客户端实例
    
    // 为文章 101 添加标签
    $redis->sadd('post:101:tags', 'php', 'database', 'redis');
    $redis->sadd('post:101:tags', 'php'); // 重复添加 'php'，但集合中只会有一个

    // 为文章 102 添加标签
    $redis->sadd('post:102:tags', 'php', 'laravel');

    // 获取文章 101 的所有标签
    $tags = $redis->smembers('post:101:tags');
    print_r($tags); // 输出: ['php', 'database', 'redis'] (顺序不保证)
    
    // 查找同时有 'php' 和 'redis' 标签的文章 (此处逻辑简化，实际需要反向索引)
    
    // 计算同时喜欢 'php' 和 'laravel' 的用户 (假设有 user:1:likes 和 user:2:likes 两个集合)
    // $common_likes = $redis->sinter('user:1:likes', 'user:2:likes');
    ```

## 5. Sorted Set (有序集合 / ZSET)

-   **定义**：类似于集合，但每个元素都关联了一个**分数 (score)**，是一个浮点数。Redis 会根据这个分数对集合中的元素进行**排序**。集合中的元素仍然是唯一的，但分数可以重复。
-   **PHP 类比**：一个根据值（分数）排序的关联数组。
-   **核心命令**：`ZADD`, `ZREM`, `ZRANGE` (按分升序获取), `ZREVRANGE` (按分降序获取), `ZRANK` (获取排名), `ZSCORE` (获取分数)。
-   **主要用途**：
    -   **排行榜**：游戏玩家积分榜、文章热度榜、销售额排行榜等。
    -   **范围查找**：可以快速获取积分在某个范围内的所有用户。
-   **Predis 示例**：
    ```php
    <?php
    // ... $redis 客户端实例
    
    // 添加玩家到游戏排行榜
    $redis->zadd('leaderboard:game1', 1500, 'player1'); // 'player1' 的分数是 1500
    $redis->zadd('leaderboard:game1', 2200, 'player2');
    $redis->zadd('leaderboard:game1', 1800, 'player3');
    
    // player1 又得了 300 分
    $redis->zincrby('leaderboard:game1', 300, 'player1'); // player1 的分数变为 1800
    
    // 获取排名前 2 的玩家（降序）
    // ZREVRANGE leaderboard:game1 0 1 WITHSCORES
    $top_players = $redis->zrevrange('leaderboard:game1', 0, 1, 'WITHSCORES');
    print_r($top_players);
    // 输出: ['player2' => 2200.0, 'player3' => 1800.0]
    
    // 获取 player3 的排名
    $rank = $redis->zrevrank('leaderboard:game1', 'player3'); // 从 0 开始
    echo "Player 3 的排名是: " . ($rank + 1);
    ```
---

## 练习任务

1.  **对象缓存**：
    创建一个 `User` 类，实例化一个对象后，将其序列化为 JSON 字符串，并使用 Redis 的 **String** 类型进行缓存。键为 `user:{id}`。然后再编写代码从 Redis 中取出该字符串，并反序列化回 `User` 对象。

2.  **购物车**：
    使用 Redis 的 **Hash** 类型来模拟一个用户的购物车。键为 `cart:{user_id}`，字段为 `product_id`，值为购买数量。实现添加商品、增加商品数量、获取购物车所有商品的功能。

3.  **实时活动流**：
    使用 Redis 的 **List** 类型，实现一个记录用户最近 10 次操作的功能。每次用户有新操作（如“登录”、“发表文章”），就使用 `LPUSH` 将操作记录推入列表，并使用 `LTRIM` 命令保持列表长度不超过 10。

# 6.5.1 Redis 基础与 PHP 客户端

## 概述

**Redis (Remote Dictionary Server)** 是一个开源的、基于**内存**的、高性能**键值对 (Key-Value)** 数据库。由于其所有数据都存储在内存中，它的读写速度快得惊人，通常可以达到数十万次每秒的 QPS（每秒查询率）。

虽然我们称之为“数据库”，但它与 MySQL 等关系型数据库（RDBMS）有本质不同。Redis 的数据模型不是表，而是一个巨大的“字典”或“哈希表”，你通过一个“键 (Key)”来存储和获取一个“值 (Value)”。

本节将介绍 Redis 的核心特性、为什么它非常适合做缓存，以及如何在 PHP 中连接并开始使用它。

## Redis 的核心特性

1.  **极高的性能**：数据存储在内存中，所有操作几乎都是内存操作，因此延迟极低。
2.  **丰富的数据类型**：与只能存储简单字符串的 Memcached 不同，Redis 的“值”支持多种数据结构，如字符串 (Strings)、哈希 (Hashes)、列表 (Lists)、集合 (Sets)、有序集合 (Sorted Sets) 等。这使得它可以应对更复杂的业务场景。
3.  **持久化**：虽然是内存数据库，但 Redis 提供了两种持久化机制（RDB 和 AOF），可以将内存中的数据定期或实时地保存到磁盘，以防止服务器断电后数据丢失。
4.  **原子操作**：Redis 的所有单个命令操作都是**原子性**的。它的核心服务采用单线程模型来处理客户端命令，这意味着在执行一个命令时，不会被其他命令打断，从而避免了并发场景下的数据竞争问题。
5.  **多功能性**：除了作为缓存，Redis 还被广泛用于会话存储、消息队列、实时排行榜、计数器、分布式锁等多种场景。

## 为什么使用 Redis 做缓存？

在典型的 Web 应用架构中，用户的请求首先到达应用服务器（运行 PHP 代码），然后应用服务器再去查询数据库（如 MySQL）。

`用户 -> 应用服务器 -> 数据库`

数据库通常将数据存储在磁盘（如 SSD）上，磁盘的访问速度与内存相比，有几个数量级的差距。当访问量增大时，频繁的磁盘 I/O 会成为整个系统的瓶颈。

引入 Redis 作为缓存后，架构变为：

`用户 -> 应用服务器 -> Redis 缓存 -> 数据库`

**工作流程**：
1.  应用服务器收到一个读请求后，**首先查询 Redis 缓存**。
2.  如果缓存在，则直接将数据返回给用户（称为**缓存命中, Cache Hit**），整个过程极快，无需访问数据库。
3.  如果缓存不存在（称为**缓存未命中, Cache Miss**），应用服务器再去查询数据库，获取数据。
4.  获取到数据后，应用服务器一方面将数据返回给用户，另一方面将这份数据**存入 Redis 缓存**中，以便下一次相同的请求能够命中缓存。

通过这种方式，Redis 帮助数据库“吸收”了绝大部分的读请求，极大地降低了数据库的负载，提升了整个应用的响应速度和吞吐量。

## 安装和运行 Redis

对于本地开发，最简单、最推荐的方式是使用 **Docker**。

1.  **拉取并运行 Redis 镜像**:
    在你的终端中执行以下命令，它会自动下载 Redis 镜像并在后台启动一个名为 `my-redis` 的容器。
    ```bash
    docker run --name my-redis -p 6379:6379 -d redis
    ```
    - `-p 6379:6379` 将你本机的 6379 端口映射到容器的 6379 端口（Redis 默认端口）。
    - `-d` 表示在后台运行。

2.  **测试连接**:
    你可以使用 `redis-cli` 工具来测试连接。
    ```bash
    # 如果你本地安装了 redis-cli
    redis-cli ping
    
    # 或者通过 Docker 执行
    docker exec -it my-redis redis-cli ping
    ```
    如果服务器正常运行，它会返回 `PONG`。

## PHP 的 Redis 客户端

PHP 需要通过一个“客户端”库来与 Redis 服务器进行通信。最流行的选择有两个：

1.  **PhpRedis (PHP 扩展)**
    -   **描述**：一个用 C 语言编写的 PHP 扩展。因为它更接近底层，所以性能非常高，是**生产环境的首选**。
    -   **安装**：需要系统权限来安装。例如 `pecl install redis`，然后在 `php.ini` 中添加 `extension=redis.so`。
    -   **使用**：`$redis = new Redis(); $redis->connect('127.0.0.1', 6379);`

2.  **Predis (Composer 包)**
    -   **描述**：一个纯 PHP 实现的客户端库。无需修改 `php.ini`，只需通过 Composer 安装即可使用。
    -   **优点**：非常方便，特别适合开发环境或无法安装 C 扩展的虚拟主机环境。
    -   **安装**：`composer require predis/predis`

**为了方便学习和快速上手，本教程将使用 Predis 作为示例。**

## 使用 Predis 连接 Redis

### 1. 安装 Predis
```bash
composer require predis/predis
```

### 2. 连接并执行命令

创建一个 PHP 文件，引入 Composer 的自动加载器，然后实例化 `Predis\Client`。

**`redis_test.php`**
```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

// 连接参数可以是一个 URL，或一个数组
// 默认连接到 'tcp://127.0.0.1:6379'
$redis_params = [
    'scheme' => 'tcp',
    'host'   => '127.0.0.1',
    'port'   => 6379,
];

try {
    // 1. 实例化 Predis 客户端
    $redis = new Predis\Client($redis_params);
    echo "成功连接到 Redis！\n";

    // 2. 使用 SET 命令设置一个值
    // SET user:1:name "Alice"
    $redis->set('user:1:name', 'Alice');

    // 3. 使用 GET 命令获取一个值
    $name = $redis->get('user:1:name');
    echo "获取到用户姓名: " . $name . "\n"; // 输出: Alice

    // 4. 使用 EXISTS 命令检查键是否存在
    $exists = $redis->exists('user:1:name');
    echo "键 'user:1:name' 是否存在? " . ($exists ? '是' : '否') . "\n"; // 输出: 是

    // 5. 使用 DEL 命令删除一个键
    $redis->del('user:1:name');
    echo "键 'user:1:name' 已被删除。\n";

    // 6. 再次检查
    $exists = $redis->exists('user:1:name');
    echo "键 'user:1:name' 是否存在? " . ($exists ? '是' : '否') . "\n"; // 输出: 否
    
    // 7. 给键设置过期时间 (单位：秒)
    $redis->set('session:xyz', 'some_data', 'EX', 10); // 10秒后自动过期
    echo "session:xyz 已设置，10秒后过期。\n";


} catch (\Exception $e) {
    die("无法连接到 Redis: " . $e->getMessage());
}
```
**键命名约定**：
在 Redis 中，一个常见的良好实践是使用冒号 `:` 来对键进行分层，形成类似命名空间的效果，例如 `object-type:id:field`（`user:1:name`）。这使得键更具可读性，并且便于管理。

---

## 练习任务

1.  **环境搭建**：
    使用 Docker 在你的本地计算机上成功启动一个 Redis 容器，并使用 `redis-cli ping` 命令确认其正在运行。

2.  **安装 Predis 并测试**：
    在一个新项目中，使用 Composer 安装 Predis。编写一个 PHP 脚本，连接到你刚刚启动的 Redis 容器，并成功执行 `SET` 和 `GET` 命令。

3.  **封装连接**：
    创建一个 `RedisConnection` 类或一个函数，它负责返回一个单例的 `Predis\Client` 实例，以避免在代码中重复创建连接。在你的其他脚本中调用这个类/函数来获取 Redis 客户端。

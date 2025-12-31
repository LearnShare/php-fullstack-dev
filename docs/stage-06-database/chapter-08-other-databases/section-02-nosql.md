# 6.8.2 NoSQL 数据库

## 概述

NoSQL（Not Only SQL）数据库是一类非关系型数据库，适用于大规模数据存储和高并发场景。了解 NoSQL 数据库的特点和使用场景，有助于选择合适的数据库系统。

本节详细介绍 NoSQL 数据库的概念、分类、特点、使用场景、PHP 连接方式等，帮助零基础学员了解 NoSQL 数据库的特点和选择方法。

**主要内容**：
- NoSQL 数据库概述
- NoSQL 数据库分类（文档型、键值型、列族型、图型）
- MongoDB 介绍和使用
- Redis 作为 NoSQL 的使用
- NoSQL 与关系型数据库的对比
- 数据库选择建议
- 完整示例和最佳实践

---

## 特性

- **多样性**：了解不同 NoSQL 数据库
- **高性能**：适合高并发场景
- **可扩展性**：易于水平扩展
- **灵活性**：灵活的数据模型
- **适用场景**：了解适用场景

---

## NoSQL 数据库概述

### 什么是 NoSQL

NoSQL（Not Only SQL）是一类非关系型数据库，不使用传统的表结构，而是使用其他数据模型。

**主要特点**：

- **非关系型**：不使用表结构
- **灵活模式**：灵活的数据模型
- **高性能**：适合高并发场景
- **可扩展**：易于水平扩展

### NoSQL 的优势

**主要优势**：

- **高性能**：读写性能高
- **可扩展性**：易于水平扩展
- **灵活性**：灵活的数据模型
- **成本**：开源方案成本低

### NoSQL 的劣势

**主要劣势**：

- **一致性**：一致性保证较弱
- **功能**：功能相对简单
- **学习曲线**：学习曲线较陡
- **工具**：工具和生态相对较少

---

## NoSQL 数据库分类

### 文档型数据库

文档型数据库以文档（如 JSON）为单位存储数据。

**代表**：

- **MongoDB**：最流行的文档型数据库
- **CouchDB**：Apache 的文档数据库
- **RavenDB**：.NET 平台的文档数据库

**特点**：

- **灵活结构**：文档结构灵活
- **嵌套数据**：支持嵌套数据结构
- **查询能力**：强大的查询能力

### 键值型数据库

键值型数据库以键值对的形式存储数据。

**代表**：

- **Redis**：内存键值数据库
- **Memcached**：分布式内存缓存
- **Amazon DynamoDB**：AWS 的键值数据库

**特点**：

- **简单**：数据结构简单
- **高性能**：读写性能极高
- **适用场景**：缓存、会话存储

### 列族型数据库

列族型数据库以列族为单位存储数据。

**代表**：

- **Cassandra**：Apache 的列族数据库
- **HBase**：Hadoop 的列族数据库
- **Amazon SimpleDB**：AWS 的列族数据库

**特点**：

- **大规模**：适合大规模数据
- **分布式**：天然支持分布式
- **适用场景**：大数据分析

### 图型数据库

图型数据库以图结构存储数据，适合关系复杂的数据。

**代表**：

- **Neo4j**：最流行的图数据库
- **ArangoDB**：多模型数据库
- **Amazon Neptune**：AWS 的图数据库

**特点**：

- **关系查询**：强大的关系查询能力
- **复杂关系**：适合复杂关系数据
- **适用场景**：社交网络、推荐系统

---

## MongoDB

### MongoDB 简介

MongoDB 是一个流行的文档型 NoSQL 数据库，使用 BSON（Binary JSON）格式存储数据。

**主要特点**：

- **文档存储**：以文档为单位存储
- **灵活模式**：无需预定义模式
- **查询能力**：强大的查询能力
- **水平扩展**：易于水平扩展

### MongoDB 安装

**Ubuntu/Debian**：

```bash
# 导入 MongoDB 公钥
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -

# 添加 MongoDB 仓库
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# 安装 MongoDB
sudo apt-get update
sudo apt-get install -y mongodb-org

# 启动服务
sudo systemctl start mongod
sudo systemctl enable mongod
```

**CentOS/RHEL**：

```bash
# 创建 MongoDB 仓库文件
sudo vi /etc/yum.repos.d/mongodb-org-7.0.repo

# 添加以下内容
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc

# 安装 MongoDB
sudo yum install -y mongodb-org

# 启动服务
sudo systemctl start mongod
sudo systemctl enable mongod
```

### PHP 连接 MongoDB

使用 MongoDB PHP 扩展连接 MongoDB。

**安装扩展**：

```bash
# 使用 PECL 安装
pecl install mongodb

# 在 php.ini 中添加
extension=mongodb.so
```

**连接示例**：

```php
<?php
declare(strict_types=1);

require_once 'vendor/autoload.php';

use MongoDB\Client;

try {
    $client = new Client('mongodb://localhost:27017');
    $db = $client->selectDatabase('mydb');
    
    echo "连接成功\n";
} catch (Exception $e) {
    die('连接失败: ' . $e->getMessage());
}
```

### MongoDB 基本操作

**插入文档**：

```php
<?php
declare(strict_types=1);

use MongoDB\Client;

$client = new Client('mongodb://localhost:27017');
$collection = $client->mydb->users;

// 插入单个文档
$result = $collection->insertOne([
    'username' => 'john',
    'email' => 'john@example.com',
    'created_at' => new MongoDB\BSON\UTCDateTime()
]);

echo "插入的文档 ID: " . $result->getInsertedId() . "\n";

// 插入多个文档
$result = $collection->insertMany([
    [
        'username' => 'jane',
        'email' => 'jane@example.com',
        'created_at' => new MongoDB\BSON\UTCDateTime()
    ],
    [
        'username' => 'bob',
        'email' => 'bob@example.com',
        'created_at' => new MongoDB\BSON\UTCDateTime()
    ]
]);

echo "插入的文档数量: " . $result->getInsertedCount() . "\n";
```

**查询文档**：

```php
<?php
declare(strict_types=1);

use MongoDB\Client;

$client = new Client('mongodb://localhost:27017');
$collection = $client->mydb->users;

// 查询单个文档
$user = $collection->findOne(['username' => 'john']);
print_r($user);

// 查询多个文档
$users = $collection->find(['status' => 'active']);

foreach ($users as $user) {
    echo $user['username'] . "\n";
}

// 条件查询
$users = $collection->find([
    'age' => ['$gte' => 18, '$lte' => 65]
]);

// 排序和限制
$users = $collection->find()
    ->sort(['created_at' => -1])
    ->limit(10);
```

**更新文档**：

```php
<?php
declare(strict_types=1);

use MongoDB\Client;

$client = new Client('mongodb://localhost:27017');
$collection = $client->mydb->users;

// 更新单个文档
$result = $collection->updateOne(
    ['username' => 'john'],
    ['$set' => ['email' => 'newemail@example.com']]
);

echo "更新的文档数量: " . $result->getModifiedCount() . "\n";

// 更新多个文档
$result = $collection->updateMany(
    ['status' => 'inactive'],
    ['$set' => ['status' => 'active']]
);

echo "更新的文档数量: " . $result->getModifiedCount() . "\n";
```

**删除文档**：

```php
<?php
declare(strict_types=1);

use MongoDB\Client;

$client = new Client('mongodb://localhost:27017');
$collection = $client->mydb->users;

// 删除单个文档
$result = $collection->deleteOne(['username' => 'john']);
echo "删除的文档数量: " . $result->getDeletedCount() . "\n";

// 删除多个文档
$result = $collection->deleteMany(['status' => 'inactive']);
echo "删除的文档数量: " . $result->getDeletedCount() . "\n";
```

---

## Redis 作为 NoSQL

### Redis 的 NoSQL 特性

Redis 虽然主要用于缓存，但也可以作为 NoSQL 数据库使用。

**特点**：

- **键值存储**：键值对存储
- **多种数据类型**：支持多种数据类型
- **持久化**：支持数据持久化
- **高性能**：读写性能极高

### Redis 数据模型

**String**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 存储字符串
$redis->set('user:1:name', 'John');
$name = $redis->get('user:1:name');
```

**Hash**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 存储哈希
$redis->hSet('user:1', 'name', 'John');
$redis->hSet('user:1', 'email', 'john@example.com');

// 获取哈希
$user = $redis->hGetAll('user:1');
```

**List**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 存储列表
$redis->lPush('users', 'John');
$redis->lPush('users', 'Jane');

// 获取列表
$users = $redis->lRange('users', 0, -1);
```

**Set**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 存储集合
$redis->sAdd('tags', 'php');
$redis->sAdd('tags', 'mysql');

// 获取集合
$tags = $redis->sMembers('tags');
```

---

## NoSQL 与关系型数据库对比

### 数据模型

**关系型数据库**：

- **表结构**：固定的表结构
- **关系**：表之间建立关系
- **模式**：需要预定义模式

**NoSQL 数据库**：

- **灵活结构**：灵活的数据结构
- **文档/键值**：文档或键值对
- **无模式**：无需预定义模式

### 性能

**关系型数据库**：

- **复杂查询**：支持复杂查询
- **事务**：支持 ACID 事务
- **一致性**：强一致性

**NoSQL 数据库**：

- **简单查询**：简单查询性能高
- **最终一致性**：最终一致性
- **高并发**：适合高并发场景

### 扩展性

**关系型数据库**：

- **垂直扩展**：主要通过垂直扩展
- **水平扩展**：水平扩展较困难

**NoSQL 数据库**：

- **水平扩展**：易于水平扩展
- **分布式**：天然支持分布式

---

## 数据库选择建议

### 选择因素

选择数据库时需要考虑的因素：

- **数据模型**：数据结构和关系
- **性能要求**：读写性能要求
- **一致性要求**：一致性要求
- **扩展性**：扩展性需求
- **团队经验**：团队技术栈和经验

### 选择建议

**关系型数据库（MySQL/PostgreSQL）**：

- **适用场景**：需要复杂查询、事务、强一致性
- **优势**：功能丰富、成熟稳定
- **劣势**：扩展性相对较差

**文档型数据库（MongoDB）**：

- **适用场景**：灵活数据结构、高并发读写
- **优势**：灵活模式、易于扩展
- **劣势**：复杂查询能力较弱

**键值型数据库（Redis）**：

- **适用场景**：缓存、会话存储、简单数据
- **优势**：高性能、简单易用
- **劣势**：功能相对简单

**图型数据库（Neo4j）**：

- **适用场景**：复杂关系数据、社交网络
- **优势**：强大的关系查询能力
- **劣势**：学习曲线较陡

---

## 完整示例

### MongoDB 用户管理系统

```php
<?php
declare(strict_types=1);

require_once 'vendor/autoload.php';

use MongoDB\Client;
use MongoDB\BSON\UTCDateTime;

class UserService
{
    private $collection;
    
    public function __construct(Client $client, string $dbName, string $collectionName)
    {
        $this->collection = $client->selectCollection($dbName, $collectionName);
    }
    
    public function createUser(array $userData): string
    {
        $userData['created_at'] = new UTCDateTime();
        $userData['updated_at'] = new UTCDateTime();
        
        $result = $this->collection->insertOne($userData);
        return (string)$result->getInsertedId();
    }
    
    public function getUserById(string $id): ?array
    {
        $user = $this->collection->findOne(['_id' => new MongoDB\BSON\ObjectId($id)]);
        return $user ? $user->toArray() : null;
    }
    
    public function getUserByEmail(string $email): ?array
    {
        $user = $this->collection->findOne(['email' => $email]);
        return $user ? $user->toArray() : null;
    }
    
    public function updateUser(string $id, array $updateData): bool
    {
        $updateData['updated_at'] = new UTCDateTime();
        
        $result = $this->collection->updateOne(
            ['_id' => new MongoDB\BSON\ObjectId($id)],
            ['$set' => $updateData]
        );
        
        return $result->getModifiedCount() > 0;
    }
    
    public function deleteUser(string $id): bool
    {
        $result = $this->collection->deleteOne(['_id' => new MongoDB\BSON\ObjectId($id)]);
        return $result->getDeletedCount() > 0;
    }
    
    public function listUsers(int $page = 1, int $perPage = 10): array
    {
        $skip = ($page - 1) * $perPage;
        
        $users = $this->collection->find()
            ->sort(['created_at' => -1])
            ->skip($skip)
            ->limit($perPage);
        
        $result = [];
        foreach ($users as $user) {
            $result[] = $user->toArray();
        }
        
        return $result;
    }
}

// 使用
$client = new Client('mongodb://localhost:27017');
$userService = new UserService($client, 'mydb', 'users');

// 创建用户
$userId = $userService->createUser([
    'username' => 'john',
    'email' => 'john@example.com',
    'status' => 'active'
]);

// 查询用户
$user = $userService->getUserById($userId);
print_r($user);

// 更新用户
$userService->updateUser($userId, ['status' => 'inactive']);

// 列出用户
$users = $userService->listUsers(1, 10);
print_r($users);
```

---

## 使用场景

### 高并发场景

NoSQL 数据库适合高并发读写场景。

### 灵活数据结构

适合数据结构灵活、变化频繁的场景。

### 大规模数据

适合大规模数据存储和分析。

---

## 注意事项

### 一致性

NoSQL 数据库通常提供最终一致性，不是强一致性。

### 功能限制

NoSQL 数据库的功能相对简单，复杂查询能力较弱。

### 学习曲线

NoSQL 数据库的学习曲线可能较陡。

---

## 常见问题

### NoSQL 和关系型数据库有什么区别？

NoSQL 使用灵活的数据模型，关系型数据库使用固定的表结构。

### 什么时候使用 NoSQL？

适合高并发、灵活数据结构、大规模数据的场景。

### MongoDB 和 MySQL 如何选择？

根据数据模型、性能要求、一致性需求等因素选择。

---

## 最佳实践

### 了解特点

了解不同 NoSQL 数据库的特点和适用场景。

### 混合使用

可以混合使用关系型数据库和 NoSQL 数据库。

### 数据一致性

注意 NoSQL 数据库的一致性特点。

---

## 练习任务

1. **MongoDB 使用**
   - 安装 MongoDB
   - 使用 PHP 连接 MongoDB
   - 执行基本 CRUD 操作
   - 测试 MongoDB 功能

2. **Redis 作为 NoSQL**
   - 使用 Redis 存储用户数据
   - 实现用户 CRUD 操作
   - 测试 Redis 功能
   - 对比性能差异

3. **数据模型设计**
   - 设计 MongoDB 数据模型
   - 实现复杂查询
   - 测试查询性能
   - 优化数据模型

4. **数据库对比**
   - 对比 MySQL 和 MongoDB
   - 测试性能差异
   - 分析适用场景
   - 编写对比报告

5. **综合应用**
   - 创建一个使用 NoSQL 数据库的应用
   - 实现所有功能
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.8.1 其他关系型数据库](section-01-relational.md)
- [6.5 Redis 缓存策略与性能优化](../chapter-05-redis/readme.md)
- [6.1 数据库设计基础](../chapter-01-database-design/readme.md)

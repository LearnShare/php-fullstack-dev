# 6.8.3 集成测试

## 概述

集成测试验证组件之间的交互。本节介绍集成测试概念、数据库测试、API 测试、测试数据管理等内容。

## 集成测试概念

### 什么是集成测试

集成测试验证多个组件协同工作的正确性。

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserServiceIntegrationTest extends TestCase
{
    private PDO $pdo;
    private UserService $service;
    
    protected function setUp(): void
    {
        // 使用测试数据库
        $this->pdo = new PDO('sqlite::memory:');
        $this->createSchema();
        $this->service = new UserService($this->pdo);
    }
    
    private function createSchema(): void
    {
        $this->pdo->exec("
            CREATE TABLE users (
                id INTEGER PRIMARY KEY,
                name TEXT,
                email TEXT
            )
        ");
    }
    
    public function testCreateAndRetrieveUser(): void
    {
        $user = $this->service->createUser('John', 'john@example.com');
        $retrieved = $this->service->getUser($user->getId());
        
        $this->assertEquals('John', $retrieved->getName());
    }
}
```

## 数据库测试

### 使用事务回滚

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class DatabaseTest extends TestCase
{
    private PDO $pdo;
    
    protected function setUp(): void
    {
        $this->pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
        $this->pdo->beginTransaction();
    }
    
    protected function tearDown(): void
    {
        $this->pdo->rollBack();
    }
    
    public function testInsert(): void
    {
        $stmt = $this->pdo->prepare("INSERT INTO users (name) VALUES (?)");
        $stmt->execute(['John']);
        $id = $this->pdo->lastInsertId();
        
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $user = $stmt->fetch();
        
        $this->assertEquals('John', $user['name']);
    }
}
```

## API 测试

### 使用 HTTP 客户端

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use GuzzleHttp\Client;

class ApiTest extends TestCase
{
    private Client $client;
    
    protected function setUp(): void
    {
        $this->client = new Client(['base_uri' => 'http://localhost:8000']);
    }
    
    public function testGetUser(): void
    {
        $response = $this->client->get('/api/users/1');
        $this->assertEquals(200, $response->getStatusCode());
        
        $data = json_decode($response->getBody(), true);
        $this->assertEquals('John', $data['name']);
    }
    
    public function testCreateUser(): void
    {
        $response = $this->client->post('/api/users', [
            'json' => [
                'name' => 'Jane',
                'email' => 'jane@example.com',
            ],
        ]);
        
        $this->assertEquals(201, $response->getStatusCode());
    }
}
```

## 测试数据管理

### 使用 Fixtures

```php
<?php
declare(strict_types=1);

class UserFixtures
{
    public static function createUser(PDO $pdo): int
    {
        $stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
        $stmt->execute(['Test User', 'test@example.com']);
        return (int) $pdo->lastInsertId();
    }
    
    public static function createUsers(PDO $pdo, int $count): array
    {
        $ids = [];
        for ($i = 0; $i < $count; $i++) {
            $ids[] = self::createUser($pdo);
        }
        return $ids;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserServiceIntegrationTest extends TestCase
{
    private PDO $pdo;
    private UserService $service;
    
    protected function setUp(): void
    {
        $this->pdo = new PDO('sqlite::memory:');
        $this->createSchema();
        $this->service = new UserService($this->pdo);
    }
    
    private function createSchema(): void
    {
        $this->pdo->exec("
            CREATE TABLE users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT NOT NULL UNIQUE
            )
        ");
    }
    
    public function testUserLifecycle(): void
    {
        // 创建
        $user = $this->service->createUser('John', 'john@example.com');
        $this->assertNotNull($user->getId());
        
        // 读取
        $retrieved = $this->service->getUser($user->getId());
        $this->assertEquals('John', $retrieved->getName());
        
        // 更新
        $this->service->updateUser($user->getId(), ['name' => 'Jane']);
        $updated = $this->service->getUser($user->getId());
        $this->assertEquals('Jane', $updated->getName());
        
        // 删除
        $this->service->deleteUser($user->getId());
        $this->assertNull($this->service->getUser($user->getId()));
    }
}
```

## 最佳实践

1. **隔离测试**：每个测试使用独立数据
2. **清理数据**：测试后清理测试数据
3. **使用 Fixtures**：复用测试数据创建逻辑
4. **真实环境**：尽可能接近生产环境

## 注意事项

1. 避免测试之间的依赖
2. 使用事务或内存数据库加速测试
3. 清理测试数据
4. 测试应该可重复执行

## 练习

1. 编写一个集成测试，测试用户服务的完整生命周期。

2. 实现 API 集成测试，测试 REST API 端点。

3. 创建测试 Fixtures，管理测试数据。

4. 实现测试数据库管理，使用事务回滚。

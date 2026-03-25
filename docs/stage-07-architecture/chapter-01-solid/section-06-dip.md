# 7.1.6 依赖倒置原则（DIP）

## 概述

依赖倒置原则（Dependency Inversion Principle，简称 DIP）是 SOLID 原则中的最后一个原则，也是最具影响力的原则之一。该原则包含两个核心要点：

1. 高层模块不应该依赖低层模块，两者都应该依赖抽象
2. 抽象不应该依赖细节，细节应该依赖抽象

这个原则彻底改变了我们思考依赖关系的方式。在传统的分层架构中，高层模块（如业务逻辑层）依赖于低层模块（如数据访问层）。但依赖倒置原则建议我们通过引入抽象来反转这种依赖关系，使高层和低层都依赖于抽象。

依赖倒置原则是实现松耦合、可测试、可维护代码的基础。通过依赖抽象而非具体实现，我们可以轻松地替换依赖的实现，而不需要修改使用这些依赖的代码。这对于单元测试、模块解耦、系统扩展都至关重要。

理解并应用依赖倒置原则是成为高级 PHP 开发者的关键一步。本节将详细介绍依赖倒置原则的概念、依赖注入的实现方法、控制反转的概念，以及如何在实际开发中应用这一原则。

**主要内容**：
- 依赖倒置原则的定义和重要性
- 依赖注入的三种方式
- 控制反转和 IoC 容器
- 违反 DIP 的常见情况
- 符合 DIP 的重构示例
- 依赖管理和最佳实践

## 特性

依赖倒置原则具有以下核心特性：

- **依赖抽象**：依赖接口或抽象类，而不是具体实现
- **解耦**：降低模块之间的耦合度
- **可替换性**：可以轻松替换依赖的实现
- **可测试性**：更容易进行单元测试
- **灵活性**：系统更容易适应变化

## 核心概念

### 什么是依赖倒置

"倒置"指的是与传统依赖关系的反向。在传统架构中：
- 高层模块 → 依赖 → 低层模块

在依赖倒置原则中：
- 高层模块 → 依赖 → 抽象（接口/抽象类）
- 低层模块 → 实现 → 抽象（接口/抽象类）

这就是"倒置"：依赖关系从"高层依赖低层"变成了"高层和低层都依赖抽象"。

### 高层模块与低层模块

- **高层模块**：包含核心业务逻辑、策略、规则的模块
- **低层模块**：提供基础设施功能（如数据库访问、文件操作、网络请求等）

高层模块是系统存在的理由，低层模块是支撑高层模块的基础。

### 为什么要倒置

倒置依赖关系的好处：
- 高层模块不依赖于低层模块的实现细节
- 低层模块可以独立变化和演进
- 系统更加灵活，更容易扩展
- 更利于单元测试

## 依赖注入的三种方式

### 方式 1：构造函数注入

通过构造函数将依赖传入类：

```php
<?php
declare(strict_types=1);

interface Logger
{
    public function log(string $message): void;
}

class FileLogger implements Logger
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function log(string $message): void
    {
        $line = date('Y-m-d H:i:s') . " " . $message . "\n";
        file_put_contents($this->filename, $line, FILE_APPEND);
    }
}

class UserService
{
    private Logger $logger;
    
    // 构造函数注入
    public function __construct(Logger $logger)
    {
        $this->logger = $logger;
    }
    
    public function createUser(string $name): void
    {
        $this->logger->log("Creating user: {$name}");
        // 创建用户的逻辑
        $this->logger->log("User created: {$name}");
    }
}
```

**优点**：
- 依赖在对象创建时就确定
- 对象创建后状态不可变
- 易于理解和使用

**缺点**：
- 如果依赖很多，构造函数会变得复杂
- 不适合可选依赖

### 方式 2：Setter 注入

通过 setter 方法注入依赖：

```php
<?php
declare(strict_types=1);

interface Cache
{
    public function get(string $key): ?string;
    public function set(string $key, string $value): void;
}

class RedisCache implements Cache
{
    private string $host;
    
    public function __construct(string $host)
    {
        $this->host = $host;
    }
    
    public function get(string $key): ?string
    {
        // Redis 获取逻辑
        return null;
    }
    
    public function set(string $key, string $value): void
    {
        // Redis 设置逻辑
    }
}

class ProductService
{
    private ?Cache $cache = null;
    
    // Setter 注入
    public function setCache(Cache $cache): void
    {
        $this->cache = $cache;
    }
    
    public function getProduct(int $id): array
    {
        // 尝试从缓存获取
        if ($this->cache !== null) {
            $cached = $this->cache->get("product:{$id}");
            if ($cached !== null) {
                return json_decode($cached, true);
            }
        }
        
        // 从数据库获取
        $product = ['id' => $id, 'name' => 'Product'];
        
        // 存入缓存
        if ($this->cache !== null) {
            $this->cache->set("product:{$id}", json_encode($product));
        }
        
        return $product;
    }
}
```

**优点**：
- 依赖可以在创建后设置
- 适合可选依赖
- 可以动态切换依赖实现

**缺点**：
- 对象状态可能变化
- 依赖可能在使用时未设置

### 方式 3：接口注入

通过接口定义注入方法：

```php
<?php
declare(strict_types=1);

interface EmailAware
{
    public function setEmailService(EmailService $service): void;
}

class EmailService
{
    public function send(string $to, string $subject, string $body): void
    {
        echo "Sending email to: {$to}\n";
    }
}

class NotificationService implements EmailAware
{
    private ?EmailService $emailService = null;
    
    public function setEmailService(EmailService $service): void
    {
        $this->emailService = $service;
    }
    
    public function notify(string $email, string $message): void
    {
        if ($this->emailService === null) {
            throw new RuntimeException("Email service not configured");
        }
        
        $this->emailService->send($email, "Notification", $message);
    }
}
```

**优点**：
- 明确声明依赖需求
- IDE 可以提示需要的依赖

**缺点**：
- 需要定义额外的接口
- 可能导致接口数量增加

## 违反 DIP 的示例

### 示例 1：直接依赖具体类

```php
<?php
declare(strict_types=1);

// 违反 DIP：直接依赖具体类
class OrderService
{
    private MySQLConnection $connection;
    
    public function __construct()
    {
        // 直接创建具体类的实例
        $this->connection = new MySQLConnection(
            'localhost',
            'shop',
            'root',
            'password'
        );
    }
    
    public function createOrder(array $data): int
    {
        // 直接使用具体类
        $stmt = $this->connection->prepare(
            "INSERT INTO orders (customer_id, total) VALUES (?, ?)"
        );
        $stmt->execute([$data['customer_id'], $data['total']]);
        
        return (int)$this->connection->lastInsertId();
    }
}

class MySQLConnection
{
    private string $host;
    private string $database;
    private string $username;
    private string $password;
    private ?PDO $pdo = null;
    
    public function __construct(
        string $host,
        string $database,
        string $username,
        string $password
    ) {
        $this->host = $host;
        $this->database = $database;
        $this->username = $username;
        $this->password = $password;
    }
    
    public function prepare(string $sql): PDOStatement
    {
        $this->connect();
        return $this->pdo->prepare($sql);
    }
    
    public function lastInsertId(): string
    {
        $this->connect();
        return $this->pdo->lastInsertId();
    }
    
    private function connect(): void
    {
        if ($this->pdo === null) {
            $dsn = "mysql:host={$this->host};dbname={$this->database}";
            $this->pdo = new PDO($dsn, $this->username, $this->password);
        }
    }
}
```

**问题分析**：
- `OrderService` 直接依赖于 `MySQLConnection` 具体类
- 无法切换到其他数据库（如 PostgreSQL）
- 难以进行单元测试（无法 mock 数据库连接）
- `OrderService` 的创建逻辑与数据库配置耦合

### 示例 2：在方法中创建依赖

```php
<?php
declare(strict_types=1);

// 违反 DIP：在方法中创建依赖
class UserExporter
{
    public function exportToCsv(array $users, string $filename): void
    {
        // 在方法内部创建文件写入器
        $file = new CsvFileWriter($filename);
        
        foreach ($users as $user) {
            $file->writeLine([
                $user['id'],
                $user['name'],
                $user['email']
            ]);
        }
    }
    
    public function exportToJson(array $users, string $filename): void
    {
        // 在方法内部创建 JSON 写入器
        $file = new JsonFileWriter($filename);
        
        $file->write(json_encode($users));
    }
}

class CsvFileWriter
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function writeLine(array $data): void
    {
        $line = implode(',', $data) . "\n";
        file_put_contents($this->filename, $line, FILE_APPEND);
    }
}

class JsonFileWriter
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function write(string $data): void
    {
        file_put_contents($this->filename, $data);
    }
}
```

**问题分析**：
- `UserExporter` 与具体的写入器类耦合
- 无法在运行时切换输出格式
- 难以测试（无法 mock 写入器）

## 符合 DIP 的重构示例

### 重构示例 1：使用构造函数注入

```php
<?php
declare(strict_types=1);

// 定义抽象接口
interface DatabaseConnection
{
    public function prepare(string $sql): PDOStatement;
    public function lastInsertId(): string;
}

interface UserRepository
{
    public function save(array $data): int;
    public function findById(int $id): ?array;
}

// MySQL 实现
class MySQLConnection implements DatabaseConnection
{
    private string $host;
    private string $database;
    private string $username;
    private string $password;
    private ?PDO $pdo = null;
    
    public function __construct(
        string $host,
        string $database,
        string $username,
        string $password
    ) {
        $this->host = $host;
        $this->database = $database;
        $this->username = $username;
        $this->password = $password;
    }
    
    public function prepare(string $sql): PDOStatement
    {
        $this->connect();
        return $this->pdo->prepare($sql);
    }
    
    public function lastInsertId(): string
    {
        $this->connect();
        return $this->pdo->lastInsertId();
    }
    
    private function connect(): void
    {
        if ($this->pdo === null) {
            $dsn = "mysql:host={$this->host};dbname={$this->database}";
            $this->pdo = new PDO($dsn, $this->username, $this->password);
        }
    }
}

// PostgreSQL 实现（可以轻松切换）
class PostgreSQLConnection implements DatabaseConnection
{
    private string $host;
    private string $database;
    private string $username;
    private string $password;
    private ?PDO $pdo = null;
    
    public function __construct(
        string $host,
        string $database,
        string $username,
        string $password
    ) {
        $this->host = $host;
        $this->database = $database;
        $this->username = $username;
        $this->password = $password;
    }
    
    public function prepare(string $sql): PDOStatement
    {
        $this->connect();
        return $this->pdo->prepare($sql);
    }
    
    public function lastInsertId(): string
    {
        $this->connect();
        return $this->pdo->lastInsertId();
    }
    
    private function connect(): void
    {
        if ($this->pdo === null) {
            $dsn = "pgsql:host={$this->host};dbname={$this->database}";
            $this->pdo = new PDO($dsn, $this->username, $this->password);
        }
    }
}

// 用户仓储实现
class MySQLUserRepository implements UserRepository
{
    private DatabaseConnection $connection;
    
    public function __construct(DatabaseConnection $connection)
    {
        $this->connection = $connection;
    }
    
    public function save(array $data): int
    {
        $stmt = $this->connection->prepare(
            "INSERT INTO users (name, email) VALUES (?, ?)"
        );
        $stmt->execute([$data['name'], $data['email']]);
        
        return (int)$this->connection->lastInsertId();
    }
    
    public function findById(int $id): ?array
    {
        $stmt = $this->connection->prepare(
            "SELECT * FROM users WHERE id = ?"
        );
        $stmt->execute([$id]);
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        
        return $result ?: null;
    }
}

// 订单服务（高层模块）
class OrderService
{
    private UserRepository $userRepository;
    
    // 依赖接口，而不是具体实现
    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }
    
    public function createOrder(array $data): int
    {
        // 使用仓储保存订单
        // 不需要关心数据如何存储
        return 1;
    }
}

// 使用示例：切换数据库只需要修改配置
$config = [
    'driver' => 'mysql',  // 可以改为 'pgsql'
    'host' => 'localhost',
    'database' => 'shop',
    'username' => 'root',
    'password' => 'password'
];

// 根据配置创建连接
$connection = match($config['driver']) {
    'mysql' => new MySQLConnection(
        $config['host'],
        $config['database'],
        $config['username'],
        $config['password']
    ),
    'pgsql' => new PostgreSQLConnection(
        $config['host'],
        $config['database'],
        $config['username'],
        $config['password']
    ),
};

// 创建仓储
$userRepository = new MySQLUserRepository($connection);

// 创建服务
$orderService = new OrderService($userRepository);
```

### 重构示例 2：使用接口抽象输出

```php
<?php
declare(strict_types=1);

// 定义输出接口
interface DataExporter
{
    public function export(array $data, string $filename): void;
}

// CSV 导出实现
class CsvExporter implements DataExporter
{
    public function export(array $data, string $filename): void
    {
        $handle = fopen($filename, 'w');
        
        // 写入表头
        if (!empty($data)) {
            fputcsv($handle, array_keys($data[0]));
        }
        
        // 写入数据
        foreach ($data as $row) {
            fputcsv($handle, $row);
        }
        
        fclose($handle);
    }
}

// JSON 导出实现
class JsonExporter implements DataExporter
{
    public function export(array $data, string $filename): void
    {
        $json = json_encode($data, JSON_PRETTY_PRINT);
        file_put_contents($filename, $json);
    }
}

// XML 导出实现
class XmlExporter implements DataExporter
{
    public function export(array $data, string $filename): void
    {
        $xml = new SimpleXMLElement('<root/>');
        
        foreach ($data as $item) {
            $xmlItem = $xml->addChild('item');
            foreach ($item as $key => $value) {
                $xmlItem->addChild($key, htmlspecialchars($value));
            }
        }
        
        $xml->asXML($filename);
    }
}

// 用户导出服务
class UserExporter
{
    private DataExporter $exporter;
    
    public function __construct(DataExporter $exporter)
    {
        $this->exporter = $exporter;
    }
    
    public function export(array $users, string $filename): void
    {
        $this->exporter->export($users, $filename);
    }
}

// 使用示例：根据配置选择导出格式
$config = ['format' => 'csv']; // 可以是 'json', 'xml'

$exporter = match($config['format']) {
    'csv' => new CsvExporter(),
    'json' => new JsonExporter(),
    'xml' => new XmlExporter(),
    default => new CsvExporter(),
};

$userExporter = new UserExporter($exporter);

$users = [
    ['id' => 1, 'name' => 'John', 'email' => 'john@example.com'],
    ['id' => 2, 'name' => 'Jane', 'email' => 'jane@example.com'],
];

$userExporter->export($users, 'users.csv');
```

### 重构示例 3：完整的服务架构

```php
<?php
declare(strict_types=1);

// 定义核心仓储接口
interface OrderRepository
{
    public function save(Order $order): void;
    public function findById(int $id): ?Order;
    public function findByCustomerId(int $customerId): array;
}

// 定义通知接口
interface Notifier
{
    public function send(Order $order): void;
}

// 定义日志接口
interface Logger
{
    public function info(string $message): void;
    public function error(string $message): void;
}

// 实体
class Order
{
    public function __construct(
        public readonly int $id,
        public readonly int $customerId,
        public readonly float $total,
        public readonly string $status
    ) {}
}

// MySQL 仓储实现
class MySQLOrderRepository implements OrderRepository
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function save(Order $order): void
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO orders (customer_id, total, status) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $order->customerId,
            $order->total,
            $order->status
        ]);
    }
    
    public function findById(int $id): ?Order
    {
        $stmt = $this->pdo->prepare("SELECT * FROM orders WHERE id = ?");
        $stmt->execute([$id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($row === false) {
            return null;
        }
        
        return new Order(
            (int)$row['id'],
            (int)$row['customer_id'],
            (float)$row['total'],
            $row['status']
        );
    }
    
    public function findByCustomerId(int $customerId): array
    {
        $stmt = $this->pdo->prepare(
            "SELECT * FROM orders WHERE customer_id = ?"
        );
        $stmt->execute([$customerId]);
        
        $orders = [];
        while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
            $orders[] = new Order(
                (int)$row['id'],
                (int)$row['customer_id'],
                (float)$row['total'],
                $row['status']
            );
        }
        
        return $orders;
    }
}

// 邮件通知实现
class EmailNotifier implements Notifier
{
    private string $fromEmail;
    
    public function __construct(string $fromEmail)
    {
        $this->fromEmail = $fromEmail;
    }
    
    public function send(Order $order): void
    {
        echo "Sending email for order #{$order->id}\n";
    }
}

// 文件日志实现
class FileLogger implements Logger
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function info(string $message): void
    {
        $this->log('INFO', $message);
    }
    
    public function error(string $message): void
    {
        $this->log('ERROR', $message);
    }
    
    private function log(string $level, string $message): void
    {
        $line = sprintf(
            "[%s] [%s] %s\n",
            date('Y-m-d H:i:s'),
            $level,
            $message
        );
        file_put_contents($this->filename, $line, FILE_APPEND);
    }
}

// 订单服务（高层模块，依赖抽象）
class OrderService
{
    private OrderRepository $repository;
    private Notifier $notifier;
    private Logger $logger;
    
    public function __construct(
        OrderRepository $repository,
        Notifier $notifier,
        Logger $logger
    ) {
        $this->repository = $repository;
        $this->notifier = $notifier;
        $this->logger = $logger;
    }
    
    public function createOrder(int $customerId, float $total): Order
    {
        $this->logger->info("Creating order for customer #{$customerId}");
        
        $order = new Order(
            0,
            $customerId,
            $total,
            'pending'
        );
        
        $this->repository->save($order);
        
        $this->notifier->send($order);
        
        $this->logger->info("Order created successfully");
        
        return $order;
    }
}

// 使用依赖注入容器
class Container
{
    private array $bindings = [];
    
    public function bind(string $abstract, callable $factory): void
    {
        $this->bindings[$abstract] = $factory;
    }
    
    public function make(string $abstract): mixed
    {
        if (!isset($this->bindings[$abstract])) {
            throw new RuntimeException("No binding found for: {$abstract}");
        }
        
        return ($this->bindings[$abstract])($this);
    }
}

// 配置容器
$container = new Container();

$container->bind(Logger::class, function($c) {
    return new FileLogger('app.log');
});

$container->bind(Notifier::class, function($c) {
    return new EmailNotifier('noreply@example.com');
});

$container->bind(OrderRepository::class, function($c) {
    $pdo = new PDO('mysql:host=localhost;dbname=shop', 'root', '');
    return new MySQLOrderRepository($pdo);
});

// 从容器解析服务
$orderService = $container->make(OrderService::class);
$order = $orderService->createOrder(1, 99.99);

echo "Order created: {$order->id}\n";
```

## 使用场景

依赖倒置原则适用于以下场景：

- **需要频繁更换实现的系统**：如数据库、缓存、消息队列等
- **需要单元测试的项目**：依赖抽象使 mock 更容易
- **框架和库开发**：为用户提供扩展点
- **大型企业应用**：需要解耦各模块
- **微服务架构**：服务之间通过接口通信

## 注意事项

### 不要过度抽象

依赖倒置原则并不意味着要为什么都创建接口。对于：
- 不会变化的具体类（如 StringUtils）
- 简单工具类
- 只有一个实现的模块

可以直接依赖，不需要过度抽象。

### 依赖的方向

- 高层模块应该依赖抽象
- 低层模块应该实现抽象
- 抽象不应该依赖细节
- 细节应该依赖抽象

### 接口的数量

- 每个"可替换"的依赖都应该抽象为接口
- 不要为每个类都创建接口
- 接口应该是稳定的

## 常见问题

### 什么时候应该使用接口？

- 当有多种实现时
- 当需要 mock 进行测试时
- 当实现可能变化时

### 依赖注入会导致类过多吗？

可能会，但这是值得的。可以通过 IoC 容器来管理依赖。

### 如何选择注入方式？

- **构造函数注入**：适合必需的依赖
- **Setter 注入**：适合可选的依赖
- **接口注入**：适合需要明确声明依赖的场景

### DIP 和 DI 是一样的吗？

DIP 是设计原则，DI 是实现 DIP 的一种技术。DI 是实现 DIP 最常用的方式。

## 最佳实践

1. **依赖抽象而非具体**：为可替换的依赖定义接口

2. **使用构造函数注入**：这是最常用的方式，依赖关系清晰

3. **遵循最小依赖原则**：只依赖需要的接口

4. **使用 IoC 容器**：管理复杂的依赖关系

5. **编写可测试的代码**：依赖抽象使单元测试更容易

6. **保持接口稳定**：接口应该是稳定的，不要频繁修改

7. **依赖指向稳定方向**：依赖应该指向更稳定的模块

## 练习任务

1. **重构现有代码**：找一个直接依赖具体类的代码，重构为依赖接口

2. **设计电商系统**：使用依赖倒置原则设计电商系统，包括订单、支付、通知等模块

3. **实现 IoC 容器**：自己实现一个简单的依赖注入容器

4. **编写单元测试**：为一个使用依赖注入的类编写单元测试，使用 mock 对象

5. **分析依赖关系**：分析你项目的依赖图，找出违反 DIP 的地方

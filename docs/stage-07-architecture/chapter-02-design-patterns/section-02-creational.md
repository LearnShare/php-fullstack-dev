# 7.2.2 创建型模式

## 概述

创建型设计模式关注对象的创建机制，旨在封装对象创建的过程，使代码与具体类的创建解耦。这些模式提供了一种在创建对象时隐藏创建逻辑的方式，而不是直接使用 `new` 关键字实例化对象。

在软件开发中，对象的创建是一个常见且重要的任务。创建型模式帮助我们解决以下问题：
- 复杂对象的创建过程
- 对象创建方式的灵活性
- 依赖关系的管理
- 资源的统一管理

掌握创建型模式对于编写高质量的 PHP 代码至关重要。本节将详细介绍工厂模式、抽象工厂模式、单例模式、建造者模式和原型模式的概念、实现方法和适用场景。

**主要内容**：
- 工厂模式（简单工厂、工厂方法）
- 抽象工厂模式
- 单例模式
- 建造者模式
- 原型模式

## 工厂模式

### 简单工厂模式

简单工厂模式是最基础的工厂模式。它通过一个工厂类根据参数创建不同的产品对象。

```php
<?php
declare(strict_types=1);

// 产品接口
interface PaymentMethod
{
    public function pay(float $amount): bool;
    public function refund(string $transactionId): bool;
}

// 具体产品：信用卡支付
class CreditCardPayment implements PaymentMethod
{
    private string $cardNumber;
    
    public function __construct(string $cardNumber)
    {
        $this->cardNumber = $cardNumber;
    }
    
    public function pay(float $amount): bool
    {
        echo "Processing credit card payment: \${$amount}\n";
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        echo "Refunding to card: {$this->cardNumber}\n";
        return true;
    }
}

// 具体产品：PayPal 支付
class PayPalPayment implements PaymentMethod
{
    private string $email;
    
    public function __construct(string $email)
    {
        $this->email = $email;
    }
    
    public function pay(float $amount): bool
    {
        echo "Processing PayPal payment: \${$amount}\n";
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        echo "Refunding to PayPal: {$this->email}\n";
        return true;
    }
}

// 简单工厂
class PaymentFactory
{
    public static function create(string $type, array $options = []): PaymentMethod
    {
        return match($type) {
            'credit_card' => new CreditCardPayment($options['card_number'] ?? ''),
            'paypal' => new PayPalPayment($options['email'] ?? ''),
            default => throw new InvalidArgumentException("Unknown payment type: {$type}"),
        };
    }
}

// 使用示例
$creditCard = PaymentFactory::create('credit_card', [
    'card_number' => '4111111111111111'
]);
$creditCard->pay(100.0);

$paypal = PaymentFactory::create('paypal', [
    'email' => 'user@example.com'
]);
$paypal->pay(50.0);
```

**输出**：

```
Processing credit card payment: $100
Processing PayPal payment: $50
```

### 工厂方法模式

工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化哪个类。

```php
<?php
declare(strict_types=1);

// 产品接口
interface Logger
{
    public function log(string $message): void;
}

// 具体产品：文件日志
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

// 具体产品：数据库日志
class DatabaseLogger implements Logger
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function log(string $message): void
    {
        $stmt = $this->pdo->prepare("INSERT INTO logs (message, created_at) VALUES (?, ?)");
        $stmt->execute([$message, date('Y-m-d H:i:s')]);
    }
}

// 抽象工厂类
abstract class LoggerFactory
{
    abstract public function createLogger(): Logger;
    
    // 模板方法：提供通用的日志创建流程
    public function log(string $message): void
    {
        $logger = $this->createLogger();
        $logger->log($message);
    }
}

// 具体工厂：文件日志工厂
class FileLoggerFactory extends LoggerFactory
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function createLogger(): Logger
    {
        return new FileLogger($this->filename);
    }
}

// 具体工厂：数据库日志工厂
class DatabaseLoggerFactory extends LoggerFactory
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function createLogger(): Logger
    {
        return new DatabaseLogger($this->pdo);
    }
}

// 使用示例
$fileFactory = new FileLoggerFactory('app.log');
$fileFactory->log("Application started");

$dbFactory = new DatabaseLoggerFactory(
    new PDO('mysql:host=localhost;dbname=test', 'root', '')
);
$dbFactory->log("Database connected");
```

## 抽象工厂模式

抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

```php
<?php
declare(strict_types=1);

// 抽象产品 A：按钮
interface Button
{
    public function render(): string;
}

// 抽象产品 B：输入框
interface Input
{
    public function render(): string;
}

// 具体产品 A1：Windows 风格按钮
class WindowsButton implements Button
{
    public function render(): string
    {
        return '<button style="border: 2px outset; background: #c0c0c0">Click me</button>';
    }
}

// 具体产品 A2：Mac 风格按钮
class MacButton implements Button
{
    public function render(): string
    {
        return '<button style="border-radius: 4px; background: #007aff; color: white">Click me</button>';
    }
}

// 具体产品 B1：Windows 风格输入框
class WindowsInput implements Input
{
    public function render(): string
    {
        return '<input style="border: 2 inset; background: white">';
    }
}

// 具体产品 B2：Mac 风格输入框
class MacInput implements Input
{
    public function render(): string
    {
        return '<input style="border-radius: 4px; border: 1px solid #007aff">';
    }
}

// 抽象工厂
interface UIFactory
{
    public function createButton(): Button;
    public function createInput(): Input;
}

// 具体工厂：Windows UI 工厂
class WindowsUIFactory implements UIFactory
{
    public function createButton(): Button
    {
        return new WindowsButton();
    }
    
    public function createInput(): Input
    {
        return new WindowsInput();
    }
}

// 具体工厂：Mac UI 工厂
class MacUIFactory implements UIFactory
{
    public function createButton(): Button
    {
        return new MacButton();
    }
    
    public function createInput(): Input
    {
        return new MacInput();
    }
}

// 客户端代码
class Application
{
    private Button $button;
    private Input $input;
    
    public function __construct(UIFactory $factory)
    {
        $this->button = $factory->createButton();
        $this->input = $factory->createInput();
    }
    
    public function render(): void
    {
        echo $this->button->render() . "\n";
        echo $this->input->render() . "\n";
    }
}

// 使用示例：根据配置选择主题
$config = ['theme' => 'mac']; // 可以是 'windows' 或 'mac'

$factory = match($config['theme']) {
    'windows' => new WindowsUIFactory(),
    'mac' => new MacUIFactory(),
    default => new WindowsUIFactory(),
};

$app = new Application($factory);
$app->render();
```

**输出**（Mac 主题）：

```
<button style="border-radius: 4px; background: #007aff; color: white">Click me</button>
<input style="border-radius: 4px; border: 1px solid #007aff">
```

## 单例模式

单例模式确保一个类只有一个实例，并提供一个全局访问点。

```php
<?php
declare(strict_types=1);

// 基本的单例模式
class Database
{
    private static ?Database $instance = null;
    private PDO $pdo;
    
    // 私有构造函数，防止外部实例化
    private function __construct()
    {
        $config = [
            'host' => 'localhost',
            'dbname' => 'test',
            'username' => 'root',
            'password' => ''
        ];
        
        $dsn = sprintf(
            "mysql:host=%s;dbname=%s;charset=utf8mb4",
            $config['host'],
            $config['dbname']
        );
        
        $this->pdo = new PDO(
            $dsn,
            $config['username'],
            $config['password'],
            [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            ]
        );
    }
    
    // 获取唯一实例
    public static function getInstance(): Database
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        
        return self::$instance;
    }
    
    // 防止克隆
    private function __clone() {}
    
    // 防止反序列化
    public function __wakeup()
    {
        throw new Exception("Cannot unserialize singleton");
    }
    
    public function query(string $sql, array $params = []): array
    {
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll();
    }
}

// 使用示例
$db1 = Database::getInstance();
$db2 = Database::getInstance();

echo "Same instance: " . ($db1 === $db2 ? "Yes" : "No") . "\n";
```

**输出**：

```
Same instance: Yes
```

### 延迟加载单例

```php
<?php
declare(strict_types=1);

// 延迟加载单例（线程安全）
class LazySingleton
{
    private static ?LazySingleton $instance = null;
    
    private function __construct()
    {
        // 初始化操作
    }
    
    public static function getInstance(): LazySingleton
    {
        if (self::$instance === null) {
            // 双重检查锁定
            if (self::$instance === null) {
                self::$instance = new self();
            }
        }
        
        return self::$instance;
    }
}
```

## 建造者模式

建造者模式将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

```php
<?php
declare(strict_types=1);

// 产品：电子邮件
class Email
{
    private string $from;
    private string $to;
    private string $subject;
    private string $body;
    private array $cc = [];
    private array $attachments = [];
    
    // 私有构造函数，只能通过 Builder 创建
    private function __construct() {}
    
    // Getter 方法
    public function getFrom(): string { return $this->from; }
    public function getTo(): string { return $this->to; }
    public function getSubject(): string { return $this->subject; }
    public function getBody(): string { return $this->body; }
    public function getCc(): array { return $this->cc; }
    public function getAttachments(): array { return $this->attachments; }
    
    public function __toString(): string
    {
        return sprintf(
            "From: %s\nTo: %s\nSubject: %s\nBody: %s\n",
            $this->from,
            $this->to,
            $this->subject,
            $this->body
        );
    }
}

// 建造者接口
interface EmailBuilder
{
    public function setFrom(string $from): self;
    public function setTo(string $to): self;
    public function setSubject(string $subject): self;
    public function setBody(string $body): self;
    public function addCc(string $cc): self;
    public function addAttachment(string $path): self;
    public function build(): Email;
}

// 具体建造者
class EmailBuilderImpl implements EmailBuilder
{
    private Email $email;
    
    public function __construct()
    {
        $this->email = new Email();
    }
    
    public function setFrom(string $from): self
    {
        $this->email->from = $from;
        return $this;
    }
    
    public function setTo(string $to): self
    {
        $this->email->to = $to;
        return $this;
    }
    
    public function setSubject(string $subject): self
    {
        $this->email->subject = $subject;
        return $this;
    }
    
    public function setBody(string $body): self
    {
        $this->email->body = $body;
        return $this;
    }
    
    public function addCc(string $cc): self
    {
        $this->email->cc[] = $cc;
        return $this;
    }
    
    public function addAttachment(string $path): self
    {
        $this->email->attachments[] = $path;
        return $this;
    }
    
    public function build(): Email
    {
        return $this->email;
    }
}

// 指导者
class EmailDirector
{
    public function buildWelcomeEmail(EmailBuilder $builder): Email
    {
        return $builder
            ->setFrom('noreply@example.com')
            ->setSubject('Welcome!')
            ->setBody('Welcome to our service!')
            ->build();
    }
    
    public function buildResetPasswordEmail(EmailBuilder $builder, string $token): Email
    {
        return $builder
            ->setFrom('noreply@example.com')
            ->setSubject('Reset Password')
            ->setBody("Click here to reset your password: https://example.com/reset/{$token}")
            ->build();
    }
}

// 使用示例
$builder = new EmailBuilderImpl();
$director = new EmailDirector();

$welcomeEmail = $director->buildWelcomeEmail($builder);
echo $welcomeEmail;

$resetEmail = $director->buildResetPasswordEmail(
    (new EmailBuilderImpl())->setTo('user@example.com'),
    'abc123'
);
echo $resetEmail;
```

## 原型模式

原型模式用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

```php
<?php
declare(strict_types=1);

// 原型接口
interface Prototype
{
    public function clone(): self;
}

// 具体原型
class UserProfile implements Prototype
{
    private string $name;
    private string $email;
    private array $roles = [];
    private array $preferences = [];
    
    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
    
    // 克隆方法
    public function clone(): self
    {
        $clone = new self($this->name, $this->email);
        $clone->roles = $this->roles;
        $clone->preferences = $this->preferences;
        return $clone;
    }
    
    public function setName(string $name): void
    {
        $this->name = $name;
    }
    
    public function addRole(string $role): void
    {
        $this->roles[] = $role;
    }
    
    public function setPreference(string $key, mixed $value): void
    {
        $this->preferences[$key] = $value;
    }
    
    public function __toString(): string
    {
        return sprintf(
            "User: %s <%s>\nRoles: %s\nPreferences: %s\n",
            $this->name,
            $this->email,
            implode(', ', $this->roles),
            json_encode($this->preferences)
        );
    }
}

// 使用示例：创建用户模板
$defaultProfile = new UserProfile('Default User', 'default@example.com');
$defaultProfile->addRole('user');
$defaultProfile->setPreference('theme', 'dark');
$defaultProfile->setPreference('language', 'en');

// 克隆并个性化
$user1 = $defaultProfile->clone();
$user1->setName('John Doe');
$user1->setEmail('john@example.com');

$user2 = $defaultProfile->clone();
$user2->setName('Jane Smith');
$user2->setEmail('jane@example.com');
$user2->addRole('admin');

echo "=== Default Profile ===\n";
echo $defaultProfile;

echo "=== User 1 ===\n";
echo $user1;

echo "=== User 2 ===\n";
echo $user2;
```

**输出**：

```
=== Default Profile ===
User: Default User <default@example.com>
Roles: user
Preferences: {"theme":"dark","language":"en"}

=== User 1 ===
User: John Doe <john@example.com>
Roles: user
Preferences: {"theme":"dark","language":"en"}

=== User 2 ===
User: Jane Smith <jane@example.com>
Roles: user,admin
Preferences: {"theme":"dark","language":"en"}
```

## 使用场景

### 工厂模式适用场景

- 对象创建逻辑复杂
- 需要根据参数创建不同类型的对象
- 需要在不修改客户端代码的情况下添加新产品

### 抽象工厂适用场景

- 需要创建一系列相关的产品对象
- 需要确保产品之间的兼容性
- 需要隐藏具体产品的实现细节

### 单例模式适用场景

- 需要全局唯一实例（如配置管理、数据库连接）
- 需要严格控制资源访问
- 需要懒加载资源

### 建造者模式适用场景

- 对象创建步骤多、过程复杂
- 需要创建不同表示的对象
- 需要链式构建对象

### 原型模式适用场景

- 对象创建成本高
- 需要创建大量相似对象
- 需要保存对象状态快照

## 注意事项

### 工厂模式的注意事项

- 简单工厂适用于产品类型少、变化不频繁的场景
- 工厂方法适用于需要扩展产品类型的场景
- 抽象工厂适用于产品族场景

### 单例模式的注意事项

- PHP 中单例需要考虑多线程环境（虽然在 PHP FPM 中不常见）
- 过度使用单例会导致代码难以测试
- 考虑使用依赖注入替代全局单例

### 建造者模式的注意事项

- 建造者模式可能导致代码复杂度增加
- 只在真正需要复杂构建过程时使用

## 常见问题

### 工厂模式和抽象工厂的区别？

工厂模式创建一个产品，抽象工厂创建一系列相关的产品。工厂方法通过子类决定创建哪个产品，抽象工厂通过具体工厂创建整个产品族。

### 单例模式的线程安全？

在多线程环境中，需要使用双重检查锁定或其他同步机制确保只创建一个实例。

### 何时使用建造者模式？

当对象创建涉及多个步骤、不同配置选项或复杂初始化逻辑时，应该使用建造者模式。

### 创建型模式如何选择？

根据具体需求：
- 需要灵活创建对象 → 工厂模式
- 需要创建产品族 → 抽象工厂
- 需要唯一实例 → 单例模式
- 需要复杂构建 → 建造者模式
- 需要克隆对象 → 原型模式

## 最佳实践

1. **优先使用依赖注入**：在现代 PHP 开发中，依赖注入容器通常是比单例更好的选择

2. **工厂模式与依赖注入结合**：可以在依赖注入容器中使用工厂模式

3. **建造者模式链式调用**：使用链式调用使代码更简洁

4. **原型模式与克隆**：PHP 中使用 `clone` 关键字实现原型模式

5. **避免过度使用**：只在真正需要时使用这些模式

## 练习任务

1. **实现工厂方法**：为一个支付系统实现工厂方法模式

2. **设计 UI 主题系统**：使用抽象工厂模式设计一个支持多主题的 UI 系统

3. **实现配置管理**：使用单例模式实现配置管理类

4. **构建复杂对象**：使用建造者模式构建一个 SQL 查询构建器

5. **实现对象克隆**：使用原型模式实现用户模板和克隆功能

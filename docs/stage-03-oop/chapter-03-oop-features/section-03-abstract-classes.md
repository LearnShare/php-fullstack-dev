# 3.3.3 抽象类（Abstract Class）

## 概述

抽象类（Abstract Class）是不能被直接实例化的类，它位于普通类和接口之间。抽象类可以包含抽象方法（只有方法签名，没有实现）和普通方法（有具体实现）。子类继承抽象类后，必须实现所有抽象方法才能被实例化。

理解抽象类对于掌握面向对象编程非常重要。抽象类提供了一种"部分实现"的机制，可以在基类中提供通用的实现，同时要求子类实现特定的部分。这种设计模式特别适用于"模板方法模式"（Template Method Pattern），定义了算法的骨架，让子类填充具体步骤。

抽象类与接口的区别在于：接口只定义契约（方法签名），不提供实现；抽象类可以提供部分实现，要求子类实现剩余部分。抽象类通常用于需要共享代码的场景，而接口用于定义契约的场景。

**主要内容**：
- 抽象类的定义语法
- 抽象方法的定义和使用
- 抽象类与普通类的区别
- 抽象类与接口的区别
- 模板方法模式（Template Method Pattern）
- 抽象类的使用场景
- 抽象类的最佳实践

## 特性

- **不能实例化**：抽象类不能被直接实例化，只能被继承
- **部分实现**：可以提供部分方法的实现
- **抽象方法**：可以定义抽象方法，要求子类实现
- **代码复用**：可以在抽象类中共享通用代码
- **模板方法**：适合实现模板方法模式

## 语法/定义

### 抽象类定义

**语法**：`abstract class AbstractClassName { ... }`

**组成部分**：
- `abstract` 关键字：声明抽象类
- `class` 关键字：声明类
- `AbstractClassName`：抽象类名称

**特点**：
- 使用 `abstract` 关键字声明抽象类
- 抽象类不能被直接实例化
- 抽象类可以包含抽象方法和普通方法
- 抽象类可以包含属性、构造函数等

### 抽象方法定义

**语法**：`abstract public function methodName(): ReturnType;`

**组成部分**：
- `abstract` 关键字：声明抽象方法
- 可见性修饰符：必须是 `public`、`protected` 或 `private`
- 方法名和参数
- 返回类型声明

**特点**：
- 只有方法签名，没有方法体
- 包含抽象方法的类必须是抽象类
- 子类必须实现所有抽象方法
- 抽象方法不能是 `private`（PHP 8.0+ 之前）

## 基本用法

### 示例 1：基础抽象类

```php
<?php
declare(strict_types=1);

abstract class Animal
{
    protected string $name;
    
    public function __construct(string $name)
    {
        $this->name = $name;
    }
    
    // 抽象方法：子类必须实现
    abstract public function makeSound(): string;
    
    // 普通方法：子类可以继承
    public function sleep(): string
    {
        return "{$this->name} is sleeping";
    }
    
    public function introduce(): string
    {
        return "I am {$this->name}, {$this->makeSound()}";
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return "Woof!";
    }
}

class Cat extends Animal
{
    public function makeSound(): string
    {
        return "Meow!";
    }
}

// 使用
$dog = new Dog("Buddy");
echo $dog->makeSound() . "\n";     // Woof!
echo $dog->sleep() . "\n";         // Buddy is sleeping
echo $dog->introduce() . "\n";     // I am Buddy, Woof!

$cat = new Cat("Fluffy");
echo $cat->makeSound() . "\n";     // Meow!
echo $cat->introduce() . "\n";     // I am Fluffy, Meow!

// 错误：不能实例化抽象类
// $animal = new Animal("Generic");  // Fatal error: Cannot instantiate abstract class
```

**输出**：

```
Woof!
Buddy is sleeping
I am Buddy, Woof!
Meow!
I am Fluffy, Meow!
```

**说明**：
- `Animal` 是抽象类，不能被直接实例化
- `makeSound()` 是抽象方法，子类必须实现
- `sleep()` 和 `introduce()` 是普通方法，子类可以继承使用

### 示例 2：多个抽象方法

```php
<?php
declare(strict_types=1);

abstract class DatabaseConnection
{
    protected string $host;
    protected string $database;
    
    public function __construct(string $host, string $database)
    {
        $this->host = $host;
        $this->database = $database;
    }
    
    // 抽象方法：连接数据库
    abstract public function connect(): void;
    
    // 抽象方法：执行查询
    abstract public function query(string $sql): array;
    
    // 抽象方法：断开连接
    abstract public function disconnect(): void;
    
    // 普通方法：执行查询并自动管理连接
    public function execute(string $sql): array
    {
        $this->connect();
        try {
            $result = $this->query($sql);
            return $result;
        } finally {
            $this->disconnect();
        }
    }
}

class MySQLConnection extends DatabaseConnection
{
    public function connect(): void
    {
        echo "Connecting to MySQL at {$this->host}/{$this->database}\n";
    }
    
    public function query(string $sql): array
    {
        echo "Executing MySQL query: {$sql}\n";
        return [];  // 模拟返回结果
    }
    
    public function disconnect(): void
    {
        echo "Disconnecting from MySQL\n";
    }
}

class PostgreSQLConnection extends DatabaseConnection
{
    public function connect(): void
    {
        echo "Connecting to PostgreSQL at {$this->host}/{$this->database}\n";
    }
    
    public function query(string $sql): array
    {
        echo "Executing PostgreSQL query: {$sql}\n";
        return [];
    }
    
    public function disconnect(): void
    {
        echo "Disconnecting from PostgreSQL\n";
    }
}

$mysql = new MySQLConnection("localhost", "mydb");
$mysql->execute("SELECT * FROM users");

$postgres = new PostgreSQLConnection("localhost", "mydb");
$postgres->execute("SELECT * FROM products");
```

**输出**：

```
Connecting to MySQL at localhost/mydb
Executing MySQL query: SELECT * FROM users
Disconnecting from MySQL
Connecting to PostgreSQL at localhost/mydb
Executing PostgreSQL query: SELECT * FROM products
Disconnecting from PostgreSQL
```

**说明**：
- 抽象类可以定义多个抽象方法
- 子类必须实现所有抽象方法
- 抽象类中的普通方法（如 `execute()`）可以调用抽象方法

### 示例 3：模板方法模式

```php
<?php
declare(strict_types=1);

abstract class DataProcessor
{
    // 模板方法：定义算法骨架
    final public function process(): void
    {
        $this->load();
        $this->transform();
        $this->save();
        $this->log("Processing completed");
    }
    
    // 抽象方法：子类必须实现
    abstract protected function load(): void;
    abstract protected function transform(): void;
    abstract protected function save(): void;
    
    // 钩子方法：子类可以重写
    protected function log(string $message): void
    {
        echo "[LOG] {$message}\n";
    }
}

class FileProcessor extends DataProcessor
{
    protected function load(): void
    {
        echo "Loading data from file...\n";
    }
    
    protected function transform(): void
    {
        echo "Transforming file data...\n";
    }
    
    protected function save(): void
    {
        echo "Saving processed data to file...\n";
    }
}

class DatabaseProcessor extends DataProcessor
{
    protected function load(): void
    {
        echo "Loading data from database...\n";
    }
    
    protected function transform(): void
    {
        echo "Transforming database data...\n";
    }
    
    protected function save(): void
    {
        echo "Saving processed data to database...\n";
    }
    
    protected function log(string $message): void
    {
        echo "[DATABASE LOG] {$message}\n";
    }
}

$fileProcessor = new FileProcessor();
$fileProcessor->process();

echo "\n";

$dbProcessor = new DatabaseProcessor();
$dbProcessor->process();
```

**输出**：

```
Loading data from file...
Transforming file data...
Saving processed data to file...
[LOG] Processing completed

Loading data from database...
Transforming database data...
Saving processed data to database...
[DATABASE LOG] Processing completed
```

**说明**：
- `process()` 方法是模板方法，定义了算法骨架
- 使用 `final` 关键字防止子类重写模板方法
- 抽象方法由子类实现具体步骤
- 钩子方法（如 `log()`）子类可以选择性重写

### 示例 4：抽象类中的属性和构造函数

```php
<?php
declare(strict_types=1);

abstract class Vehicle
{
    protected string $brand;
    protected string $model;
    protected int $year;
    
    public function __construct(string $brand, string $model, int $year)
    {
        $this->brand = $brand;
        $this->model = $model;
        $this->year = $year;
    }
    
    // 抽象方法
    abstract public function start(): string;
    abstract public function stop(): string;
    
    // 普通方法
    public function getInfo(): string
    {
        return "{$this->year} {$this->brand} {$this->model}";
    }
}

class Car extends Vehicle
{
    public function __construct(string $brand, string $model, int $year)
    {
        parent::__construct($brand, $model, $year);
    }
    
    public function start(): string
    {
        return "Car engine started";
    }
    
    public function stop(): string
    {
        return "Car engine stopped";
    }
}

class Motorcycle extends Vehicle
{
    public function start(): string
    {
        return "Motorcycle engine started";
    }
    
    public function stop(): string
    {
        return "Motorcycle engine stopped";
    }
}

$car = new Car("Toyota", "Camry", 2023);
echo $car->getInfo() . "\n";        // 2023 Toyota Camry
echo $car->start() . "\n";          // Car engine started

$motorcycle = new Motorcycle("Honda", "CBR", 2023);
echo $motorcycle->getInfo() . "\n";  // 2023 Honda CBR
echo $motorcycle->start() . "\n";    // Motorcycle engine started
```

**输出**：

```
2023 Toyota Camry
Car engine started
2023 Honda CBR
Motorcycle engine started
```

**说明**：
- 抽象类可以包含属性和构造函数
- 子类构造函数应该调用父类构造函数
- 抽象类的属性可以被子类继承和使用

### 示例 5：抽象类实现接口

```php
<?php
declare(strict_types=1);

interface Drawable
{
    public function draw(): void;
}

abstract class Shape implements Drawable
{
    protected string $color;
    
    public function __construct(string $color)
    {
        $this->color = $color;
    }
    
    // 实现接口方法
    public function draw(): void
    {
        echo "Drawing a {$this->color} shape\n";
        $this->drawSpecific();
    }
    
    // 抽象方法：子类实现具体绘制逻辑
    abstract protected function drawSpecific(): void;
    
    // 抽象方法：计算面积
    abstract public function getArea(): float;
}

class Circle extends Shape
{
    private float $radius;
    
    public function __construct(string $color, float $radius)
    {
        parent::__construct($color);
        $this->radius = $radius;
    }
    
    protected function drawSpecific(): void
    {
        echo "Drawing a circle with radius {$this->radius}\n";
    }
    
    public function getArea(): float
    {
        return pi() * $this->radius * $this->radius;
    }
}

class Rectangle extends Shape
{
    private float $width;
    private float $height;
    
    public function __construct(string $color, float $width, float $height)
    {
        parent::__construct($color);
        $this->width = $width;
        $this->height = $height;
    }
    
    protected function drawSpecific(): void
    {
        echo "Drawing a rectangle {$this->width}x{$this->height}\n";
    }
    
    public function getArea(): float
    {
        return $this->width * $this->height;
    }
}

$circle = new Circle("red", 5);
$circle->draw();
echo "Area: " . $circle->getArea() . "\n";

$rectangle = new Rectangle("blue", 4, 6);
$rectangle->draw();
echo "Area: " . $rectangle->getArea() . "\n";
```

**输出**：

```
Drawing a red shape
Drawing a circle with radius 5
Area: 78.539816339745
Drawing a blue shape
Drawing a rectangle 4x6
Area: 24
```

**说明**：
- 抽象类可以实现接口
- 可以在抽象类中实现接口的部分方法
- 将剩余的实现交给子类完成

### 示例 6：抽象类中的静态方法

```php
<?php
declare(strict_types=1);

abstract class Logger
{
    // 抽象方法
    abstract public function log(string $message, string $level): void;
    
    // 静态方法：可以包含实现
    public static function create(string $type): self
    {
        return match ($type) {
            'file' => new FileLogger(),
            'console' => new ConsoleLogger(),
            default => throw new InvalidArgumentException("Unknown logger type: {$type}"),
        };
    }
}

class FileLogger extends Logger
{
    public function log(string $message, string $level): void
    {
        $logLine = date('Y-m-d H:i:s') . " [{$level}] {$message}\n";
        file_put_contents('app.log', $logLine, FILE_APPEND);
        echo "Logged to file: {$message}\n";
    }
}

class ConsoleLogger extends Logger
{
    public function log(string $message, string $level): void
    {
        echo "[{$level}] {$message}\n";
    }
}

// 使用静态工厂方法
$fileLogger = Logger::create('file');
$fileLogger->log("Application started", "INFO");

$consoleLogger = Logger::create('console');
$consoleLogger->log("User logged in", "INFO");
```

**输出**：

```
Logged to file: Application started
[INFO] User logged in
```

**说明**：
- 抽象类可以包含静态方法
- 静态方法可以包含完整实现
- 可以使用静态方法作为工厂方法

## 使用场景

### 场景 1：模板方法模式

抽象类非常适合实现模板方法模式，定义算法骨架。

**示例**：支付处理器

```php
<?php
declare(strict_types=1);

abstract class PaymentProcessor
{
    protected float $amount;
    
    public function __construct(float $amount)
    {
        $this->amount = $amount;
    }
    
    // 模板方法
    final public function process(): bool
    {
        $this->validate();
        $this->authorize();
        $result = $this->charge();
        $this->log($result);
        return $result;
    }
    
    abstract protected function validate(): void;
    abstract protected function authorize(): void;
    abstract protected function charge(): bool;
    
    protected function log(bool $success): void
    {
        $status = $success ? 'SUCCESS' : 'FAILED';
        echo "Payment {$status}: \${$this->amount}\n";
    }
}

class CreditCardProcessor extends PaymentProcessor
{
    protected function validate(): void
    {
        echo "Validating credit card...\n";
    }
    
    protected function authorize(): void
    {
        echo "Authorizing credit card payment...\n";
    }
    
    protected function charge(): bool
    {
        echo "Charging credit card...\n";
        return true;
    }
}

class PayPalProcessor extends PaymentProcessor
{
    protected function validate(): void
    {
        echo "Validating PayPal account...\n";
    }
    
    protected function authorize(): void
    {
        echo "Authorizing PayPal payment...\n";
    }
    
    protected function charge(): bool
    {
        echo "Processing PayPal payment...\n";
        return true;
    }
}
```

### 场景 2：提供部分实现

抽象类用于提供部分通用实现，子类实现特定部分。

**示例**：数据导出器

```php
<?php
declare(strict_types=1);

abstract class DataExporter
{
    protected array $data;
    
    public function __construct(array $data)
    {
        $this->data = $data;
    }
    
    // 模板方法
    final public function export(): string
    {
        $this->validate();
        return $this->format();
    }
    
    protected function validate(): void
    {
        if (empty($this->data)) {
            throw new InvalidArgumentException("Data cannot be empty");
        }
    }
    
    abstract protected function format(): string;
}

class JsonExporter extends DataExporter
{
    protected function format(): string
    {
        return json_encode($this->data, JSON_PRETTY_PRINT);
    }
}

class XmlExporter extends DataExporter
{
    protected function format(): string
    {
        // XML 格式化逻辑
        return "<data>" . json_encode($this->data) . "</data>";
    }
}
```

## 注意事项

### 抽象类不能实例化

- 抽象类不能被直接实例化
- 只能通过子类实例化
- 试图实例化抽象类会导致致命错误

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Example
{
    abstract public function method(): void;
}

// 错误：不能实例化抽象类
// $obj = new Example();  // Fatal error: Cannot instantiate abstract class

// 正确：通过子类实例化
class Concrete extends Example
{
    public function method(): void {}
}

$obj = new Concrete();  // 正确
```

### 抽象方法必须被实现

- 子类必须实现抽象类的所有抽象方法
- 未实现所有抽象方法的子类也必须是抽象类
- 缺少抽象方法实现会导致致命错误

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Base
{
    abstract public function method1(): void;
    abstract public function method2(): void;
}

// 错误：未实现所有抽象方法
// class Incomplete extends Base
// {
//     public function method1(): void {}
//     // 缺少 method2
// }

// 正确：实现所有抽象方法
class Complete extends Base
{
    public function method1(): void {}
    public function method2(): void {}
}

// 或者：子类也是抽象类
abstract class AlsoAbstract extends Base
{
    public function method1(): void {}
    // method2 留给更下层的子类实现
}
```

### 抽象方法的可见性

- 抽象方法可以是 `public`、`protected`
- PHP 8.0+ 之前抽象方法不能是 `private`
- PHP 8.0+ 支持 `private` 抽象方法

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Example
{
    abstract public function publicMethod(): void;
    abstract protected function protectedMethod(): void;
    // PHP 8.0+ 支持
    // abstract private function privateMethod(): void;
}
```

### 单继承限制

- 抽象类仍然受 PHP 单继承限制
- 一个类只能继承一个抽象类
- 可以通过实现接口补充功能

## 常见问题

### 问题 1：抽象类实例化错误

**错误信息**：`Fatal error: Cannot instantiate abstract class`

**原因**：试图直接实例化抽象类

**解决方案**：通过子类实例化，或确保子类实现了所有抽象方法

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Example
{
    abstract public function method(): void;
}

// 错误：不能实例化抽象类
// $obj = new Example();

// 正确：通过子类实例化
class Concrete extends Example
{
    public function method(): void {}
}

$obj = new Concrete();
```

### 问题 2：抽象方法未实现错误

**错误信息**：`Fatal error: Class X contains Y abstract method(s) and must therefore be declared abstract`

**原因**：子类未实现所有抽象方法

**解决方案**：实现所有抽象方法，或将类声明为抽象类

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Base
{
    abstract public function method1(): void;
    abstract public function method2(): void;
}

// 错误：未实现所有抽象方法
// class Incomplete extends Base
// {
//     public function method1(): void {}
// }

// 正确：实现所有抽象方法
class Complete extends Base
{
    public function method1(): void {}
    public function method2(): void {}
}
```

### 问题 3：抽象方法签名不匹配

**错误信息**：`Fatal error: Declaration must be compatible`

**原因**：子类实现的方法签名与抽象方法不匹配

**解决方案**：确保方法签名完全匹配（包括参数类型和返回类型）

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Base
{
    abstract public function process(string $data): bool;
}

// 错误：方法签名不匹配
// class Wrong extends Base
// {
//     public function process($data): void {}  // 参数类型和返回类型不匹配
// }

// 正确：方法签名匹配
class Correct extends Base
{
    public function process(string $data): bool
    {
        return true;
    }
}
```

## 最佳实践

### 1. 合理使用抽象类

- **适用场景**：需要提供部分实现，子类共享通用逻辑
- **不适用场景**：只需要定义契约（使用接口）
- **避免过度使用**：不要创建过多的抽象层次

**示例**：

```php
<?php
declare(strict_types=1);

// 好的使用：需要共享代码
abstract class BaseProcessor
{
    protected function commonLogic(): void
    {
        // 通用逻辑
    }
    
    abstract protected function specificLogic(): void;
}

// 不好的使用：不需要共享代码，应该使用接口
// abstract class Contract
// {
//     abstract public function method(): void;  // 应该使用接口
// }
```

### 2. 使用 final 保护模板方法

模板方法应该使用 `final` 关键字保护，防止子类重写。

**示例**：

```php
<?php
declare(strict_types=1);

abstract class Processor
{
    // 模板方法使用 final
    final public function process(): void
    {
        $this->step1();
        $this->step2();
        $this->step3();
    }
    
    abstract protected function step1(): void;
    abstract protected function step2(): void;
    abstract protected function step3(): void;
}
```

### 3. 设计清晰的抽象层次

- 抽象类应该包含通用的实现
- 抽象方法应该定义子类必须实现的部分
- 保持抽象类的职责单一

**示例**：

```php
<?php
declare(strict_types=1);

// 好的设计：职责清晰
abstract class DataProcessor
{
    // 通用实现
    protected function validate(): void
    {
        // 通用验证逻辑
    }
    
    // 抽象方法：子类实现
    abstract protected function process(): void;
}

// 不好的设计：职责混杂
// abstract class BadProcessor
// {
//     abstract protected function process(): void;
//     abstract protected function log(): void;
//     abstract protected function cache(): void;  // 职责过多
// }
```

### 4. 理解抽象类与接口的区别

根据需求选择合适的机制：
- **抽象类**：需要提供部分实现
- **接口**：只需要定义契约

**示例**：

```php
<?php
declare(strict_types=1);

// 接口：只定义契约
interface Loggable
{
    public function log(string $message): void;
}

// 抽象类：提供部分实现
abstract class BaseLogger implements Loggable
{
    protected function formatMessage(string $message): string
    {
        return date('Y-m-d H:i:s') . " - {$message}";
    }
    
    // 子类实现 log 方法
    abstract public function log(string $message): void;
}
```

## 对比分析

### 抽象类 vs 接口

| 特性         | 抽象类（Abstract Class）          | 接口（Interface）                 |
|:-------------|:--------------------------------|:--------------------------------|
| **实例化**    | ❌ 不能实例化                     | ❌ 不能实例化                     |
| **方法实现**  | ✅ 可以提供部分实现                | ❌ 不能提供实现（PHP 8.0+ 除外）  |
| **属性**      | ✅ 可以有属性                     | ❌ 不能有属性（可以有常量）        |
| **构造函数**  | ✅ 可以有构造函数                  | ❌ 不能有构造函数                  |
| **继承数量**  | 单继承                           | 可以实现多个接口                  |
| **可见性**    | 方法可以有不同可见性              | 方法必须是 public                |
| **适用场景**  | 需要部分实现，代码复用             | 定义契约，多态设计                |

### 抽象类 vs 普通类

| 特性         | 抽象类                           | 普通类                            |
|:-------------|:--------------------------------|:--------------------------------|
| **实例化**    | ❌ 不能实例化                     | ✅ 可以实例化                     |
| **抽象方法**  | ✅ 可以包含抽象方法                | ❌ 不能包含抽象方法                |
| **使用场景**  | 作为基类，提供模板                | 可以直接使用的完整类              |

### 抽象类 vs 接口 vs 普通类

| 特性         | 抽象类                           | 接口                               | 普通类                            |
|:-------------|:--------------------------------|:-----------------------------------|:--------------------------------|
| **实例化**    | ❌                                | ❌                                  | ✅                                |
| **实现**      | 部分实现                         | 无实现（PHP 8.0+ 除外）             | 完整实现                          |
| **继承**      | 单继承                           | 多实现                              | 单继承                            |
| **属性**      | ✅                                | ❌（常量）                          | ✅                                |
| **方法可见性**| 可以不同                         | 必须是 public                       | 可以不同                          |

## 练习任务

1. **创建抽象类**：创建一个 `Vehicle` 抽象类，包含抽象方法 `start()` 和 `stop()`，创建 `Car` 和 `Motorcycle` 子类实现这些方法。

2. **模板方法模式**：创建一个 `DataProcessor` 抽象类，使用模板方法模式定义数据处理流程（加载、转换、保存），创建具体子类实现。

3. **抽象类实现接口**：创建一个 `Drawable` 接口，创建一个 `Shape` 抽象类实现该接口，提供部分实现，创建 `Circle` 和 `Rectangle` 子类。

4. **多个抽象方法**：创建一个 `Notification` 抽象类，包含多个抽象方法，创建 `EmailNotification` 和 `SmsNotification` 子类。

5. **抽象类属性**：创建一个 `Employee` 抽象类，包含通用属性（name、id）和抽象方法 `calculateSalary()`，创建 `FullTimeEmployee` 和 `PartTimeEmployee` 子类。

## 相关章节

- **[3.3.1 继承（Inheritance）](section-01-inheritance.md)**：了解继承的基础知识
- **[3.3.2 接口（Interface）](section-02-interfaces.md)**：了解接口与抽象类的区别
- **[3.3.4 多态（Polymorphism）](section-04-polymorphism.md)**：了解抽象类在多态中的应用

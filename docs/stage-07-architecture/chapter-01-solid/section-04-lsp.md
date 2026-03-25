# 7.1.4 里氏替换原则（LSP）

## 概述

里氏替换原则（Liskov Substitution Principle，简称 LSP）是由计算机科学家 Barbara Liskov 在 1987 年提出的。该原则的核心思想是：子类对象应该能够替换其基类（或父类）对象而不影响程序的正确性。

换句话说，任何基类出现的地方，都应该可以使用子类来替代，而不会导致程序行为异常。这个原则是实现多态和面向对象设计的基础原则之一。它确保了继承关系的正确使用，防止不当的继承设计导致系统脆弱。

里氏替换原则的重要性在于它保证了代码的可扩展性和多态的正确性。如果子类不能正确地替换基类，那么依赖这些对象的代码就会出现问题。这不仅影响代码的正确性，也会破坏系统的可维护性。

理解里氏替换原则对于正确使用继承至关重要。很多时候，我们可能因为表面上看起来合理的继承关系而违反了 LSP，导致代码出现隐藏的 bug。本节将详细介绍里氏替换原则的概念、违反情况、以及正确的继承设计方法。

**主要内容**：
- 里氏替换原则的定义和重要性
- 子类与父类的正确关系
- 违反 LSP 的常见情况
- 如何正确设计继承关系
- 里氏替换原则与多态的关系
- 代码示例和最佳实践

## 特性

里氏替换原则具有以下核心特性：

- **可替换性**：子类对象可以出现在任何父类对象出现的地方
- **行为一致性**：子类方法的行为应该与父类方法的行为一致或更严格
- **契约保持**：子类必须遵守父类定义的契约
- **类型安全**：子类必须接受父类接受的所有输入，产生父类产生的所有输出
- **多态基础**：LSP 是实现多态的基础，确保多态调用的正确性

## 核心概念

### 什么是里氏替换

里氏替换原则可以用一句话概括：子类型必须能够替换其基类型。

这意味着：
- 如果函数接受类型 T 的对象，那么它也应该能够接受类型 T 的任何子类型 S 的对象
- 程序的行为应该是正确的，无论使用的是基类还是子类
- 客户代码不应该知道它使用的是基类还是子类

### 契约与规范

里氏替换原则与"契约式设计"（Design by Contract）的概念密切相关。每个方法都有：
- **前置条件**：方法执行前必须为真的条件
- **后置条件**：方法执行后必须为真的条件
- **不变量**：对象生命周期内始终为真的条件

子类不能放宽前置条件（必须接受父类接受的所有输入），但可以加严后置条件（产生父类产生的所有输出，加上更多）。

### 子类不是父类的"缩减版"

继承不是简单地"减去"一些功能。子类应该是父类的扩展，而不是父类的缩减。子类应该能够做父类能做的所有事情，加上更多。

## 违反 LSP 的示例

### 示例 1：违反方法签名兼容性

```php
<?php
declare(strict_types=1);

// 基类：矩形
class Rectangle
{
    protected float $width;
    protected float $height;
    
    public function __construct(float $width, float $height)
    {
        $this->width = $width;
        $this->height = $height;
    }
    
    public function setWidth(float $width): void
    {
        $this->width = $width;
    }
    
    public function setHeight(float $height): void
    {
        $this->height = $height;
    }
    
    public function getWidth(): float
    {
        return $this->width;
    }
    
    public function getHeight(): float
    {
        return $this->height;
    }
    
    public function area(): float
    {
        return $this->width * $this->height;
    }
}

// 子类：正方形（违反 LSP）
class Square extends Rectangle
{
    public function __construct(float $size)
    {
        parent::__construct($size, $size);
    }
    
    // 重写方法，但改变了行为
    public function setWidth(float $width): void
    {
        // 正方形需要保持宽高相等
        $this->width = $width;
        $this->height = $width;
    }
    
    public function setHeight(float $height): void
    {
        // 正方形需要保持宽高相等
        $this->width = $height;
        $this->height = $height;
    }
}

// 使用 Rectangle 的函数
function calculateArea(Rectangle $rectangle): float
{
    // 期望设置宽高后计算面积
    $rectangle->setWidth(5);
    $rectangle->setHeight(4);
    return $rectangle->area();
}

// 测试
$rectangle = new Rectangle(3, 4);
echo "Rectangle area: " . calculateArea($rectangle) . "\n"; // 20

$square = new Square(3);
echo "Square area: " . calculateArea($square) . "\n"; // 16 (错误！应该是 20)
```

**问题分析**：
- 正方形是特殊的矩形，这似乎是一个合理的继承关系
- 但是正方形的 setWidth() 和 setHeight() 方法相互影响
- 当 calculateArea() 先设置宽度再设置高度时，正方形的行为与预期不符
- 子类 Square 不能替换父类 Rectangle 而保持行为正确

### 示例 2：违反返回值类型兼容性

```php
<?php
declare(strict_types=1);

// 基类：数据获取器
interface DataFetcher
{
    public function fetch(): array;
}

// 子类：返回类型更严格（违反 LSP）
class StrictDataFetcher implements DataFetcher
{
    public function fetch(): array
    {
        // 可能返回空数组
        return [];
    }
}

// 使用 DataFetcher 的代码
function processData(DataFetcher $fetcher): void
{
    $data = $fetcher->fetch();
    
    // 假设至少有一个数据项
    $firstItem = $data[0]; // 如果返回空数组，这里会报错
    
    echo "Processing: " . $firstItem . "\n";
}

// 测试
$fetcher = new StrictDataFetcher();
processData($fetcher); // 可能产生 Notice 或 Warning
```

### 示例 3：抛出父类不允许的异常

```php
<?php
declare(strict_types=1);

// 基类：文件读取器
class FileReader
{
    protected string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function read(): string
    {
        if (!file_exists($this->filename)) {
            return '';
        }
        
        return file_get_contents($this->filename);
    }
}

// 子类：抛出父类不允许的异常（违反 LSP）
class StrictFileReader extends FileReader
{
    public function read(): string
    {
        if (!file_exists($this->filename)) {
            // 抛出了父类方法不抛出的异常
            throw new RuntimeException(
                "File not found: {$this->filename}"
            );
        }
        
        return file_get_contents($this->filename);
    }
}

// 使用 FileReader 的代码
function readFileContent(FileReader $reader): void
{
    try {
        $content = $reader->read();
        echo "Content: " . substr($content, 0, 100) . "\n";
    } catch (RuntimeException $e) {
        // 只捕获预期的异常
        echo "Error: " . $e->getMessage() . "\n";
    }
}

// 测试
$reader = new StrictFileReader('nonexistent.txt');
readFileContent($reader); // 可能抛出未捕获的异常
```

### 示例 4：违反前置条件

```php
<?php
declare(strict_types=1);

// 基类：账户
class Account
{
    protected float $balance = 0;
    
    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException(
                "Deposit amount must be positive"
            );
        }
        
        $this->balance += $amount;
    }
    
    public function getBalance(): float
    {
        return $this->balance;
    }
}

// 子类：放宽了前置条件（违反 LSP）
class FlexibleAccount extends Account
{
    public function deposit(float $amount): void
    {
        // 允许负数存款（实际上是取款）
        // 这放宽了前置条件，可能导致问题
        $this->balance += $amount;
    }
}

// 使用 Account 的代码
function depositSalary(Account $account): void
{
    // 代码假设只能存正数
    $account->deposit(-1000); // 这会在某些账户上出问题
}

$account = new FlexibleAccount();
depositSalary($account);
echo "Balance: " . $account->getBalance() . "\n"; // -1000
```

## 符合 LSP 的正确设计

### 重构示例 1：使用接口而非继承

```php
<?php
declare(strict_types=1);

// 使用接口定义契约，而不是继承
interface Shape
{
    public function area(): float;
}

interface Scalable
{
    public function scale(float $factor): void;
}

// 矩形实现
class Rectangle implements Shape, Scalable
{
    private float $width;
    private float $height;
    
    public function __construct(float $width, float $height)
    {
        $this->width = $width;
        $this->height = $height;
    }
    
    public function area(): float
    {
        return $this->width * $this->height;
    }
    
    public function scale(float $factor): void
    {
        $this->width *= $factor;
        $this->height *= $factor;
    }
    
    public function getWidth(): float
    {
        return $this->width;
    }
    
    public function getHeight(): float
    {
        return $this->height;
    }
}

// 正方形实现（不再继承 Rectangle）
class Square implements Shape, Scalable
{
    private float $side;
    
    public function __construct(float $side)
    {
        $this->side = $side;
    }
    
    public function area(): float
    {
        return $this->side * $this->side;
    }
    
    public function scale(float $factor): void
    {
        $this->side *= $factor;
    }
    
    public function getSide(): float
    {
        return $this->side;
    }
}

// 使用 Shape 接口的函数
function calculateTotalArea(array $shapes): float
{
    $total = 0;
    
    foreach ($shapes as $shape) {
        $total += $shape->area();
    }
    
    return $total;
}

// 测试
$shapes = [
    new Rectangle(3, 4),
    new Square(5),
    new Rectangle(2, 3)
];

echo "Total area: " . calculateTotalArea($shapes) . "\n"; // 12 + 25 + 6 = 43
```

### 重构示例 2：正确处理异常

```php
<?php
declare(strict_types=1);

// 基类：数据存储
interface DataStore
{
    /**
     * @throws DataStoreException 如果存储失败
     */
    public function save(string $key, mixed $value): void;
    
    /**
     * @throws DataStoreException 如果数据不存在
     */
    public function load(string $key): mixed;
    
    public function exists(string $key): bool;
}

// 基础异常类
class DataStoreException extends Exception {}

// 文件存储实现
class FileDataStore implements DataStore
{
    private string $directory;
    
    public function __construct(string $directory)
    {
        $this->directory = $directory;
        
        if (!is_dir($directory)) {
            mkdir($directory, 0755, true);
        }
    }
    
    public function save(string $key, mixed $value): void
    {
        $path = $this->getPath($key);
        $result = file_put_contents(
            $path,
            serialize($value)
        );
        
        if ($result === false) {
            throw new DataStoreException(
                "Failed to save data: {$key}"
            );
        }
    }
    
    public function load(string $key): mixed
    {
        $path = $this->getPath($key);
        
        if (!file_exists($path)) {
            throw new DataStoreException(
                "Data not found: {$key}"
            );
        }
        
        $content = file_get_contents($path);
        
        if ($content === false) {
            throw new DataStoreException(
                "Failed to load data: {$key}"
            );
        }
        
        return unserialize($content);
    }
    
    public function exists(string $key): bool
    {
        return file_exists($this->getPath($key));
    }
    
    private function getPath(string $key): string
    {
        return $this->directory . '/' . md5($key) . '.dat';
    }
}

// 缓存存储实现（子类不抛出新异常类型）
class CacheDataStore implements DataStore
{
    private array $cache = [];
    
    public function save(string $key, mixed $value): void
    {
        // 缓存可能失败，但不抛出异常
        $this->cache[$key] = $value;
    }
    
    public function load(string $key): mixed
    {
        if (!isset($this->cache[$key])) {
            // 缓存未命中时，抛出标准异常
            throw new DataStoreException(
                "Cache miss: {$key}"
            );
        }
        
        return $this->cache[$key];
    }
    
    public function exists(string $key): bool
    {
        return isset($this->cache[$key]);
    }
}

// 使用 DataStore 的代码
function loadUserData(DataStore $store, string $userId): array
{
    $key = "user:{$userId}";
    
    if (!$store->exists($key)) {
        return ['error' => 'User not found'];
    }
    
    try {
        return $store->load($key);
    } catch (DataStoreException $e) {
        return ['error' => $e->getMessage()];
    }
}

// 测试
$fileStore = new FileDataStore('/tmp/data');
$cacheStore = new CacheDataStore();

$fileStore->save('test', ['name' => 'John']);
echo "File store exists: " . ($fileStore->exists('test') ? 'yes' : 'no') . "\n";

$cacheStore->save('test', ['name' => 'Jane']);
echo "Cache store exists: " . ($cacheStore->exists('test') ? 'yes' : 'no') . "\n";
```

### 重构示例 3：正确的继承设计

```php
<?php
declare(strict_types=1);

// 基类：鸟
class Bird
{
    public function fly(): string
    {
        return "Bird is flying";
    }
    
    public function eat(): string
    {
        return "Bird is eating";
    }
}

// 子类：企鹅（不应该继承 Bird）
class Penguin extends Bird
{
    // 企鹅不会飞，违反 LSP
    // 如果必须继承，需要重写 fly() 方法
    public function fly(): string
    {
        throw new RuntimeException("Penguins cannot fly");
    }
}

// 更好的设计：使用组合和接口
interface Flyable
{
    public function fly(): string;
}

interface Eater
{
    public function eat(): string;
}

// 会飞的鸟
class FlyingBird implements Flyable, Eater
{
    public function fly(): string
    {
        return "Bird is flying";
    }
    
    public function eat(): string
    {
        return "Bird is eating";
    }
}

// 不会飞的鸟
class FlightlessBird implements Eater
{
    public function eat(): string
    {
        return "Bird is eating";
    }
}

// 企鹅
class PenguinBird extends FlightlessBird
{
    // 企鹅不会飞，但可以吃
}

// 使用示例
function makeItFly(Flyable $bird): void
{
    echo $bird->fly() . "\n";
}

function makeItEat(Eater $bird): void
{
    echo $bird->eat() . "\n";
}

$penguin = new PenguinBird();
makeItEat($penguin); // 可以吃
// makeItFly($penguin); // 不可以飞，编译时会报错
```

## 基本用法

### 使用测试验证 LSP

```php
<?php
declare(strict_types=1);

// 基类
abstract class PaymentProcessor
{
    abstract public function process(float $amount): bool;
    abstract public function refund(string $transactionId): bool;
}

// 测试基类：验证子类是否符合 LSP
function testLSP(PaymentProcessor $processor): void
{
    // 测试处理支付
    $result = $processor->process(100.0);
    echo "Process result: " . ($result ? 'success' : 'failed') . "\n";
    
    // 测试退款
    $refundResult = $processor->refund('txn_123');
    echo "Refund result: " . ($refundResult ? 'success' : 'failed') . "\n";
}

// 测试各种支付处理器
class CreditCardProcessor extends PaymentProcessor
{
    public function process(float $amount): bool
    {
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        return true;
    }
}

class PayPalProcessor extends PaymentProcessor
{
    public function process(float $amount): bool
    {
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        return true;
    }
}

// 测试
testLSP(new CreditCardProcessor());
testLSP(new PayPalProcessor());
```

## 使用场景

里氏替换原则适用于以下场景：

- **使用继承时**：任何使用继承的地方都应该检查是否符合 LSP
- **多态设计时**：确保多态调用不会产生意外行为
- **定义契约时**：定义接口和抽象类时，考虑子类是否能正确实现
- **代码审查时**：检查继承关系是否合理

## 注意事项

### 继承 vs 组合

继承并不总是最佳选择。当子类不能完全替代父类时，应该考虑：
- 使用接口定义契约
- 使用组合而非继承
- 重新设计类的层次结构

### "是一个" vs "像是一个"

判断继承是否合理的简单方法：
- "子类是一个父类" → 可能符合 LSP
- "子类像是一个父类，但..." → 可能违反 LSP

### 契约思维

在设计继承时，始终考虑契约：
- 子类不能放宽前置条件
- 子类不能加严后置条件（不能减少输出）
- 子类不能抛出父类方法声明之外的异常

## 常见问题

### 什么时候可以使用继承？

继承应该是 "is-a" 关系：
- 企鹅是鸟（但不会飞，所以不应该直接继承会飞的鸟）
- 正方形是矩形（但行为不同，需要重新考虑）
- 猫是动物（通常符合）

### 子类可以添加新方法吗？

可以。子类可以添加父类没有的方法，这是扩展。但子类不能改变父类方法的行为（除非更严格）。

### 如何发现违反 LSP 的代码？

信号包括：
- 需要检查对象的具体类型
- 使用 `instanceof` 判断
- 子类重写方法后行为与预期不符
- 需要在调用方做特殊处理

### LSP 与 PHP 类型系统

PHP 8.1+ 支持更强的类型系统。使用：
- 严格类型声明
- 返回类型声明
- 参数类型声明

可以帮助确保 LSP。

## 最佳实践

1. **优先使用接口**：使用接口定义行为，而不是通过继承

2. **遵循契约**：子类必须遵守父类定义的契约

3. **测试多态**：使用基类引用测试子类，确保行为正确

4. **最小化继承层次**：过深的继承层次会增加违反 LSP 的风险

5. **使用组合**：当继承不合适时，使用组合代替

6. **编写 LSP 测试**：为每个继承关系编写测试，确保子类可以替换父类

7. **代码审查**：在代码审查中特别关注继承关系

## 练习任务

1. **分析继承关系**：分析你过去的代码，找出违反 LSP 的继承关系，并思考如何重构

2. **设计形状系统**：设计一个处理各种形状（圆、矩形、三角形等）的系统，使用接口而非不当的继承

3. **验证 LSP**：为一个现有类层次编写测试，验证子类是否可以正确替换父类

4. **思考正方形问题**：深入思考"正方形是否是矩形的子类"这个问题，写出你的分析和结论

5. **重构现有代码**：找到一个使用 `instanceof` 或类型检查的代码，尝试使用 LSP 重构

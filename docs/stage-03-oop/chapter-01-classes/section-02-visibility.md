# 3.1.2 可见性修饰符

## 概述

可见性修饰符控制类成员的访问权限，是封装（Encapsulation）的核心机制。本节详细介绍 `public`、`protected`、`private` 三种可见性修饰符的区别、使用场景、访问规则，以及如何通过可见性修饰符实现良好的封装设计。

封装是面向对象编程的三大特性之一（封装、继承、多态），通过控制类成员的访问权限，可以隐藏实现细节，只暴露必要的接口，提高代码的安全性和可维护性。可见性修饰符决定了类成员（属性和方法）在哪些地方可以被访问。

**主要内容**：
- `public` 修饰符的定义和使用
- `protected` 修饰符的定义和使用
- `private` 修饰符的定义和使用
- 三种修饰符的访问范围对比
- 可见性修饰符的最佳实践
- 属性封装的重要性
- Getter 和 Setter 方法
- 可见性修饰符的继承规则（详细说明见 [3.3.1 继承（Inheritance）](../chapter-03-oop-features/section-01-inheritance.md)）

## 特性

- **public（公开）**：任何地方都可以访问，包括类外部、子类、类内部
- **protected（受保护）**：类内部和子类可以访问，类外部不能访问
- **private（私有）**：只有类内部可以访问，子类和类外部都不能访问
- **封装性**：通过可见性修饰符隐藏实现细节，只暴露必要的接口
- **安全性**：防止外部代码直接访问和修改内部数据

## 语法/定义

### public 修饰符

**语法**：`public Type $property;` 或 `public function methodName(): ReturnType`

**访问范围**：
- 类内部：可以访问
- 子类：可以访问
- 类外部：可以访问

**特点**：
- 最宽松的访问权限
- 任何地方都可以访问
- 适合需要公开的接口

### protected 修饰符

**语法**：`protected Type $property;` 或 `protected function methodName(): ReturnType`

**访问范围**：
- 类内部：可以访问
- 子类：可以访问
- 类外部：不能访问

**特点**：
- 中等访问权限
- 子类可以访问，适合需要被子类继承的成员
- 类外部不能访问，保护内部实现

### private 修饰符

**语法**：`private Type $property;` 或 `private function methodName(): ReturnType`

**访问范围**：
- 类内部：可以访问
- 子类：不能访问
- 类外部：不能访问

**特点**：
- 最严格的访问权限
- 只有类内部可以访问
- 完全隐藏实现细节

## 基本用法

### 示例 1：public 成员的使用

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;  // 公开属性
    
    public function getName(): string  // 公开方法
    {
        return $this->name;
    }
    
    public function setName(string $name): void  // 公开方法
    {
        $this->name = $name;
    }
}

// 类外部可以直接访问 public 成员
$user = new User();
$user->name = "John";              // 直接访问属性
echo $user->name . "\n";           // John
echo $user->getName() . "\n";      // John（通过方法访问）

$user->setName("Jane");
echo $user->getName() . "\n";      // Jane
```

**输出**：

```
John
John
Jane
```

**说明**：
- `public` 成员可以在类外部直接访问
- 可以直接访问属性或通过方法访问
- `public` 提供了最大的访问灵活性

### 示例 2：protected 成员的使用

```php
<?php
declare(strict_types=1);

class Animal
{
    protected string $name;  // 受保护属性
    
    public function setName(string $name): void
    {
        $this->name = $name;  // 类内部可以访问
    }
    
    protected function getName(): string  // 受保护方法
    {
        return $this->name;
    }
    
    public function displayInfo(): string
    {
        return "Animal: {$this->getName()}";  // 类内部可以调用 protected 方法
    }
}

class Dog extends Animal
{
    public function getDogName(): string
    {
        return $this->name;  // 子类可以访问 protected 属性
    }
    
    public function callProtectedMethod(): string
    {
        return $this->getName();  // 子类可以调用 protected 方法
    }
}

$animal = new Animal();
$animal->setName("Cat");
echo $animal->displayInfo() . "\n";  // Animal: Cat

// $animal->name = "Dog";  // 错误：不能访问 protected 属性
// echo $animal->getName();  // 错误：不能调用 protected 方法

$dog = new Dog();
$dog->setName("Buddy");
echo $dog->getDogName() . "\n";              // Buddy（子类可以访问）
echo $dog->callProtectedMethod() . "\n";     // Buddy（子类可以调用）
```

**输出**：

```
Animal: Cat
Buddy
Buddy
```

**说明**：
- `protected` 成员可以在类内部和子类中访问
- 类外部不能直接访问 `protected` 成员
- `protected` 适合需要被子类继承但不需要公开的成员

### 示例 3：private 成员的使用

```php
<?php
declare(strict_types=1);

class BankAccount
{
    private string $accountNumber;  // 私有属性
    private float $balance = 0.0;   // 私有属性
    
    public function __construct(string $accountNumber)
    {
        $this->accountNumber = $accountNumber;  // 类内部可以访问
    }
    
    private function validateAmount(float $amount): bool  // 私有方法
    {
        return $amount > 0;
    }
    
    public function deposit(float $amount): bool
    {
        if (!$this->validateAmount($amount)) {  // 类内部可以调用 private 方法
            return false;
        }
        
        $this->balance += $amount;  // 类内部可以访问 private 属性
        return true;
    }
    
    public function withdraw(float $amount): bool
    {
        if (!$this->validateAmount($amount)) {
            return false;
        }
        
        if ($amount > $this->balance) {
            return false;
        }
        
        $this->balance -= $amount;
        return true;
    }
    
    public function getBalance(): float
    {
        return $this->balance;  // 通过公开方法访问私有属性
    }
    
    public function getAccountNumber(): string
    {
        return $this->accountNumber;
    }
}

$account = new BankAccount("ACC001");
$account->deposit(1000);
echo "Balance: " . $account->getBalance() . "\n";  // Balance: 1000

// $account->balance = 9999;  // 错误：不能访问 private 属性
// $account->validateAmount(100);  // 错误：不能调用 private 方法
// echo $account->accountNumber;  // 错误：不能访问 private 属性

$account->withdraw(300);
echo "Balance: " . $account->getBalance() . "\n";  // Balance: 700
```

**输出**：

```
Balance: 1000
Balance: 700
```

**说明**：
- `private` 成员只能在类内部访问
- 子类和类外部都不能访问 `private` 成员
- `private` 完全隐藏实现细节，提供最强的封装

### 示例 4：三种修饰符的对比

```php
<?php
declare(strict_types=1);

class Example
{
    public string $publicProperty = "Public";
    protected string $protectedProperty = "Protected";
    private string $privateProperty = "Private";
    
    public function publicMethod(): string
    {
        return $this->publicProperty . " - " . 
               $this->protectedProperty . " - " . 
               $this->privateProperty;  // 类内部可以访问所有成员
    }
    
    protected function protectedMethod(): string
    {
        return "Protected Method";
    }
    
    private function privateMethod(): string
    {
        return "Private Method";
    }
    
    public function testInternalAccess(): void
    {
        echo $this->publicMethod() . "\n";      // 可以调用 public 方法
        echo $this->protectedMethod() . "\n";   // 可以调用 protected 方法
        echo $this->privateMethod() . "\n";     // 可以调用 private 方法
    }
}

class ChildExample extends Example
{
    public function testChildAccess(): void
    {
        echo $this->publicProperty . "\n";      // 可以访问 public
        echo $this->protectedProperty . "\n";   // 可以访问 protected
        // echo $this->privateProperty . "\n";  // 错误：不能访问 private
        
        $this->publicMethod();                  // 可以调用 public
        $this->protectedMethod();               // 可以调用 protected
        // $this->privateMethod();              // 错误：不能调用 private
    }
}

$example = new Example();
echo $example->publicProperty . "\n";           // Public（可以访问）
// echo $example->protectedProperty . "\n";     // 错误：不能访问 protected
// echo $example->privateProperty . "\n";       // 错误：不能访问 private

$example->testInternalAccess();
// $example->protectedMethod();                 // 错误：不能调用 protected
// $example->privateMethod();                   // 错误：不能调用 private

$child = new ChildExample();
$child->testChildAccess();
```

**输出**：

```
Public
Public - Protected - Private
Protected Method
Private Method
Public
Protected
```

**说明**：
- 类内部可以访问所有可见性的成员
- 子类可以访问 `public` 和 `protected`，但不能访问 `private`
- 类外部只能访问 `public` 成员

### 示例 5：属性封装（Getter 和 Setter）

```php
<?php
declare(strict_types=1);

class Person
{
    private string $name;
    private int $age;
    private string $email;
    
    // Getter 方法：获取属性值
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
    
    // Setter 方法：设置属性值（可以添加验证）
    public function setName(string $name): void
    {
        if (empty(trim($name))) {
            throw new InvalidArgumentException("Name cannot be empty");
        }
        $this->name = trim($name);
    }
    
    public function setAge(int $age): void
    {
        if ($age < 0 || $age > 150) {
            throw new InvalidArgumentException("Age must be between 0 and 150");
        }
        $this->age = $age;
    }
    
    public function setEmail(string $email): void
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email format");
        }
        $this->email = $email;
    }
}

$person = new Person();
$person->setName("John Doe");
$person->setAge(25);
$person->setEmail("john@example.com");

echo "Name: " . $person->getName() . "\n";
echo "Age: " . $person->getAge() . "\n";
echo "Email: " . $person->getEmail() . "\n";

// 属性是 private，不能直接访问
// $person->name = "Jane";  // 错误：不能访问 private 属性
// echo $person->age;       // 错误：不能访问 private 属性

// Setter 方法可以添加验证
try {
    $person->setAge(200);  // 抛出异常
} catch (InvalidArgumentException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

**输出**：

```
Name: John Doe
Age: 25
Email: john@example.com
Error: Age must be between 0 and 150
```

**说明**：
- 属性使用 `private`，完全隐藏内部实现
- 通过 `public` 的 Getter 和 Setter 方法访问属性
- Setter 方法可以添加验证逻辑，确保数据有效性
- 这是封装的最佳实践

## 使用场景

### 场景 1：public 成员

**适用场景**：
- 需要对外公开的接口
- 类的核心功能方法
- 简单数据容器（如 DTO）

**示例**：简单的数据容器

```php
<?php
declare(strict_types=1);

class Point
{
    public float $x;
    public float $y;
    
    public function __construct(float $x, float $y)
    {
        $this->x = $x;
        $this->y = $y;
    }
    
    public function distance(Point $other): float
    {
        $dx = $this->x - $other->x;
        $dy = $this->y - $other->y;
        return sqrt($dx * $dx + $dy * $dy);
    }
}

$point1 = new Point(0, 0);
$point2 = new Point(3, 4);
echo "Distance: " . $point1->distance($point2) . "\n";  // Distance: 5
```

### 场景 2：protected 成员

**适用场景**：
- 需要被子类继承但不公开的成员
- 模板方法模式中的钩子方法
- 需要子类扩展的方法

**示例**：模板方法模式

```php
<?php
declare(strict_types=1);

abstract class PaymentProcessor
{
    public function processPayment(float $amount): bool
    {
        $this->validateAmount($amount);  // 模板方法
        $this->authenticate();           // 模板方法
        return $this->executePayment($amount);  // 子类实现
    }
    
    protected function validateAmount(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Amount must be positive");
        }
    }
    
    protected abstract function authenticate(): void;
    protected abstract function executePayment(float $amount): bool;
}

class CreditCardProcessor extends PaymentProcessor
{
    protected function authenticate(): void
    {
        echo "Authenticating credit card...\n";
    }
    
    protected function executePayment(float $amount): bool
    {
        echo "Processing credit card payment: {$amount}\n";
        return true;
    }
}

$processor = new CreditCardProcessor();
$processor->processPayment(100.0);
```

### 场景 3：private 成员

**适用场景**：
- 内部实现细节
- 辅助方法
- 不应该被外部或子类访问的成员

**示例**：内部实现细节

```php
<?php
declare(strict_types=1);

class PasswordHasher
{
    private const SALT_LENGTH = 16;
    
    public function hash(string $password): string
    {
        $salt = $this->generateSalt();
        return $this->combinePasswordAndSalt($password, $salt);
    }
    
    public function verify(string $password, string $hash): bool
    {
        // 验证逻辑
        return password_verify($password, $hash);
    }
    
    private function generateSalt(): string
    {
        return bin2hex(random_bytes(self::SALT_LENGTH));
    }
    
    private function combinePasswordAndSalt(string $password, string $salt): string
    {
        return password_hash($password . $salt, PASSWORD_DEFAULT);
    }
}

$hasher = new PasswordHasher();
$hash = $hasher->hash("myPassword");
echo "Hash created: " . (strlen($hash) > 0 ? "Yes" : "No") . "\n";
// $hasher->generateSalt();  // 错误：不能调用 private 方法
```

## 注意事项

### 最小权限原则

- **使用最小必要的可见性**：默认使用 `private`，只有在需要时才提升可见性
- **优先使用 private**：属性通常应该是 `private`，通过方法访问
- **谨慎使用 public**：只有真正需要公开的接口才使用 `public`

**示例**：

```php
<?php
declare(strict_types=1);

// 不好的设计：过度使用 public
class BadDesign
{
    public string $name;
    public int $age;
    public string $email;
    public float $balance;
}

// 好的设计：使用 private 和 Getter/Setter
class GoodDesign
{
    private string $name;
    private int $age;
    private string $email;
    private float $balance;
    
    public function getName(): string { return $this->name; }
    public function setName(string $name): void { $this->name = $name; }
    
    public function getAge(): int { return $this->age; }
    public function setAge(int $age): void 
    { 
        if ($age < 0) {
            throw new InvalidArgumentException("Age cannot be negative");
        }
        $this->age = $age;
    }
}
```

### 属性封装的重要性

- **数据保护**：防止外部代码直接修改属性，可能导致数据不一致
- **验证逻辑**：在 Setter 方法中可以添加验证，确保数据有效性
- **灵活性**：将来可以改变内部实现，而不影响外部代码

**示例**：

```php
<?php
declare(strict_types=1);

class Counter
{
    private int $count = 0;
    
    public function increment(): void
    {
        $this->count++;
    }
    
    public function getCount(): int
    {
        return $this->count;
    }
    
    // 如果直接暴露属性，外部代码可能这样做：
    // $counter->count = -100;  // 破坏数据完整性
    // 使用 private + getter 可以防止这种情况
}

$counter = new Counter();
$counter->increment();
$counter->increment();
echo $counter->getCount() . "\n";  // 2

// $counter->count = -100;  // 错误：不能访问 private 属性
```

### 可见性修饰符的继承规则

- **public**：子类可以访问，保持 `public`
- **protected**：子类可以访问，可以改为 `public`，但不能改为 `private`
- **private**：子类不能访问，子类可以定义同名的 `private` 成员（但这是不同的成员）

详细说明见 [3.3.1 继承（Inheritance）](../chapter-03-oop-features/section-01-inheritance.md)。

### 方法可见性的设计

- **公共接口**：对外提供的功能使用 `public`
- **内部实现**：辅助方法、工具方法使用 `private`
- **扩展点**：需要子类扩展的方法使用 `protected`

## 常见问题

### 问题 1：过度使用 public

**症状**：所有属性和方法都使用 `public`

**原因**：不理解封装的重要性，图方便

**解决方案**：
- 遵循最小权限原则
- 属性默认使用 `private`
- 只有需要对外公开的接口才使用 `public`

**示例**：

```php
<?php
declare(strict_types=1);

// 不好的设计
class BadUser
{
    public string $name;
    public int $age;
    public string $password;  // 危险：密码应该是 private
}

// 好的设计
class GoodUser
{
    private string $name;
    private int $age;
    private string $password;  // 私有，保护敏感数据
    
    public function getName(): string { return $this->name; }
    public function setName(string $name): void { $this->name = $name; }
    
    public function setPassword(string $password): void
    {
        $this->password = password_hash($password, PASSWORD_DEFAULT);
    }
}
```

### 问题 2：可见性设计不当

**症状**：应该使用 `private` 的用了 `public`，应该使用 `protected` 的用了 `private`

**原因**：不理解三种修饰符的区别和适用场景

**解决方案**：
- 理解三种修饰符的访问范围
- 根据实际需求选择合适的可见性
- 遵循最小权限原则

### 问题 3：试图访问 private 成员

**错误信息**：`Fatal error: Cannot access private property` 或 `Fatal error: Cannot access private method`

**原因**：
- 在类外部或子类中试图访问 `private` 成员
- 不理解 `private` 的访问限制

**解决方案**：
- 使用 `public` 的 Getter/Setter 方法访问 `private` 属性
- 将需要子类访问的成员改为 `protected`
- 理解可见性修饰符的访问规则

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    private string $privateProperty = "Private";
    
    public function getPrivateProperty(): string
    {
        return $this->privateProperty;  // 通过 public 方法访问
    }
}

$example = new Example();
// echo $example->privateProperty;  // 错误：不能访问 private 属性
echo $example->getPrivateProperty() . "\n";  // 正确：通过方法访问
```

### 问题 4：子类无法访问父类的 private 成员

**错误信息**：`Fatal error: Cannot access private property in ...`

**原因**：
- 子类试图访问父类的 `private` 成员
- `private` 成员不能被继承

**解决方案**：
- 将需要子类访问的成员改为 `protected`
- 理解 `private`、`protected`、`public` 在继承中的区别

**示例**：

```php
<?php
declare(strict_types=1);

class ParentClass
{
    private string $privateProperty = "Private";
    protected string $protectedProperty = "Protected";
    public string $publicProperty = "Public";
}

class ChildClass extends ParentClass
{
    public function testAccess(): void
    {
        // echo $this->privateProperty;  // 错误：不能访问 private
        echo $this->protectedProperty . "\n";  // 正确：可以访问 protected
        echo $this->publicProperty . "\n";     // 正确：可以访问 public
    }
}

$child = new ChildClass();
$child->testAccess();
```

## 最佳实践

### 1. 属性通常使用 private，通过 Getter/Setter 访问

**原则**：
- 属性默认使用 `private`
- 提供 `public` 的 Getter 和 Setter 方法
- Setter 方法可以添加验证逻辑

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    private string $name;
    private int $age;
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function setName(string $name): void
    {
        if (empty(trim($name))) {
            throw new InvalidArgumentException("Name cannot be empty");
        }
        $this->name = trim($name);
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
    
    public function setAge(int $age): void
    {
        if ($age < 0 || $age > 150) {
            throw new InvalidArgumentException("Invalid age");
        }
        $this->age = $age;
    }
}
```

### 2. 方法根据使用场景选择合适的可见性

**原则**：
- **public**：对外提供的接口
- **protected**：需要子类扩展的方法
- **private**：内部辅助方法

**示例**：

```php
<?php
declare(strict_types=1);

class DataProcessor
{
    // public：对外接口
    public function process(array $data): array
    {
        $this->validate($data);
        $this->normalize($data);
        return $this->transform($data);
    }
    
    // protected：需要子类扩展
    protected function transform(array $data): array
    {
        // 默认实现，子类可以覆盖
        return $data;
    }
    
    // private：内部实现
    private function validate(array $data): void
    {
        // 验证逻辑
    }
    
    private function normalize(array $data): void
    {
        // 规范化逻辑
    }
}
```

### 3. 遵循最小权限原则

**原则**：
- 默认使用最严格的可见性（`private`）
- 只有在需要时才提升可见性
- 不要过度暴露内部实现

**决策流程**：
1. 是否需要外部访问？→ 不需要：使用 `private`
2. 是否需要子类访问？→ 需要：使用 `protected`
3. 是否需要完全公开？→ 需要：使用 `public`

### 4. 理解封装的重要性

**封装的好处**：
- **数据保护**：防止外部代码直接修改数据
- **灵活性**：可以改变内部实现而不影响外部代码
- **可维护性**：集中管理数据访问逻辑
- **安全性**：保护敏感数据（如密码）

**示例**：

```php
<?php
declare(strict_types=1);

class SecureUser
{
    private string $password;
    
    public function setPassword(string $password): void
    {
        // 集中处理密码哈希
        $this->password = password_hash($password, PASSWORD_DEFAULT);
    }
    
    public function verifyPassword(string $password): bool
    {
        return password_verify($password, $this->password);
    }
    
    // 不提供 getPassword() 方法，保护密码
}
```

### 5. 在文档中说明可见性设计

**原则**：
- 在类文档中说明哪些是公开接口
- 说明 `protected` 方法的用途和扩展方式
- 帮助使用者理解类的设计意图

## 对比分析

### public vs protected vs private

| 特性         | public                         | protected                      | private                        |
|:-------------|:-------------------------------|:-------------------------------|:-------------------------------|
| **类内部**    | ✅ 可以访问                     | ✅ 可以访问                     | ✅ 可以访问                     |
| **子类**      | ✅ 可以访问                     | ✅ 可以访问                     | ❌ 不能访问                     |
| **类外部**    | ✅ 可以访问                     | ❌ 不能访问                     | ❌ 不能访问                     |
| **使用场景**  | 对外接口                       | 需要子类扩展的成员               | 内部实现细节                    |
| **封装程度**  | 最低                           | 中等                           | 最高                           |

### 直接访问属性 vs Getter/Setter

| 特性         | 直接访问属性（public）           | Getter/Setter（private + 方法） |
|:-------------|:--------------------------------|:--------------------------------|
| **访问方式**  | `$object->property`              | `$object->getProperty()`         |
| **数据保护**  | ❌ 无保护                       | ✅ 可以添加验证                  |
| **灵活性**    | ❌ 难以改变实现                  | ✅ 可以改变内部实现               |
| **代码量**    | ✅ 简洁                         | ❌ 需要更多代码                  |
| **适用场景**  | 简单的数据容器                   | 需要保护的业务对象                |

**示例对比**：

```php
<?php
declare(strict_types=1);

// 方式 1：直接访问（简单但不安全）
class SimplePoint
{
    public float $x;
    public float $y;
}

$point = new SimplePoint();
$point->x = 10;
$point->y = -999;  // 可能是不合理的值

// 方式 2：Getter/Setter（更安全）
class SafePoint
{
    private float $x;
    private float $y;
    
    public function setX(float $x): void
    {
        if ($x < 0) {
            throw new InvalidArgumentException("X must be non-negative");
        }
        $this->x = $x;
    }
    
    public function setY(float $y): void
    {
        if ($y < 0) {
            throw new InvalidArgumentException("Y must be non-negative");
        }
        $this->y = $y;
    }
    
    public function getX(): float { return $this->x; }
    public function getY(): float { return $this->y; }
}

$safePoint = new SafePoint();
$safePoint->setX(10);
// $safePoint->setY(-999);  // 抛出异常，保护数据完整性
```

## 练习任务

1. **创建用户类**：定义一个 `User` 类，包含 `name`、`email`、`password` 属性。`name` 和 `email` 使用 `private`，提供 Getter 和 Setter 方法；`password` 使用 `private`，只提供 `setPassword()` 和 `verifyPassword()` 方法，不提供 Getter。

2. **创建银行账户类**：定义一个 `BankAccount` 类，包含 `accountNumber` 和 `balance` 属性。`balance` 使用 `private`，提供 `deposit()`、`withdraw()`、`getBalance()` 方法，不允许直接修改余额。

3. **创建继承关系**：定义一个 `Animal` 基类，包含 `name`（protected）和 `sound`（private）属性。定义一个 `Dog` 子类，尝试访问这两个属性，观察哪些可以访问，哪些不能访问。

4. **可见性设计练习**：设计一个 `Logger` 类，包含日志记录的公开接口，以及内部的文件操作、格式化等私有方法，展示可见性的合理使用。

5. **Getter/Setter 验证**：创建一个 `Product` 类，包含 `name`、`price`、`stock` 属性（全部 private），在 Setter 方法中添加验证：
   - `name` 不能为空
   - `price` 必须大于 0
   - `stock` 必须大于等于 0

## 相关章节

- **[3.1.1 类与对象基础](section-01-basics.md)**：回顾类与对象的基础知识
- **[3.1.3 构造函数与析构函数](section-03-constructors-destructors.md)**：了解如何初始化对象
- **[3.3.1 继承（Inheritance）](../chapter-03-oop-features/section-01-inheritance.md)**：了解可见性修饰符在继承中的规则

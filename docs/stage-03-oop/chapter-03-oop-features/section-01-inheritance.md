# 3.3.1 继承（Inheritance）

## 概述

继承（Inheritance）是面向对象编程的核心特性之一，允许子类（Child Class）继承父类（Parent Class）的属性和方法，实现代码复用和层次化设计。通过继承，子类可以获得父类的所有 `public` 和 `protected` 成员，可以重写父类方法，也可以添加新的方法和属性。

理解继承对于掌握面向对象编程至关重要。继承提供了一种组织代码的方式，可以建立类的层次结构，实现代码复用。同时，继承也是多态的基础，为多态提供了必要的类层次结构。

PHP 支持单继承，即一个类只能继承一个父类。虽然这限制了继承的灵活性，但通过接口（Interface）和 Traits（详见 [3.4.1 Traits](../chapter-04-metaprogramming/section-01-traits.md)）可以补充这种限制。

**主要内容**：
- 继承的基础语法和 `extends` 关键字
- 访问父类方法和属性
- `parent` 关键字的使用
- 构造函数的继承和调用
- `final` 关键字的使用（防止继承和方法重写）
- 继承中的可见性规则
- 继承层次设计
- 继承的最佳实践

## 特性

- **代码复用**：子类继承父类的属性和方法，避免重复代码
- **层次化设计**：建立类的层次结构，体现"是一个"（is-a）关系
- **方法重写**：子类可以重写父类方法，实现特定行为
- **多态基础**：继承为多态提供了类层次结构基础
- **单继承**：PHP 只支持单继承（一个类只能继承一个父类）

## 语法/定义

### 继承语法

**语法**：`class ChildClass extends ParentClass { ... }`

**组成部分**：
- `class` 关键字：声明类
- `ChildClass`：子类名称
- `extends` 关键字：表示继承关系
- `ParentClass`：父类名称

**特点**：
- 使用 `extends` 关键字声明继承关系
- PHP 支持单继承（一个类只能继承一个父类）
- 子类可以访问父类的 `public` 和 `protected` 成员
- 子类不能访问父类的 `private` 成员

### parent 关键字

**语法**：`parent::methodName()` 或 `parent::$propertyName`

**组成部分**：
- `parent` 关键字：引用父类
- `::` 操作符：范围解析操作符
- `methodName` 或 `$propertyName`：父类的方法或属性

**特点**：
- 用于在子类中调用父类的方法
- 用于在子类构造函数中调用父类构造函数
- 可以是静态方法或实例方法

### final 关键字

**语法**：
- `final class ClassName { ... }`：防止类被继承
- `final public function methodName() { ... }`：防止方法被重写

**特点**：
- `final class`：类不能被继承
- `final method`：方法不能被重写
- 用于防止不必要的继承和方法重写

## 基本用法

### 示例 1：基础继承

```php
<?php
declare(strict_types=1);

class Animal
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
    
    public function makeSound(): string
    {
        return "Some sound";
    }
    
    public function move(): string
    {
        return "Moving";
    }
    
    public function getInfo(): string
    {
        return "{$this->name} is {$this->age} years old";
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return "Woof!";
    }
    
    public function fetch(): string
    {
        return "Fetching the ball";
    }
}

class Cat extends Animal
{
    public function makeSound(): string
    {
        return "Meow!";
    }
    
    public function climb(): string
    {
        return "Climbing the tree";
    }
}

// 使用继承
$dog = new Dog("Buddy", 3);
echo $dog->getInfo() . "\n";        // Buddy is 3 years old（继承自父类）
echo $dog->makeSound() . "\n";      // Woof!（重写了父类方法）
echo $dog->move() . "\n";           // Moving（继承自父类）
echo $dog->fetch() . "\n";          // Fetching the ball（子类新增方法）

$cat = new Cat("Fluffy", 2);
echo $cat->makeSound() . "\n";      // Meow!（重写了父类方法）
echo $cat->climb() . "\n";          // Climbing the tree（子类新增方法）
```

**输出**：

```
Buddy is 3 years old
Woof!
Moving
Fetching the ball
Meow!
Climbing the tree
```

**说明**：
- `Dog` 和 `Cat` 都继承自 `Animal` 类
- 子类可以使用父类的属性和方法（如 `getInfo()`、`move()`）
- 子类可以重写父类方法（如 `makeSound()`）
- 子类可以添加新方法（如 `fetch()`、`climb()`）

### 示例 2：使用 parent 关键字调用父类方法

```php
<?php
declare(strict_types=1);

class Vehicle
{
    public function start(): string
    {
        return "Engine started";
    }
    
    public function stop(): string
    {
        return "Engine stopped";
    }
}

class Car extends Vehicle
{
    public function start(): string
    {
        $parentResult = parent::start();
        return $parentResult . " - Car is ready";
    }
    
    public function honk(): string
    {
        return "Honk! Honk!";
    }
}

class Motorcycle extends Vehicle
{
    public function start(): string
    {
        $parentResult = parent::start();
        return $parentResult . " - Motorcycle is ready";
    }
}

$car = new Car();
echo $car->start() . "\n";  // Engine started - Car is ready
echo $car->stop() . "\n";   // Engine stopped（继承自父类）

$motorcycle = new Motorcycle();
echo $motorcycle->start() . "\n";  // Engine started - Motorcycle is ready
```

**输出**：

```
Engine started - Car is ready
Engine stopped
Engine started - Motorcycle is ready
```

**说明**：
- 使用 `parent::start()` 调用父类的 `start()` 方法
- 可以在调用父类方法的基础上添加额外的逻辑
- 这种方式实现了方法的扩展而不是完全替换

### 示例 3：构造函数继承

```php
<?php
declare(strict_types=1);

class Person
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
    
    public function introduce(): string
    {
        return "I am {$this->name}, {$this->age} years old";
    }
}

class Student extends Person
{
    public function __construct(
        string $name,
        int $age,
        public string $studentId
    ) {
        parent::__construct($name, $age);
    }
    
    public function introduce(): string
    {
        $base = parent::introduce();
        return $base . ", student ID: {$this->studentId}";
    }
}

class Teacher extends Person
{
    public function __construct(
        string $name,
        int $age,
        public string $subject
    ) {
        parent::__construct($name, $age);
    }
    
    public function introduce(): string
    {
        $base = parent::introduce();
        return $base . ", teaching {$this->subject}";
    }
}

$student = new Student("Alice", 20, "S12345");
echo $student->introduce() . "\n";  // I am Alice, 20 years old, student ID: S12345

$teacher = new Teacher("Bob", 35, "Mathematics");
echo $teacher->introduce() . "\n";  // I am Bob, 35 years old, teaching Mathematics
```

**输出**：

```
I am Alice, 20 years old, student ID: S12345
I am Bob, 35 years old, teaching Mathematics
```

**说明**：
- 子类构造函数必须调用父类构造函数（如果需要初始化父类属性）
- 使用 `parent::__construct()` 调用父类构造函数
- 可以在调用父类构造函数后初始化子类特有的属性

### 示例 4：可见性规则在继承中的表现

```php
<?php
declare(strict_types=1);

class BaseClass
{
    public string $publicProperty = "Public";
    protected string $protectedProperty = "Protected";
    private string $privateProperty = "Private";
    
    public function publicMethod(): string
    {
        return "Public method";
    }
    
    protected function protectedMethod(): string
    {
        return "Protected method";
    }
    
    private function privateMethod(): string
    {
        return "Private method";
    }
    
    public function testAccess(): void
    {
        echo "Base - Public: {$this->publicProperty}\n";
        echo "Base - Protected: {$this->protectedProperty}\n";
        echo "Base - Private: {$this->privateProperty}\n";
    }
}

class ChildClass extends BaseClass
{
    public function testChildAccess(): void
    {
        // 可以访问 public
        echo "Child - Public: {$this->publicProperty}\n";
        
        // 可以访问 protected
        echo "Child - Protected: {$this->protectedProperty}\n";
        
        // 不能访问 private
        // echo "Child - Private: {$this->privateProperty}\n";  // 错误：不能访问 private
        
        // 可以调用 public 方法
        $this->publicMethod();
        
        // 可以调用 protected 方法
        $this->protectedMethod();
        
        // 不能调用 private 方法
        // $this->privateMethod();  // 错误：不能调用 private 方法
    }
}

$child = new ChildClass();
$child->testAccess();        // 可以调用继承的 public 方法
$child->testChildAccess();   // 可以调用子类的方法

// 外部访问
echo "Public property: {$child->publicProperty}\n";  // 可以访问
// echo $child->protectedProperty;  // 错误：不能访问 protected
// echo $child->privateProperty;    // 错误：不能访问 private
```

**输出**：

```
Base - Public: Public
Base - Protected: Protected
Base - Private: Private
Child - Public: Public
Child - Protected: Protected
Public property: Public
```

**说明**：
- 子类可以访问父类的 `public` 和 `protected` 成员
- 子类不能访问父类的 `private` 成员
- 外部代码只能访问 `public` 成员

### 示例 5：final 关键字的使用

```php
<?php
declare(strict_types=1);

// final 类：不能被继承
final class MathConstants
{
    public const PI = 3.14159;
    public const E = 2.71828;
}

// 错误：不能继承 final 类
// class ExtendedConstants extends MathConstants {}

// final 方法：不能被子类重写
class BaseClass
{
    final public function criticalMethod(): string
    {
        return "This method cannot be overridden";
    }
    
    public function canBeOverridden(): string
    {
        return "This can be overridden";
    }
}

class ChildClass extends BaseClass
{
    // 错误：不能重写 final 方法
    // public function criticalMethod(): string
    // {
    //     return "Override";  // 错误：Cannot override final method
    // }
    
    // 可以重写非 final 方法
    public function canBeOverridden(): string
    {
        return "Overridden in child";
    }
}

$child = new ChildClass();
echo $child->criticalMethod() . "\n";  // This method cannot be overridden
echo $child->canBeOverridden() . "\n";  // Overridden in child
```

**输出**：

```
This method cannot be overridden
Overridden in child
```

**说明**：
- `final class` 防止类被继承
- `final method` 防止方法被重写
- 用于保护关键实现，防止意外的修改

### 示例 6：多级继承

```php
<?php
declare(strict_types=1);

class Animal
{
    public function __construct(
        public string $name
    ) {}
    
    public function eat(): string
    {
        return "{$this->name} is eating";
    }
    
    public function sleep(): string
    {
        return "{$this->name} is sleeping";
    }
}

class Mammal extends Animal
{
    public function giveBirth(): string
    {
        return "{$this->name} gives birth to live young";
    }
    
    public function eat(): string
    {
        return parent::eat() . " (mammal style)";
    }
}

class Dog extends Mammal
{
    public function bark(): string
    {
        return "{$this->name} is barking: Woof! Woof!";
    }
    
    public function eat(): string
    {
        return parent::eat() . " - dog food";
    }
}

$dog = new Dog("Buddy");
echo $dog->eat() . "\n";           // Buddy is eating (mammal style) - dog food
echo $dog->sleep() . "\n";         // Buddy is sleeping（继承自 Animal）
echo $dog->giveBirth() . "\n";     // Buddy gives birth to live young（继承自 Mammal）
echo $dog->bark() . "\n";          // Buddy is barking: Woof! Woof!（Dog 类的方法）
```

**输出**：

```
Buddy is eating (mammal style) - dog food
Buddy is sleeping
Buddy gives birth to live young
Buddy is barking: Woof! Woof!
```

**说明**：
- PHP 支持多级继承（Animal -> Mammal -> Dog）
- 子类可以访问所有祖先类的 `public` 和 `protected` 成员
- 方法可以沿着继承链被重写，使用 `parent::` 调用直接父类的方法

### 示例 7：继承层次设计

```php
<?php
declare(strict_types=1);

// 基类：员工
class Employee
{
    public function __construct(
        protected int $id,
        protected string $name,
        protected float $salary
    ) {}
    
    public function getId(): int
    {
        return $this->id;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getSalary(): float
    {
        return $this->salary;
    }
    
    public function calculateBonus(): float
    {
        return $this->salary * 0.1;  // 默认 10% 奖金
    }
    
    public function getInfo(): string
    {
        return "Employee {$this->id}: {$this->name}, Salary: {$this->salary}";
    }
}

// 子类：经理
class Manager extends Employee
{
    public function __construct(
        int $id,
        string $name,
        float $salary,
        protected int $teamSize
    ) {
        parent::__construct($id, $name, $salary);
    }
    
    public function calculateBonus(): float
    {
        $baseBonus = parent::calculateBonus();
        $teamBonus = $this->teamSize * 100;
        return $baseBonus + $teamBonus;
    }
    
    public function getInfo(): string
    {
        $base = parent::getInfo();
        return $base . ", Team Size: {$this->teamSize}";
    }
}

// 子类：开发者
class Developer extends Employee
{
    public function __construct(
        int $id,
        string $name,
        float $salary,
        protected string $programmingLanguage
    ) {
        parent::__construct($id, $name, $salary);
    }
    
    public function calculateBonus(): float
    {
        // 开发者有固定的奖金
        return $this->salary * 0.15;  // 15% 奖金
    }
    
    public function getInfo(): string
    {
        $base = parent::getInfo();
        return $base . ", Language: {$this->programmingLanguage}";
    }
}

$manager = new Manager(1, "Alice", 100000, 5);
echo $manager->getInfo() . "\n";
echo "Bonus: " . $manager->calculateBonus() . "\n";  // 15000 (10000 + 500)

$developer = new Developer(2, "Bob", 80000, "PHP");
echo $developer->getInfo() . "\n";
echo "Bonus: " . $developer->calculateBonus() . "\n";  // 12000
```

**输出**：

```
Employee 1: Alice, Salary: 100000, Team Size: 5
Bonus: 15000
Employee 2: Bob, Salary: 80000, Language: PHP
Bonus: 12000
```

**说明**：
- 建立了清晰的继承层次（Employee -> Manager/Developer）
- 子类重写了 `calculateBonus()` 方法，实现不同的奖金计算逻辑
- 使用 `parent::` 调用父类方法，实现方法的扩展

## 使用场景

### 场景 1：代码复用

继承用于复用父类的代码，避免重复。

**示例**：图形类层次

```php
<?php
declare(strict_types=1);

class Shape
{
    public function __construct(
        protected string $color
    ) {}
    
    public function getColor(): string
    {
        return $this->color;
    }
    
    public function setColor(string $color): void
    {
        $this->color = $color;
    }
}

class Circle extends Shape
{
    public function __construct(
        string $color,
        private float $radius
    ) {
        parent::__construct($color);
    }
    
    public function getArea(): float
    {
        return pi() * $this->radius * $this->radius;
    }
}

class Rectangle extends Shape
{
    public function __construct(
        string $color,
        private float $width,
        private float $height
    ) {
        parent::__construct($color);
    }
    
    public function getArea(): float
    {
        return $this->width * $this->height;
    }
}
```

### 场景 2：层次化设计

继承用于建立类的层次结构，体现"是一个"关系。

**示例**：交通工具层次

```php
<?php
declare(strict_types=1);

class Vehicle
{
    public function __construct(
        protected string $brand,
        protected string $model
    ) {}
    
    public function start(): string
    {
        return "Starting {$this->brand} {$this->model}";
    }
}

class Car extends Vehicle
{
    public function __construct(
        string $brand,
        string $model,
        private int $doors
    ) {
        parent::__construct($brand, $model);
    }
    
    public function honk(): string
    {
        return "Honk! Honk!";
    }
}

class Motorcycle extends Vehicle
{
    public function rev(): string
    {
        return "Vroom! Vroom!";
    }
}
```

## 注意事项

### 单继承限制

- **PHP 只支持单继承**：一个类只能继承一个父类
- **替代方案**：使用接口（Interface）实现多重继承的效果
- **Traits**：使用 Traits 实现代码复用（详见 [3.4.1 Traits](../chapter-04-metaprogramming/section-01-traits.md)）

**示例**：

```php
<?php
declare(strict_types=1);

class Parent1 {}
class Parent2 {}

// 错误：PHP 不支持多重继承
// class Child extends Parent1, Parent2 {}

// 解决方案：使用接口
interface Interface1 {}
interface Interface2 {}

class Child implements Interface1, Interface2 {}  // 可以实现多个接口
```

### 可见性规则

- **public**：子类和外部都可以访问
- **protected**：子类可以访问，外部不能访问
- **private**：只有定义它的类可以访问，子类不能访问

**示例**：

```php
<?php
declare(strict_types=1);

class Base
{
    public string $public = "Public";
    protected string $protected = "Protected";
    private string $private = "Private";
}

class Child extends Base
{
    public function test(): void
    {
        echo $this->public . "\n";     // 可以访问
        echo $this->protected . "\n";  // 可以访问
        // echo $this->private . "\n";  // 错误：不能访问 private
    }
}
```

### 构造函数继承

- **规则**：如果父类有构造函数，子类构造函数应该调用父类构造函数
- **方式**：使用 `parent::__construct()` 调用
- **时机**：在子类构造函数体的开始调用

**示例**：

```php
<?php
declare(strict_types=1);

class Base
{
    public function __construct(protected string $name) {}
}

class Child extends Base
{
    public function __construct(
        string $name,
        private int $age
    ) {
        parent::__construct($name);  // 必须调用父类构造函数
    }
}
```

### 方法重写规则

- **可见性**：子类重写方法的可见性不能比父类更严格
- **参数**：方法签名必须兼容（PHP 8.0+ 支持协变返回类型）
- **final**：不能重写 `final` 方法

**示例**：

```php
<?php
declare(strict_types=1);

class Base
{
    protected function method(): void {}  // protected
}

class Child extends Base
{
    // 正确：可以改为 public（更宽松）
    public function method(): void {}
    
    // 错误：不能改为 private（更严格）
    // private function method(): void {}
}
```

## 常见问题

### 问题 1：多重继承错误

**错误信息**：`Parse error: syntax error`

**原因**：试图继承多个父类

**解决方案**：
- 使用接口实现多重继承的效果
- 使用 Traits 实现代码复用
- 重新设计类层次结构

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：PHP 不支持多重继承
// class Child extends Parent1, Parent2 {}

// 解决方案 1：使用接口
interface Interface1 {}
interface Interface2 {}
class Child implements Interface1, Interface2 {}

// 解决方案 2：使用 Traits
trait Trait1 {}
trait Trait2 {}
class Child extends Parent1
{
    use Trait1, Trait2;
}
```

### 问题 2：未调用父类构造函数

**症状**：父类属性未初始化或出现错误

**原因**：子类构造函数未调用父类构造函数

**解决方案**：在子类构造函数中使用 `parent::__construct()` 调用父类构造函数

**示例**：

```php
<?php
declare(strict_types=1);

class Base
{
    public function __construct(protected string $name) {}
}

class Child extends Base
{
    public function __construct(
        string $name,
        private int $age
    ) {
        parent::__construct($name);  // 必须调用
    }
}
```

### 问题 3：访问 private 成员错误

**错误信息**：`Fatal error: Cannot access private property`

**原因**：子类试图访问父类的 `private` 成员

**解决方案**：
- 将 `private` 改为 `protected`（如果需要子类访问）
- 提供 `public` 或 `protected` 的访问方法

**示例**：

```php
<?php
declare(strict_types=1);

class Base
{
    private string $private = "Private";
    protected string $protected = "Protected";
    
    // 提供 protected 方法访问 private 属性
    protected function getPrivate(): string
    {
        return $this->private;
    }
}

class Child extends Base
{
    public function test(): void
    {
        // echo $this->private;  // 错误：不能访问 private
        echo $this->protected . "\n";  // 可以访问 protected
        echo $this->getPrivate() . "\n";  // 可以通过方法访问
    }
}
```

### 问题 4：final 类/方法被继承/重写

**错误信息**：`Fatal error: Class X may not inherit from final class Y` 或 `Fatal error: Cannot override final method`

**原因**：试图继承 `final` 类或重写 `final` 方法

**解决方案**：
- 不要继承 `final` 类
- 不要重写 `final` 方法
- 如果需要扩展功能，使用组合而不是继承

## 最佳实践

### 1. 合理使用继承

- **适用场景**：存在"是一个"（is-a）关系
- **避免过度继承**：继承层次不宜过深（建议不超过 3-4 层）
- **组合优于继承**：优先考虑组合（has-a）关系

**示例**：

```php
<?php
declare(strict_types=1);

// 好的设计：清晰的 is-a 关系
class Animal {}
class Dog extends Animal {}  // Dog is an Animal

// 不好的设计：过度继承
// class A {}
// class B extends A {}
// class C extends B {}
// class D extends C {}
// class E extends D {}  // 继承层次太深
```

### 2. 使用 final 保护关键实现

- 使用 `final class` 防止关键类被继承
- 使用 `final method` 防止关键方法被重写
- 明确表达设计意图

**示例**：

```php
<?php
declare(strict_types=1);

// 工具类使用 final，防止继承
final class MathUtils
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
}

// 关键方法使用 final，防止重写
class Base
{
    final public function criticalMethod(): void
    {
        // 关键实现，不能被重写
    }
}
```

### 3. 设计清晰的类层次结构

- 基类应该包含通用的属性和方法
- 子类应该添加特定于子类的功能
- 避免在基类中包含子类特定的逻辑

**示例**：

```php
<?php
declare(strict_types=1);

// 好的设计：基类包含通用功能
class Animal
{
    public function __construct(protected string $name) {}
    public function eat(): string { return "Eating"; }
    public function sleep(): string { return "Sleeping"; }
}

class Dog extends Animal
{
    public function bark(): string { return "Woof!"; }  // 特定于 Dog
}

// 不好的设计：基类包含特定逻辑
// class Animal
// {
//     public function makeSound(): string
//     {
//         return "Dog sound";  // 不好：包含特定逻辑
//     }
// }
```

### 4. 正确调用父类构造函数

- 子类构造函数必须调用父类构造函数（如果父类有构造函数）
- 在构造函数体开始处调用
- 传递正确的参数

**示例**：

```php
<?php
declare(strict_types=1);

class Base
{
    public function __construct(protected string $name) {}
}

class Child extends Base
{
    public function __construct(
        string $name,
        private int $age
    ) {
        parent::__construct($name);  // 正确：在开始处调用
        // 然后初始化子类属性
    }
}
```

### 5. 理解继承 vs 组合

- **继承（is-a）**：表示"是一个"关系
- **组合（has-a）**：表示"有一个"关系
- 优先考虑组合，更灵活

**示例**：

```php
<?php
declare(strict_types=1);

// 继承：Dog is an Animal
class Animal {}
class Dog extends Animal {}

// 组合：Car has an Engine
class Engine {}
class Car
{
    public function __construct(private Engine $engine) {}  // 组合
}

// 不要用继承实现组合关系
// class Car extends Engine {}  // 不好：Car is not an Engine
```

## 对比分析

### 继承 vs 组合

| 特性         | 继承（Inheritance）              | 组合（Composition）              |
|:-------------|:--------------------------------|:--------------------------------|
| **关系**      | is-a（是一个）                   | has-a（有一个）                  |
| **灵活性**    | 编译时确定，较不灵活              | 运行时确定，更灵活                |
| **耦合度**    | 高耦合                           | 低耦合                           |
| **代码复用**  | 通过继承复用                     | 通过组合复用                     |
| **适用场景**  | 真正的 is-a 关系                 | has-a 关系，需要灵活性           |

### public vs protected vs private（继承中的可见性）

| 可见性       | 类内部 | 子类 | 外部 | 继承中的表现                     |
|:-------------|:-------|:-----|:-----|:--------------------------------|
| **public**   | ✅     | ✅   | ✅   | 子类可以访问和重写               |
| **protected**| ✅     | ✅   | ❌   | 子类可以访问和重写，外部不能访问  |
| **private**  | ✅     | ❌   | ❌   | 子类不能访问，只有定义类可以访问  |

### 继承 vs 接口

| 特性         | 继承                               | 接口                               |
|:-------------|:-----------------------------------|:-----------------------------------|
| **实现**      | 提供实现（代码）                    | 只定义契约（方法签名）               |
| **数量**      | 单继承                             | 可以实现多个接口                    |
| **方法**      | 可以有具体方法和抽象方法            | 只有方法签名（PHP 8.0+ 可以有默认实现）|
| **属性**      | 可以有属性                         | 不能有属性（可以有常量）            |
| **适用场景**  | 代码复用，is-a 关系                 | 定义契约，实现多态                  |

## 练习任务

1. **创建继承层次**：创建一个 `Vehicle` 基类，包含 `start()` 和 `stop()` 方法，然后创建 `Car` 和 `Motorcycle` 子类，每个子类添加特定的方法。

2. **方法重写练习**：创建一个 `Animal` 基类，包含 `makeSound()` 方法，创建 `Dog`、`Cat`、`Bird` 子类，每个子类重写 `makeSound()` 方法。

3. **使用 parent 关键字**：创建一个 `Employee` 基类，包含 `getInfo()` 方法，创建 `Manager` 和 `Developer` 子类，使用 `parent::getInfo()` 扩展信息。

4. **final 关键字练习**：创建一个包含 `final` 方法的基类，尝试在子类中重写该方法，观察错误信息。

5. **多级继承练习**：创建一个三级继承层次（如：Animal -> Mammal -> Dog），理解继承链中的方法调用和属性访问。

## 相关章节

- **[3.1.2 可见性修饰符](../chapter-01-classes/section-02-visibility.md)**：了解可见性修饰符在继承中的作用
- **[3.3.2 接口（Interface）](section-02-interfaces.md)**：了解如何使用接口补充单继承的限制
- **[3.4.1 Traits](../chapter-04-metaprogramming/section-01-traits.md)**：了解如何使用 Traits 实现代码复用
- **[3.3.4 多态（Polymorphism）](section-04-polymorphism.md)**：了解继承如何为多态提供基础

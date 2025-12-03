# 3.3.1 继承（Inheritance）

## 概述

继承是面向对象编程的核心特性之一，允许子类继承父类的属性和方法，实现代码复用和层次化设计。

## 基础语法

### extends 关键字

- **语法**：`class ChildClass extends ParentClass { ... }`
- 子类继承父类的所有 `public` 和 `protected` 属性和方法。
- 子类可以重写（override）父类方法，也可以添加新方法。

```php
class Animal
{
    public function __construct(
        public string $name
    ) {
    }

    public function makeSound(): string
    {
        return 'Some sound';
    }

    public function move(): string
    {
        return 'Moving';
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return 'Woof!';
    }

    public function fetch(): string
    {
        return 'Fetching the ball';
    }
}

$dog = new Dog('Buddy');
echo $dog->makeSound(); // Woof!
echo $dog->move();      // Moving（继承自父类）
echo $dog->fetch();     // Fetching the ball
```

## 访问父类方法

### parent 关键字

- 使用 `parent::methodName()` 调用父类方法。

```php
class Vehicle
{
    public function start(): string
    {
        return 'Engine started';
    }
}

class Car extends Vehicle
{
    public function start(): string
    {
        $parentResult = parent::start();
        return $parentResult . ' - Car is ready';
    }
}

$car = new Car();
echo $car->start(); // Engine started - Car is ready
```

### 调用父类构造函数

```php
class Person
{
    public function __construct(
        public string $name,
        public int $age
    ) {
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
}

$student = new Student('Alice', 20, 'S12345');
echo $student->name;       // Alice
echo $student->studentId; // S12345
```

## final 关键字

### final class

- `final class`：禁止被继承。

```php
final class Math
{
    public static function add(float $a, float $b): float
    {
        return $a + $b;
    }
}

// class AdvancedMath extends Math {} // 错误：Cannot extend final class
```

### final method

- `final function`：禁止被子类重写。

```php
class Base
{
    public final function criticalMethod(): void
    {
        // 关键逻辑，不允许重写
    }
}

class Derived extends Base
{
    // public function criticalMethod(): void {} // 错误：Cannot override final method
}
```

## 继承层次示例

```php
class Shape
{
    public function __construct(
        public string $color
    ) {
    }

    public function getColor(): string
    {
        return $this->color;
    }

    public function getArea(): float
    {
        return 0.0; // 基类默认实现
    }
}

class Rectangle extends Shape
{
    public function __construct(
        string $color,
        public float $width,
        public float $height
    ) {
        parent::__construct($color);
    }

    public function getArea(): float
    {
        return $this->width * $this->height;
    }
}

class Circle extends Shape
{
    public function __construct(
        string $color,
        public float $radius
    ) {
        parent::__construct($color);
    }

    public function getArea(): float
    {
        return pi() * $this->radius ** 2;
    }
}

$rect = new Rectangle('red', 10, 5);
$circle = new Circle('blue', 3);

echo $rect->getArea();  // 50
echo $circle->getArea(); // 约 28.27
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Employee
{
    public function __construct(
        public int $id,
        public string $name,
        protected float $salary
    ) {
    }

    public function getSalary(): float
    {
        return $this->salary;
    }

    public function calculateBonus(): float
    {
        return $this->salary * 0.1; // 默认 10% 奖金
    }
}

class Manager extends Employee
{
    public function __construct(
        int $id,
        string $name,
        float $salary,
        public int $teamSize
    ) {
        parent::__construct($id, $name, $salary);
    }

    public function calculateBonus(): float
    {
        $baseBonus = parent::calculateBonus();
        return $baseBonus + ($this->teamSize * 100); // 额外团队奖金
    }
}

class Developer extends Employee
{
    public function __construct(
        int $id,
        string $name,
        float $salary,
        public string $programmingLanguage
    ) {
        parent::__construct($id, $name, $salary);
    }
}

$manager = new Manager(1, 'Alice', 100000, 5);
echo $manager->calculateBonus(); // 15000 (10000 + 500)

$developer = new Developer(2, 'Bob', 80000, 'PHP');
echo $developer->calculateBonus(); // 8000
```

## 注意事项

1. **单继承**：PHP 只支持单继承，一个类只能继承一个父类。

2. **可见性**：子类可以访问父类的 `public` 和 `protected` 成员，不能访问 `private` 成员。

3. **方法重写**：子类可以重写父类方法，使用 `parent::` 调用父类实现。

4. **final 关键字**：使用 `final` 防止类被继承或方法被重写。

5. **构造函数**：子类构造函数应该调用父类构造函数。

## 练习

1. 创建一个 `Vehicle` 基类，包含 `start()` 和 `stop()` 方法，然后创建 `Car` 和 `Motorcycle` 子类。

2. 实现一个 `Animal` 基类，包含 `name` 属性和 `makeSound()` 方法，创建 `Dog`、`Cat`、`Bird` 子类。

3. 创建一个 `Shape` 基类，包含 `getArea()` 和 `getPerimeter()` 抽象方法，创建 `Rectangle` 和 `Circle` 子类实现。

4. 实现一个 `User` 基类，创建 `Admin` 和 `RegularUser` 子类，子类有不同的权限方法。

5. 创建一个 `PaymentProcessor` 基类，包含通用的支付逻辑，创建 `CreditCardProcessor` 和 `PayPalProcessor` 子类。

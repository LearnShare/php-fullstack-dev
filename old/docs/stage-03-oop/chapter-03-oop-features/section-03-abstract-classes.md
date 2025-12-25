# 3.3.3 抽象类（Abstract Class）

## 概述

抽象类（Abstract Class）是不能被实例化的类，可以包含抽象方法（无实现）和普通方法（有实现）。抽象类介于普通类和接口之间。

## 基础语法

### abstract class

- **语法**：`abstract class AbstractClassName { ... }`
- 抽象类不能被实例化，只能被继承。
- 可以包含抽象方法（无实现）和普通方法（有实现）。

```php
abstract class Animal
{
    public function __construct(
        public string $name
    ) {
    }

    // 抽象方法：子类必须实现
    abstract public function makeSound(): string;

    // 普通方法：子类可以继承或重写
    public function sleep(): string
    {
        return "{$this->name} is sleeping";
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return 'Woof!';
    }
}

class Cat extends Animal
{
    public function makeSound(): string
    {
        return 'Meow!';
    }
}

$dog = new Dog('Buddy');
echo $dog->makeSound(); // Woof!
echo $dog->sleep();     // Buddy is sleeping
```

## 抽象方法

### abstract method

- 抽象方法只有签名，没有实现。
- 子类必须实现所有抽象方法。

```php
abstract class DatabaseConnection
{
    abstract public function connect(): void;
    abstract public function query(string $sql): array;
    abstract public function disconnect(): void;

    public function execute(string $sql): array
    {
        $this->connect();
        $result = $this->query($sql);
        $this->disconnect();
        return $result;
    }
}

class MySQLConnection extends DatabaseConnection
{
    public function connect(): void
    {
        echo "Connecting to MySQL...\n";
    }

    public function query(string $sql): array
    {
        echo "Executing: {$sql}\n";
        return [];
    }

    public function disconnect(): void
    {
        echo "Disconnecting from MySQL...\n";
    }
}
```

## 抽象类 vs 接口

### 对比

| 特性           | 抽象类                     | 接口                       |
| :------------- | :------------------------- | :------------------------- |
| 实例化         | 不能实例化                 | 不能实例化                 |
| 方法实现       | 可以包含实现               | 不能包含实现（PHP 8.0+ 除外） |
| 属性           | 可以包含属性               | 只能包含常量               |
| 继承           | 单继承                     | 多实现                     |
| 使用场景       | 提供部分实现的模板         | 定义契约                   |
| 构造函数       | 可以有构造函数             | 不能有构造函数             |

### 选择建议

- **使用抽象类**：当需要提供部分实现，子类共享通用逻辑时。
- **使用接口**：当只需要定义契约，不需要实现时。

## 模板方法模式

### 实现示例

```php
abstract class PaymentProcessor
{
    public function __construct(
        protected float $amount
    ) {
    }

    // 模板方法模式
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
        echo "Payment {$status}: {$this->amount}\n";
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
        echo "Authorizing payment...\n";
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
        echo "Charging PayPal...\n";
        return true;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

abstract class DataExporter
{
    public function __construct(
        protected array $data
    ) {
    }

    // 模板方法
    final public function export(): string
    {
        $this->validate();
        $formatted = $this->format();
        $this->save($formatted);
        return $formatted;
    }

    abstract protected function validate(): void;
    abstract protected function format(): string;

    protected function save(string $content): void
    {
        echo "Saving exported data...\n";
    }
}

class JsonExporter extends DataExporter
{
    protected function validate(): void
    {
        if (empty($this->data)) {
            throw new InvalidArgumentException('Data cannot be empty');
        }
    }

    protected function format(): string
    {
        return json_encode($this->data, JSON_PRETTY_PRINT);
    }
}

class XmlExporter extends DataExporter
{
    protected function validate(): void
    {
        if (empty($this->data)) {
            throw new InvalidArgumentException('Data cannot be empty');
        }
    }

    protected function format(): string
    {
        $xml = new SimpleXMLElement('<root/>');
        $this->arrayToXml($this->data, $xml);
        return $xml->asXML();
    }

    private function arrayToXml(array $data, SimpleXMLElement $xml): void
    {
        foreach ($data as $key => $value) {
            if (is_array($value)) {
                $subnode = $xml->addChild($key);
                $this->arrayToXml($value, $subnode);
            } else {
                $xml->addChild($key, $value);
            }
        }
    }
}
```

## 注意事项

1. **不能实例化**：抽象类不能被直接实例化，只能被继承。

2. **抽象方法**：包含抽象方法的类必须是抽象类。

3. **实现要求**：子类必须实现所有抽象方法。

4. **模板方法**：使用 `final` 关键字保护模板方法不被重写。

5. **选择原则**：需要部分实现时使用抽象类，只需要契约时使用接口。

## 练习

1. 创建一个 `Vehicle` 抽象类，包含抽象方法 `start()` 和 `stop()`，然后创建 `Car` 和 `Motorcycle` 子类。

2. 实现一个 `Shape` 抽象类，包含 `color` 属性和抽象方法 `getArea()`，创建 `Rectangle` 和 `Circle` 子类。

3. 创建一个 `Notification` 抽象类，使用模板方法模式定义通知流程，创建 `EmailNotification` 和 `SmsNotification` 子类。

4. 实现一个 `DataProcessor` 抽象类，包含通用的数据处理逻辑，子类实现特定的处理方式。

5. 创建一个 `ReportGenerator` 抽象类，使用模板方法模式，子类实现不同的报告格式。

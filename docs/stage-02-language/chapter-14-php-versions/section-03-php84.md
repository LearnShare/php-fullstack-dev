# 2.14.3 PHP 8.4 新特性与示例

## 概述

PHP 8.4 引入了属性钩子、不对称可见性、`array_first/array_last`、`#[\NoDiscard]`、DOM API 改进等新特性。

理解 PHP 8.4 的新特性对于编写更现代、更灵活的 PHP 代码非常重要。掌握这些新特性可以帮助提高代码质量、开发效率和代码可维护性。

## 特性

- **属性钩子**：属性的 getter 和 setter，类似其他语言的属性访问器
- **不对称可见性**：属性的读写可见性可以不同
- **array_first/array_last**：获取数组首尾元素的便捷函数
- **#[\NoDiscard]**：标记不应丢弃的返回值，提醒检查返回值
- **DOM API 改进**：改进的 DOM API，提供更好的功能

## 语法/定义

### 属性钩子（Property Hooks）

**语法**：
```php
private string $name {
    get {
        return $this->name;
    }
    set(string $value) {
        $this->name = trim($value);
    }
}
```

**要求**：PHP 8.4+

**特点**：
- 属性的 getter 和 setter
- 可以控制属性访问
- 类似其他语言的属性访问器

### 不对称可见性

**语法**：`private(set) public(get) string $name;`

**要求**：PHP 8.4+

**特点**：
- 属性的读写可见性可以不同
- 可以只读或只写
- 提供更灵活的访问控制

### array_first() - 获取第一个元素

**语法**：`array_first(array $array): mixed`

**参数**：
- `$array`：要处理的数组

**返回值**：数组的第一个元素，如果数组为空返回 `null`

**要求**：PHP 8.4+

### array_last() - 获取最后一个元素

**语法**：`array_last(array $array): mixed`

**参数**：
- `$array`：要处理的数组

**返回值**：数组的最后一个元素，如果数组为空返回 `null`

**要求**：PHP 8.4+

### #[\NoDiscard] 属性

**语法**：`#[\NoDiscard]`

**要求**：PHP 8.4+

**功能**：标记不应丢弃的返回值

**特点**：
- 如果返回值被丢弃，静态分析工具会警告
- 提醒检查返回值
- 提高代码质量

## 基本用法

### 示例 1：属性钩子

```php
<?php
declare(strict_types=1);

class User
{
    private string $name {
        get {
            return $this->name ?? 'Unknown';
        }
        set(string $value) {
            if (empty(trim($value))) {
                throw new InvalidArgumentException("Name cannot be empty");
            }
            $this->name = trim($value);
        }
    }
    
    private int $age {
        get {
            return $this->age;
        }
        set(int $value) {
            if ($value < 0) {
                throw new InvalidArgumentException("Age must be non-negative");
            }
            $this->age = $value;
        }
    }
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;  // 调用 setter
        $this->age = $age;    // 调用 setter
    }
    
    public function getName(): string
    {
        return $this->name;  // 调用 getter
    }
}

// 使用
$user = new User('John', 25);
echo $user->getName() . "\n";  // John
```

### 示例 2：不对称可见性

```php
<?php
declare(strict_types=1);

class User
{
    // 只读属性：外部可以读取，但不能写入
    public(readonly) private(set) string $id;
    
    // 只写属性：外部可以写入，但不能读取
    private(get) public(set) string $password;
    
    public function __construct(string $id)
    {
        $this->id = $id;
    }
    
    public function getId(): string
    {
        return $this->id;  // 可以读取
    }
    
    public function setPassword(string $password): void
    {
        $this->password = $password;  // 可以写入
    }
}

// 使用
$user = new User('user123');
echo $user->getId() . "\n";  // user123
$user->setPassword('secret');
// echo $user->password;  // 错误：不能读取
```

### 示例 3：array_first/array_last

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 获取第一个元素
$first = array_first($numbers);
echo $first . "\n";  // 1

// 获取最后一个元素
$last = array_last($numbers);
echo $last . "\n";  // 5

// 空数组
$empty = [];
$first = array_first($empty);  // null
$last = array_last($empty);     // null
```

### 示例 4：#[\NoDiscard]

```php
<?php
declare(strict_types=1);

class Result
{
    public function __construct(
        public bool $success,
        public string $message
    ) {}
}

#[\NoDiscard]
function processData(array $data): Result
{
    // 处理数据
    return new Result(true, "Processed successfully");
}

// 使用
$result = processData([]);  // 正确：检查返回值
if ($result->success) {
    echo $result->message . "\n";
}

// 错误：丢弃返回值（静态分析工具会警告）
// processData([]);  // 警告：返回值不应被丢弃
```

## 完整代码示例

### 示例 1：属性钩子实现

```php
<?php
declare(strict_types=1);

class Email
{
    private string $address {
        get {
            return $this->address;
        }
        set(string $value) {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException("Invalid email address: {$value}");
            }
            $this->address = strtolower(trim($value));
        }
    }
    
    public function __construct(string $address)
    {
        $this->address = $address;  // 调用 setter，自动验证
    }
    
    public function getAddress(): string
    {
        return $this->address;  // 调用 getter
    }
}

// 使用
$email = new Email('USER@EXAMPLE.COM');
echo $email->getAddress() . "\n";  // user@example.com（自动转换为小写）

// 错误：无效邮箱
// $email = new Email('invalid');  // InvalidArgumentException
```

### 示例 2：不对称可见性使用

```php
<?php
declare(strict_types=1);

class BankAccount
{
    // 余额：外部可以读取，但不能直接写入
    public(readonly) private(set) float $balance = 0.0;
    
    // 账户号：只读
    public(readonly) private(set) string $accountNumber;
    
    public function __construct(string $accountNumber)
    {
        $this->accountNumber = $accountNumber;
    }
    
    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Amount must be positive");
        }
        $this->balance += $amount;  // 内部可以写入
    }
    
    public function withdraw(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Amount must be positive");
        }
        if ($this->balance < $amount) {
            throw new RuntimeException("Insufficient balance");
        }
        $this->balance -= $amount;  // 内部可以写入
    }
    
    public function getBalance(): float
    {
        return $this->balance;  // 外部可以读取
    }
}

// 使用
$account = new BankAccount('123456');
$account->deposit(100.0);
echo $account->getBalance() . "\n";  // 100.0
// $account->balance = 1000;  // 错误：不能直接写入
```

### 示例 3：数组工具函数

```php
<?php
declare(strict_types=1);

class ArrayHelper
{
    public static function getFirst(array $array): mixed
    {
        return array_first($array);
    }
    
    public static function getLast(array $array): mixed
    {
        return array_last($array);
    }
    
    public static function getFirstAndLast(array $array): array
    {
        return [
            'first' => array_first($array),
            'last' => array_last($array)
        ];
    }
}

// 使用
$numbers = [1, 2, 3, 4, 5];
$result = ArrayHelper::getFirstAndLast($numbers);
print_r($result);
// Array
// (
//     [first] => 1
//     [last] => 5
// )
```

### 示例 4：#[\NoDiscard] 使用

```php
<?php
declare(strict_types=1);

class Database
{
    #[\NoDiscard]
    public function execute(string $sql): Result
    {
        // 执行 SQL
        return new Result(true, "Executed successfully");
    }
    
    #[\NoDiscard]
    public function transaction(callable $callback): Result
    {
        // 执行事务
        return new Result(true, "Transaction completed");
    }
}

// 使用
$db = new Database();

// 正确：检查返回值
$result = $db->execute("SELECT * FROM users");
if ($result->success) {
    echo "Query executed\n";
}

// 错误：丢弃返回值（静态分析工具会警告）
// $db->execute("SELECT * FROM users");  // 警告
```

## 使用场景

### 属性钩子

- **属性验证**：在 setter 中验证属性值
- **属性转换**：在 setter 中转换属性值
- **属性计算**：在 getter 中计算属性值

### 不对称可见性

- **只读属性**：创建只读属性
- **只写属性**：创建只写属性（如密码）
- **访问控制**：更灵活的访问控制

### array_first/array_last

- **数组操作**：简化数组首尾元素获取
- **数据处理**：处理数据流
- **队列操作**：队列的首尾操作

### #[\NoDiscard]

- **返回值检查**：提醒检查返回值
- **错误处理**：确保错误被处理
- **代码质量**：提高代码质量

## 注意事项

### 版本要求

- **PHP 8.4+**：所有新特性都需要 PHP 8.4 或更高版本
- **兼容性**：注意向后兼容性
- **迁移**：评估迁移成本

### 属性钩子限制

- **语法复杂**：语法相对复杂
- **性能考虑**：每次访问都会调用 getter/setter
- **工具支持**：确保工具支持

### 不对称可见性

- **可读性**：可能影响代码可读性
- **使用场景**：合理使用，不要过度使用
- **文档说明**：为复杂可见性添加文档

## 常见问题

### 问题 1：属性钩子语法

**症状**：属性钩子语法错误

**原因**：不理解属性钩子语法

**解决方法**：

```php
<?php
declare(strict_types=1);

class User
{
    // 正确：属性钩子语法
    private string $name {
        get {
            return $this->name;
        }
        set(string $value) {
            $this->name = $value;
        }
    }
}
```

### 问题 2：array_first/array_last 空数组

**症状**：空数组返回 `null`

**原因**：`array_first/array_last` 对空数组返回 `null`

**解决方法**：

```php
<?php
declare(strict_types=1);

$array = [];

// 检查返回值
$first = array_first($array);
if ($first !== null) {
    echo $first . "\n";
} else {
    echo "Array is empty\n";
}

// 或使用默认值
$first = array_first($array) ?? 'default';
echo $first . "\n";  // default
```

## 最佳实践

### 新特性使用

- **了解场景**：了解新特性的适用场景
- **合理使用**：合理使用，不要过度使用
- **逐步采用**：逐步采用新特性

### 代码质量

- **属性控制**：使用属性钩子控制属性访问
- **访问控制**：使用不对称可见性实现灵活的访问控制
- **返回值检查**：使用 `#[\NoDiscard]` 提醒检查返回值

## 相关章节

- **2.8.3 数组操作函数**：了解数组函数
- **阶段三：面向对象编程基础**：了解类和属性
- **2.13.3 静态代码分析**：了解静态分析工具

## 练习任务

1. **属性钩子练习**：
   - 创建带属性钩子的类
   - 实现属性验证
   - 测试属性访问
   - 理解钩子机制

2. **不对称可见性练习**：
   - 创建只读属性
   - 创建只写属性
   - 测试访问控制
   - 理解可见性规则

3. **数组函数练习**：
   - 使用 `array_first/array_last`
   - 实现数组工具函数
   - 测试各种数组场景
   - 对比传统方法

4. **实际应用练习**：
   - 在项目中使用新特性
   - 重构现有代码
   - 测试兼容性
   - 评估效果

5. **综合练习**：
   - 创建一个使用 PHP 8.4 新特性的项目
   - 实现各种新特性
   - 进行代码审查
   - 测试功能改进

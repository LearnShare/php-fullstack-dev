# 2.6.4 三元运算符与空合并运算符

## 概述

三元运算符和空合并运算符提供了简洁的条件赋值方式，可以替代简单的 if-else 语句，使代码更加简洁易读。

## 三元运算符（?:）

### 基本语法

```php
条件 ? 真值 : 假值
```

### 基本用法

```php
<?php
declare(strict_types=1);

$age = 20;
$status = $age >= 18 ? 'Adult' : 'Minor';
echo $status . "\n";  // Adult

// 等价于
if ($age >= 18) {
    $status = 'Adult';
} else {
    $status = 'Minor';
}
```

### 嵌套三元运算符

```php
<?php
declare(strict_types=1);

$score = 85;
$grade = $score >= 90 ? 'A' : ($score >= 80 ? 'B' : ($score >= 70 ? 'C' : 'F'));
echo $grade . "\n";  // B

// 注意：嵌套三元运算符可读性较差，建议使用 if-else 或 match
```

### 省略中间值（Elvis 运算符）

PHP 7.0+ 支持省略中间值，如果条件为真则返回条件本身的值：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$displayName = $name ?: 'Guest';  // 如果 $name 为真，返回 $name，否则返回 'Guest'
echo $displayName . "\n";  // Alice

$emptyName = '';
$displayName2 = $emptyName ?: 'Guest';
echo $displayName2 . "\n";  // Guest
```

### 在表达式中使用

```php
<?php
declare(strict_types=1);

function getPrice(bool $isMember): float
{
    return $isMember ? 9.99 : 19.99;
}

echo getPrice(true) . "\n";   // 9.99
echo getPrice(false) . "\n";  // 19.99
```

## 空合并运算符（??）

### 基本语法

```php
$value ?? $default
```

### 基本用法

空合并运算符（PHP 7.0+）用于提供默认值，如果左侧操作数为 `null` 或未定义，则返回右侧操作数。

```php
<?php
declare(strict_types=1);

$name = $input['name'] ?? 'Guest';
echo $name . "\n";

// 等价于
$name = isset($input['name']) ? $input['name'] : 'Guest';
```

### 链式空合并

```php
<?php
declare(strict_types=1);

// 链式使用，返回第一个非 null 的值
$value = $a ?? $b ?? $c ?? 'default';
echo $value . "\n";

// 实际应用
$config = [
    'database' => [
        'host' => 'localhost'
    ]
];

$host = $config['database']['host'] ?? $config['db_host'] ?? 'localhost';
echo $host . "\n";  // localhost
```

### 与 isset() 的区别

```php
<?php
declare(strict_types=1);

// isset() 检查变量是否存在且不为 null
$var = null;
var_dump(isset($var));  // bool(false)

// ?? 只检查是否为 null
$result = $var ?? 'default';
echo $result . "\n";  // default

// 未定义的变量
$result2 = $undefined ?? 'default';
echo $result2 . "\n";  // default（不会产生警告）
```

## 空合并赋值运算符（??=）

### 基本语法

```php
$variable ??= $default;
```

### 基本用法

空合并赋值运算符（PHP 7.4+）只在变量为 `null` 或未定义时赋值。

```php
<?php
declare(strict_types=1);

$name ??= 'Guest';  // 如果 $name 为 null 或未定义，赋值为 'Guest'
echo $name . "\n";  // Guest

$name = 'Alice';
$name ??= 'Guest';  // $name 已有值，不会重新赋值
echo $name . "\n";  // Alice
```

### 实际应用

```php
<?php
declare(strict_types=1);

class Config
{
    private array $settings = [];
    
    public function get(string $key, mixed $default = null): mixed
    {
        return $this->settings[$key] ?? $default;
    }
    
    public function set(string $key, mixed $value): void
    {
        $this->settings[$key] ??= $value;  // 只在未设置时赋值
    }
}

$config = new Config();
$config->set('host', 'localhost');
$config->set('host', 'example.com');  // 不会覆盖，因为已设置
echo $config->get('host') . "\n";  // localhost
```

## 优先级

### 三元运算符优先级

三元运算符的优先级较低，使用时建议加括号：

```php
<?php
declare(strict_types=1);

// 需要加括号
$result = ($condition ? $a : $b) + $c;

// 否则可能不是预期结果
$result2 = $condition ? $a : $b + $c;  // 可能不是预期
```

### 空合并运算符优先级

空合并运算符的优先级高于三元运算符，但低于大多数其他运算符：

```php
<?php
declare(strict_types=1);

// ?? 优先级高于 ?:
$result = $a ?? $b ? $c : $d;  // 等价于 ($a ?? $b) ? $c : $d

// 建议加括号明确意图
$result2 = ($a ?? $b) ? $c : $d;
```

## 完整示例

```php
<?php
declare(strict_types=1);

class UserService
{
    public function getUserDisplayName(?array $user): string
    {
        // 使用空合并运算符提供默认值
        $firstName = $user['first_name'] ?? 'Guest';
        $lastName = $user['last_name'] ?? '';
        
        // 使用三元运算符格式化
        $fullName = $lastName 
            ? "{$firstName} {$lastName}" 
            : $firstName;
        
        return $fullName;
    }
    
    public function getUserStatus(?array $user): string
    {
        // 链式空合并
        $status = $user['status'] 
            ?? $user['active'] 
            ?? 'unknown';
        
        // 三元运算符
        return $status === 'active' ? 'Active' : 'Inactive';
    }
    
    public function getPrice(bool $isMember, ?float $discount = null): float
    {
        $basePrice = 99.99;
        
        // 使用空合并和三元运算符
        $finalPrice = $isMember 
            ? $basePrice * (1 - ($discount ?? 0.1))  // 会员默认 10% 折扣
            : $basePrice;
        
        return round($finalPrice, 2);
    }
}

$service = new UserService();

$user1 = ['first_name' => 'Alice', 'last_name' => 'Smith', 'status' => 'active'];
echo $service->getUserDisplayName($user1) . "\n";  // Alice Smith
echo $service->getUserStatus($user1) . "\n";      // Active

$user2 = ['first_name' => 'Bob'];
echo $service->getUserDisplayName($user2) . "\n";  // Bob
echo $service->getUserStatus($user2) . "\n";      // Inactive

echo $service->getPrice(true, 0.2) . "\n";   // 79.99（会员，20% 折扣）
echo $service->getPrice(true) . "\n";        // 89.99（会员，默认 10% 折扣）
echo $service->getPrice(false) . "\n";       // 99.99（非会员）
```

## 注意事项

1. **可读性**：嵌套三元运算符可读性差，复杂逻辑应使用 if-else 或 match。

2. **优先级**：注意运算符优先级，必要时使用括号。

3. **空合并 vs isset**：`??` 只检查 `null`，`isset()` 还检查变量是否存在。

4. **性能**：空合并运算符比 `isset()` + 三元运算符更简洁，性能相同。

5. **链式使用**：空合并运算符可以链式使用，返回第一个非 null 的值。

## 练习

1. 创建一个函数，使用三元运算符根据分数返回等级（A、B、C、D、F）。

2. 编写一个配置读取函数，使用空合并运算符提供默认值。

3. 实现一个用户信息格式化函数，使用空合并和三元运算符处理可选字段。

4. 创建一个函数，演示空合并赋值运算符的使用场景。

5. 编写一个函数，比较 `isset()` + 三元运算符与空合并运算符的差异。


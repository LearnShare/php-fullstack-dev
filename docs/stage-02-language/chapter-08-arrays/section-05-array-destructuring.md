# 2.8.5 数组解构

## 概述

数组解构（Array Destructuring）是 PHP 7.1+ 引入的特性，允许将数组的元素直接赋值给多个变量。这大大简化了代码，提高了可读性。

## 索引数组解构

### 基本语法

```php
[$var1, $var2, $var3] = $array;
```

### 基本示例

```php
<?php
declare(strict_types=1);

$data = ['Alice', 25, 'alice@example.com'];
[$name, $age, $email] = $data;

echo "Name: {$name}, Age: {$age}, Email: {$email}\n";
// 输出：Name: Alice, Age: 25, Email: alice@example.com
```

### 跳过元素

使用空位置跳过不需要的元素：

```php
<?php
declare(strict_types=1);

$data = ['Alice', 25, 'alice@example.com', 'Beijing'];
[$name, , $email] = $data;  // 跳过第二个元素（年龄）

echo "Name: {$name}, Email: {$email}\n";
// 输出：Name: Alice, Email: alice@example.com
```

## 关联数组解构

### 基本语法

```php
['key1' => $var1, 'key2' => $var2] = $array;
```

### 基本示例

```php
<?php
declare(strict_types=1);

$user = [
    'name' => 'Alice',
    'age' => 25,
    'email' => 'alice@example.com'
];

['name' => $name, 'age' => $age, 'email' => $email] = $user;

echo "Name: {$name}, Age: {$age}, Email: {$email}\n";
```

### 部分解构

只解构需要的键：

```php
<?php
declare(strict_types=1);

$user = [
    'name' => 'Alice',
    'age' => 25,
    'email' => 'alice@example.com',
    'city' => 'Beijing'
];

['name' => $name, 'email' => $email] = $user;
echo "Name: {$name}, Email: {$email}\n";
```

## 使用默认值

### 索引数组默认值

```php
<?php
declare(strict_types=1);

$data = ['Alice', 25];
[$name, $age, $email = 'N/A'] = $data;  // $email 使用默认值

echo "Name: {$name}, Age: {$age}, Email: {$email}\n";
// 输出：Name: Alice, Age: 25, Email: N/A
```

### 关联数组默认值

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice', 'age' => 25];
['name' => $name, 'age' => $age, 'email' => $email = 'N/A'] = $user;

echo "Name: {$name}, Age: {$age}, Email: {$email}\n";
// 输出：Name: Alice, Age: 25, Email: N/A
```

## 交换变量

数组解构提供了一种简洁的变量交换方式：

```php
<?php
declare(strict_types=1);

$a = 10;
$b = 20;

[$a, $b] = [$b, $a];  // 交换变量

echo "a = {$a}, b = {$b}\n";  // a = 20, b = 10
```

## 函数返回值解构

```php
<?php
declare(strict_types=1);

function getUserInfo(): array
{
    return [
        'name' => 'Alice',
        'age' => 25,
        'email' => 'alice@example.com'
    ];
}

['name' => $name, 'age' => $age] = getUserInfo();
echo "Name: {$name}, Age: {$age}\n";
```

## 嵌套解构

### 嵌套数组解构

```php
<?php
declare(strict_types=1);

$data = [
    'user' => ['name' => 'Alice', 'age' => 25],
    'address' => ['city' => 'Beijing', 'country' => 'China']
];

[
    'user' => ['name' => $name, 'age' => $age],
    'address' => ['city' => $city]
] = $data;

echo "Name: {$name}, Age: {$age}, City: {$city}\n";
```

## 在循环中使用

### foreach 中的解构

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 30],
    ['name' => 'Charlie', 'age' => 20]
];

foreach ($users as ['name' => $name, 'age' => $age]) {
    echo "{$name} is {$age} years old\n";
}
```

### 关联数组循环解构

```php
<?php
declare(strict_types=1);

$users = [
    'user1' => ['name' => 'Alice', 'age' => 25],
    'user2' => ['name' => 'Bob', 'age' => 30]
];

foreach ($users as $id => ['name' => $name, 'age' => $age]) {
    echo "{$id}: {$name} ({$age})\n";
}
```

## 命名参数解构（PHP 8.1+）

PHP 8.1+ 支持使用数组解构作为命名参数：

```php
<?php
declare(strict_types=1);

function createUser(string $name, int $age, string $email = ''): array
{
    return [
        'name' => $name,
        'age' => $age,
        'email' => $email
    ];
}

// 使用数组解构传递参数
$params = ['name' => 'Alice', 'age' => 25, 'email' => 'alice@example.com'];
$user = createUser(...$params);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ArrayDestructuring
{
    public static function demonstrate(): void
    {
        echo "=== 索引数组解构 ===\n";
        $data = ['Alice', 25, 'alice@example.com'];
        [$name, $age, $email] = $data;
        echo "Name: {$name}, Age: {$age}, Email: {$email}\n";
        
        echo "\n=== 跳过元素 ===\n";
        [$name, , $email] = $data;
        echo "Name: {$name}, Email: {$email}\n";
        
        echo "\n=== 关联数组解构 ===\n";
        $user = ['name' => 'Alice', 'age' => 25, 'email' => 'alice@example.com'];
        ['name' => $name, 'age' => $age] = $user;
        echo "Name: {$name}, Age: {$age}\n";
        
        echo "\n=== 默认值 ===\n";
        $data = ['Alice', 25];
        [$name, $age, $email = 'N/A'] = $data;
        echo "Name: {$name}, Age: {$age}, Email: {$email}\n";
        
        echo "\n=== 交换变量 ===\n";
        $a = 10;
        $b = 20;
        [$a, $b] = [$b, $a];
        echo "a = {$a}, b = {$b}\n";
        
        echo "\n=== 嵌套解构 ===\n";
        $data = [
            'user' => ['name' => 'Alice', 'age' => 25],
            'address' => ['city' => 'Beijing']
        ];
        ['user' => ['name' => $name, 'age' => $age], 'address' => ['city' => $city]] = $data;
        echo "Name: {$name}, Age: {$age}, City: {$city}\n";
        
        echo "\n=== 循环中解构 ===\n";
        $users = [
            ['name' => 'Alice', 'age' => 25],
            ['name' => 'Bob', 'age' => 30]
        ];
        foreach ($users as ['name' => $name, 'age' => $age]) {
            echo "{$name} is {$age} years old\n";
        }
    }
}

ArrayDestructuring::demonstrate();
```

## 注意事项

1. **元素数量**：解构的元素数量可以少于数组元素数量，但不能访问不存在的元素（除非使用默认值）。

2. **键的存在性**：关联数组解构时，如果键不存在且没有默认值，会产生警告。

3. **性能**：数组解构的性能与手动赋值相近，但代码更简洁。

4. **可读性**：对于复杂结构，解构可以提高代码可读性。

5. **兼容性**：数组解构需要 PHP 7.1+，旧版本可以使用 `list()` 函数。

## 与 list() 的对比

### list() 语法（旧语法）

```php
<?php
declare(strict_types=1);

$data = ['Alice', 25, 'alice@example.com'];
list($name, $age, $email) = $data;  // 旧语法
```

### 数组解构语法（推荐，PHP 7.1+）

```php
<?php
declare(strict_types=1);

$data = ['Alice', 25, 'alice@example.com'];
[$name, $age, $email] = $data;  // 新语法，推荐使用
```

## 练习

1. 创建一个函数，返回多个值，使用数组解构接收返回值。

2. 编写一个函数，使用数组解构交换两个变量的值。

3. 实现一个函数，从嵌套数组中解构出需要的值。

4. 创建一个函数，在 `foreach` 循环中使用数组解构处理数据。

5. 编写一个函数，使用数组解构和默认值处理可能缺失的数据。

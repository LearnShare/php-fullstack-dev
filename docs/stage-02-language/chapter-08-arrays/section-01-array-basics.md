# 2.8.1 数组基础

## 概述

数组是 PHP 中最重要的数据结构之一。PHP 数组是有序映射（ordered map），可以同时包含数字键和字符串键，这使得它既能作为列表使用，也能作为字典使用。

## 数组特性

### 有序映射

PHP 数组是有序映射，这意味着：
- 键值对按照插入顺序保存
- 可以同时包含数字键和字符串键
- 键可以是整数或字符串
- 值可以是任意类型

### 与 JavaScript 对比

| 特性 | JavaScript | PHP |
| :--- | :--------- | :-- |
| 数组 | `[1, 2, 3]` | `[1, 2, 3]` |
| 对象 | `{id: 1, name: 'Alice'}` | `['id' => 1, 'name' => 'Alice']` |
| 统一结构 | 数组和对象是不同类型 | 数组统一处理 |

## 创建数组

### 使用方括号语法（推荐，PHP 5.4+）

```php
<?php
declare(strict_types=1);

// 索引数组
$numbers = [1, 2, 3, 4, 5];

// 关联数组
$user = [
    'id' => 1,
    'name' => 'Alice',
    'email' => 'alice@example.com'
];

// 混合数组
$mixed = [
    0 => 'first',
    'key' => 'value',
    1 => 'second'
];
```

### 使用 array() 语法

```php
<?php
declare(strict_types=1);

// 索引数组
$numbers = array(1, 2, 3, 4, 5);

// 关联数组
$user = array(
    'id' => 1,
    'name' => 'Alice',
    'email' => 'alice@example.com'
);
```

### 动态创建

```php
<?php
declare(strict_types=1);

// 空数组
$arr = [];

// 逐个添加元素
$arr[] = 'first';        // 自动索引：$arr[0] = 'first'
$arr[] = 'second';       // 自动索引：$arr[1] = 'second'
$arr['key'] = 'value';   // 指定键：$arr['key'] = 'value'

print_r($arr);
```

## 访问数组元素

### 基本访问

```php
<?php
declare(strict_types=1);

$user = [
    'name' => 'Alice',
    'age' => 25,
    'email' => 'alice@example.com'
];

// 访问元素
echo $user['name'] . "\n";   // Alice
echo $user['age'] . "\n";    // 25

// 检查键是否存在
if (isset($user['email'])) {
    echo $user['email'] . "\n";
}

// 使用 null 合并运算符
$phone = $user['phone'] ?? 'N/A';
echo $phone . "\n";
```

### 嵌套数组访问

```php
<?php
declare(strict_types=1);

$data = [
    'user' => [
        'name' => 'Alice',
        'address' => [
            'city' => 'Beijing',
            'country' => 'China'
        ]
    ]
];

echo $data['user']['name'] . "\n";                    // Alice
echo $data['user']['address']['city'] . "\n";        // Beijing

// 安全访问（PHP 8.0+）
$country = $data['user']['address']['country'] ?? 'Unknown';
echo $country . "\n";
```

## 修改数组元素

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice', 'age' => 25];

// 修改现有元素
$user['age'] = 26;

// 添加新元素
$user['email'] = 'alice@example.com';

// 修改嵌套数组
$user['address'] = ['city' => 'Beijing'];
$user['address']['country'] = 'China';

print_r($user);
```

## 删除数组元素

### 使用 unset()

```php
<?php
declare(strict_types=1);

$arr = ['a' => 1, 'b' => 2, 'c' => 3];

unset($arr['b']);  // 删除键 'b'

print_r($arr);
// 输出：
// Array
// (
//     [a] => 1
//     [c] => 3
// )
```

### 注意：unset() 不会重新索引

```php
<?php
declare(strict_types=1);

$arr = [0 => 'a', 1 => 'b', 2 => 'c'];
unset($arr[1]);

print_r($arr);
// 输出：
// Array
// (
//     [0] => a
//     [2] => c
// )

// 如果需要重新索引，使用 array_values()
$arr = array_values($arr);
print_r($arr);
// 输出：
// Array
// (
//     [0] => a
//     [1] => c
// )
```

## 数组键的类型

### 整数键

```php
<?php
declare(strict_types=1);

$arr = [];
$arr[0] = 'zero';
$arr[1] = 'one';
$arr[2] = 'two';

// 字符串形式的数字会被转换为整数
$arr['3'] = 'three';  // 等同于 $arr[3] = 'three'
```

### 字符串键

```php
<?php
declare(strict_types=1);

$arr = [];
$arr['name'] = 'Alice';
$arr['age'] = 25;
```

### 浮点数键

浮点数键会被截断为整数：

```php
<?php
declare(strict_types=1);

$arr = [];
$arr[3.14] = 'pi';    // 等同于 $arr[3] = 'pi'
$arr[5.99] = 'almost'; // 等同于 $arr[5] = 'almost'
```

### 布尔值和 null 键

```php
<?php
declare(strict_types=1);

$arr = [];
$arr[true] = 'true';   // 等同于 $arr[1] = 'true'
$arr[false] = 'false'; // 等同于 $arr[0] = 'false'
$arr[null] = 'null';   // 等同于 $arr[''] = 'null'
```

## 类型检测

### is_array()

**语法**：`is_array(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是数组类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_array([1, 2, 3]));      // bool(true)
var_dump(is_array(['key' => 'value'])); // bool(true)
var_dump(is_array("string"));      // bool(false)
```

### 检查数组是否为空

```php
<?php
declare(strict_types=1);

$arr1 = [];
$arr2 = [1, 2, 3];

var_dump(empty($arr1));  // bool(true)
var_dump(empty($arr2));  // bool(false)

// 检查数组是否有元素
if (count($arr1) > 0) {
    echo "Array has elements\n";
}
```

## 嵌套数组

### 多维数组

```php
<?php
declare(strict_types=1);

$matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

echo $matrix[0][1] . "\n";  // 2
echo $matrix[1][2] . "\n";  // 6
```

### 复杂嵌套结构

```php
<?php
declare(strict_types=1);

$users = [
    [
        'id' => 1,
        'name' => 'Alice',
        'roles' => ['admin', 'user'],
        'metadata' => [
            'created_at' => '2024-01-01',
            'last_login' => '2024-01-15'
        ]
    ],
    [
        'id' => 2,
        'name' => 'Bob',
        'roles' => ['user'],
        'metadata' => [
            'created_at' => '2024-01-02',
            'last_login' => null
        ]
    ]
];

echo $users[0]['name'] . "\n";                    // Alice
echo $users[0]['roles'][0] . "\n";                // admin
echo $users[0]['metadata']['created_at'] . "\n";  // 2024-01-01
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ArrayBasics
{
    public static function demonstrate(): void
    {
        echo "=== 创建数组 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        $user = [
            'id' => 1,
            'name' => 'Alice',
            'email' => 'alice@example.com'
        ];
        
        echo "Numbers: " . implode(', ', $numbers) . "\n";
        echo "User: {$user['name']} ({$user['email']})\n";
        
        echo "\n=== 修改数组 ===\n";
        $user['age'] = 25;
        $user['email'] = 'newemail@example.com';
        print_r($user);
        
        echo "\n=== 嵌套数组 ===\n";
        $data = [
            'users' => [
                ['id' => 1, 'name' => 'Alice'],
                ['id' => 2, 'name' => 'Bob']
            ],
            'total' => 2
        ];
        echo "First user: {$data['users'][0]['name']}\n";
        echo "Total: {$data['total']}\n";
        
        echo "\n=== 键的类型 ===\n";
        $mixed = [
            0 => 'zero',
            '1' => 'one',      // 字符串 '1' 转为整数 1
            2.5 => 'two',      // 浮点数 2.5 截断为 2
            'key' => 'value'
        ];
        print_r($mixed);
    }
}

ArrayBasics::demonstrate();
```

## 注意事项

1. **键的唯一性**：数组键必须唯一，重复的键会覆盖之前的值。

2. **自动索引**：使用 `$arr[] = value` 时，PHP 会自动分配下一个可用的整数键。

3. **类型转换**：字符串形式的数字键会被转换为整数，浮点数键会被截断。

4. **性能考虑**：关联数组的性能略低于索引数组，但在大多数应用中差异可以忽略。

5. **内存使用**：PHP 数组使用哈希表实现，内存使用相对较大。

## 练习

1. 创建一个函数，将两个数组合并为一个新数组，处理键冲突的情况。

2. 编写一个函数，安全地访问嵌套数组的值，如果路径不存在返回默认值。

3. 实现一个函数，将扁平数组转换为嵌套数组结构。

4. 创建一个函数，检查数组是否为关联数组（包含字符串键）。

5. 编写一个函数，将嵌套数组扁平化为一维数组。

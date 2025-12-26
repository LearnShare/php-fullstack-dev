# 2.8.1 数组基础

## 概述

数组是 PHP 中最重要的数据结构之一，是有序映射。本节详细介绍数组的特性、创建方式、访问元素、修改删除、嵌套数组、类型检测、键的类型等基础知识。

理解数组的本质（有序映射）对于掌握 PHP 数组操作非常重要。PHP 数组同时扮演了其他语言中数组和字典/对象的角色，这种设计提供了极大的灵活性。

## 特性

- **有序映射**：数组是有序的键值对集合，保持插入顺序
- **混合键**：可以同时包含数字键和字符串键
- **动态大小**：数组大小可以动态变化，无需预先声明
- **类型灵活**：数组元素可以是任意类型（标量、数组、对象等）
- **值可重复**：数组值可以重复，但键必须唯一

## 语法/定义

### 数组创建

**短数组语法**：`[]`（PHP 5.4+，推荐）
**长数组语法**：`array()`（兼容旧版本）

**索引数组**：使用数字键（从 0 开始）
**关联数组**：使用字符串键
**混合数组**：同时包含数字键和字符串键

### 访问元素

**语法**：`$array[key]`

**键类型**：
- 整数：`$array[0]`、`$array[1]`
- 字符串：`$array['key']`、`$array["key"]`
- 变量：`$array[$key]`
- 表达式：`$array[$key + 1]`

### 键的类型规则

- **整数键**：自动从 0 开始编号（如果未指定）
- **字符串数字键**：会被转换为整数（如 `"1"` → `1`）
- **浮点数键**：会被转换为整数（如 `1.5` → `1`）
- **布尔键**：会被转换为整数（`true` → `1`，`false` → `0`）
- **null 键**：会被转换为空字符串 `""`

## 基本用法

### 示例 1：索引数组

```php
<?php
declare(strict_types=1);

// 创建索引数组
$fruits = ['apple', 'banana', 'orange'];
echo $fruits[0] . "\n";  // apple
echo $fruits[1] . "\n";  // banana

// 使用 array() 语法
$numbers = array(1, 2, 3, 4, 5);
echo $numbers[0] . "\n";  // 1

// 指定索引
$arr = [0 => 'first', 1 => 'second', 2 => 'third'];
echo $arr[1] . "\n";  // second

// 不连续索引
$arr = [0 => 'first', 5 => 'second', 10 => 'third'];
print_r($arr);
// Array
// (
//     [0] => first
//     [5] => second
//     [10] => third
// )
```

### 示例 2：关联数组

```php
<?php
declare(strict_types=1);

// 创建关联数组
$user = [
    'name' => 'John',
    'age' => 25,
    'email' => 'john@example.com'
];

echo $user['name'] . "\n";   // John
echo $user['age'] . "\n";    // 25
echo $user['email'] . "\n";  // john@example.com

// 使用 array() 语法
$config = array(
    'host' => 'localhost',
    'port' => 3306,
    'database' => 'mydb'
);
```

### 示例 3：混合数组

```php
<?php
declare(strict_types=1);

// 混合数组：同时包含数字键和字符串键
$mixed = [
    0 => 'first',
    'key' => 'value',
    1 => 'second',
    'name' => 'John'
];

echo $mixed[0] . "\n";        // first
echo $mixed['key'] . "\n";   // value
echo $mixed[1] . "\n";       // second
echo $mixed['name'] . "\n";  // John

print_r($mixed);
```

### 示例 4：修改和删除元素

```php
<?php
declare(strict_types=1);

$arr = ['a' => 1, 'b' => 2, 'c' => 3];

// 修改元素
$arr['a'] = 10;
echo $arr['a'] . "\n";  // 10

// 添加新元素
$arr['d'] = 4;
echo $arr['d'] . "\n";  // 4

// 删除元素
unset($arr['b']);
var_dump(isset($arr['b']));  // bool(false)

// 添加元素（索引数组）
$numbers = [1, 2, 3];
$numbers[] = 4;  // 自动使用下一个数字键
print_r($numbers);  // Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 4 )
```

### 示例 5：嵌套数组

```php
<?php
declare(strict_types=1);

// 二维数组
$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30],
    ['name' => 'Bob', 'age' => 35]
];

echo $users[0]['name'] . "\n";  // John
echo $users[1]['age'] . "\n";   // 30

// 多维数组
$data = [
    'users' => [
        'admin' => ['name' => 'Admin', 'role' => 'administrator'],
        'user1' => ['name' => 'User1', 'role' => 'user']
    ],
    'settings' => [
        'theme' => 'dark',
        'language' => 'en'
    ]
];

echo $data['users']['admin']['name'] . "\n";  // Admin
echo $data['settings']['theme'] . "\n";       // dark
```

## 完整代码示例

### 示例 1：数组工具类

```php
<?php
declare(strict_types=1);

class ArrayHelper
{
    public static function get(array $array, string|int $key, mixed $default = null): mixed
    {
        return $array[$key] ?? $default;
    }
    
    public static function has(array $array, string|int $key): bool
    {
        return array_key_exists($key, $array);
    }
    
    public static function set(array &$array, string|int $key, mixed $value): void
    {
        $array[$key] = $value;
    }
    
    public static function remove(array &$array, string|int $key): bool
    {
        if (array_key_exists($key, $array)) {
            unset($array[$key]);
            return true;
        }
        return false;
    }
    
    public static function isEmpty(array $array): bool
    {
        return empty($array);
    }
}

// 使用
$arr = ['name' => 'John', 'age' => 25];
echo ArrayHelper::get($arr, 'name', 'Unknown') . "\n";  // John
echo ArrayHelper::get($arr, 'email', 'N/A') . "\n";     // N/A
var_dump(ArrayHelper::has($arr, 'age'));  // bool(true)
```

### 示例 2：配置管理

```php
<?php
declare(strict_types=1);

class Config
{
    private array $config = [];
    
    public function __construct(array $config = [])
    {
        $this->config = $config;
    }
    
    public function get(string $key, mixed $default = null): mixed
    {
        $keys = explode('.', $key);
        $value = $this->config;
        
        foreach ($keys as $k) {
            if (!is_array($value) || !array_key_exists($k, $value)) {
                return $default;
            }
            $value = $value[$k];
        }
        
        return $value;
    }
    
    public function set(string $key, mixed $value): void
    {
        $keys = explode('.', $key);
        $config = &$this->config;
        
        foreach ($keys as $k) {
            if (!is_array($config)) {
                $config = [];
            }
            if (!array_key_exists($k, $config)) {
                $config[$k] = [];
            }
            $config = &$config[$k];
        }
        
        $config = $value;
    }
}

// 使用
$config = new Config([
    'database' => [
        'host' => 'localhost',
        'port' => 3306
    ]
]);

echo $config->get('database.host') . "\n";  // localhost
$config->set('database.name', 'mydb');
echo $config->get('database.name') . "\n";   // mydb
```

### 示例 3：键的类型转换

```php
<?php
declare(strict_types=1);

// 字符串数字键转换为整数
$arr = [
    "0" => "zero",
    "1" => "one",
    "2" => "two"
];
print_r($arr);
// Array
// (
//     [0] => zero
//     [1] => one
//     [2] => two
// )

// 浮点数键转换为整数
$arr = [
    1.5 => "one point five",
    2.7 => "two point seven"
];
print_r($arr);
// Array
// (
//     [1] => one point five
//     [2] => two point seven
// )

// 布尔键转换为整数
$arr = [
    true => "true",
    false => "false"
];
print_r($arr);
// Array
// (
//     [1] => true
//     [0] => false
// )

// null 键转换为空字符串
$arr = [
    null => "null value"
];
print_r($arr);
// Array
// (
//     [] => null value
// )
```

## 使用场景

### 数据集合

- **列表数据**：存储列表数据（如用户列表、商品列表）
- **配置信息**：存储配置项（如数据库配置、应用配置）
- **映射关系**：建立键值映射（如 ID 到名称的映射）

### 数据结构

- **树形结构**：使用嵌套数组表示树形结构
- **图结构**：使用数组表示图结构
- **表格数据**：使用二维数组表示表格数据

### 数据传递

- **函数参数**：传递多个相关参数
- **返回值**：返回多个值
- **API 数据**：处理 API 返回的数据

## 注意事项

### 键的类型

- **自动转换**：字符串数字、浮点数、布尔值、null 会被转换
- **唯一性**：键必须唯一，重复键会覆盖前面的值
- **类型检查**：使用 `array_key_exists()` 检查键是否存在

### 未定义键

- **访问未定义键**：会产生 Notice 错误
- **检查键存在**：使用 `isset()` 或 `array_key_exists()` 检查
- **使用空合并运算符**：使用 `??` 提供默认值

### 性能考虑

- **短数组语法**：优先使用 `[]`（性能更好）
- **预分配**：已知大小时可以预分配
- **避免频繁修改**：避免频繁修改数组结构

## 常见问题

### 问题 1：键类型混淆

**症状**：数组键的行为不符合预期

**原因**：不理解数组键的类型转换规则

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = [
    "1" => "one",
    "2" => "two"
];
echo $arr[1] . "\n";  // one（字符串 "1" 被转换为整数 1）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 理解键的类型转换
$arr = [
    "1" => "one",  // 会被转换为整数 1
    "2" => "two"   // 会被转换为整数 2
];

// 如果需要字符串键，使用非数字字符串
$arr = [
    "key1" => "one",
    "key2" => "two"
];
echo $arr["key1"] . "\n";  // one
```

### 问题 2：未定义键错误

**症状**：访问不存在的键产生 Notice 错误

**原因**：没有检查键是否存在

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = ['name' => 'John'];
echo $arr['age'] . "\n";  // Notice: Undefined index: age
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = ['name' => 'John'];

// 方法1：使用 isset()
if (isset($arr['age'])) {
    echo $arr['age'] . "\n";
} else {
    echo "Age not set\n";
}

// 方法2：使用 array_key_exists()
if (array_key_exists('age', $arr)) {
    echo $arr['age'] . "\n";
}

// 方法3：使用空合并运算符
echo $arr['age'] ?? 'N/A' . "\n";  // N/A
```

### 问题 3：数组赋值引用问题

**症状**：修改数组后影响原数组

**原因**：数组赋值是引用赋值（PHP 7.1+ 优化后是写时复制）

**说明**：

```php
<?php
declare(strict_types=1);

$arr1 = [1, 2, 3];
$arr2 = $arr1;  // 写时复制，不会立即复制

$arr2[0] = 10;  // 此时才会复制
echo $arr1[0] . "\n";  // 1（不受影响）
echo $arr2[0] . "\n";  // 10

// 如果需要引用，使用 &
$arr3 = &$arr1;
$arr3[0] = 20;
echo $arr1[0] . "\n";  // 20（受影响）
```

## 最佳实践

### 数组创建

- **使用短数组语法**：优先使用 `[]`（PHP 5.4+）
- **明确键名**：使用有意义的键名
- **保持一致性**：保持数组结构的一致性

### 键的使用

- **检查键存在**：访问前检查键是否存在
- **使用默认值**：使用空合并运算符提供默认值
- **理解类型转换**：理解键的类型转换规则

### 代码可读性

- **使用关联数组**：使用关联数组提高可读性
- **添加注释**：为复杂数组结构添加注释
- **提取常量**：将常用键名提取为常量

## 对比分析

### 索引数组 vs 关联数组

| 特性 | 索引数组 | 关联数组 |
|:-----|:---------|:---------|
| 键类型 | 整数 | 字符串 |
| 可读性 | 低 | 高 |
| 适用场景 | 有序列表 | 配置、映射 |
| 推荐度 | 按需使用 | 推荐（配置） |

**选择建议**：
- **有序列表**：使用索引数组
- **配置、映射**：使用关联数组

### 短数组语法 vs array() 语法

| 特性 | `[]` | `array()` |
|:-----|:-----|:----------|
| 语法简洁 | 是 | 否 |
| 性能 | 更好 | 稍差 |
| 版本要求 | PHP 5.4+ | 所有版本 |
| 推荐度 | 推荐 | 兼容旧版本 |

**选择建议**：
- **PHP 5.4+**：使用 `[]`
- **兼容旧版本**：使用 `array()`

## 相关章节

- **2.4.2 复合类型**：了解数组类型
- **2.8.2 数组遍历**：了解数组遍历
- **2.8.5 数组解构**：了解数组解构

## 练习任务

1. **数组创建练习**：
   - 练习创建索引数组、关联数组、混合数组
   - 理解键的类型转换规则
   - 测试各种键类型
   - 观察数组结构

2. **数组操作练习**：
   - 练习访问、修改、删除数组元素
   - 练习添加新元素
   - 理解数组的写时复制机制
   - 测试各种操作场景

3. **嵌套数组练习**：
   - 创建二维、多维数组
   - 访问嵌套数组元素
   - 修改嵌套数组元素
   - 测试各种嵌套结构

4. **实际应用练习**：
   - 实现配置管理类
   - 实现数据存储结构
   - 处理 API 返回数据
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个数组工具类
   - 实现各种数组操作方法
   - 处理各种数组结构
   - 进行代码审查，确保正确性

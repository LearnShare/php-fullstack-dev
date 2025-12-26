# 2.8.5 数组解构

## 概述

数组解构是 PHP 7.1+ 引入的特性，可以方便地从数组提取值。本节详细介绍索引数组解构、关联数组解构、跳过元素、使用默认值、交换变量、函数返回值解构、嵌套解构、在循环中使用、命名参数解构（PHP 8.1+）等。

数组解构可以大大简化代码，提高可读性。理解解构的语法、限制和最佳实践可以帮助编写更现代的 PHP 代码。

## 特性

- **简洁性**：简化数组值提取代码
- **灵活性**：支持跳过元素、默认值
- **可读性**：提高代码可读性
- **版本要求**：需要 PHP 7.1+（部分特性需要 PHP 8.1+）

## 语法/定义

### 索引数组解构

**基本语法**：`[$var1, $var2, $var3] = $array;`

**特点**：
- 按位置提取值
- 支持跳过元素
- 支持默认值

### 关联数组解构

**基本语法**：`['key1' => $var1, 'key2' => $var2] = $array;`

**特点**：
- 按键名提取值
- 支持默认值
- 键名必须匹配

### 跳过元素

**语法**：`[$var1, , $var3] = $array;`

**说明**：使用空位（逗号之间无变量）跳过元素

### 默认值

**语法**：`[$var1 = 'default', $var2] = $array;`

**说明**：如果数组元素不存在，使用默认值

### 命名参数解构（PHP 8.1+）

**语法**：`['key1' => $var1, 'key2' => $var2] = $array;`

**特点**：与关联数组解构相同，但在函数参数中使用

## 基本用法

### 示例 1：索引数组解构

```php
<?php
declare(strict_types=1);

// 基本解构
[$a, $b, $c] = [1, 2, 3];
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 1, b: 2, c: 3

// 跳过元素
[$first, , $third] = [1, 2, 3];
echo "first: {$first}, third: {$third}\n";  // first: 1, third: 3

// 使用默认值
[$a = 0, $b = 0, $c = 0] = [1, 2];
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 1, b: 2, c: 0
```

### 示例 2：关联数组解构

```php
<?php
declare(strict_types=1);

// 基本解构
['name' => $name, 'age' => $age] = ['name' => 'John', 'age' => 25];
echo "Name: {$name}, Age: {$age}\n";  // Name: John, Age: 25

// 使用默认值
['name' => $name, 'age' => $age, 'email' => $email = 'N/A'] = ['name' => 'John', 'age' => 25];
echo "Name: {$name}, Age: {$age}, Email: {$email}\n";  // Name: John, Age: 25, Email: N/A

// 部分解构
['name' => $name] = ['name' => 'John', 'age' => 25, 'email' => 'john@example.com'];
echo "Name: {$name}\n";  // Name: John
```

### 示例 3：交换变量

```php
<?php
declare(strict_types=1);

// 交换变量值
$a = 10;
$b = 20;
[$a, $b] = [$b, $a];
echo "a: {$a}, b: {$b}\n";  // a: 20, b: 10

// 交换多个变量
$x = 1;
$y = 2;
$z = 3;
[$x, $y, $z] = [$z, $x, $y];
echo "x: {$x}, y: {$y}, z: {$z}\n";  // x: 3, y: 1, z: 2
```

### 示例 4：函数返回值解构

```php
<?php
declare(strict_types=1);

function getUserInfo(): array
{
    return ['name' => 'John', 'age' => 25, 'email' => 'john@example.com'];
}

// 解构函数返回值
['name' => $name, 'age' => $age] = getUserInfo();
echo "Name: {$name}, Age: {$age}\n";  // Name: John, Age: 25

function divide(int $a, int $b): array
{
    return [
        'quotient' => intdiv($a, $b),
        'remainder' => $a % $b
    ];
}

['quotient' => $q, 'remainder' => $r] = divide(10, 3);
echo "Quotient: {$q}, Remainder: {$r}\n";  // Quotient: 3, Remainder: 1
```

### 示例 5：嵌套解构

```php
<?php
declare(strict_types=1);

// 嵌套数组解构
$data = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30]
];

[['name' => $name1, 'age' => $age1], ['name' => $name2, 'age' => $age2]] = $data;
echo "User 1: {$name1}, {$age1}\n";  // User 1: John, 25
echo "User 2: {$name2}, {$age2}\n";  // User 2: Jane, 30

// 复杂嵌套结构
$config = [
    'database' => ['host' => 'localhost', 'port' => 3306],
    'cache' => ['enabled' => true, 'ttl' => 3600]
];

[
    'database' => ['host' => $dbHost, 'port' => $dbPort],
    'cache' => ['enabled' => $cacheEnabled, 'ttl' => $cacheTtl]
] = $config;

echo "DB Host: {$dbHost}, Port: {$dbPort}\n";  // DB Host: localhost, Port: 3306
echo "Cache Enabled: " . ($cacheEnabled ? 'Yes' : 'No') . ", TTL: {$cacheTtl}\n";
```

### 示例 6：在循环中使用

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30],
    ['name' => 'Bob', 'age' => 35]
];

// 在 foreach 中使用解构
foreach ($users as ['name' => $name, 'age' => $age]) {
    echo "{$name} is {$age} years old\n";
}
// John is 25 years old
// Jane is 30 years old
// Bob is 35 years old

// 在 foreach 中使用索引数组解构
$points = [[1, 2], [3, 4], [5, 6]];
foreach ($points as [$x, $y]) {
    echo "Point: ({$x}, {$y})\n";
}
```

### 示例 7：命名参数解构（PHP 8.1+）

```php
<?php
declare(strict_types=1);

// PHP 8.1+ 支持在函数参数中使用数组解构
function processUser(array $user): void
{
    ['name' => $name, 'age' => $age, 'email' => $email = 'N/A'] = $user;
    echo "Processing: {$name}, {$age}, {$email}\n";
}

// 使用
processUser(['name' => 'John', 'age' => 25]);
// Processing: John, 25, N/A

processUser(['name' => 'Jane', 'age' => 30, 'email' => 'jane@example.com']);
// Processing: Jane, 30, jane@example.com
```

## 完整代码示例

### 示例 1：配置解析

```php
<?php
declare(strict_types=1);

class ConfigParser
{
    public static function parse(array $config): array
    {
        [
            'database' => [
                'host' => $dbHost = 'localhost',
                'port' => $dbPort = 3306,
                'name' => $dbName = 'mydb'
            ],
            'cache' => [
                'enabled' => $cacheEnabled = false,
                'ttl' => $cacheTtl = 3600
            ]
        ] = $config;
        
        return [
            'database' => [
                'host' => $dbHost,
                'port' => $dbPort,
                'name' => $dbName
            ],
            'cache' => [
                'enabled' => $cacheEnabled,
                'ttl' => $cacheTtl
            ]
        ];
    }
}

// 使用
$config = [
    'database' => ['host' => 'remote.host', 'port' => 5432],
    'cache' => ['enabled' => true]
];

$parsed = ConfigParser::parse($config);
print_r($parsed);
```

### 示例 2：数据提取工具

```php
<?php
declare(strict_types=1);

class DataExtractor
{
    public static function extractUser(array $data): array
    {
        [
            'id' => $id,
            'name' => $name,
            'email' => $email = null,
            'age' => $age = null
        ] = $data;
        
        return [
            'id' => $id,
            'name' => $name,
            'email' => $email,
            'age' => $age
        ];
    }
    
    public static function extractCoordinates(array $point): array
    {
        [$x, $y, $z = 0] = $point;
        return ['x' => $x, 'y' => $y, 'z' => $z];
    }
}

// 使用
$userData = ['id' => 1, 'name' => 'John', 'email' => 'john@example.com'];
$user = DataExtractor::extractUser($userData);
print_r($user);

$point = [10, 20];
$coords = DataExtractor::extractCoordinates($point);
print_r($coords);  // ['x' => 10, 'y' => 20, 'z' => 0]
```

### 示例 3：批量处理

```php
<?php
declare(strict_types=1);

function processUsers(array $users): array
{
    $result = [];
    foreach ($users as ['name' => $name, 'age' => $age, 'email' => $email = null]) {
        $result[] = [
            'name' => strtoupper($name),
            'age' => $age,
            'email' => $email ?? 'no-email@example.com'
        ];
    }
    return $result;
}

// 使用
$users = [
    ['name' => 'John', 'age' => 25, 'email' => 'john@example.com'],
    ['name' => 'Jane', 'age' => 30],
    ['name' => 'Bob', 'age' => 35, 'email' => 'bob@example.com']
];

$processed = processUsers($users);
print_r($processed);
```

## 使用场景

### 函数返回值

- **多值返回**：解构函数返回的多个值
- **配置读取**：从配置数组提取值
- **API 响应**：处理 API 返回的数据

### 变量操作

- **变量交换**：交换两个或多个变量的值
- **值提取**：从数组中提取需要的值
- **默认值处理**：使用默认值处理缺失值

### 循环处理

- **foreach 解构**：在循环中使用解构
- **批量处理**：批量处理数组数据
- **数据转换**：转换数据格式

## 注意事项

### 版本要求

- **PHP 7.1+**：基本数组解构需要 PHP 7.1+
- **PHP 8.1+**：命名参数解构需要 PHP 8.1+
- **兼容性**：注意代码兼容性

### 数组结构

- **结构匹配**：确保数组结构与解构模式匹配
- **默认值**：使用默认值处理缺失元素
- **错误处理**：处理结构不匹配的情况

### 性能考虑

- **解构开销**：解构有轻微的性能开销
- **可读性优先**：优先考虑可读性
- **避免过度**：避免过度使用嵌套解构

## 常见问题

### 问题 1：数组结构不匹配

**症状**：解构失败或结果不符合预期

**原因**：解构模式与数组结构不匹配

**错误示例**：

```php
<?php
declare(strict_types=1);

// 数组元素不足
[$a, $b, $c] = [1, 2];  // $c 是 null
echo "c: ";
var_dump($c);  // NULL
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：提供默认值
[$a, $b, $c = 0] = [1, 2];
echo "c: {$c}\n";  // c: 0

// 方法2：检查数组长度
$arr = [1, 2];
if (count($arr) >= 3) {
    [$a, $b, $c] = $arr;
} else {
    [$a, $b] = $arr;
    $c = 0;
}
```

### 问题 2：关联数组键不匹配

**症状**：关联数组解构失败

**原因**：解构模式中的键名与数组键名不匹配

**错误示例**：

```php
<?php
declare(strict_types=1);

// 键名不匹配
['name' => $name, 'age' => $age] = ['username' => 'John', 'age' => 25];
// $name 是 null，因为键名是 'username' 不是 'name'
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 使用正确的键名
['username' => $name, 'age' => $age] = ['username' => 'John', 'age' => 25];
echo "Name: {$name}, Age: {$age}\n";  // Name: John, Age: 25

// 或使用默认值
['name' => $name = 'Unknown', 'age' => $age] = ['username' => 'John', 'age' => 25];
echo "Name: {$name}, Age: {$age}\n";  // Name: Unknown, Age: 25
```

### 问题 3：版本兼容性

**症状**：代码在旧版本 PHP 中无法运行

**原因**：使用了不支持的 PHP 版本

**解决方法**：

```php
<?php
declare(strict_types=1);

// 检查 PHP 版本
if (PHP_VERSION_ID >= 70100) {
    // 使用数组解构
    [$a, $b] = [1, 2];
} else {
    // 降级方案
    $a = [1, 2][0];
    $b = [1, 2][1];
}
```

## 最佳实践

### 解构使用

- **简化代码**：使用解构简化代码
- **使用默认值**：为可能缺失的元素提供默认值
- **匹配结构**：确保解构模式与数组结构匹配

### 代码可读性

- **明确意图**：使用解构明确代码意图
- **避免过度嵌套**：避免过度使用嵌套解构
- **添加注释**：为复杂的解构添加注释

### 错误处理

- **检查结构**：解构前检查数组结构
- **使用默认值**：使用默认值处理缺失值
- **处理异常**：处理可能的异常情况

## 对比分析

### 解构 vs 传统方式

| 特性 | 数组解构 | 传统方式 |
|:-----|:---------|:---------|
| 代码简洁 | 是 | 否 |
| 可读性 | 高 | 中 |
| 性能 | 稍差 | 稍好 |
| 版本要求 | PHP 7.1+ | 所有版本 |
| 推荐度 | 推荐 | 兼容旧版本 |

**选择建议**：
- **PHP 7.1+**：使用数组解构
- **兼容旧版本**：使用传统方式

### 索引解构 vs 关联解构

| 特性 | 索引解构 | 关联解构 |
|:-----|:---------|:---------|
| 键类型 | 位置 | 键名 |
| 可读性 | 中 | 高 |
| 适用场景 | 有序列表 | 配置、映射 |
| 推荐度 | 按需使用 | 推荐（配置） |

**选择建议**：
- **有序列表**：使用索引解构
- **配置、映射**：使用关联解构

## 相关章节

- **2.8.1 数组基础**：了解数组基础
- **2.8.2 数组遍历**：了解数组遍历
- **2.6.2 赋值运算符**：了解赋值运算符

## 练习任务

1. **基本解构练习**：
   - 练习索引数组和关联数组解构
   - 理解跳过元素和默认值
   - 测试各种解构场景
   - 观察解构行为

2. **函数返回值解构练习**：
   - 解构函数返回的数组
   - 实现多值返回函数
   - 测试各种返回值场景
   - 使用默认值处理缺失值

3. **嵌套解构练习**：
   - 练习嵌套数组解构
   - 实现复杂结构解构
   - 测试各种嵌套结构
   - 优化解构模式

4. **循环解构练习**：
   - 在 foreach 中使用解构
   - 实现批量数据处理
   - 测试各种循环场景
   - 优化代码可读性

5. **综合练习**：
   - 创建一个使用数组解构的程序
   - 实现配置解析功能
   - 实现数据提取功能
   - 进行代码审查，确保正确性

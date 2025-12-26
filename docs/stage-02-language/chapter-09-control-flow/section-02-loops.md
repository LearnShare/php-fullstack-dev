# 2.9.2 循环结构

## 概述

循环结构用于重复执行代码。本节详细介绍 `while`、`do-while`、`for`、`foreach` 循环的语法、用法、适用场景、嵌套循环及完整示例。

理解不同循环结构的特点和适用场景对于编写高效、可读的代码非常重要。选择合适的循环结构可以提高代码的性能和可维护性。

## 特性

- **while**：条件未知的循环，先检查条件
- **do-while**：至少执行一次的循环，先执行后检查
- **for**：已知次数的循环，适合计数循环
- **foreach**：遍历数组或可迭代对象，最高效的数组遍历方式

## 语法/定义

### while 循环

**基本语法**：`while (condition) { ... }`

**特点**：
- 先检查条件，条件为真时执行
- 可能不执行（条件初始为假）
- 适合条件未知的循环

### do-while 循环

**基本语法**：`do { ... } while (condition);`

**特点**：
- 先执行后检查条件
- 至少执行一次
- 适合需要至少执行一次的循环

### for 循环

**基本语法**：`for (init; condition; increment) { ... }`

**特点**：
- 包含初始化、条件、递增三部分
- 适合已知次数的循环
- 可以省略任意部分

### foreach 循环

**基本语法**：
- `foreach ($array as $value) { ... }`
- `foreach ($array as $key => $value) { ... }`
- `foreach ($array as &$value) { ... }`（引用遍历）

**特点**：
- 最高效的数组遍历方式
- 自动处理数组指针
- 支持引用遍历修改元素

## 基本用法

### 示例 1：while 循环

```php
<?php
declare(strict_types=1);

// 基本 while 循环
$i = 0;
while ($i < 5) {
    echo $i . "\n";
    $i++;
}
// 输出：0, 1, 2, 3, 4

// 条件未知的循环
$data = [];
while (($line = fgets(STDIN)) !== false) {
    $data[] = trim($line);
    if (count($data) >= 10) {
        break;  // 限制最多读取 10 行
    }
}
```

### 示例 2：do-while 循环

```php
<?php
declare(strict_types=1);

// 基本 do-while 循环
$i = 0;
do {
    echo $i . "\n";
    $i++;
} while ($i < 5);
// 输出：0, 1, 2, 3, 4

// 至少执行一次的循环
$attempts = 0;
do {
    $attempts++;
    $result = performOperation();
} while ($result === false && $attempts < 3);
```

### 示例 3：for 循环

```php
<?php
declare(strict_types=1);

// 基本 for 循环
for ($i = 0; $i < 5; $i++) {
    echo $i . "\n";
}
// 输出：0, 1, 2, 3, 4

// 倒序循环
for ($i = 4; $i >= 0; $i--) {
    echo $i . "\n";
}
// 输出：4, 3, 2, 1, 0

// 多变量循环
for ($i = 0, $j = 10; $i < 5; $i++, $j--) {
    echo "i: {$i}, j: {$j}\n";
}

// 省略部分
$i = 0;
for (; $i < 5; $i++) {
    echo $i . "\n";
}
```

### 示例 4：foreach 循环

```php
<?php
declare(strict_types=1);

// 值遍历
$fruits = ['apple', 'banana', 'orange'];
foreach ($fruits as $fruit) {
    echo $fruit . "\n";
}

// 键值遍历
$user = ['name' => 'John', 'age' => 25, 'email' => 'john@example.com'];
foreach ($user as $key => $value) {
    echo "{$key}: {$value}\n";
}

// 引用遍历
$numbers = [1, 2, 3, 4, 5];
foreach ($numbers as &$number) {
    $number *= 2;
}
unset($number);  // 重要：解除引用
print_r($numbers);  // [2, 4, 6, 8, 10]
```

### 示例 5：嵌套循环

```php
<?php
declare(strict_types=1);

// 嵌套 for 循环
for ($i = 1; $i <= 3; $i++) {
    for ($j = 1; $j <= 3; $j++) {
        echo "({$i}, {$j}) ";
    }
    echo "\n";
}
// 输出：
// (1, 1) (1, 2) (1, 3)
// (2, 1) (2, 2) (2, 3)
// (3, 1) (3, 2) (3, 3)

// 嵌套 foreach 循环
$users = [
    ['name' => 'John', 'hobbies' => ['reading', 'coding']],
    ['name' => 'Jane', 'hobbies' => ['music', 'travel']]
];

foreach ($users as $user) {
    echo "{$user['name']}:\n";
    foreach ($user['hobbies'] as $hobby) {
        echo "  - {$hobby}\n";
    }
}
```

## 完整代码示例

### 示例 1：循环工具类

```php
<?php
declare(strict_types=1);

class LoopHelper
{
    public static function repeat(callable $callback, int $times): void
    {
        for ($i = 0; $i < $times; $i++) {
            $callback($i);
        }
    }
    
    public static function iterateUntil(callable $condition, callable $action): void
    {
        while (!$condition()) {
            $action();
        }
    }
    
    public static function processArray(array $array, callable $processor): array
    {
        $result = [];
        foreach ($array as $key => $value) {
            $result[$key] = $processor($value, $key);
        }
        return $result;
    }
}

// 使用
LoopHelper::repeat(fn($i) => print("Iteration {$i}\n"), 3);

$count = 0;
LoopHelper::iterateUntil(
    fn() => $count >= 5,
    function() use (&$count) {
        echo "Count: {$count}\n";
        $count++;
    }
);

$numbers = [1, 2, 3];
$doubled = LoopHelper::processArray($numbers, fn($n) => $n * 2);
print_r($doubled);  // [2, 4, 6]
```

### 示例 2：数据处理

```php
<?php
declare(strict_types=1);

function processUsers(array $users): array
{
    $processed = [];
    foreach ($users as $user) {
        // 处理每个用户
        $processed[] = [
            'name' => strtoupper($user['name']),
            'age' => $user['age'],
            'status' => $user['age'] >= 18 ? 'adult' : 'minor'
        ];
    }
    return $processed;
}

function findUser(array $users, string $name): ?array
{
    foreach ($users as $user) {
        if ($user['name'] === $name) {
            return $user;
        }
    }
    return null;
}

// 使用
$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 17],
    ['name' => 'Bob', 'age' => 30]
];

$processed = processUsers($users);
print_r($processed);

$user = findUser($users, 'Jane');
print_r($user);
```

### 示例 3：生成器模式

```php
<?php
declare(strict_types=1);

function generateRange(int $start, int $end, int $step = 1): Generator
{
    for ($i = $start; $i <= $end; $i += $step) {
        yield $i;
    }
}

// 使用生成器
foreach (generateRange(1, 10, 2) as $number) {
    echo $number . " ";
}
// 输出：1 3 5 7 9

// 传统方式（占用更多内存）
function getRange(int $start, int $end, int $step = 1): array
{
    $result = [];
    for ($i = $start; $i <= $end; $i += $step) {
        $result[] = $i;
    }
    return $result;
}
```

## 使用场景

### while 循环

- **条件未知**：循环次数未知
- **读取数据**：从文件、数据库等读取数据
- **等待条件**：等待某个条件满足

### do-while 循环

- **至少执行一次**：需要至少执行一次的循环
- **重试逻辑**：重试操作直到成功
- **用户输入**：至少提示一次用户输入

### for 循环

- **计数循环**：已知循环次数
- **数组索引**：使用索引访问数组（不推荐，优先使用 foreach）
- **范围遍历**：遍历数字范围

### foreach 循环

- **数组遍历**：遍历数组（推荐）
- **对象遍历**：遍历实现了 `Iterator` 接口的对象
- **数据转换**：转换数组数据

## 注意事项

### 无限循环

- **避免无限循环**：确保循环条件会改变
- **使用 break**：必要时使用 `break` 退出循环
- **设置超时**：长时间运行的循环设置超时

### 性能考虑

- **foreach 最快**：遍历数组时 `foreach` 性能最好
- **避免重复计算**：避免在循环条件中重复计算
- **提前退出**：满足条件时提前退出循环

### 变量作用域

- **循环变量**：理解循环变量的作用域
- **引用遍历**：引用遍历后记得 `unset()`
- **闭包捕获**：注意闭包中变量的捕获

## 常见问题

### 问题 1：无限循环

**症状**：程序陷入无限循环

**原因**：循环条件永远为真

**错误示例**：

```php
<?php
declare(strict_types=1);

$i = 0;
while ($i < 5) {
    echo $i . "\n";
    // 忘记递增 $i，导致无限循环
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$i = 0;
while ($i < 5) {
    echo $i . "\n";
    $i++;  // 确保条件会改变
}

// 或使用 for 循环
for ($i = 0; $i < 5; $i++) {
    echo $i . "\n";
}
```

### 问题 2：foreach 引用遍历陷阱

**症状**：后续代码意外修改数组元素

**原因**：引用变量在循环后仍然存在

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;
}
// 未 unset($value)

$value = 100;  // 意外修改最后一个元素
print_r($arr);  // [2, 4, 100]
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;
}
unset($value);  // 解除引用

$value = 100;  // 不会影响数组
print_r($arr);  // [2, 4, 6]
```

### 问题 3：循环中修改数组结构

**症状**：循环结果不符合预期

**原因**：循环时修改数组结构可能导致问题

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3, 4, 5];

// 方法1：遍历数组副本
foreach ($arr as $key => $value) {
    if ($value % 2 === 0) {
        unset($arr[$key]);
    }
}

// 方法2：收集要删除的键
$keysToRemove = [];
foreach ($arr as $key => $value) {
    if ($value % 2 === 0) {
        $keysToRemove[] = $key;
    }
}
foreach ($keysToRemove as $key) {
    unset($arr[$key]);
}

// 方法3：使用 array_filter()
$arr = array_filter($arr, fn($value) => $value % 2 !== 0);
```

## 最佳实践

### 循环选择

- **遍历数组**：使用 `foreach`（推荐）
- **计数循环**：使用 `for`
- **条件未知**：使用 `while`
- **至少执行一次**：使用 `do-while`

### 性能优化

- **使用 foreach**：遍历数组时使用 `foreach`
- **提前退出**：满足条件时使用 `break` 提前退出
- **避免重复计算**：缓存循环条件中的计算结果

### 代码可读性

- **使用有意义的变量名**：使用有意义的循环变量名
- **避免嵌套过深**：避免过度嵌套循环
- **提取函数**：复杂循环逻辑提取到函数中

## 对比分析

### 循环性能对比

| 循环类型 | 性能 | 适用场景 |
|:---------|:-----|:---------|
| `foreach` | 最好 | 数组遍历 |
| `for` | 好 | 计数循环 |
| `while` | 好 | 条件未知 |
| `do-while` | 好 | 至少执行一次 |

**选择建议**：
- **数组遍历**：使用 `foreach`
- **计数循环**：使用 `for`
- **条件未知**：使用 `while`

### foreach vs for（数组遍历）

| 特性 | foreach | for |
|:-----|:---------|:----|
| 性能 | 最好 | 稍差 |
| 可读性 | 高 | 中 |
| 键访问 | 自动 | 手动 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **数组遍历**：使用 `foreach`（推荐）
- **需要索引计算**：可以使用 `for`

## 相关章节

- **2.8.2 数组遍历**：详细了解 foreach 循环
- **2.9.3 跳转语句**：了解 break 和 continue
- **2.10 函数与作用域**：了解函数中的循环

## 练习任务

1. **循环结构练习**：
   - 练习使用 `while`、`do-while`、`for`、`foreach`
   - 理解不同循环的适用场景
   - 测试各种循环场景
   - 观察循环行为

2. **嵌套循环练习**：
   - 练习嵌套循环的使用
   - 实现二维数组处理
   - 测试各种嵌套场景
   - 优化嵌套循环性能

3. **实际应用练习**：
   - 实现数据处理功能
   - 实现搜索功能
   - 实现循环工具类
   - 测试各种应用场景

4. **性能优化练习**：
   - 对比不同循环的性能
   - 优化循环代码
   - 使用生成器处理大数据
   - 测试性能改进

5. **综合练习**：
   - 创建一个循环工具类
   - 实现各种循环功能
   - 处理各种数据结构
   - 进行代码审查，确保正确性

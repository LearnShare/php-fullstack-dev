# 2.9.3 跳转语句

## 概述

跳转语句用于改变程序的执行流程。本节详细介绍 `break`、`continue`、`goto` 的语法、用法、跳出多层循环、使用限制及注意事项。

理解跳转语句的作用和限制对于编写清晰、可控的代码非常重要。合理使用 `break` 和 `continue` 可以提高代码效率，而 `goto` 应该避免使用。

## 特性

- **break**：跳出循环或 switch 语句
- **continue**：跳过当前循环迭代，继续下一次迭代
- **goto**：跳转到标签（不推荐使用）

## 语法/定义

### break 语句

**基本语法**：`break;`

**跳出多层语法**：`break $levels;`

**参数**：
- `$levels`：要跳出的层数（1 表示当前循环，2 表示外层循环）

**功能**：
- 跳出当前循环或 switch 语句
- 可以指定跳出层数

### continue 语句

**基本语法**：`continue;`

**跳过多层语法**：`continue $levels;`

**参数**：
- `$levels`：要跳过的层数

**功能**：
- 跳过当前循环迭代，继续下一次迭代
- 可以指定跳过层数

### goto 语句

**基本语法**：
```php
goto label;
// ...
label:
```

**功能**：跳转到标签位置

**限制**：
- 不能跳入函数、类、循环内部
- 不能跳入 switch 或 match 内部
- 可以跳出函数、类、循环

## 基本用法

### 示例 1：break 语句

```php
<?php
declare(strict_types=1);

// 基本 break
for ($i = 0; $i < 10; $i++) {
    if ($i === 5) {
        break;  // 跳出循环
    }
    echo $i . "\n";
}
// 输出：0, 1, 2, 3, 4

// 在 switch 中使用
$value = 1;
switch ($value) {
    case 1:
        echo "One\n";
        break;  // 跳出 switch
    case 2:
        echo "Two\n";
        break;
}

// 跳出多层循环
for ($i = 0; $i < 3; $i++) {
    for ($j = 0; $j < 3; $j++) {
        if ($i === 1 && $j === 1) {
            break 2;  // 跳出两层循环
        }
        echo "({$i}, {$j}) ";
    }
    echo "\n";
}
```

### 示例 2：continue 语句

```php
<?php
declare(strict_types=1);

// 基本 continue
for ($i = 0; $i < 10; $i++) {
    if ($i % 2 === 0) {
        continue;  // 跳过偶数
    }
    echo $i . "\n";
}
// 输出：1, 3, 5, 7, 9

// 跳过多层循环
for ($i = 0; $i < 3; $i++) {
    for ($j = 0; $j < 3; $j++) {
        if ($j === 1) {
            continue 2;  // 跳过外层循环的当前迭代
        }
        echo "({$i}, {$j}) ";
    }
    echo "\n";
}
```

### 示例 3：goto 语句（不推荐）

```php
<?php
declare(strict_types=1);

// goto 基本使用（不推荐）
$i = 0;
start:
if ($i < 5) {
    echo $i . "\n";
    $i++;
    goto start;
}

// goto 用于错误处理（可接受的使用场景）
function processData(array $data): bool
{
    if (empty($data)) {
        goto error;
    }
    
    // 处理数据
    if (!validateData($data)) {
        goto error;
    }
    
    // 成功处理
    return true;
    
    error:
    error_log("Data processing failed");
    return false;
}
```

## 完整代码示例

### 示例 1：搜索功能

```php
<?php
declare(strict_types=1);

class SearchHelper
{
    public static function findInArray(array $array, mixed $value): ?int
    {
        foreach ($array as $index => $item) {
            if ($item === $value) {
                return $index;  // 找到后立即返回
            }
        }
        return null;
    }
    
    public static function findInNestedArray(array $array, mixed $value): ?array
    {
        foreach ($array as $i => $row) {
            foreach ($row as $j => $item) {
                if ($item === $value) {
                    return [$i, $j];  // 找到后立即返回
                }
            }
        }
        return null;
    }
    
    public static function findFirst(array $array, callable $callback): mixed
    {
        foreach ($array as $item) {
            if ($callback($item)) {
                return $item;  // 找到后立即返回
            }
        }
        return null;
    }
}

// 使用
$numbers = [1, 2, 3, 4, 5];
$index = SearchHelper::findInArray($numbers, 3);
echo "Found at index: {$index}\n";  // Found at index: 2
```

### 示例 2：数据处理（使用 continue）

```php
<?php
declare(strict_types=1);

function processUsers(array $users): array
{
    $processed = [];
    foreach ($users as $user) {
        // 跳过无效用户
        if (!isset($user['name']) || !isset($user['age'])) {
            continue;  // 跳过当前迭代
        }
        
        // 跳过未成年用户
        if ($user['age'] < 18) {
            continue;
        }
        
        // 处理有效用户
        $processed[] = [
            'name' => strtoupper($user['name']),
            'age' => $user['age']
        ];
    }
    return $processed;
}

// 使用
$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane'],  // 缺少 age，会被跳过
    ['name' => 'Bob', 'age' => 17],  // 未成年，会被跳过
    ['name' => 'Alice', 'age' => 30]
];

$processed = processUsers($users);
print_r($processed);
```

### 示例 3：避免 goto 的替代方案

```php
<?php
declare(strict_types=1);

// 使用 goto 的错误处理（不推荐）
function processWithGoto(array $data): bool
{
    if (empty($data)) {
        goto error;
    }
    
    if (!validateData($data)) {
        goto error;
    }
    
    return true;
    
    error:
    error_log("Processing failed");
    return false;
}

// 更好的替代方案：使用函数和早期返回
function processWithoutGoto(array $data): bool
{
    if (empty($data)) {
        error_log("Processing failed: empty data");
        return false;
    }
    
    if (!validateData($data)) {
        error_log("Processing failed: invalid data");
        return false;
    }
    
    return true;
}

// 或使用异常处理
function processWithException(array $data): bool
{
    try {
        if (empty($data)) {
            throw new InvalidArgumentException("Data is empty");
        }
        
        if (!validateData($data)) {
            throw new InvalidArgumentException("Data is invalid");
        }
        
        return true;
    } catch (Exception $e) {
        error_log("Processing failed: " . $e->getMessage());
        return false;
    }
}
```

## 使用场景

### break

- **提前退出**：满足条件时提前退出循环
- **搜索找到**：找到目标后立即退出
- **错误处理**：遇到错误时退出循环
- **性能优化**：避免不必要的迭代

### continue

- **跳过无效数据**：跳过不符合条件的数据
- **过滤处理**：只处理符合条件的数据
- **性能优化**：跳过不必要的处理

### goto（不推荐）

- **错误处理**：某些错误处理场景（但应使用异常）
- **状态机**：简单状态机实现（但应使用其他方法）
- **避免使用**：应避免使用，使用函数或重构代码

## 注意事项

### break/continue 层级

- **默认层级**：默认跳出/跳过当前循环
- **指定层级**：可以指定跳出/跳过的层数
- **理解层级**：理解层级编号（1=当前，2=外层）

### goto 限制

- **不能跳入**：不能跳入函数、类、循环
- **可以跳出**：可以跳出函数、类、循环
- **可读性**：过度使用降低代码可读性

### 替代方案

- **使用函数**：使用函数替代 goto
- **早期返回**：使用早期返回替代 goto
- **异常处理**：使用异常处理错误情况

## 常见问题

### 问题 1：goto 使用限制

**症状**：goto 语句报错

**原因**：试图跳入函数、类或循环

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：不能跳入循环
goto inside_loop;
for ($i = 0; $i < 5; $i++) {
    inside_loop:  // 错误
    echo $i . "\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：重构代码，避免使用 goto
for ($i = 0; $i < 5; $i++) {
    echo $i . "\n";
}

// 方法2：使用函数
function processLoop(): void
{
    for ($i = 0; $i < 5; $i++) {
        echo $i . "\n";
    }
}
processLoop();
```

### 问题 2：跳出多层循环

**症状**：需要跳出嵌套循环

**原因**：break 默认只跳出当前循环

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：使用 break $levels
$found = false;
for ($i = 0; $i < 3; $i++) {
    for ($j = 0; $j < 3; $j++) {
        if ($i === 1 && $j === 1) {
            $found = true;
            break 2;  // 跳出两层循环
        }
    }
}

// 方法2：使用标志变量
$found = false;
for ($i = 0; $i < 3 && !$found; $i++) {
    for ($j = 0; $j < 3 && !$found; $j++) {
        if ($i === 1 && $j === 1) {
            $found = true;
        }
    }
}

// 方法3：提取为函数
function findInNested(array $array, mixed $value): ?array
{
    foreach ($array as $i => $row) {
        foreach ($row as $j => $item) {
            if ($item === $value) {
                return [$i, $j];  // 使用 return 跳出
            }
        }
    }
    return null;
}
```

### 问题 3：continue 跳过逻辑错误

**症状**：continue 跳过逻辑不符合预期

**原因**：不理解 continue 的作用

**解决方法**：

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 正确使用 continue
foreach ($numbers as $number) {
    if ($number % 2 === 0) {
        continue;  // 跳过偶数，继续下一次迭代
    }
    echo $number . "\n";  // 只输出奇数
}
// 输出：1, 3, 5

// 错误：使用 break 会提前退出
foreach ($numbers as $number) {
    if ($number % 2 === 0) {
        break;  // 错误：遇到第一个偶数就退出
    }
    echo $number . "\n";
}
// 输出：1（只输出第一个奇数）
```

## 最佳实践

### break/continue 使用

- **合理使用**：合理使用 break 和 continue
- **明确意图**：使用注释说明 break/continue 的意图
- **避免过度**：避免过度使用，考虑重构代码

### goto 避免

- **避免使用**：应避免使用 goto
- **使用函数**：使用函数替代 goto
- **早期返回**：使用早期返回替代 goto
- **异常处理**：使用异常处理错误情况

### 代码可读性

- **提取函数**：复杂逻辑提取到函数中
- **使用标志**：使用标志变量控制循环
- **重构代码**：重构代码提高可读性

## 对比分析

### break vs continue

| 特性 | break | continue |
|:-----|:------|:---------|
| 作用 | 退出循环 | 跳过当前迭代 |
| 后续执行 | 不执行 | 继续下一次迭代 |
| 适用场景 | 找到目标 | 跳过无效数据 |
| 推荐度 | 推荐 | 推荐 |

**选择建议**：
- **找到目标**：使用 `break`
- **跳过数据**：使用 `continue`

### goto vs 函数/异常

| 特性 | goto | 函数/异常 |
|:-----|:-----|:----------|
| 可读性 | 低 | 高 |
| 维护性 | 差 | 好 |
| 推荐度 | 不推荐 | 推荐 |

**选择建议**：
- **避免 goto**：使用函数或异常处理
- **重构代码**：重构代码提高可读性

## 相关章节

- **2.9.1 条件语句**：了解条件判断
- **2.9.2 循环结构**：了解循环结构
- **2.10 函数与作用域**：了解函数中的跳转

## 练习任务

1. **跳转语句练习**：
   - 练习使用 `break` 和 `continue`
   - 理解跳出多层循环
   - 测试各种跳转场景
   - 观察跳转行为

2. **实际应用练习**：
   - 实现搜索功能
   - 实现数据过滤功能
   - 实现循环控制逻辑
   - 测试各种应用场景

3. **代码重构练习**：
   - 将使用 goto 的代码重构为函数
   - 使用早期返回替代 goto
   - 使用异常处理错误情况
   - 提高代码可读性

4. **综合练习**：
   - 创建一个使用跳转语句的程序
   - 实现各种控制逻辑
   - 避免使用 goto
   - 进行代码审查，确保正确性

# 2.9.3 跳转语句

## 概述

跳转语句用于改变程序的执行流程。PHP 提供了 `break`、`continue` 和 `goto` 三种跳转语句，用于控制循环和代码执行。

## break 语句

### 基本语法

```php
break;
break $levels;  // PHP 5.4+，跳出多层循环
```

### 在循环中使用

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

foreach ($numbers as $number) {
    if ($number === 3) {
        break;  // 跳出循环
    }
    echo $number . "\n";
}
// 输出：1, 2
```

### 在 switch 中使用

```php
<?php
declare(strict_types=1);

$status = 200;

switch ($status) {
    case 200:
        echo "OK\n";
        break;  // 必须使用 break，否则会贯穿
    case 404:
        echo "Not Found\n";
        break;
}
```

### 跳出多层循环

```php
<?php
declare(strict_types=1);

$matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

foreach ($matrix as $row) {
    foreach ($row as $value) {
        if ($value === 5) {
            break 2;  // 跳出两层循环
        }
        echo $value . " ";
    }
}
// 输出：1 2 3 4
```

## continue 语句

### 基本语法

```php
continue;
continue $levels;  // PHP 5.4+，跳过多层循环
```

### 在循环中使用

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

foreach ($numbers as $number) {
    if ($number % 2 === 0) {
        continue;  // 跳过本次循环，继续下一次
    }
    echo $number . "\n";
}
// 输出：1, 3, 5（跳过偶数）
```

### 跳过多层循环

```php
<?php
declare(strict_types=1);

$matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

foreach ($matrix as $rowIndex => $row) {
    foreach ($row as $colIndex => $value) {
        if ($value === 5) {
            continue 2;  // 跳过两层循环，继续外层循环的下一次
        }
        echo "Matrix[{$rowIndex}][{$colIndex}] = {$value}\n";
    }
}
```

## goto 语句

### 基本语法

```php
goto label;
// ...
label:
// 代码
```

### 基本用法

```php
<?php
declare(strict_types=1);

$value = 10;

if ($value > 5) {
    goto large;
}

echo "Small value\n";
goto end;

large:
echo "Large value\n";

end:
echo "Done\n";
```

### 使用限制

`goto` 有以下限制：
- 不能跳转到函数内部
- 不能跳转到类定义内部
- 不能跳转到循环或 switch 内部
- 可以跳出循环或 switch

### 跳出循环

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 10; $i++) {
    for ($j = 0; $j < 10; $j++) {
        if ($i * $j === 25) {
            goto found;
        }
    }
}

found:
echo "Found at i = {$i}, j = {$j}\n";
```

### 替代方案

大多数情况下，可以使用其他方式替代 `goto`：

```php
<?php
declare(strict_types=1);

// 使用 goto
function processWithGoto(array $data): void
{
    foreach ($data as $item) {
        if ($item === null) {
            goto error;
        }
        // 处理逻辑
    }
    return;
    
error:
    echo "Error: null value found\n";
}

// 使用函数返回（推荐）
function processWithoutGoto(array $data): bool
{
    foreach ($data as $item) {
        if ($item === null) {
            echo "Error: null value found\n";
            return false;
        }
        // 处理逻辑
    }
    return true;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class JumpStatements
{
    public static function demonstrateBreak(): void
    {
        echo "=== break 语句 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        foreach ($numbers as $number) {
            if ($number === 3) {
                break;
            }
            echo $number . " ";
        }
        echo "\n";
    }
    
    public static function demonstrateContinue(): void
    {
        echo "\n=== continue 语句 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        foreach ($numbers as $number) {
            if ($number % 2 === 0) {
                continue;
            }
            echo $number . " ";
        }
        echo "\n";
    }
    
    public static function demonstrateGoto(): void
    {
        echo "\n=== goto 语句 ===\n";
        $value = 10;
        
        if ($value > 5) {
            goto large;
        }
        
        echo "Small value\n";
        goto end;
        
        large:
        echo "Large value\n";
        
        end:
        echo "Done\n";
    }
}

JumpStatements::demonstrateBreak();
JumpStatements::demonstrateContinue();
JumpStatements::demonstrateGoto();
```

## 注意事项

1. **break 和 continue 的层级**：使用数字参数可以控制跳出或跳过的循环层数。

2. **goto 的使用**：`goto` 会降低代码可读性，应尽量避免使用，优先使用函数、异常等替代方案。

3. **switch 中的 break**：`switch` 中必须使用 `break`，否则会贯穿到下一个 `case`。

4. **性能考虑**：`break` 和 `continue` 对性能影响很小，`goto` 可能影响代码优化。

5. **代码可读性**：过度使用跳转语句会降低代码可读性，应谨慎使用。

## 练习

1. 创建一个函数，使用 `break` 在找到目标值后立即退出循环。

2. 编写一个函数，使用 `continue` 跳过不符合条件的元素。

3. 实现一个函数，使用 `break 2` 跳出嵌套循环。

4. 创建一个函数，将使用 `goto` 的代码重构为不使用 `goto` 的版本。

5. 编写一个函数，演示 `break` 和 `continue` 在不同循环结构中的使用。

# 2.9 控制结构

## 目标

- 熟悉条件分支、循环、跳转语句在 PHP 中的使用方式。
- 能根据业务需求选择合适的控制结构（`if`、`match`、`switch`、循环）。
- 了解组合语句（`break` 多层、`goto`）及其使用谨慎点。

## 条件语句

### `if / elseif / else`

```php
if ($score >= 90) {
    $grade = 'A';
} elseif ($score >= 80) {
    $grade = 'B';
} else {
    $grade = 'C';
}
```

### `switch`

- 用于多分支匹配，比较使用宽松模式。
- `break` 防止贯穿。

```php
switch ($status) {
    case 'pending':
        $label = '待处理';
        break;
    case 'done':
        $label = '完成';
        break;
    default:
        $label = '未知';
}
```

### `match`（PHP 8）

- 使用严格比较，返回值。

```php
$label = match ($status) {
    'pending' => '待处理',
    'done' => '完成',
    default => '未知',
};
```

## 循环结构

### `while` 与 `do ... while`

```php
while (($line = fgets($handle)) !== false) {
    // 处理
}

do {
    $choice = trim(fgets(STDIN));
} while (!in_array($choice, ['y', 'n'], true));
```

### `for`

- 适合已知次数的迭代。

```php
for ($i = 0; $i < count($items); $i++) {
    echo $items[$i];
}
```

### `foreach`

- 遍历数组或实现 `Traversable` 接口的对象。

```php
foreach ($users as $id => $user) {
    echo "{$id}: {$user['name']}";
}
```

### `break` 与 `continue`

- `break` 跳出循环；`break 2` 可跳出两层。
- `continue` 跳过当前迭代；`continue 2` 跳过外层循环的当前迭代。

## `goto`（谨慎使用）

- 语法：`goto label; ... label:`
- 易导致代码结构混乱，仅在异常流程或生成器中偶尔使用。

## `declare`

- `declare(strict_types=1);`：启用严格类型。
- `declare(ticks=1);`：注册 tick 处理器。
- `declare(encoding='UTF-8');`（已废弃）。

## `try/catch/finally`

- 虽属于异常处理，但常与控制流搭配。

```php
try {
    $result = $service->handle($payload);
} catch (Throwable $e) {
    logger()->error('handle.failed', ['error' => $e->getMessage()]);
    throw $e;
} finally {
    // 清理资源
}
```

## 练习

1. 使用 `match` 重构多层 `if/else`，实现订单状态到标签、颜色的映射。
2. 编写 `retry(callable $operation, int $times)` 函数，使用循环与 `try/catch` 实现重试机制。
3. 使用 `foreach` 和 `break 2` 统计二维数组中某个值首次出现的位置。

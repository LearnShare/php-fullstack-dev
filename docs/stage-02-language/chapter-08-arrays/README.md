# 2.8 数组完整指南

## 目标

- 掌握索引数组、关联数组、嵌套数组的创建与访问。
- 熟悉常用数组函数（遍历、过滤、映射、排序、解构）。
- 能在 JSON、表单、配置文件等场景中高效处理数组数据。

## 数组基础

- PHP 数组是有序映射，可同时包含数字键与字符串键。
- 创建语法：`[]` 或 `array()`。
- 读取元素：`$arr['key']`、`$arr[0]`。

```php
$user = [
    'id' => 1,
    'name' => 'Alice',
    'roles' => ['admin', 'editor'],
];
```

## 与 JavaScript 对比

如果你熟悉 JavaScript，理解 PHP 数组的关键差异：

### 数组 vs 对象

**JavaScript**：
- 数组：`[1, 2, 3]`（数字索引）
- 对象：`{id: 1, name: 'Alice'}`（字符串键）

**PHP**：
- **数组统一处理**：`[1, 2, 3]` 和 `['id' => 1, 'name' => 'Alice']` 都是数组
- PHP 数组可以同时包含数字键和字符串键

```php
// PHP：混合数组
$mixed = [
    0 => 'first',      // 数字键
    'key' => 'value',  // 字符串键
    1 => 'second',     // 数字键
];
```

### 数组创建对比

| 操作 | JavaScript | PHP |
| :--- | :--------- | :-- |
| 索引数组 | `[1, 2, 3]` | `[1, 2, 3]` 或 `array(1, 2, 3)` |
| 关联数组 | `{id: 1, name: 'Alice'}` | `['id' => 1, 'name' => 'Alice']` |
| 访问元素 | `arr[0]` 或 `obj.key` | `$arr[0]` 或 `$arr['key']` |
| 添加元素 | `arr.push(4)` | `$arr[] = 4` 或 `array_push($arr, 4)` |

### 遍历对比

**JavaScript**：
```javascript
// forEach
[1, 2, 3].forEach((value, index) => {
    console.log(index, value);
});

// for...of
for (const value of [1, 2, 3]) {
    console.log(value);
}

// for...in（对象）
for (const key in {id: 1, name: 'Alice'}) {
    console.log(key, obj[key]);
}
```

**PHP**：
```php
// foreach（类似 forEach 和 for...of）
foreach ([1, 2, 3] as $index => $value) {
    echo "{$index}: {$value}\n";
}

// 只遍历值（类似 for...of）
foreach ([1, 2, 3] as $value) {
    echo "{$value}\n";
}

// 遍历关联数组（类似 for...in）
foreach (['id' => 1, 'name' => 'Alice'] as $key => $value) {
    echo "{$key}: {$value}\n";
}
```

### 高阶函数对比

| 操作 | JavaScript | PHP |
| :--- | :--------- | :-- |
| 映射 | `arr.map(x => x * 2)` | `array_map(fn($x) => $x * 2, $arr)` |
| 过滤 | `arr.filter(x => x > 0)` | `array_filter($arr, fn($x) => $x > 0)` |
| 归约 | `arr.reduce((acc, x) => acc + x, 0)` | `array_reduce($arr, fn($acc, $x) => $acc + $x, 0)` |
| 查找 | `arr.find(x => x > 5)` | `array_values(array_filter($arr, fn($x) => $x > 5))[0] ?? null` |

**示例对比**：

```javascript
// JavaScript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(x => x * 2);
const evens = numbers.filter(x => x % 2 === 0);
const sum = numbers.reduce((acc, x) => acc + x, 0);
```

```php
// PHP
$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(fn($x) => $x * 2, $numbers);
$evens = array_filter($numbers, fn($x) => $x % 2 === 0);
$sum = array_reduce($numbers, fn($acc, $x) => $acc + $x, 0);
```

### 解构对比

**JavaScript**：
```javascript
// 数组解构
const [x, y] = [10, 20];

// 对象解构
const {name, id} = {id: 1, name: 'Alice'};
```

**PHP**：
```php
// 数组解构（PHP 7.1+）
[$x, $y] = [10, 20];

// 关联数组解构
['name' => $name, 'id' => $id] = ['id' => 1, 'name' => 'Alice'];
```

### 关键差异总结

1. **统一数据结构**：PHP 数组同时扮演 JavaScript 中数组和对象的角色
2. **键的类型**：PHP 数组键可以是整数或字符串，JavaScript 数组键只能是数字
3. **函数式编程**：PHP 的高阶函数需要将数组作为第二个参数，JavaScript 是方法调用
4. **类型检查**：PHP 8+ 支持类型声明，但数组仍然是动态的

## 遍历

- `foreach ($array as $value)`：遍历值。
- `foreach ($array as $key => $value)`：同时获取键和值。
- 引用遍历：`foreach ($array as &$value)`（遍历结束后需 `unset($value)`，避免引用残留）。

## 解构与 `list`

- 支持数组解构（PHP 7.1+）：

```php
$point = [10, 20];
[$x, $y] = $point;
$user = ['id' => 1, 'name' => 'Alice'];
['name' => $name] = $user;
```

## 数组操作函数

| 函数         | 语法                                                   | 说明                         | 版本要求 |
| :----------- | :----------------------------------------------------- | :--------------------------- | :------- |
| `array_key_exists` | `array_key_exists(mixed $key, array $array): bool` | 判断键是否存在             | -        |
| `in_array`   | `in_array(mixed $needle, array $haystack, bool $strict = false): bool` | 判断值是否存在 | -        |
| `array_keys` | `array_keys(array $array, mixed $filter_value = null, bool $strict = false): array` | 返回所有键 | -        |
| `array_values` | `array_values(array $array): array`                   | 返回所有值                 | -        |
| `array_merge` | `array_merge(array ...$arrays): array`                 | 合并数组                     | -        |
| `array_map`  | `array_map(callable $callback, array $array, array ...$arrays): array` | 对每个元素执行回调 | -        |
| `array_filter` | `array_filter(array $array, ?callable $callback = null, int $mode = 0): array` | 过滤数组 | -        |
| `array_first` | `array_first(array $array): mixed`                     | 获取第一个元素（不影响指针） | PHP 8.4+ |
| `array_last`  | `array_last(array $array): mixed`                      | 获取最后一个元素（不影响指针） | PHP 8.4+ |

### array_first() 和 array_last()（PHP 8.4+）

- **语法**：
  - `array_first(array $array): mixed`：获取数组的第一个元素
  - `array_last(array $array): mixed`：获取数组的最后一个元素
- **说明**：获取元素但不影响数组内部指针，比 `reset()` 和 `end()` 更安全。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// PHP 8.4+ 新函数
$first = array_first($numbers); // 1
$last = array_last($numbers);   // 5

// 不影响数组内部指针
current($numbers); // 仍然是 1（如果之前没有移动过）

// 空数组返回 null
$empty = [];
$first = array_first($empty); // null
$last = array_last($empty);   // null
```

### 与 reset() 和 end() 的区别

```php
<?php
declare(strict_types=1);

$array = [1, 2, 3, 4, 5];

// 传统方式（会移动内部指针）
reset($array);
$first = current($array); // 1，但指针已移动

end($array);
$last = current($array); // 5，指针在末尾

// PHP 8.4+ 方式（不影响指针）
$first = array_first($array); // 1，指针位置不变
$last = array_last($array);    // 5，指针位置不变
```
| `array_reduce` | `array_reduce(array $array, callable $callback, mixed $initial = null): mixed` | 聚合计算 |
| `array_slice` | `array_slice(array $array, int $offset, ?int $length = null, bool $preserve_keys = false): array` | 取子数组 |

示例：

```php
$scores = [80, 90, 70];
$avg = array_reduce($scores, fn ($carry, $score) => $carry + $score, 0) / count($scores);
```

## 排序

- 基于值：`sort`（升序）、`rsort`（降序）。
- 保留键：`asort`、`arsort`。
- 基于键：`ksort`、`krsort`。
- 自定义：`usort($array, $callback)`。

示例：

```php
$users = [
    ['id' => 2, 'name' => 'Bob'],
    ['id' => 1, 'name' => 'Alice'],
];
usort($users, fn ($a, $b) => $a['id'] <=> $b['id']);
```

## 数组与 JSON

- `json_encode($array, JSON_UNESCAPED_UNICODE)`：数组转 JSON。
- `json_decode($json, true, 512, JSON_THROW_ON_ERROR)`：JSON 转关联数组。

```php
$payload = ['id' => 1, 'name' => 'PHP'];
$json = json_encode($payload, JSON_UNESCAPED_UNICODE);
$decoded = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
```

## 数组与对象转换

- `(object) $array`：数组转对象。
- `(array) $object`：对象转数组。
- 谨慎使用，结构复杂时建议使用 DTO（数据传输对象）。

## 数组解构传参（PHP 8.1+）

- `function foo(int $x, int $y) { ... }`
- 调用：`foo(...['x' => 1, 'y' => 2]);`
- 使用命名参数结合扩展运算符可以动态构建参数列表。

## 生成器与迭代器

- 对大量数据可使用 `yield` 返回生成器，避免一次性加载数组。
  ```php
  function getRows(): Generator
  {
      foreach (DB::cursor('SELECT * FROM users') as $row) {
          yield $row;
      }
  }
  ```

## 数组与引用

- `array_walk(&$array, $callback)`：回调可直接修改原数组。
- `array_map` 返回新数组，不会修改原数组。
- 结构复杂时，可组合 `array_replace_recursive`、`array_diff_assoc` 等进行差异比对。

## 练习

1. 编写 `flatten(array $input): array`，将任意嵌套数组展开为一维数组。
2. 基于 `array_reduce` 实现 `groupBy(array $items, string $key): array`。
3. 编写 `array_pluck(array $items, string $keyPath): array`，支持 `profile.email` 这类点号路径。

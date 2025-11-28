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

| 函数         | 语法                                                   | 说明                         |
| :----------- | :----------------------------------------------------- | :--------------------------- |
| `array_key_exists` | `array_key_exists(mixed $key, array $array): bool` | 判断键是否存在             |
| `in_array`   | `in_array(mixed $needle, array $haystack, bool $strict = false): bool` | 判断值是否存在 |
| `array_keys` | `array_keys(array $array, mixed $filter_value = null, bool $strict = false): array` | 返回所有键 |
| `array_values` | `array_values(array $array): array`                   | 返回所有值                 |
| `array_merge` | `array_merge(array ...$arrays): array`                 | 合并数组                     |
| `array_map`  | `array_map(callable $callback, array $array, array ...$arrays): array` | 对每个元素执行回调 |
| `array_filter` | `array_filter(array $array, ?callable $callback = null, int $mode = 0): array` | 过滤数组 |
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

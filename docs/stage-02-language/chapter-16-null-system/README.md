# 2.16 isset / empty / Null 体系

## 目标

- 理解 `null`、未定义变量、空值之间的差异。
- 熟悉 `isset`、`empty`、`is_null`、空合并运算符 `??` 的使用场景。
- 避免因空值判断错误导致的逻辑漏洞。

## `null` 与未定义变量

- 未定义变量访问会触发 `E_NOTICE`，但 `isset` 会返回 `false`。
- `null` 表示“显式为空”，常用于占位或可选值。

```php
$value = null;
echo $value === null ? 'null' : 'not null';
```

## `isset`

- **语法**：`isset(mixed $var, mixed ...$vars): bool`
- 返回 `true` 当变量存在且不为 `null`。
- 多个参数时只有全部存在且不为 `null` 才返回 `true`。

```php
if (isset($_GET['page'])) {
    $page = (int) $_GET['page'];
}
```

## `empty`

- **语法**：`empty(mixed $var): bool`
- 当变量不存在或值为以下之一返回 `true`：`""`、`0`、`"0"`、`0.0`、`null`、`false`、`[]`。

```php
if (empty($input['keyword'])) {
    throw new InvalidArgumentException('Keyword required');
}
```

> **注意**：`empty('0')` 为 `true`，在处理数字字符串时要慎用。

## `is_null`

- **语法**：`is_null(mixed $value): bool`
- 等价于 `$value === null`。

## 空合并运算符 `??`

- 语法：`$value = $input['key'] ?? 'default';`
- 仅在左侧变量不存在或为 `null` 时返回右侧。
- 与 `?:` 不同，`?:` 会把空字符串或 `0` 当作 `false`。

```php
$page = $_GET['page'] ?? 1;
$keyword = $input['keyword'] ?? fn () => throw new InvalidArgumentException();
```

## Null 合并赋值（PHP 7.4+）

- 语法：`$array['key'] ??= 'default';`
- 等价于 `$array['key'] = $array['key'] ?? 'default';`

```php
$config['timezone'] ??= 'UTC';
```

## 判空策略

1. **必须存在**：使用 `array_key_exists` + `isset` 结合类型校验。
2. **可选值**：使用 `??` 提供默认值。
3. **数字字符串**：避免 `empty`，改用 `trim($value) === ''`。
4. **对象属性**：统一初始化，减少 `isset($obj->prop)`。

## 练习

1. 封装 `data_get($array, $path, $default = null)`，支持点号路径并使用 `??`。
2. 实现 `coalesce(...$values)`，返回第一个非 `null` 值，类似 SQL `COALESCE`。
3. 审查项目中 `empty` 的使用场景，找出潜在的“`'0'` 被视为空”的问题并修复。

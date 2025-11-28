# 2.10 函数与作用域

## 目标

- 掌握函数定义、参数类型、默认值、可变参数与返回类型。
- 理解作用域（局部、全局、静态）和闭包中的 `use` 语义。
- 熟悉函数式工具（回调、可变参数、尾调用）与常用内置函数。

## 函数定义

```php
function greet(string $name, string $title = 'Mr.'): string
{
    return "Hello {$title} {$name}";
}
```

- 参数可声明类型；默认值必须在必填参数之后。
- 返回类型使用 `: type` 指定，支持联合类型与 `void`。

## 可变参数

- 语法：`function sum(int ...$numbers): int`
- `...$numbers` 会接收所有剩余参数，并以数组形式提供。

```php
function sum(int ...$nums): int
{
    return array_sum($nums);
}
```

## 传值与传引用

- 默认按值传递；使用 `&` 表示引用传参。

```php
function increment(int &$value): void
{
    $value++;
}
```

- 引用传参适用于需要修改实参的场景，使用时需标注注释说明。

## 箭头函数（PHP 7.4+）

- 语法：`fn ($x) => $x + 1`
- 自动继承父作用域变量（按值）。
- 常用于回调、简洁的映射逻辑。

```php
$names = array_map(fn ($user) => $user['name'], $users);
```

## 闭包与 `use`

- 使用匿名函数捕获外部变量。

```php
$factor = 10;
$multiply = function (int $value) use ($factor): int {
    return $value * $factor;
};
```

- `use (&$var)` 以引用形式捕获，可在闭包内修改。

## 作用域

- **局部作用域**：函数内部定义的变量外部不可见。
- **全局作用域**：脚本顶级作用域。
- **静态变量**：函数内部的 `static $count = 0;`，在多次调用间持久化。
- 使用 `global $var;` 或 `$GLOBALS['var']` 访问全局变量（应尽量避免）。

## 内置函数

| 函数           | 作用                              |
| :------------- | :-------------------------------- |
| `func_get_args` | 获取当前函数参数数组（传统写法） |
| `call_user_func` | 调用可调用类型                  |
| `call_user_func_array` | 以数组形式传参函数        |
| `is_callable`  | 判断变量是否可调用               |
| `array_map`、`array_filter` | 常见回调函数         |

## 函数的返回策略

- **早返回**：尽早退出函数，提高可读性。
  ```php
  function handle(?User $user): void
  {
      if ($user === null) {
          return;
      }
      // ...
  }
  ```
- **尾调用**：函数最后一行直接返回另一个函数结果，减少临时变量。

## 可调用类型（`callable`）

- 函数名字符串：`'trim'`
- 静态方法：`[ClassName::class, 'method']`
- 对象方法：`[$object, 'method']`
- 匿名函数：`function () {}`
- 实现 `__invoke()` 的对象。

示例：

```php
function apply(callable $callback, array $items): array
{
    return array_map($callback, $items);
}
```

## 命名参数（PHP 8）

- 调用函数时指定参数名，提升可读性。

```php
sendEmail(
    to: 'user@example.com',
    subject: 'Welcome',
    body: 'Hello!'
);
```

## 练习

1. 编写 `memoize(callable $fn): callable`，缓存纯函数的返回值。
2. 使用 `callable` 实现一个通用的 `pipeline(array $stages): callable`，依次执行多个处理阶段。
3. 改写已有函数，利用命名参数与默认值提升可读性。

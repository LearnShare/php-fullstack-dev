# 2.11 匿名函数与闭包

## 目标

- 理解匿名函数、闭包、箭头函数的核心概念。
- 熟悉 `use` 关键字捕获变量、按值/按引用捕获的差异。
- 掌握闭包在回调、事件、依赖注入中的实战用法。

## 匿名函数基础

```php
$logger = function (string $message): void {
    echo "[LOG] {$message}\n";
};
$logger('Service started');
```

- 函数无名称，赋值给变量或传入其他函数。
- 可声明参数类型与返回类型。

## `use` 捕获变量

- 默认按值捕获：

```php
$prefix = '[INFO]';
$log = function (string $text) use ($prefix): void {
    echo "{$prefix} {$text}\n";
};
```

- 按引用捕获：`use (&$prefix)`，闭包内修改会影响外部。

## 箭头函数

- 语法：`fn ($arg) => expression`
- 自动按值捕获父作用域变量。

```php
$multiply = fn (int $n) => $n * $factor;
```

## 闭包与面向对象

- PHP 提供 `Closure` 类，可使用以下方法：
  - `Closure::bindTo($object, ?string $class = null)`
  - `Closure::fromCallable(callable $callable)`

示例：

```php
$closure = function (): string {
    return $this->secret;
};
$bound = $closure->bindTo($object, $object);
echo $bound();
```

## 高阶函数应用

1. **回调**：`array_map`, `array_filter`, `usort`.
2. **事件监听**：将闭包注册到事件总线。
3. **依赖注入（DI）**：在容器中延迟构建服务。

```php
class Container
{
    private array $bindings = [];

    public function bind(string $id, callable $factory): void
    {
        $this->bindings[$id] = $factory;
    }

    public function make(string $id): mixed
    {
        return ($this->bindings[$id])($this);
    }
}
```

## 使用 Generator 与闭包

- 闭包可返回 `Generator`，用于惰性计算：

```php
function chunk(array $items, int $size): Generator
{
    $total = count($items);
    for ($i = 0; $i < $total; $i += $size) {
        yield array_slice($items, $i, $size);
    }
}
```

## 常见陷阱

- 引用捕获后忘记 `unset`，导致变量引用保留。
- 闭包序列化：默认不可序列化，若需持久化可使用 `opis/closure`。
- 过度嵌套闭包会降低可读性，必要时提取为私有方法。

## 练习

1. 编写 `once(callable $fn): callable`，使得函数只执行一次并缓存结果。
2. 使用闭包实现一个简单的任务队列，每个任务可访问共享配置。
3. 尝试使用 `Closure::bind` 访问对象的私有属性，并讨论其风险。

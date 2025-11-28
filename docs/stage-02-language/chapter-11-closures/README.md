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

## 与 JavaScript 对比

如果你熟悉 JavaScript，理解 PHP 闭包的关键差异：

### 语法对比

**JavaScript**：
```javascript
// 箭头函数（自动捕获外部变量）
const factor = 10;
const multiply = (n) => n * factor;

// 普通函数（需要手动捕获）
const multiply2 = function(n) {
    return n * factor;  // 自动捕获外部变量
};
```

**PHP**：
```php
// 箭头函数（PHP 7.4+，自动按值捕获）
$factor = 10;
$multiply = fn(int $n) => $n * $factor;

// 匿名函数（需要 use 关键字）
$multiply2 = function(int $n) use ($factor): int {
    return $n * $factor;  // 必须使用 use 捕获
};
```

### 变量捕获对比

| 特性 | JavaScript | PHP |
| :--- | :--------- | :-- |
| 箭头函数捕获 | 自动捕获外部变量 | 自动按值捕获（PHP 7.4+） |
| 普通函数捕获 | 自动捕获外部变量 | 必须使用 `use` 关键字 |
| 按引用捕获 | 默认按引用 | 使用 `use (&$var)` |
| 修改外部变量 | 可以修改（非 const） | 按值捕获不能修改，按引用可以 |

**示例对比**：

```javascript
// JavaScript：箭头函数自动捕获
let count = 0;
const increment = () => {
    count++;  // 可以修改外部变量
    return count;
};
increment();  // count 变为 1
```

```php
// PHP：箭头函数按值捕获，不能修改外部变量
$count = 0;
$increment = fn() => ++$count;  // 错误：不能修改外部变量

// 需要使用匿名函数 + use(&$count)
$increment = function() use (&$count): int {
    $count++;  // 按引用捕获，可以修改
    return $count;
};
$increment();  // $count 变为 1
```

### 闭包作用域对比

**JavaScript**：
```javascript
// 闭包可以访问外部作用域的变量
function createCounter() {
    let count = 0;
    return () => {
        count++;
        return count;
    };
}

const counter = createCounter();
counter();  // 1
counter();  // 2
```

**PHP**：
```php
// PHP 闭包需要显式使用 use 捕获
function createCounter(): Closure {
    $count = 0;
    return function() use (&$count): int {
        $count++;
        return $count;
    };
}

$counter = createCounter();
$counter();  // 1
$counter();  // 2
```

### 回调函数对比

**JavaScript**：
```javascript
// 数组方法
const numbers = [1, 2, 3];
const doubled = numbers.map(x => x * 2);
const evens = numbers.filter(x => x % 2 === 0);

// 事件监听
button.addEventListener('click', () => {
    console.log('clicked');
});
```

**PHP**：
```php
// 数组函数（需要将数组作为参数）
$numbers = [1, 2, 3];
$doubled = array_map(fn($x) => $x * 2, $numbers);
$evens = array_filter($numbers, fn($x) => $x % 2 === 0);

// 事件系统（框架中常见）
$dispatcher->listen('user.created', function($user) {
    echo "User created: {$user->name}\n";
});
```

### 关键差异总结

1. **变量捕获**：
   - JavaScript：闭包自动捕获外部变量
   - PHP：必须使用 `use` 关键字显式捕获

2. **修改外部变量**：
   - JavaScript：箭头函数可以修改外部变量（如果变量不是 const）
   - PHP：箭头函数按值捕获，不能修改；需要使用 `use (&$var)` 按引用捕获

3. **语法**：
   - JavaScript：`(arg) => expression` 或 `function() { ... }`
   - PHP：`fn($arg) => expression` 或 `function() use ($var) { ... }`

4. **this 绑定**：
   - JavaScript：箭头函数不绑定 `this`，普通函数绑定 `this`
   - PHP：闭包可以使用 `$this`（在类中），但需要 `use ($this)`

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

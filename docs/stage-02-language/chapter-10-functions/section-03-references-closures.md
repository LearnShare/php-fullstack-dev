# 2.10.3 引用、箭头函数与闭包

## 概述

引用传参、箭头函数和闭包是 PHP 函数的高级特性。理解这些特性对于编写高效、灵活的代码至关重要。

## 引用传参

### 概念与原理

**引用传参**（Pass by Reference）是 PHP 函数参数传递的一种方式。与值传参（Pass by Value）不同，引用传参允许函数直接操作外部变量，而不是操作变量的副本。

#### 值传参与引用传参的区别

**值传参**（默认方式）：
- 函数接收的是变量的**副本**
- 函数内部对参数的修改**不会影响**外部变量
- 适用于大多数场景，避免意外的副作用

**引用传参**：
- 函数接收的是变量的**引用**（可以理解为变量的别名）
- 函数内部对参数的修改**会直接影响**外部变量
- 适用于需要修改外部变量的场景

#### 工作原理

当使用引用传参时：
1. 函数参数前添加 `&` 符号
2. 调用函数时，传递的变量与函数参数建立引用关系
3. 函数内部对参数的修改，实际上是在修改外部变量本身
4. 函数执行完毕后，外部变量的值已经被修改

#### 使用场景

引用传参适用于以下场景：
- **需要修改外部变量**：如交换两个变量的值、递增计数器等
- **避免大数组复制**：传递大数组时，引用传参可以避免复制整个数组，提高性能
- **返回多个值**：通过引用参数可以"返回"多个值（虽然 PHP 也支持返回数组）
- **与某些内置函数配合**：如 `sort()`、`array_walk()` 等函数需要引用传参

#### 注意事项

- **副作用**：引用传参会修改外部变量，可能导致意外的副作用，使用需谨慎
- **可读性**：代码阅读者可能不容易发现函数会修改外部变量
- **调试困难**：引用传参可能导致调试时难以追踪变量的变化
- **性能考虑**：对于小数据类型（如整数、字符串），引用传参的性能提升微乎其微，甚至可能更慢

### 基本语法

在函数定义时，在参数名前添加 `&` 符号：

```php
function functionName(type &$parameter): returnType
{
    // 修改 $parameter 会影响外部变量
}
```

**语法说明**：
- `&` 符号必须紧跟在类型声明之后、参数名之前
- 调用函数时，传递的变量会自动建立引用关系
- 函数内部对 `$parameter` 的任何修改都会反映到外部变量

### 基本用法

**示例：递增函数**

```php
<?php
declare(strict_types=1);

function increment(int &$value): void
{
    $value++;
}

$num = 5;
increment($num);
echo $num . "\n";  // 6（原值被修改）
```

**执行过程分析**：
1. 定义变量 `$num = 5`
2. 调用 `increment($num)` 时，`$value` 成为 `$num` 的引用（别名）
3. 函数内部执行 `$value++`，实际上是在修改 `$num`
4. 函数返回后，`$num` 的值已经变为 6

**对比：值传参的情况**

```php
<?php
declare(strict_types=1);

// 值传参（没有 &）
function incrementByValue(int $value): void
{
    $value++;  // 只修改副本，不影响外部变量
}

$num = 5;
incrementByValue($num);
echo $num . "\n";  // 仍然是 5（原值未被修改）
```

### 交换变量

引用传参的典型应用场景是交换两个变量的值：

```php
<?php
declare(strict_types=1);

function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

$x = 10;
$y = 20;
swap($x, $y);
echo "x = {$x}, y = {$y}\n";  // x = 20, y = 10
```

**工作原理**：
1. `swap($x, $y)` 调用时，`$a` 成为 `$x` 的引用，`$b` 成为 `$y` 的引用
2. 函数内部交换 `$a` 和 `$b` 的值
3. 实际上是在交换 `$x` 和 `$y` 的值
4. 函数返回后，`$x` 和 `$y` 的值已经交换

**为什么需要引用传参**：
如果不使用引用传参，函数只能交换参数的副本，无法影响外部变量。使用引用传参可以直接修改外部变量，实现真正的交换。

### 引用返回

函数不仅可以接收引用参数，还可以**返回引用**。引用返回允许函数返回一个变量的引用，调用者可以通过这个引用直接修改原变量。

**语法**：在函数名前添加 `&` 符号，表示返回引用。

```php
function &functionName(): returnType
{
    // 返回变量的引用
}
```

**使用场景**：
- 需要返回数组或对象的某个元素，并允许直接修改
- 实现类似数组访问的语法（如 `$obj->getValue() = 10`）

**示例**：

```php
<?php
declare(strict_types=1);

$values = [10, 20, 30];

function &getValue(array &$arr, int $index): int
{
    return $arr[$index];
}

$ref =& getValue($values, 1);
$ref = 200;  // 直接修改 $values[1]

print_r($values);  // [10, 200, 30]
```

## 箭头函数（PHP 7.4+）

### 概念与特性

**箭头函数**（Arrow Functions）是 PHP 7.4 引入的语法糖，提供了一种更简洁的方式来定义匿名函数。箭头函数特别适合用于简单的回调函数和单行表达式。

#### 主要特性

1. **简洁语法**：比传统匿名函数更简洁，适合单行表达式
2. **自动变量捕获**：自动按值捕获外部作用域中的变量，无需显式使用 `use` 子句
3. **单表达式**：只能包含一个表达式，不能包含多条语句
4. **性能优化**：相比传统闭包，箭头函数有轻微的性能优势

#### 适用场景

- **数组操作**：与 `array_map()`、`array_filter()` 等函数配合使用
- **简单回调**：作为回调函数传递给其他函数
- **单行逻辑**：只需要一行代码就能表达的逻辑

#### 限制

- **不能包含多条语句**：只能是一个表达式，不能使用 `{}` 包含多条语句
- **不能按引用捕获**：只能按值捕获变量，不能使用 `&` 按引用捕获
- **不能修改外部变量**：捕获的变量是副本，修改不会影响外部变量

### 基本语法

```php
fn(parameters) => expression
```

**语法说明**：
- `fn` 是关键字，表示箭头函数
- `parameters` 是参数列表，支持类型声明
- `=>` 是箭头操作符
- `expression` 是单个表达式，其值会自动返回

### 基本用法

```php
<?php
declare(strict_types=1);

$double = fn(int $x): int => $x * 2;
echo $double(5) . "\n";  // 10

$add = fn(int $a, int $b): int => $a + $b;
echo $add(3, 4) . "\n";  // 7
```

### 与匿名函数对比

```php
<?php
declare(strict_types=1);

// 箭头函数
$arrow = fn(int $x): int => $x * 2;

// 匿名函数（等价）
$anonymous = function(int $x): int {
    return $x * 2;
};

echo $arrow(5) . "\n";      // 10
echo $anonymous(5) . "\n";  // 10
```

### 自动捕获变量

箭头函数**自动按值捕获**外部作用域中的变量，无需显式声明：

```php
<?php
declare(strict_types=1);

$multiplier = 3;
$arrow = fn(int $x): int => $x * $multiplier;  // 自动捕获 $multiplier

echo $arrow(5) . "\n";  // 15

$multiplier = 5;  // 修改外部变量不影响箭头函数
echo $arrow(5) . "\n";  // 仍然是 15
```

**工作原理**：
1. 箭头函数在定义时，会自动捕获外部作用域中的 `$multiplier` 变量
2. 捕获是按值进行的，即创建了 `$multiplier` 的副本
3. 后续修改外部的 `$multiplier` 不会影响箭头函数中已捕获的值
4. 箭头函数中使用的始终是捕获时的值（3）

**对比：传统闭包需要显式捕获**

```php
<?php
declare(strict_types=1);

$multiplier = 3;
// 传统闭包需要显式使用 use 子句
$closure = function(int $x) use ($multiplier): int {
    return $x * $multiplier;
};

echo $closure(5) . "\n";  // 15
```

## 闭包（Closure）

### 概念与特性

**闭包**（Closure）是一种能够访问外部作用域中变量的函数。在 PHP 中，闭包通常指匿名函数，它们可以"记住"定义时的环境（包括外部变量）。

#### 核心概念

**闭包的本质**：
- 闭包是一个函数，但它可以访问定义时所在作用域的变量
- 即使外部作用域已经结束，闭包仍然可以访问那些变量
- 闭包"封闭"了外部环境，形成了自己的作用域

#### 变量捕获机制

闭包通过 `use` 子句显式声明要捕获的外部变量：

- **按值捕获**：`use ($variable)` - 创建变量的副本，修改不影响外部变量
- **按引用捕获**：`use (&$variable)` - 直接引用外部变量，修改会影响外部变量

#### 使用场景

- **回调函数**：作为参数传递给其他函数
- **函数工厂**：创建具有不同行为的函数
- **事件处理**：在事件驱动的代码中使用
- **延迟执行**：将代码封装在闭包中，稍后执行

#### 与普通函数的区别

| 特性       | 普通函数           | 闭包               |
| :--------- | :----------------- | :----------------- |
| 变量访问   | 只能访问参数和全局变量 | 可以访问外部作用域变量 |
| 变量捕获   | 不支持             | 支持（通过 `use`） |
| 定义方式   | 函数声明或表达式   | 匿名函数表达式     |
| 命名       | 必须有名称         | 可以匿名           |

### 基本语法

```php
$closure = function(parameters) use ($variables) {
    // 函数体
};
```

**语法说明**：
- `function(parameters)` 定义匿名函数
- `use ($variables)` 声明要捕获的外部变量
- 可以捕获多个变量：`use ($var1, $var2, &$var3)`
- 使用 `&` 可以按引用捕获变量

### 基本用法

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$greet = function() use ($name): string {
    return "Hello, {$name}!";
};

echo $greet() . "\n";  // Hello, Alice!
```

### 按值捕获

**按值捕获**是闭包的默认捕获方式。捕获的变量是外部变量的**副本**，闭包内部对变量的修改不会影响外部变量。

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use ($count): void {
    $count++;  // 修改的是副本，不影响外部变量
};

$increment();
echo $count . "\n";  // 仍然是 0
```

**工作原理**：
1. 闭包定义时，通过 `use ($count)` 捕获外部变量 `$count`
2. 捕获是按值进行的，创建了 `$count` 的副本（值为 0）
3. 闭包内部修改的是副本，外部的 `$count` 不受影响
4. 即使多次调用闭包，外部的 `$count` 仍然是 0

**适用场景**：
- 需要"冻结"某个时刻的变量值
- 不希望闭包修改外部变量
- 需要多个闭包各自维护独立的变量副本

### 按引用捕获

使用 `&` 可以**按引用捕获**变量。这样闭包内部对变量的修改会直接影响外部变量。

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use (&$count): void {
    $count++;  // 修改外部变量
};

$increment();
echo $count . "\n";  // 1
```

**工作原理**：
1. 闭包定义时，通过 `use (&$count)` 按引用捕获外部变量 `$count`
2. 闭包内部的 `$count` 和外部变量指向同一个内存位置
3. 闭包内部修改 `$count`，实际上是在修改外部变量
4. 每次调用闭包，外部的 `$count` 都会递增

**适用场景**：
- 需要闭包修改外部变量（如计数器、累加器）
- 需要在多个闭包之间共享状态
- 需要实现状态持久化

### 捕获多个变量

闭包可以同时捕获多个外部变量，可以混合使用按值捕获和按引用捕获：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age = 25;
$city = 'Beijing';

$info = function() use ($name, $age, $city): string {
    return "{$name}, {$age}, {$city}";
};

echo $info() . "\n";  // Alice, 25, Beijing
```

**混合捕获示例**：

```php
<?php
declare(strict_types=1);

$name = 'Alice';      // 按值捕获
$age = 25;            // 按值捕获
$counter = 0;         // 按引用捕获

$info = function() use ($name, $age, &$counter): string {
    $counter++;  // 修改外部变量
    return "{$name}, {$age}, count: {$counter}";
};

echo $info() . "\n";  // Alice, 25, count: 1
echo $info() . "\n";  // Alice, 25, count: 2
echo $counter . "\n"; // 2（外部变量被修改）
```

## 闭包作为返回值

闭包可以作为函数的返回值，这种模式称为**函数工厂**（Function Factory）。函数工厂可以根据不同的参数创建具有不同行为的函数。

```php
<?php
declare(strict_types=1);

function createMultiplier(int $factor): Closure
{
    return function(int $x) use ($factor): int {
        return $x * $factor;
    };
}

$double = createMultiplier(2);
$triple = createMultiplier(3);

echo $double(5) . "\n";  // 10
echo $triple(5) . "\n";  // 15
```

**工作原理**：
1. `createMultiplier(2)` 调用时，参数 `$factor = 2` 被闭包捕获
2. 返回的闭包"记住"了 `$factor = 2` 这个值
3. 调用 `$double(5)` 时，闭包使用捕获的 `$factor` 值（2）进行计算
4. 每个闭包都有自己独立的 `$factor` 值

**应用场景**：
- **配置函数**：根据配置创建不同行为的函数
- **缓存函数**：创建带有缓存功能的函数
- **装饰器模式**：为函数添加额外功能

## 闭包作为参数

闭包可以作为参数传递给其他函数，实现**高阶函数**（Higher-Order Function）模式。高阶函数是函数式编程的重要概念。

```php
<?php
declare(strict_types=1);

function processNumbers(array $numbers, callable $callback): array
{
    return array_map($callback, $numbers);
}

$numbers = [1, 2, 3, 4, 5];
$multiplier = 2;

$result = processNumbers($numbers, function($n) use ($multiplier) {
    return $n * $multiplier;
});

print_r($result);  // [2, 4, 6, 8, 10]
```

**工作原理**：
1. `processNumbers()` 是一个高阶函数，接收数组和回调函数作为参数
2. 闭包 `function($n) use ($multiplier)` 捕获了外部变量 `$multiplier`
3. `array_map()` 对数组中的每个元素调用闭包
4. 闭包使用捕获的 `$multiplier` 值处理每个元素

**高阶函数的优势**：
- **灵活性**：可以传入不同的处理逻辑，无需修改函数本身
- **可复用性**：同一个函数可以处理不同的数据和处理方式
- **函数式编程**：支持函数式编程范式，代码更简洁

## 闭包高级特性

### 闭包与类

闭包可以在类的方法中定义，并访问类的属性和方法。

#### 访问类属性

```php
<?php
declare(strict_types=1);

class Calculator
{
    private int $factor = 2;
    
    public function getMultiplier(): Closure
    {
        return function(int $x): int {
            return $x * $this->factor;  // 访问 $this
        };
    }
}

$calc = new Calculator();
$multiplier = $calc->getMultiplier();
echo $multiplier(5) . "\n";  // 10
```

**工作原理**：
- 在类方法中定义的闭包可以自动访问 `$this`
- 闭包可以访问类的私有、受保护和公共属性
- 闭包可以调用类的私有、受保护和公共方法

#### 静态方法中的闭包

```php
<?php
declare(strict_types=1);

class Math
{
    private static int $multiplier = 3;
    
    public static function getMultiplier(): Closure
    {
        return function(int $x): int {
            return $x * self::$multiplier;  // 访问静态属性
        };
    }
}

$multiplier = Math::getMultiplier();
echo $multiplier(5) . "\n";  // 15
```

### Closure::bindTo() - 绑定作用域

`bindTo()` 方法可以将闭包绑定到不同的对象和作用域，这对于动态改变闭包的行为非常有用。

**语法**：`Closure::bindTo(?object $newThis, ?string $newScope = "static"): ?Closure`

**参数**：
- `$newThis`：可选，要绑定到的对象，如果为 `null` 则解绑
- `$newScope`：可选，类作用域（类名或 `"static"`），默认为 `"static"`

**返回值**：成功返回新的闭包对象，失败返回 `null`。

**基本用法**：

```php
<?php
declare(strict_types=1);

class User
{
    private string $name = 'Alice';
}

$closure = function(): string {
    return $this->name;
};

$user = new User();
$bound = $closure->bindTo($user, User::class);
echo $bound() . "\n";  // Alice
```

**使用场景**：
- 动态绑定闭包到不同的对象
- 改变闭包的作用域，访问私有或受保护的成员
- 实现依赖注入和回调机制

### 变量生命周期

闭包会延长被捕获变量的生命周期。即使外部作用域已经结束，闭包仍然可以访问那些变量。

```php
<?php
declare(strict_types=1);

function createClosure(): Closure
{
    $local = 'This variable will persist';
    
    return function() use ($local): string {
        return $local;
    };
}

$closure = createClosure();
// $local 在 createClosure() 执行完后仍然存在
echo $closure() . "\n";  // This variable will persist
```

**工作原理**：
1. 函数 `createClosure()` 执行时，创建局部变量 `$local`
2. 闭包通过 `use ($local)` 捕获了这个变量
3. 函数返回后，`$local` 通常应该被销毁
4. 但由于闭包引用了它，`$local` 的生命周期被延长
5. 只要闭包存在，`$local` 就不会被销毁

**注意事项**：
- 闭包会持有变量的引用，可能导致内存泄漏
- 如果不再需要闭包，应该将其设置为 `null` 以释放内存
- 对于大对象，考虑只捕获需要的部分

### 与 JavaScript 对比

如果你熟悉 JavaScript，理解 PHP 闭包的关键差异很重要：

#### JavaScript 闭包

```javascript
// JavaScript
let count = 0;
const increment = () => {
    count++;  // 自动捕获外部变量
    return count;
};
```

#### PHP 闭包

```php
<?php
// PHP
$count = 0;
$increment = function() use (&$count): int {
    $count++;  // 需要显式使用 use
    return $count;
};
```

#### 关键差异

1. **变量捕获**：
   - JavaScript：闭包自动捕获外部变量
   - PHP：必须使用 `use` 关键字显式捕获

2. **修改外部变量**：
   - JavaScript：箭头函数可以修改外部变量（如果变量不是 const）
   - PHP：箭头函数按值捕获，不能修改；需要使用 `use (&$var)` 按引用捕获

3. **语法**：
   - JavaScript：`(arg) => expression` 或 `function() { ... }`
   - PHP：`fn($arg) => expression` 或 `function() use ($var) { ... }`

## 箭头函数 vs 闭包

| 特性       | 箭头函数           | 闭包               |
| :--------- | :----------------- | :----------------- |
| 语法       | `fn() => expr`     | `function() {}`    |
| 变量捕获   | 自动按值捕获       | 需要 `use` 子句    |
| 按引用捕获 | 不支持             | 支持（`use (&$var)`） |
| 返回值     | 单表达式           | 可以有多条语句     |
| 性能       | 稍快               | 稍慢               |

### 示例对比

```php
<?php
declare(strict_types=1);

$multiplier = 2;
$numbers = [1, 2, 3, 4, 5];

// 箭头函数（自动捕获）
$arrow = array_map(fn($n) => $n * $multiplier, $numbers);

// 闭包（需要 use）
$closure = array_map(function($n) use ($multiplier) {
    return $n * $multiplier;
}, $numbers);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ReferencesClosures
{
    public static function demonstrateReferences(): void
    {
        echo "=== 引用传参 ===\n";
        $num = 5;
        increment($num);
        echo "After increment: {$num}\n";  // 6
        
        $x = 10;
        $y = 20;
        swap($x, $y);
        echo "After swap: x = {$x}, y = {$y}\n";  // x = 20, y = 10
    }
    
    public static function demonstrateArrowFunctions(): void
    {
        echo "\n=== 箭头函数 ===\n";
        $multiplier = 3;
        $arrow = fn(int $x): int => $x * $multiplier;
        echo $arrow(5) . "\n";  // 15
    }
    
    public static function demonstrateClosures(): void
    {
        echo "\n=== 闭包 ===\n";
        $count = 0;
        $increment = function() use (&$count): void {
            $count++;
        };
        $increment();
        echo "Count: {$count}\n";  // 1
        
        $multiplier = createMultiplier(5);
        echo $multiplier(3) . "\n";  // 15
    }
}

function increment(int &$value): void
{
    $value++;
}

function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

function createMultiplier(int $factor): Closure
{
    return function(int $x) use ($factor): int {
        return $x * $factor;
    };
}

ReferencesClosures::demonstrateReferences();
ReferencesClosures::demonstrateArrowFunctions();
ReferencesClosures::demonstrateClosures();
```

## 注意事项

1. **引用传参**：只在确实需要修改外部变量时使用引用，避免副作用。

2. **箭头函数限制**：箭头函数只能包含单个表达式，不能包含多条语句。

3. **变量捕获**：箭头函数自动按值捕获，闭包需要显式使用 `use` 子句。

4. **按引用捕获**：只有闭包支持按引用捕获，箭头函数不支持。

5. **性能考虑**：箭头函数性能略好于闭包，但差异很小。

6. **变量生命周期**：闭包会延长被捕获变量的生命周期，注意内存使用。

7. **作用域绑定**：使用 `bindTo()` 可以改变闭包的作用域，但要谨慎使用。

## 练习

1. 创建一个函数，使用引用传参实现数组元素的批量修改。

2. 编写一个函数，使用箭头函数实现数组的高阶操作。

3. 实现一个函数工厂，使用闭包创建具有不同行为的函数。

4. 创建一个函数，演示箭头函数和闭包在变量捕获上的差异。

5. 编写一个函数，使用闭包实现计数器功能。

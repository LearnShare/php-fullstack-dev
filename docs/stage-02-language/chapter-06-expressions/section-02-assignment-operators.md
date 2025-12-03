# 2.6.2 赋值运算符

## 概述

赋值运算符用于将值赋给变量。PHP 提供了基本的赋值运算符和复合赋值运算符，可以简化代码并提高可读性。

## 基本赋值运算符（=）

### 语法

```php
$variable = value;
```

### 基本用法

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age = 25;
$price = 99.99;
$isActive = true;
```

### 链式赋值

```php
<?php
declare(strict_types=1);

$a = $b = $c = 10;  // 三个变量都赋值为 10
echo "a = {$a}, b = {$b}, c = {$c}\n";  // a = 10, b = 10, c = 10
```

## 复合赋值运算符

复合赋值运算符将运算和赋值合并为一个操作。

| 运算符 | 等价于        | 说明           |
| :----- | :------------ | :------------- |
| `+=`   | `$a = $a + $b` | 加法赋值       |
| `-=`   | `$a = $a - $b` | 减法赋值       |
| `*=`   | `$a = $a * $b` | 乘法赋值       |
| `/=`   | `$a = $a / $b` | 除法赋值       |
| `%=`   | `$a = $a % $b` | 取模赋值       |
| `**=`  | `$a = $a ** $b` | 指数赋值       |
| `.=`   | `$a = $a . $b` | 字符串连接赋值 |

### 算术复合赋值

```php
<?php
declare(strict_types=1);

$count = 10;
$count += 5;   // 等价于 $count = $count + 5
echo $count . "\n";  // 15

$count -= 3;   // 等价于 $count = $count - 3
echo $count . "\n";  // 12

$count *= 2;   // 等价于 $count = $count * 2
echo $count . "\n";  // 24

$count /= 4;   // 等价于 $count = $count / 4
echo $count . "\n";  // 6.0

$count %= 4;   // 等价于 $count = $count % 4
echo $count . "\n";  // 2.0（注意：浮点数取模）
```

### 指数赋值

```php
<?php
declare(strict_types=1);

$base = 2;
$base **= 3;   // 等价于 $base = $base ** 3
echo $base . "\n";  // 8
```

### 字符串连接赋值

```php
<?php
declare(strict_types=1);

$name = 'PHP';
$name .= ' Guide';  // 等价于 $name = $name . ' Guide'
echo $name . "\n";  // PHP Guide

$message = 'Hello';
$message .= ', ';
$message .= 'World';
$message .= '!';
echo $message . "\n";  // Hello, World!
```

## 自增和自减运算符

### 后置自增（$i++）

先使用变量的值，然后将其加 1。

```php
<?php
declare(strict_types=1);

$i = 5;
echo $i++ . "\n";  // 输出 5，然后 $i 变为 6
echo $i . "\n";    // 输出 6
```

### 前置自增（++$i）

先将变量加 1，然后使用新值。

```php
<?php
declare(strict_types=1);

$i = 5;
echo ++$i . "\n";  // 先将 $i 加 1 变为 6，然后输出 6
echo $i . "\n";    // 输出 6
```

### 后置自减（$i--）

先使用变量的值，然后将其减 1。

```php
<?php
declare(strict_types=1);

$i = 5;
echo $i-- . "\n";  // 输出 5，然后 $i 变为 4
echo $i . "\n";    // 输出 4
```

### 前置自减（--$i）

先将变量减 1，然后使用新值。

```php
<?php
declare(strict_types=1);

$i = 5;
echo --$i . "\n";  // 先将 $i 减 1 变为 4，然后输出 4
echo $i . "\n";    // 输出 4
```

### 在循环中使用

```php
<?php
declare(strict_types=1);

// 使用后置自增
for ($i = 0; $i < 5; $i++) {
    echo $i . " ";
}
echo "\n";  // 输出：0 1 2 3 4

// 使用前置自增（效果相同）
for ($i = 0; $i < 5; ++$i) {
    echo $i . " ";
}
echo "\n";  // 输出：0 1 2 3 4
```

## 数组赋值

### 数组元素赋值

```php
<?php
declare(strict_types=1);

$arr = [];
$arr[] = 'first';        // 自动索引
$arr[] = 'second';
$arr['key'] = 'value';   // 指定键

print_r($arr);
```

### 数组解构赋值（PHP 7.1+）

```php
<?php
declare(strict_types=1);

// 列表解构
[$a, $b, $c] = [1, 2, 3];
echo "a = {$a}, b = {$b}, c = {$c}\n";  // a = 1, b = 2, c = 3

// 关联数组解构
['name' => $name, 'age' => $age] = ['name' => 'Alice', 'age' => 25];
echo "Name: {$name}, Age: {$age}\n";  // Name: Alice, Age: 25

// 跳过元素
[, $b, , $d] = [1, 2, 3, 4];
echo "b = {$b}, d = {$d}\n";  // b = 2, d = 4

// 使用 list()（旧语法，PHP 7.1+ 推荐使用方括号）
list($x, $y) = [10, 20];
echo "x = {$x}, y = {$y}\n";  // x = 10, y = 20
```

### 交换变量值

```php
<?php
declare(strict_types=1);

$a = 10;
$b = 20;

// 使用临时变量
$temp = $a;
$a = $b;
$b = $temp;
echo "a = {$a}, b = {$b}\n";  // a = 20, b = 10

// 使用数组解构（PHP 7.1+）
[$a, $b] = [$b, $a];
echo "a = {$a}, b = {$b}\n";  // a = 10, b = 20
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Counter
{
    private int $count = 0;
    
    public function increment(): int
    {
        return ++$this->count;  // 前置自增，返回新值
    }
    
    public function decrement(): int
    {
        return --$this->count;  // 前置自减，返回新值
    }
    
    public function getCount(): int
    {
        return $this->count;
    }
    
    public function add(int $value): void
    {
        $this->count += $value;
    }
    
    public function multiply(int $factor): void
    {
        $this->count *= $factor;
    }
}

$counter = new Counter();
echo "Initial: " . $counter->getCount() . "\n";  // 0

$counter->increment();
echo "After increment: " . $counter->getCount() . "\n";  // 1

$counter->add(5);
echo "After add 5: " . $counter->getCount() . "\n";  // 6

$counter->multiply(2);
echo "After multiply 2: " . $counter->getCount() . "\n";  // 12
```

## 注意事项

1. **赋值返回值**：赋值运算符返回被赋的值，可以用于链式赋值。

2. **自增自减**：前置和后置自增/自减在单独使用时效果相同，但在表达式中使用时行为不同。

3. **数组解构**：PHP 7.1+ 推荐使用方括号语法，`list()` 是旧语法。

4. **类型转换**：赋值时可能发生隐式类型转换，注意类型安全。

5. **引用赋值**：使用 `&` 进行引用赋值，两个变量指向同一个值。

## 练习

1. 创建一个 `Accumulator` 类，使用复合赋值运算符实现累加功能。

2. 编写一个函数，使用数组解构交换两个变量的值。

3. 实现一个计数器类，使用自增和自减运算符。

4. 创建一个函数，使用复合赋值运算符计算数组元素的总和。

5. 编写一个函数，演示前置和后置自增/自减的区别。


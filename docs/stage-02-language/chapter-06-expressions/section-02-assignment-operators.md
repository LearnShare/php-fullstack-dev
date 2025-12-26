# 2.6.2 赋值运算符

## 概述

赋值运算符用于给变量赋值。本节详细介绍基本赋值、复合赋值、自增自减、数组解构赋值（PHP 7.1+）的语法、用法、优先级及使用场景。

理解赋值运算符的行为和优先级对于编写正确的表达式很重要。自增自减运算符的前置和后置区别、数组解构赋值等特性可以提高代码的简洁性和可读性。

## 特性

- **基本赋值**：将值赋给变量
- **复合赋值**：先运算再赋值，简化代码
- **自增自减**：变量值加1或减1，前置和后置有区别
- **数组解构**：从数组提取值赋给变量（PHP 7.1+）

## 语法/定义

### 基本赋值运算符（=）

**语法**：`$variable = value;`

**功能**：将值赋给变量

**特点**：
- 是赋值运算符，不是相等比较
- 返回赋值的值
- 支持链式赋值

### 复合赋值运算符

| 运算符 | 等价于 | 说明 | 示例 |
|:-------|:-------|:-----|:-----|
| `+=` | `$a = $a + $b` | 加法赋值 | `$a += 5` |
| `-=` | `$a = $a - $b` | 减法赋值 | `$a -= 5` |
| `*=` | `$a = $a * $b` | 乘法赋值 | `$a *= 5` |
| `/=` | `$a = $a / $b` | 除法赋值 | `$a /= 5` |
| `%=` | `$a = $a % $b` | 取模赋值 | `$a %= 5` |
| `**=` | `$a = $a ** $b` | 指数赋值 | `$a **= 5` |
| `.=` | `$a = $a . $b` | 字符串连接赋值 | `$a .= "text"` |

**特点**：
- 先进行运算，再赋值
- 简化代码，提高可读性
- 性能略好（避免重复读取变量）

### 自增自减运算符

**前置自增**：`++$a`
- 先加1，再返回新值
- 返回加1后的值

**后置自增**：`$a++`
- 先返回原值，再加1
- 返回加1前的值

**前置自减**：`--$a`
- 先减1，再返回新值
- 返回减1后的值

**后置自减**：`$a--`
- 先返回原值，再减1
- 返回减1前的值

### 数组解构赋值（PHP 7.1+）

**索引数组解构**：`[$var1, $var2, $var3] = $array;`

**关联数组解构**：`['key1' => $var1, 'key2' => $var2] = $array;`

**特点**：
- 从数组提取值赋给变量
- 支持跳过元素（使用空位）
- 支持默认值（PHP 7.1+）

## 基本用法

### 示例 1：基本赋值

```php
<?php
declare(strict_types=1);

// 基本赋值
$a = 10;
$b = $a;
echo "a: {$a}, b: {$b}\n";  // a: 10, b: 10

// 链式赋值
$a = $b = $c = 10;
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 10, b: 10, c: 10

// 表达式赋值
$a = 10 + 5;
echo "a: {$a}\n";  // a: 15

// 函数返回值赋值
function getValue(): int
{
    return 42;
}

$a = getValue();
echo "a: {$a}\n";  // a: 42
```

### 示例 2：复合赋值

```php
<?php
declare(strict_types=1);

$a = 10;

// 加法赋值
$a += 5;
echo "After += 5: {$a}\n";  // After += 5: 15

// 减法赋值
$a -= 3;
echo "After -= 3: {$a}\n";  // After -= 3: 12

// 乘法赋值
$a *= 2;
echo "After *= 2: {$a}\n";  // After *= 2: 24

// 除法赋值
$a /= 4;
echo "After /= 4: {$a}\n";  // After /= 4: 6

// 取模赋值
$a %= 4;
echo "After %= 4: {$a}\n";  // After %= 4: 2

// 指数赋值
$a **= 3;
echo "After **= 3: {$a}\n";  // After **= 3: 8

// 字符串连接赋值
$str = "Hello";
$str .= " World";
echo "String: {$str}\n";  // String: Hello World
```

### 示例 3：自增自减

```php
<?php
declare(strict_types=1);

// 前置自增
$a = 10;
$b = ++$a;
echo "a: {$a}, b: {$b}\n";  // a: 11, b: 11

// 后置自增
$a = 10;
$b = $a++;
echo "a: {$a}, b: {$b}\n";  // a: 11, b: 10

// 前置自减
$a = 10;
$b = --$a;
echo "a: {$a}, b: {$b}\n";  // a: 9, b: 9

// 后置自减
$a = 10;
$b = $a--;
echo "a: {$a}, b: {$b}\n";  // a: 9, b: 10

// 在表达式中使用
$a = 10;
$result = $a++ + ++$a;
echo "a: {$a}, result: {$result}\n";  // a: 12, result: 22
```

### 示例 4：数组解构赋值

```php
<?php
declare(strict_types=1);

// 索引数组解构
[$a, $b, $c] = [1, 2, 3];
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 1, b: 2, c: 3

// 跳过元素
[$a, , $c] = [1, 2, 3];
echo "a: {$a}, c: {$c}\n";  // a: 1, c: 3

// 关联数组解构
['name' => $name, 'age' => $age] = ['name' => 'Alice', 'age' => 25];
echo "Name: {$name}, Age: {$age}\n";  // Name: Alice, Age: 25

// 默认值（PHP 7.1+）
[$a, $b = 10, $c] = [1, , 3];
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 1, b: 10, c: 3

// 交换变量值
$a = 10;
$b = 20;
[$a, $b] = [$b, $a];
echo "a: {$a}, b: {$b}\n";  // a: 20, b: 10
```

## 完整代码示例

### 示例 1：计数器实现

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
    
    public function reset(): void
    {
        $this->count = 0;
    }
}

$counter = new Counter();
echo "Initial: {$counter->getCount()}\n";  // Initial: 0
echo "After increment: {$counter->increment()}\n";  // After increment: 1
echo "After increment: {$counter->increment()}\n";  // After increment: 2
echo "After decrement: {$counter->decrement()}\n";   // After decrement: 1
```

### 示例 2：数组处理

```php
<?php
declare(strict_types=1);

// 使用数组解构处理函数返回值
function getUserInfo(): array
{
    return ['name' => 'Alice', 'age' => 25, 'email' => 'alice@example.com'];
}

['name' => $name, 'age' => $age, 'email' => $email] = getUserInfo();
echo "User: {$name}, Age: {$age}, Email: {$email}\n";

// 处理多个返回值
function divideWithRemainder(int $a, int $b): array
{
    return [
        'quotient' => intdiv($a, $b),
        'remainder' => $a % $b,
    ];
}

['quotient' => $q, 'remainder' => $r] = divideWithRemainder(10, 3);
echo "Quotient: {$q}, Remainder: {$r}\n";  // Quotient: 3, Remainder: 1
```

### 示例 3：复合赋值应用

```php
<?php
declare(strict_types=1);

// 累加器
function accumulate(array $numbers): int
{
    $sum = 0;
    foreach ($numbers as $num) {
        $sum += $num;  // 使用复合赋值
    }
    return $sum;
}

// 字符串构建
function buildMessage(array $parts): string
{
    $message = "";
    foreach ($parts as $part) {
        $message .= $part . " ";  // 使用字符串连接赋值
    }
    return trim($message);
}

// 使用
$numbers = [1, 2, 3, 4, 5];
echo "Sum: " . accumulate($numbers) . "\n";  // Sum: 15

$parts = ["Hello", "World", "from", "PHP"];
echo "Message: " . buildMessage($parts) . "\n";  // Message: Hello World from PHP
```

## 使用场景

### 基本赋值

- **变量初始化**：初始化变量
- **值传递**：将值赋给变量
- **链式赋值**：同时给多个变量赋值

### 复合赋值

- **累加器**：累加数值
- **字符串构建**：构建字符串
- **计数器**：更新计数器值
- **简化代码**：简化重复的赋值表达式

### 自增自减

- **循环计数器**：在循环中使用
- **数组索引**：访问数组元素
- **计数器**：实现计数器功能

### 数组解构

- **函数返回值**：解构函数返回的数组
- **交换变量**：交换两个变量的值
- **配置提取**：从配置数组提取值
- **简化代码**：简化数组访问代码

## 注意事项

### 赋值运算符优先级

- **优先级较低**：赋值运算符优先级较低
- **右结合**：赋值运算符是右结合的
- **使用括号**：不确定时使用括号明确优先级

### 前置和后置区别

- **前置**：先运算，再返回新值
- **后置**：先返回原值，再运算
- **在表达式中**：在表达式中使用时要注意区别

### 数组解构

- **结构匹配**：确保数组结构与解构模式匹配
- **默认值**：可以为缺失的元素提供默认值
- **跳过元素**：可以使用空位跳过元素

### 类型安全

- **类型声明**：为变量添加类型声明
- **验证赋值**：验证赋值的值是否符合类型
- **严格模式**：使用严格模式提高类型安全

## 常见问题

### 问题 1：赋值运算符优先级

**症状**：表达式结果不符合预期

**原因**：不理解赋值运算符的优先级

**错误示例**：

```php
<?php
declare(strict_types=1);

$a = 10;
$b = 20;
$result = $a = $b + 5;  // $result 是 25，不是布尔值
echo "Result: {$result}\n";  // Result: 25
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 理解赋值运算符的优先级
$a = 10;
$b = 20;
$result = ($a = $b) + 5;  // 明确优先级
echo "Result: {$result}\n";  // Result: 25

// 或分开写
$a = $b;
$result = $a + 5;
```

### 问题 2：前置和后置混淆

**症状**：自增自减结果不符合预期

**原因**：不理解前置和后置的区别

**错误示例**：

```php
<?php
declare(strict_types=1);

$a = 10;
$b = $a++;  // 期望 $b 是 11，实际是 10
echo "a: {$a}, b: {$b}\n";  // a: 11, b: 10
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 前置自增：先加1，再返回
$a = 10;
$b = ++$a;
echo "a: {$a}, b: {$b}\n";  // a: 11, b: 11

// 后置自增：先返回，再加1
$a = 10;
$b = $a++;
echo "a: {$a}, b: {$b}\n";  // a: 11, b: 10
```

### 问题 3：数组解构不匹配

**症状**：数组解构失败或结果不符合预期

**原因**：数组结构与解构模式不匹配

**错误示例**：

```php
<?php
declare(strict_types=1);

// 数组元素不足
[$a, $b, $c] = [1, 2];  // $c 是 null
echo "c: ";
var_dump($c);  // NULL
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：提供默认值
[$a, $b, $c = 0] = [1, 2];
echo "c: {$c}\n";  // c: 0

// 方法2：检查数组长度
$arr = [1, 2];
if (count($arr) >= 3) {
    [$a, $b, $c] = $arr;
} else {
    [$a, $b] = $arr;
    $c = 0;
}
```

## 最佳实践

### 赋值运算符

- **使用复合赋值**：简化代码，提高可读性
- **理解优先级**：理解赋值运算符的优先级
- **使用括号**：不确定时使用括号明确优先级

### 自增自减

- **理解区别**：理解前置和后置的区别
- **在循环中使用**：在循环中使用自增自减
- **避免复杂表达式**：避免在复杂表达式中使用

### 数组解构

- **使用解构**：使用数组解构简化代码
- **提供默认值**：为可能缺失的元素提供默认值
- **匹配结构**：确保数组结构与解构模式匹配

### 代码可读性

- **明确意图**：使用有意义的变量名
- **避免复杂表达式**：避免过于复杂的赋值表达式
- **添加注释**：为复杂的赋值逻辑添加注释

## 对比分析

### 前置 vs 后置

| 特性 | 前置（++$a） | 后置（$a++） |
|:-----|:------------|:-------------|
| 执行顺序 | 先运算，后返回 | 先返回，后运算 |
| 返回值 | 新值 | 原值 |
| 适用场景 | 需要新值 | 需要原值 |
| 推荐度 | 按需使用 | 按需使用 |

**选择建议**：
- **需要新值**：使用前置
- **需要原值**：使用后置
- **简单递增**：两者都可以

### 复合赋值 vs 普通赋值

| 特性 | 复合赋值 | 普通赋值 |
|:-----|:---------|:---------|
| 代码简洁 | 是 | 否 |
| 性能 | 略好 | 略差 |
| 可读性 | 高 | 中 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **累加、累乘等**：使用复合赋值
- **复杂表达式**：可以使用普通赋值提高可读性

## 相关章节

- **2.6.1 算术运算符**：了解算术运算符
- **2.6.6 运算符优先级与结合性**：了解运算符优先级
- **2.8 数组完整指南**：详细了解数组操作

## 练习任务

1. **基本赋值练习**：
   - 练习基本赋值和链式赋值
   - 理解赋值运算符的返回值
   - 测试不同类型值的赋值
   - 观察赋值行为

2. **复合赋值练习**：
   - 练习使用各种复合赋值运算符
   - 实现累加器和字符串构建
   - 理解复合赋值的等价性
   - 测试性能差异

3. **自增自减练习**：
   - 练习前置和后置自增自减
   - 理解执行顺序的区别
   - 在循环中使用自增自减
   - 测试不同场景下的行为

4. **数组解构练习**：
   - 练习索引数组和关联数组解构
   - 使用默认值和跳过元素
   - 解构函数返回值
   - 实现变量交换

5. **综合练习**：
   - 创建一个使用各种赋值运算符的程序
   - 实现计数器、累加器等功能
   - 使用数组解构简化代码
   - 进行代码审查，确保赋值正确

# 2.6.6 运算符优先级与结合性

## 概述

运算符优先级和结合性决定了表达式的求值顺序。本节详细介绍运算符优先级表、结合性（左结合、右结合）、常见优先级问题、使用括号的最佳实践。

理解运算符优先级和结合性对于编写正确的表达式至关重要。错误的优先级理解可能导致意外的结果。掌握优先级规则可以帮助编写更清晰、更可靠的代码。

## 特性

- **优先级**：不同运算符有不同的优先级
- **结合性**：相同优先级的运算符有结合性（左结合或右结合）
- **括号**：使用括号可以明确优先级
- **可读性**：理解优先级可以提高代码可读性

## 语法/定义

### 运算符优先级

**概念**：运算符优先级决定了表达式中运算符的执行顺序。优先级高的运算符先执行。

**优先级表**（从高到低）：

| 优先级 | 运算符 | 说明 |
|:-------|:------|:-----|
| 1 | `new` | 创建对象 |
| 2 | `**` | 指数 |
| 3 | `++` `--` `~` `(int)` `(float)` `(string)` `(array)` `(object)` `(bool)` `@` | 自增自减、类型转换、错误抑制 |
| 4 | `!` | 逻辑非 |
| 5 | `*` `/` `%` | 乘法、除法、取模 |
| 6 | `+` `-` `.` | 加法、减法、字符串连接 |
| 7 | `<<` `>>` | 位左移、位右移 |
| 8 | `<` `<=` `>` `>=` | 比较运算符 |
| 9 | `==` `!=` `===` `!==` `<>` `<=>` | 相等比较 |
| 10 | `&` | 位与、引用 |
| 11 | `^` | 位异或 |
| 12 | `\|` | 位或 |
| 13 | `&&` | 逻辑与 |
| 14 | `\|\|` | 逻辑或 |
| 15 | `??` | 空合并 |
| 16 | `?:` | 三元运算符 |
| 17 | `=` `+=` `-=` `*=` `/=` `%=` `**=` `.=` `<<=` `>>=` `&=` `\|=` `^=` `??=` | 赋值运算符 |
| 18 | `and` | 逻辑与（低优先级） |
| 19 | `xor` | 逻辑异或 |
| 20 | `or` | 逻辑或（低优先级） |

**注意**：这是简化的优先级表，完整表请参考 PHP 官方文档。

### 结合性

**概念**：结合性决定了相同优先级的运算符的执行顺序。

**左结合**：从左到右求值（大多数运算符）
- 示例：`$a + $b + $c` 等价于 `($a + $b) + $c`

**右结合**：从右到左求值（赋值运算符、三元运算符等）
- 示例：`$a = $b = $c` 等价于 `$a = ($b = $c)`
- 示例：`$a ? $b : $c ? $d : $e` 等价于 `$a ? $b : ($c ? $d : $e)`

## 基本用法

### 示例 1：优先级示例

```php
<?php
declare(strict_types=1);

// 乘法优先级高于加法
$result = 2 + 3 * 4;
echo "2 + 3 * 4 = {$result}\n";  // 2 + 3 * 4 = 14（不是 20）

// 使用括号明确优先级
$result = (2 + 3) * 4;
echo "(2 + 3) * 4 = {$result}\n";  // (2 + 3) * 4 = 20

// 逻辑与优先级高于逻辑或
$a = true;
$b = false;
$c = true;
$result = $a && $b || $c;
echo "Result: " . ($result ? 'true' : 'false') . "\n";  // Result: true

// 使用括号明确优先级
$result = ($a && $b) || $c;
echo "Result: " . ($result ? 'true' : 'false') . "\n";  // Result: true
```

### 示例 2：结合性示例

```php
<?php
declare(strict_types=1);

// 赋值运算符右结合
$a = $b = $c = 10;
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 10, b: 10, c: 10

// 等价于
$c = 10;
$b = $c;
$a = $b;

// 三元运算符右结合
$a = true;
$b = false;
$c = true;
$d = false;
$e = "default";

$result = $a ? $b : $c ? $d : $e;
echo "Result: {$result}\n";  // Result: default

// 等价于
$result = $a ? $b : ($c ? $d : $e);
```

### 示例 3：常见优先级问题

```php
<?php
declare(strict_types=1);

// 问题1：逻辑运算符优先级
$a = true;
$b = false;
$c = true;

// 期望：($a && $b) || $c
$result = $a && $b || $c;  // 实际：$a && ($b || $c)
echo "Result: " . ($result ? 'true' : 'false') . "\n";  // Result: true

// 正确方式
$result = ($a && $b) || $c;
echo "Result: " . ($result ? 'true' : 'false') . "\n";  // Result: true

// 问题2：空合并运算符优先级
$a = null;
$b = "default";
$c = "fallback";

// 期望：($a ?? $b) ?: $c
$result = $a ?? $b ?: $c;  // 实际：$a ?? ($b ?: $c)
echo "Result: {$result}\n";  // Result: default

// 正确方式
$result = ($a ?? $b) ?: $c;
echo "Result: {$result}\n";  // Result: default
```

## 完整代码示例

### 示例 1：优先级检查工具

```php
<?php
declare(strict_types=1);

class PrecedenceChecker
{
    public static function demonstratePrecedence(): void
    {
        // 算术运算符
        echo "=== Arithmetic Operators ===\n";
        $result = 2 + 3 * 4;
        echo "2 + 3 * 4 = {$result}\n";  // 14
        
        $result = (2 + 3) * 4;
        echo "(2 + 3) * 4 = {$result}\n";  // 20
        
        // 逻辑运算符
        echo "\n=== Logical Operators ===\n";
        $a = true;
        $b = false;
        $c = true;
        
        $result = $a && $b || $c;
        echo "a && b || c = " . ($result ? 'true' : 'false') . "\n";  // true
        
        $result = ($a && $b) || $c;
        echo "(a && b) || c = " . ($result ? 'true' : 'false') . "\n";  // true
        
        // 赋值运算符
        echo "\n=== Assignment Operators ===\n";
        $a = $b = $c = 10;
        echo "a = b = c = 10: a={$a}, b={$b}, c={$c}\n";  // 都是 10
        
        // 空合并运算符
        echo "\n=== Null Coalescing Operator ===\n";
        $a = null;
        $b = "default";
        $c = "fallback";
        
        $result = $a ?? $b ?? $c;
        echo "a ?? b ?? c = {$result}\n";  // default
        
        $result = ($a ?? $b) ?: $c;
        echo "(a ?? b) ?: c = {$result}\n";  // default
    }
}

PrecedenceChecker::demonstratePrecedence();
```

### 示例 2：表达式求值

```php
<?php
declare(strict_types=1);

function evaluateExpression(string $description, mixed $result): void
{
    echo "{$description}\n";
    echo "Result: ";
    var_dump($result);
    echo "\n";
}

// 复杂表达式
$a = 10;
$b = 5;
$c = 2;

// 表达式1：算术和比较
$result1 = $a + $b * $c > 15;
evaluateExpression("a + b * c > 15", $result1);  // true

// 表达式2：逻辑组合
$result2 = ($a > $b) && ($b > $c) || ($a < $c);
evaluateExpression("(a > b) && (b > c) || (a < c)", $result2);  // true

// 表达式3：赋值和运算
$result3 = $a += $b *= $c;
evaluateExpression("a += b *= c", $result3);  // 20

// 表达式4：三元运算符
$result4 = $a > $b ? $a : $b > $c ? $b : $c;
evaluateExpression("a > b ? a : b > c ? b : c", $result4);  // 10
```

### 示例 3：最佳实践

```php
<?php
declare(strict_types=1);

// 最佳实践：使用括号明确优先级
class ExpressionBuilder
{
    public static function safeExpression1(int $a, int $b, int $c): bool
    {
        // 不推荐：依赖优先级
        // return $a + $b * $c > 15;
        
        // 推荐：使用括号明确
        return ($a + ($b * $c)) > 15;
    }
    
    public static function safeExpression2(bool $a, bool $b, bool $c): bool
    {
        // 不推荐：依赖优先级
        // return $a && $b || $c;
        
        // 推荐：使用括号明确
        return ($a && $b) || $c;
    }
    
    public static function safeExpression3(?string $a, ?string $b, string $c): string
    {
        // 不推荐：依赖优先级
        // return $a ?? $b ?: $c;
        
        // 推荐：使用括号明确
        return ($a ?? $b) ?: $c;
    }
}

// 使用
var_dump(ExpressionBuilder::safeExpression1(10, 5, 2));  // bool(true)
var_dump(ExpressionBuilder::safeExpression2(true, false, true));  // bool(true)
echo ExpressionBuilder::safeExpression3(null, null, "default") . "\n";  // default
```

## 使用场景

### 表达式求值

- **理解求值顺序**：理解表达式的求值顺序
- **调试表达式**：调试复杂的表达式
- **优化表达式**：优化表达式结构

### 代码调试

- **定位优先级问题**：定位由优先级导致的问题
- **验证表达式**：验证表达式的正确性
- **重构表达式**：重构复杂的表达式

### 代码优化

- **简化表达式**：简化复杂的表达式
- **提高可读性**：使用括号提高可读性
- **避免错误**：避免优先级导致的错误

## 注意事项

### 优先级记忆

- **常用优先级**：记住常用运算符的优先级
- **参考文档**：不确定时参考官方文档
- **使用括号**：不确定时使用括号

### 结合性

- **理解结合性**：理解左结合和右结合的区别
- **赋值运算符**：赋值运算符是右结合的
- **三元运算符**：三元运算符是右结合的

### 括号使用

- **明确优先级**：使用括号明确优先级
- **提高可读性**：使用括号提高可读性
- **避免歧义**：使用括号避免歧义

## 常见问题

### 问题 1：优先级混淆

**症状**：表达式结果不符合预期

**原因**：不理解运算符优先级

**错误示例**：

```php
<?php
declare(strict_types=1);

// 期望：($a && $b) || $c
$a = true;
$b = false;
$c = true;
$result = $a && $b || $c;  // 实际：$a && ($b || $c)
var_dump($result);  // bool(true) - 可能不符合预期
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$a = true;
$b = false;
$c = true;

// 使用括号明确优先级
$result = ($a && $b) || $c;
var_dump($result);  // bool(true)
```

### 问题 2：结合性混淆

**症状**：表达式结果不符合预期

**原因**：不理解运算符结合性

**错误示例**：

```php
<?php
declare(strict_types=1);

// 期望：($a ? $b : $c) ? $d : $e
$a = false;
$b = "b";
$c = "c";
$d = "d";
$e = "e";

$result = $a ? $b : $c ? $d : $e;  // 实际：$a ? $b : ($c ? $d : $e)
echo "Result: {$result}\n";  // Result: d（可能不符合预期）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$a = false;
$b = "b";
$c = "c";
$d = "d";
$e = "e";

// 使用括号明确结合性
$result = ($a ? $b : $c) ? $d : $e;
echo "Result: {$result}\n";  // Result: d
```

### 问题 3：空合并运算符优先级

**症状**：表达式结果不符合预期

**原因**：不理解空合并运算符的优先级

**错误示例**：

```php
<?php
declare(strict_types=1);

$a = null;
$b = "";
$c = "default";

// 期望：($a ?? $b) ?: $c
$result = $a ?? $b ?: $c;  // 实际：$a ?? ($b ?: $c)
echo "Result: {$result}\n";  // Result: default（可能不符合预期）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$a = null;
$b = "";
$c = "default";

// 使用括号明确优先级
$result = ($a ?? $b) ?: $c;
echo "Result: {$result}\n";  // Result: default
```

## 最佳实践

### 使用括号

- **明确优先级**：不确定优先级时使用括号
- **提高可读性**：使用括号提高代码可读性
- **避免错误**：使用括号避免优先级错误

### 理解优先级

- **记住常用优先级**：记住常用运算符的优先级
- **参考文档**：不确定时参考官方文档
- **测试验证**：不确定时进行测试验证

### 代码可读性

- **明确意图**：使用括号明确代码意图
- **避免复杂表达式**：避免过于复杂的表达式
- **提取变量**：将复杂表达式提取为变量

## 对比分析

### 左结合 vs 右结合

| 特性 | 左结合 | 右结合 |
|:-----|:-------|:-------|
| 求值顺序 | 从左到右 | 从右到左 |
| 常见运算符 | 算术、比较、逻辑 | 赋值、三元 |
| 示例 | `$a + $b + $c` | `$a = $b = $c` |

**选择建议**：
- **理解结合性**：理解不同运算符的结合性
- **使用括号**：不确定时使用括号

### 括号使用 vs 依赖优先级

| 特性 | 使用括号 | 依赖优先级 |
|:-----|:---------|:-----------|
| 可读性 | 高 | 低 |
| 安全性 | 高 | 低 |
| 代码长度 | 长 | 短 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **优先使用括号**：提高可读性和安全性
- **简单表达式**：简单表达式可以依赖优先级
- **复杂表达式**：复杂表达式必须使用括号

## 相关章节

- **2.6.1 算术运算符**：了解算术运算符
- **2.6.2 赋值运算符**：了解赋值运算符
- **2.6.3 逻辑运算符**：了解逻辑运算符
- **2.6.4 三元运算符与空合并运算符**：了解条件运算符

## 练习任务

1. **优先级练习**：
   - 测试各种运算符的优先级
   - 理解优先级的影响
   - 使用括号明确优先级
   - 测试不同组合的结果

2. **结合性练习**：
   - 测试左结合和右结合的行为
   - 理解结合性的影响
   - 使用括号明确结合性
   - 测试不同场景

3. **常见问题练习**：
   - 重现常见的优先级问题
   - 使用括号解决问题
   - 测试各种场景
   - 理解问题的原因

4. **最佳实践练习**：
   - 使用括号改进代码
   - 提高代码可读性
   - 避免优先级错误
   - 进行代码审查

5. **综合练习**：
   - 创建一个表达式求值工具
   - 实现优先级检查功能
   - 测试各种表达式
   - 进行代码审查，确保正确性

# 2.7.2 常用字符串函数

## 概述

PHP 提供了丰富的字符串处理函数，包括长度计算、查找、替换、截取、格式化等。本节介绍最常用的字符串函数及其详细用法。

## 长度计算

### strlen() - 字节长度

**语法**：`strlen(string $string): int`

**参数**：
- `$string`：要计算长度的字符串

**返回值**：返回字符串的字节数（不是字符数）。

```php
<?php
declare(strict_types=1);

echo strlen("Hello") . "\n";        // 5
echo strlen("你好") . "\n";        // 6（UTF-8 编码，每个中文字符 3 字节）
```

### mb_strlen() - 字符长度

**语法**：`mb_strlen(string $string, ?string $encoding = null): int`

**参数**：
- `$string`：要计算长度的字符串
- `$encoding`：字符编码，默认为内部编码

**返回值**：返回字符串的字符数。

```php
<?php
declare(strict_types=1);

mb_internal_encoding('UTF-8');
echo mb_strlen("Hello", 'UTF-8') . "\n";   // 5
echo mb_strlen("你好", 'UTF-8') . "\n";     // 2（正确计算中文字符数）
```

## 查找与定位

### strpos() - 查找子串位置

**语法**：`strpos(string $haystack, string $needle, int $offset = 0): int|false`

**参数**：
- `$haystack`：要搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置

**返回值**：找到返回位置（从 0 开始），未找到返回 `false`。

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
$pos = strpos($str, "World");
if ($pos !== false) {
    echo "Found at position: {$pos}\n";  // Found at position: 7
}

// 从指定位置开始搜索
$pos2 = strpos($str, "o", 5);  // 从位置 5 开始搜索
echo $pos2 . "\n";  // 8
```

### mb_strpos() - 多字节查找

**语法**：`mb_strpos(string $haystack, string $needle, int $offset = 0, ?string $encoding = null): int|false`

**参数**：
- `$haystack`：要搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置
- `$encoding`：可选，字符编码，默认为内部编码

**返回值**：找到返回位置（从 0 开始），未找到返回 `false`。

```php
<?php
declare(strict_types=1);

mb_internal_encoding('UTF-8');
$str = "你好，世界！";
$pos = mb_strpos($str, "世界", 0, 'UTF-8');
echo $pos . "\n";  // 4
```

### strrpos() - 从后往前查找

**语法**：`strrpos(string $haystack, string $needle, int $offset = 0): int|false`

**参数**：
- `$haystack`：要搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置（负数表示从末尾开始）

**返回值**：找到返回位置（从 0 开始），未找到返回 `false`。

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
$pos = strrpos($str, "o");  // 从后往前查找
echo $pos . "\n";  // 8
```

## 截取子串

### substr() - 截取子串（按字节）

**语法**：`substr(string $string, int $offset, ?int $length = null): string`

**参数**：
- `$string`：原字符串
- `$offset`：起始位置（负数表示从末尾开始）
- `$length`：可选，要截取的长度（负数表示从末尾开始）

**返回值**：返回截取的子串。

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
echo substr($str, 0, 5) . "\n";    // Hello
echo substr($str, 7) . "\n";       // World!
echo substr($str, -6) . "\n";      // World!
echo substr($str, 0, -1) . "\n";   // Hello, World
```

### mb_substr() - 截取子串（按字符）

**语法**：`mb_substr(string $string, int $start, ?int $length = null, ?string $encoding = null): string`

**参数**：
- `$string`：原字符串
- `$start`：起始位置（负数表示从末尾开始）
- `$length`：可选，要截取的长度（负数表示从末尾开始）
- `$encoding`：可选，字符编码，默认为内部编码

**返回值**：返回截取的子串。

```php
<?php
declare(strict_types=1);

mb_internal_encoding('UTF-8');
$str = "你好，世界！";
echo mb_substr($str, 0, 2, 'UTF-8') . "\n";  // 你好
echo mb_substr($str, 3, 2, 'UTF-8') . "\n";  // 世界
```

## 替换操作

### str_replace() - 字符串替换

**语法**：`str_replace(mixed $search, mixed $replace, mixed $subject, int &$count = null): mixed`

**参数**：
- `$search`：要查找的值（可以是字符串或数组）
- `$replace`：替换值（可以是字符串或数组）
- `$subject`：要搜索替换的字符串或数组
- `$count`：可选，接收替换次数

**返回值**：返回替换后的字符串或数组。

```php
<?php
declare(strict_types=1);

// 简单替换
$str = "Hello, World!";
echo str_replace("World", "PHP", $str) . "\n";  // Hello, PHP!

// 多个替换
$str = "apple, banana, apple";
$count = 0;
echo str_replace("apple", "orange", $str, $count) . "\n";  // orange, banana, orange
echo "Replacements: {$count}\n";  // Replacements: 2

// 数组替换
$search = ['apple', 'banana'];
$replace = ['orange', 'grape'];
$str = "I like apple and banana";
echo str_replace($search, $replace, $str) . "\n";  // I like orange and grape
```

### str_ireplace() - 不区分大小写替换

**语法**：`str_ireplace(mixed $search, mixed $replace, mixed $subject, int &$count = null): mixed`

**参数**：
- `$search`：要查找的值（可以是字符串或数组）
- `$replace`：替换值（可以是字符串或数组）
- `$subject`：要搜索替换的字符串或数组
- `$count`：可选，接收替换次数

**返回值**：返回替换后的字符串或数组。

```php
<?php
declare(strict_types=1);

$str = "Hello, WORLD!";
echo str_ireplace("world", "PHP", $str) . "\n";  // Hello, PHP!
```

## 去除空白

### trim() - 去除首尾空白

**语法**：`trim(string $string, string $characters = " \n\r\t\v\0"): string`

**参数**：
- `$string`：要处理的字符串
- `$characters`：可选，要去除的字符列表

**返回值**：返回去除空白后的字符串。

```php
<?php
declare(strict_types=1);

$str = "  Hello, World!  ";
echo trim($str) . "\n";  // Hello, World!

// 去除指定字符
$str = "***Hello***";
echo trim($str, '*') . "\n";  // Hello
```

### ltrim() - 去除左侧空白

**语法**：`ltrim(string $string, string $characters = " \n\r\t\v\0"): string`

**参数**：
- `$string`：要处理的字符串
- `$characters`：可选，要去除的字符列表

**返回值**：返回去除左侧空白后的字符串。

### rtrim() - 去除右侧空白

**语法**：`rtrim(string $string, string $characters = " \n\r\t\v\0"): string`

**参数**：
- `$string`：要处理的字符串
- `$characters`：可选，要去除的字符列表

**返回值**：返回去除右侧空白后的字符串。

```php
<?php
declare(strict_types=1);

$str = "  Hello  ";
echo ltrim($str) . "\n";  // "Hello  "
echo rtrim($str) . "\n";  // "  Hello"
```

## 大小写转换

### strtolower() - 转为小写

**语法**：`strtolower(string $string): string`

**参数**：
- `$string`：要转换的字符串

**返回值**：返回转换为小写的字符串。

```php
<?php
declare(strict_types=1);

echo strtolower("Hello, World!") . "\n";  // hello, world!
```

### strtoupper() - 转为大写

**语法**：`strtoupper(string $string): string`

**参数**：
- `$string`：要转换的字符串

**返回值**：返回转换为大写的字符串。

```php
<?php
declare(strict_types=1);

echo strtoupper("Hello, World!") . "\n";  // HELLO, WORLD!
```

### ucfirst() - 首字母大写

**语法**：`ucfirst(string $string): string`

**参数**：
- `$string`：要转换的字符串

**返回值**：返回首字母大写的字符串。

```php
<?php
declare(strict_types=1);

echo ucfirst("hello, world!") . "\n";  // Hello, world!
```

### ucwords() - 每个单词首字母大写

**语法**：`ucwords(string $string, string $separators = " \t\r\n\f\v"): string`

**参数**：
- `$string`：要转换的字符串
- `$separators`：可选，单词分隔符列表

**返回值**：返回每个单词首字母大写的字符串。

```php
<?php
declare(strict_types=1);

echo ucwords("hello, world!") . "\n";  // Hello, World!
```

## 分割与拼接

### explode() - 分割字符串

**语法**：`explode(string $separator, string $string, int $limit = PHP_INT_MAX): array`

**参数**：
- `$separator`：分隔符
- `$string`：要分割的字符串
- `$limit`：可选，限制分割后的数组元素数量

**返回值**：返回分割后的数组。

```php
<?php
declare(strict_types=1);

$str = "apple,banana,orange";
$fruits = explode(",", $str);
print_r($fruits);  // ['apple', 'banana', 'orange']

// 限制分割数量
$parts = explode(",", $str, 2);
print_r($parts);  // ['apple', 'banana,orange']
```

### implode() - 拼接数组

**语法**：`implode(string $separator, array $array): string`

**参数**：
- `$separator`：分隔符
- `$array`：要拼接的数组

**返回值**：返回拼接后的字符串。

```php
<?php
declare(strict_types=1);

$fruits = ['apple', 'banana', 'orange'];
echo implode(", ", $fruits) . "\n";  // apple, banana, orange
```

## 格式化

### sprintf() - 格式化字符串

**语法**：`sprintf(string $format, mixed ...$values): string`

**参数**：
- `$format`：格式化字符串，包含格式说明符（如 `%s`、`%d`、`%f` 等）
- `...$values`：要格式化的值（可变参数）

**返回值**：返回格式化后的字符串。

```php
<?php
declare(strict_types=1);

$name = "Alice";
$age = 25;
$message = sprintf("Name: %s, Age: %d", $name, $age);
echo $message . "\n";  // Name: Alice, Age: 25
```

### number_format() - 数字格式化

**语法**：`number_format(float $number, int $decimals = 0, ?string $decimal_separator = ".", ?string $thousands_separator = ","): string`

**参数**：
- `$number`：要格式化的数字
- `$decimals`：可选，小数位数，默认为 0
- `$decimal_separator`：可选，小数点分隔符，默认为 "."
- `$thousands_separator`：可选，千位分隔符，默认为 ","

**返回值**：返回格式化后的数字字符串。

```php
<?php
declare(strict_types=1);

echo number_format(1234567.89, 2) . "\n";  // 1,234,567.89
echo number_format(1234567.89, 2, ',', ' ') . "\n";  // 1 234 567,89
```

## 完整示例

```php
<?php
declare(strict_types=1);

class StringHelper
{
    public static function truncate(string $str, int $length, string $suffix = '...'): string
    {
        if (mb_strlen($str, 'UTF-8') <= $length) {
            return $str;
        }
        return mb_substr($str, 0, $length, 'UTF-8') . $suffix;
    }
    
    public static function slugify(string $title): string
    {
        // 转为小写
        $slug = strtolower($title);
        // 替换空格为连字符
        $slug = str_replace(' ', '-', $slug);
        // 移除特殊字符
        $slug = preg_replace('/[^a-z0-9\-]/', '', $slug);
        // 移除多个连字符
        $slug = preg_replace('/-+/', '-', $slug);
        // 去除首尾连字符
        return trim($slug, '-');
    }
    
    public static function extractEmail(string $text): ?string
    {
        if (preg_match('/[\w\.-]+@[\w\.-]+\.\w+/', $text, $matches)) {
            return $matches[0];
        }
        return null;
    }
}

// 使用示例
echo StringHelper::truncate("这是一段很长的文本", 5) . "\n";
echo StringHelper::slugify("Hello, World!") . "\n";
echo StringHelper::extractEmail("Contact: user@example.com") . "\n";
```

## 注意事项

1. **多字节处理**：处理中文等多字节字符时，使用 `mb_*` 系列函数。

2. **返回值检查**：`strpos()` 等函数可能返回 `false`，使用 `!==` 严格比较。

3. **性能考虑**：简单操作使用内置函数，复杂操作考虑使用正则表达式。

4. **编码设置**：使用 `mb_*` 函数前，设置正确的内部编码。

5. **安全性**：处理用户输入时，使用适当的转义函数。

## 练习

1. 创建一个字符串工具类，封装常用的字符串操作函数。

2. 编写一个函数，安全地截取 UTF-8 字符串，避免截断中文字符。

3. 实现一个函数，将驼峰命名转换为下划线命名。

4. 创建一个函数，提取文本中的所有 URL。

5. 编写一个函数，实现字符串的模糊匹配。

# 2.7.2 常用字符串函数

## 概述

PHP 提供了丰富的字符串处理函数。本节详细介绍常用字符串函数的语法、参数、返回值和使用场景，包括长度计算、查找定位、截取、替换、去除空白、大小写转换、分割拼接、格式化等。

掌握常用字符串函数对于处理文本数据、用户输入、格式化输出等场景非常重要。理解函数的参数、返回值和注意事项可以帮助编写更可靠的代码。

## 特性

- **函数丰富**：提供大量字符串处理函数
- **多字节支持**：`mb_*` 系列函数支持多字节字符（UTF-8 等）
- **性能优化**：选择合适的函数提高性能
- **编码处理**：支持多种字符编码

## 语法/定义

### 长度计算函数

#### strlen() - 计算字节长度

**语法**：`strlen(string $string): int`

**参数**：
- `$string`：要计算长度的字符串

**返回值**：返回字符串的字节长度（类型为 `int`）

**特点**：
- 计算字节数，不是字符数
- 多字节字符（如中文）会计算错误
- 性能最好

#### mb_strlen() - 计算字符长度（多字节）

**语法**：`mb_strlen(string $string, ?string $encoding = null): int`

**参数**：
- `$string`：要计算长度的字符串
- `$encoding`：可选，字符编码，默认为内部编码

**返回值**：返回字符串的字符长度（类型为 `int`）

**特点**：
- 计算字符数，不是字节数
- 正确处理多字节字符
- 需要 mbstring 扩展

### 查找定位函数

#### strpos() - 查找子串首次出现位置

**语法**：`strpos(string $haystack, string $needle, int $offset = 0): int|false`

**参数**：
- `$haystack`：被搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置，默认为 0

**返回值**：返回首次出现的位置（从 0 开始），未找到返回 `false`

**特点**：
- 区分大小写
- 返回字节位置
- 多字节字符可能返回错误位置

#### mb_strpos() - 查找子串首次出现位置（多字节）

**语法**：`mb_strpos(string $haystack, string $needle, int $offset = 0, ?string $encoding = null): int|false`

**参数**：
- `$haystack`：被搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置，默认为 0
- `$encoding`：可选，字符编码

**返回值**：返回首次出现的位置（从 0 开始），未找到返回 `false`

**特点**：
- 区分大小写
- 返回字符位置
- 正确处理多字节字符

#### strrpos() - 查找子串最后出现位置

**语法**：`strrpos(string $haystack, string $needle, int $offset = 0): int|false`

**参数**：
- `$haystack`：被搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置

**返回值**：返回最后出现的位置，未找到返回 `false`

**特点**：
- 区分大小写
- 从右向左搜索

### 截取函数

#### substr() - 截取子串

**语法**：`substr(string $string, int $start, ?int $length = null): string`

**参数**：
- `$string`：要截取的字符串
- `$start`：起始位置（负数表示从末尾开始）
- `$length`：可选，截取长度（负数表示到倒数第 N 个字符）

**返回值**：返回截取的子串（类型为 `string`）

**特点**：
- 按字节截取
- 多字节字符可能截取错误

#### mb_substr() - 截取子串（多字节）

**语法**：`mb_substr(string $string, int $start, ?int $length = null, ?string $encoding = null): string`

**参数**：
- `$string`：要截取的字符串
- `$start`：起始位置（负数表示从末尾开始）
- `$length`：可选，截取长度
- `$encoding`：可选，字符编码

**返回值**：返回截取的子串（类型为 `string`）

**特点**：
- 按字符截取
- 正确处理多字节字符

### 替换函数

#### str_replace() - 字符串替换

**语法**：`str_replace(array|string $search, array|string $replace, string|array $subject, int &$count = null): string|array`

**参数**：
- `$search`：要查找的字符串或数组
- `$replace`：替换的字符串或数组
- `$subject`：被搜索的字符串或数组
- `$count`：可选，引用参数，返回替换次数

**返回值**：返回替换后的字符串或数组

**特点**：
- 区分大小写
- 支持数组参数

#### str_ireplace() - 字符串替换（不区分大小写）

**语法**：`str_ireplace(array|string $search, array|string $replace, string|array $subject, int &$count = null): string|array`

**参数**：与 `str_replace()` 相同

**返回值**：返回替换后的字符串或数组

**特点**：
- 不区分大小写
- 其他与 `str_replace()` 相同

### 去除空白函数

#### trim() - 去除两端空白

**语法**：`trim(string $string, string $characters = " \n\r\t\v\0"): string`

**参数**：
- `$string`：要处理的字符串
- `$characters`：可选，要去除的字符列表

**返回值**：返回去除空白后的字符串

**特点**：
- 去除两端空白
- 可指定要去除的字符

#### ltrim() - 去除左端空白

**语法**：`ltrim(string $string, string $characters = " \n\r\t\v\0"): string`

**参数**：与 `trim()` 相同

**返回值**：返回去除左端空白后的字符串

#### rtrim() - 去除右端空白

**语法**：`rtrim(string $string, string $characters = " \n\r\t\v\0"): string`

**参数**：与 `trim()` 相同

**返回值**：返回去除右端空白后的字符串

### 大小写转换函数

#### strtolower() - 转换为小写

**语法**：`strtolower(string $string): string`

**参数**：
- `$string`：要转换的字符串

**返回值**：返回小写字符串

#### strtoupper() - 转换为大写

**语法**：`strtoupper(string $string): string`

**参数**：
- `$string`：要转换的字符串

**返回值**：返回大写字符串

#### ucfirst() - 首字母大写

**语法**：`ucfirst(string $string): string`

**参数**：
- `$string`：要转换的字符串

**返回值**：返回首字母大写的字符串

#### ucwords() - 每个单词首字母大写

**语法**：`ucwords(string $string, string $separators = " \t\r\n\f\v"): string`

**参数**：
- `$string`：要转换的字符串
- `$separators`：可选，单词分隔符

**返回值**：返回每个单词首字母大写的字符串

### 分割拼接函数

#### explode() - 分割字符串为数组

**语法**：`explode(string $separator, string $string, int $limit = PHP_INT_MAX): array`

**参数**：
- `$separator`：分隔符
- `$string`：要分割的字符串
- `$limit`：可选，限制分割次数

**返回值**：返回分割后的数组

#### implode() - 将数组拼接为字符串

**语法**：`implode(string $separator, array $array): string`

**参数**：
- `$separator`：分隔符
- `$array`：要拼接的数组

**返回值**：返回拼接后的字符串

**别名**：`join()` 是 `implode()` 的别名

## 基本用法

### 示例 1：长度计算

```php
<?php
declare(strict_types=1);

// strlen() - 字节长度
$str1 = "Hello";
echo "strlen('Hello'): " . strlen($str1) . "\n";  // 5

$str2 = "你好";
echo "strlen('你好'): " . strlen($str2) . "\n";  // 6（UTF-8 每个中文字符 3 字节）

// mb_strlen() - 字符长度
echo "mb_strlen('你好'): " . mb_strlen($str2, 'UTF-8') . "\n";  // 2
```

### 示例 2：查找定位

```php
<?php
declare(strict_types=1);

$haystack = "Hello, World!";

// strpos() - 查找首次出现位置
$pos = strpos($haystack, "World");
if ($pos !== false) {
    echo "Found at position: {$pos}\n";  // Found at position: 7
} else {
    echo "Not found\n";
}

// strrpos() - 查找最后出现位置
$haystack2 = "Hello, World, World!";
$pos = strrpos($haystack2, "World");
echo "Last found at position: {$pos}\n";  // Last found at position: 14

// mb_strpos() - 多字节版本
$haystack3 = "你好，世界！";
$pos = mb_strpos($haystack3, "世界", 0, 'UTF-8');
if ($pos !== false) {
    echo "Found at position: {$pos}\n";  // Found at position: 3
}
```

### 示例 3：截取

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";

// substr() - 基本截取
echo substr($str, 0, 5) . "\n";  // Hello
echo substr($str, 7) . "\n";     // World!
echo substr($str, -6) . "\n";    // World!

// mb_substr() - 多字节版本
$str2 = "你好，世界！";
echo mb_substr($str2, 0, 2, 'UTF-8') . "\n";  // 你好
echo mb_substr($str2, 3, 2, 'UTF-8') . "\n";  // 世界
```

### 示例 4：替换

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";

// str_replace() - 基本替换
$result = str_replace("World", "PHP", $str);
echo "Result: {$result}\n";  // Result: Hello, PHP!

// 数组替换
$search = ["Hello", "World"];
$replace = ["Hi", "PHP"];
$result = str_replace($search, $replace, $str);
echo "Result: {$result}\n";  // Result: Hi, PHP!

// 统计替换次数
$count = 0;
$result = str_replace("o", "0", $str, $count);
echo "Result: {$result}, Count: {$count}\n";  // Result: Hell0, W0rld!, Count: 2
```

### 示例 5：去除空白

```php
<?php
declare(strict_types=1);

$str = "  Hello, World!  ";

// trim() - 去除两端空白
echo "'" . trim($str) . "'\n";  // 'Hello, World!'

// ltrim() - 去除左端空白
echo "'" . ltrim($str) . "'\n";  // 'Hello, World!  '

// rtrim() - 去除右端空白
echo "'" . rtrim($str) . "'\n";  // '  Hello, World!'

// 自定义字符
$str2 = "...Hello, World!...";
echo trim($str2, ".") . "\n";  // Hello, World!
```

### 示例 6：大小写转换

```php
<?php
declare(strict_types=1);

$str = "hello, world!";

// strtolower() - 转小写
echo strtolower("HELLO, WORLD!") . "\n";  // hello, world!

// strtoupper() - 转大写
echo strtoupper($str) . "\n";  // HELLO, WORLD!

// ucfirst() - 首字母大写
echo ucfirst($str) . "\n";  // Hello, world!

// ucwords() - 每个单词首字母大写
echo ucwords($str) . "\n";  // Hello, World!
```

### 示例 7：分割拼接

```php
<?php
declare(strict_types=1);

// explode() - 分割
$str = "apple,banana,orange";
$fruits = explode(",", $str);
print_r($fruits);  // Array ( [0] => apple [1] => banana [2] => orange )

// 限制分割次数
$fruits = explode(",", $str, 2);
print_r($fruits);  // Array ( [0] => apple [1] => banana,orange )

// implode() - 拼接
$array = ["apple", "banana", "orange"];
$str = implode(", ", $array);
echo $str . "\n";  // apple, banana, orange

// join() - implode() 的别名
$str = join(" - ", $array);
echo $str . "\n";  // apple - banana - orange
```

## 完整代码示例

### 示例 1：字符串工具类

```php
<?php
declare(strict_types=1);

class StringHelper
{
    public static function length(string $str, string $encoding = 'UTF-8'): int
    {
        return mb_strlen($str, $encoding);
    }
    
    public static function contains(string $haystack, string $needle, string $encoding = 'UTF-8'): bool
    {
        return mb_strpos($haystack, $needle, 0, $encoding) !== false;
    }
    
    public static function startsWith(string $haystack, string $needle, string $encoding = 'UTF-8'): bool
    {
        return mb_strpos($haystack, $needle, 0, $encoding) === 0;
    }
    
    public static function endsWith(string $haystack, string $needle, string $encoding = 'UTF-8'): bool
    {
        $needleLength = mb_strlen($needle, $encoding);
        if ($needleLength === 0) {
            return true;
        }
        $haystackLength = mb_strlen($haystack, $encoding);
        return mb_substr($haystack, -$needleLength, null, $encoding) === $needle;
    }
    
    public static function truncate(string $str, int $length, string $suffix = '...', string $encoding = 'UTF-8'): string
    {
        if (mb_strlen($str, $encoding) <= $length) {
            return $str;
        }
        return mb_substr($str, 0, $length - mb_strlen($suffix, $encoding), $encoding) . $suffix;
    }
}

// 使用
$str = "Hello, 世界！";
echo "Length: " . StringHelper::length($str) . "\n";  // Length: 9
echo "Contains '世界': " . (StringHelper::contains($str, "世界") ? 'Yes' : 'No') . "\n";  // Yes
echo "Starts with 'Hello': " . (StringHelper::startsWith($str, "Hello") ? 'Yes' : 'No') . "\n";  // Yes
echo "Truncate: " . StringHelper::truncate($str, 8) . "\n";  // Hello, ...
```

### 示例 2：文本处理

```php
<?php
declare(strict_types=1);

function processText(string $text): string
{
    // 去除空白
    $text = trim($text);
    
    // 替换多个空格为单个空格
    $text = preg_replace('/\s+/', ' ', $text);
    
    // 首字母大写
    $text = ucfirst($text);
    
    return $text;
}

function extractWords(string $text): array
{
    // 转换为小写
    $text = strtolower($text);
    
    // 去除标点符号
    $text = preg_replace('/[^\w\s]/', '', $text);
    
    // 分割为单词
    $words = explode(' ', $text);
    
    // 去除空元素
    $words = array_filter($words, fn($word) => $word !== '');
    
    return array_values($words);
}

// 使用
$text = "  hello,   world!  this is a test.  ";
echo "Processed: '" . processText($text) . "'\n";
$words = extractWords($text);
print_r($words);
```

### 示例 3：CSV 处理

```php
<?php
declare(strict_types=1);

function parseCsvLine(string $line, string $delimiter = ',', string $enclosure = '"'): array
{
    $fields = [];
    $field = '';
    $inQuotes = false;
    $len = strlen($line);
    
    for ($i = 0; $i < $len; $i++) {
        $char = $line[$i];
        
        if ($char === $enclosure) {
            if ($inQuotes && $i + 1 < $len && $line[$i + 1] === $enclosure) {
                // 转义的引号
                $field .= $enclosure;
                $i++;
            } else {
                // 切换引号状态
                $inQuotes = !$inQuotes;
            }
        } elseif ($char === $delimiter && !$inQuotes) {
            // 字段结束
            $fields[] = trim($field);
            $field = '';
        } else {
            $field .= $char;
        }
    }
    
    // 添加最后一个字段
    $fields[] = trim($field);
    
    return $fields;
}

function formatCsvLine(array $fields, string $delimiter = ',', string $enclosure = '"'): string
{
    $formatted = [];
    foreach ($fields as $field) {
        // 如果字段包含分隔符或引号，需要用引号括起来
        if (strpos($field, $delimiter) !== false || strpos($field, $enclosure) !== false) {
            $field = $enclosure . str_replace($enclosure, $enclosure . $enclosure, $field) . $enclosure;
        }
        $formatted[] = $field;
    }
    return implode($delimiter, $formatted);
}

// 使用
$line = 'John,"Doe, Jr.",25,"New York, NY"';
$fields = parseCsvLine($line);
print_r($fields);

$formatted = formatCsvLine($fields);
echo $formatted . "\n";
```

## 使用场景

### 长度计算

- **验证输入**：验证用户输入长度
- **截取文本**：根据长度截取文本
- **显示限制**：限制显示长度

### 查找定位

- **搜索功能**：实现搜索功能
- **字符串验证**：验证字符串是否包含特定内容
- **位置提取**：提取特定位置的内容

### 截取

- **摘要生成**：生成文本摘要
- **标题截取**：截取标题
- **内容预览**：生成内容预览

### 替换

- **模板处理**：处理模板变量
- **数据清理**：清理数据
- **格式化**：格式化文本

### 去除空白

- **用户输入**：清理用户输入
- **数据验证**：验证数据
- **格式化**：格式化输出

### 大小写转换

- **标准化**：标准化文本
- **比较**：不区分大小写比较
- **格式化**：格式化输出

### 分割拼接

- **CSV 处理**：处理 CSV 数据
- **URL 参数**：处理 URL 参数
- **配置解析**：解析配置字符串

## 注意事项

### 多字节字符

- **使用 mb_* 函数**：处理中文等多字节字符时使用 `mb_*` 函数
- **设置编码**：确保使用正确的字符编码
- **编码一致性**：保持编码一致性

### 性能考虑

- **选择合适的函数**：选择合适的函数提高性能
- **避免重复计算**：缓存计算结果
- **批量处理**：批量处理数据

### 错误处理

- **检查返回值**：检查函数返回值（如 `strpos()` 返回 `false`）
- **使用严格比较**：使用 `===` 比较返回值
- **处理异常**：处理可能的异常

## 常见问题

### 问题 1：中文字符长度错误

**症状**：中文字符长度计算错误

**原因**：使用 `strlen()` 计算多字节字符

**错误示例**：

```php
<?php
declare(strict_types=1);

$str = "你好";
echo strlen($str) . "\n";  // 6（错误：应该是 2）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$str = "你好";
echo mb_strlen($str, 'UTF-8') . "\n";  // 2（正确）
```

### 问题 2：中文字符截取乱码

**症状**：中文字符截取后出现乱码

**原因**：使用 `substr()` 截取多字节字符

**错误示例**：

```php
<?php
declare(strict_types=1);

$str = "你好，世界！";
echo substr($str, 0, 3) . "\n";  // 你（可能乱码）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$str = "你好，世界！";
echo mb_substr($str, 0, 2, 'UTF-8') . "\n";  // 你好（正确）
```

### 问题 3：strpos() 返回 false 判断错误

**症状**：`strpos()` 返回 0 时被误判为 false

**原因**：使用 `==` 而不是 `===` 比较

**错误示例**：

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
$pos = strpos($str, "Hello");
if ($pos == false) {  // 错误：0 == false 为 true
    echo "Not found\n";
} else {
    echo "Found\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
$pos = strpos($str, "Hello");
if ($pos === false) {  // 正确：使用严格比较
    echo "Not found\n";
} else {
    echo "Found at position: {$pos}\n";
}
```

## 最佳实践

### 多字节处理

- **使用 mb_* 函数**：处理多字节字符时使用 `mb_*` 函数
- **设置编码**：明确指定字符编码
- **编码一致性**：保持编码一致性

### 错误处理

- **检查返回值**：检查函数返回值
- **使用严格比较**：使用 `===` 比较返回值
- **处理边界情况**：处理空字符串、null 等边界情况

### 性能优化

- **选择合适的函数**：选择合适的函数
- **避免重复计算**：缓存计算结果
- **批量处理**：批量处理数据

## 对比分析

### strlen() vs mb_strlen()

| 特性 | strlen() | mb_strlen() |
|:-----|:---------|:------------|
| 计算单位 | 字节 | 字符 |
| 多字节支持 | 否 | 是 |
| 性能 | 更好 | 稍差 |
| 推荐度 | 单字节字符 | 多字节字符 |

**选择建议**：
- **单字节字符**：使用 `strlen()`
- **多字节字符**：使用 `mb_strlen()`

### substr() vs mb_substr()

| 特性 | substr() | mb_substr() |
|:-----|:---------|:------------|
| 截取单位 | 字节 | 字符 |
| 多字节支持 | 否 | 是 |
| 性能 | 更好 | 稍差 |
| 推荐度 | 单字节字符 | 多字节字符 |

**选择建议**：
- **单字节字符**：使用 `substr()`
- **多字节字符**：使用 `mb_substr()`

## 相关章节

- **2.7.1 字符串创建方式**：了解字符串创建
- **2.4.1 标量类型**：了解字符串类型
- **2.8 数组完整指南**：了解字符串与数组的转换

## 练习任务

1. **基本函数练习**：
   - 练习使用各种字符串函数
   - 理解函数的参数和返回值
   - 测试不同场景下的行为
   - 观察多字节字符的处理

2. **多字节处理练习**：
   - 对比 `strlen()` 和 `mb_strlen()`
   - 对比 `substr()` 和 `mb_substr()`
   - 处理中文等多字节字符
   - 测试各种编码

3. **实际应用练习**：
   - 实现文本处理工具
   - 实现 CSV 处理功能
   - 实现搜索功能
   - 测试各种应用场景

4. **错误处理练习**：
   - 处理 `strpos()` 返回 false 的情况
   - 处理空字符串和 null
   - 处理编码错误
   - 测试边界情况

5. **综合练习**：
   - 创建一个字符串工具类
   - 实现各种字符串处理功能
   - 处理多字节字符
   - 进行代码审查，确保正确性

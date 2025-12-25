# 2.7.1 字符串创建方式

## 概述

PHP 提供了四种创建字符串的方式：单引号、双引号、Heredoc 和 Nowdoc。每种方式都有其特点和适用场景。

## 单引号字符串

### 基本语法

```php
$string = 'Hello World';
```

### 特点

- **不解析变量**：变量不会被替换
- **不解析转义序列**：除了 `\'` 和 `\\` 外，其他转义序列不会被解析
- **性能稍好**：因为不需要解析变量和转义序列

### 示例

```php
<?php
declare(strict_types=1);

$name = 'Alice';

// 变量不会被解析
echo 'Hello, $name\n';  // 输出：Hello, $name\n（字面量）

// 只支持两种转义
echo 'It\'s a string\n';  // 输出：It's a string\n（\n 不会被解析为换行）
echo 'Path: C:\\Windows\n';  // 输出：Path: C:\Windows\n

// 单引号内使用单引号需要转义
echo 'He said: \'Hello\'';  // 输出：He said: 'Hello'
```

## 双引号字符串

### 基本语法

```php
$string = "Hello World";
```

### 特点

- **解析变量**：变量会被替换为值
- **解析转义序列**：支持所有转义序列（`\n`、`\t`、`\"` 等）
- **变量插值**：可以使用 `{$variable}` 或 `$variable` 语法

### 变量插值

#### 简单变量

```php
<?php
declare(strict_types=1);

$name = 'Alice';
echo "Hello, $name\n";  // 输出：Hello, Alice
```

#### 复杂变量（使用花括号）

```php
<?php
declare(strict_types=1);

$name = 'Alice';
echo "Hello, {$name}!\n";  // 推荐：使用花括号

// 数组元素
$user = ['name' => 'Alice', 'age' => 25];
echo "Name: {$user['name']}, Age: {$user['age']}\n";

// 对象属性
class User {
    public string $name = 'Alice';
}
$user = new User();
echo "Name: {$user->name}\n";
```

#### 转义序列

```php
<?php
declare(strict_types=1);

echo "Line 1\nLine 2\n";        // 换行
echo "Tab\tSeparated\n";         // 制表符
echo "Quote: \"Hello\"\n";       // 双引号
echo "Backslash: \\\n";          // 反斜线
echo "Unicode: \u{1F600}\n";     // Unicode 字符（PHP 7.0+）
```

### 示例

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age = 25;

// 简单插值
echo "Name: $name, Age: $age\n";

// 使用花括号（推荐）
echo "Name: {$name}, Age: {$age}\n";

// 复杂表达式
echo "Next year: " . ($age + 1) . "\n";
echo "Next year: {$age + 1}\n";  // 错误：不能使用表达式
```

## Heredoc 语法

### 基本语法

```php
$string = <<<IDENTIFIER
多行字符串内容
可以包含变量插值
IDENTIFIER;
```

### 特点

- **多行字符串**：可以包含换行符
- **解析变量**：类似双引号，支持变量插值
- **解析转义序列**：支持所有转义序列
- **标识符规则**：标识符必须单独一行，不能有缩进

### 示例

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age = 25;

$text = <<<EOT
Hello, {$name}!
You are {$age} years old.
This is a multi-line string.
EOT;

echo $text;
// 输出：
// Hello, Alice!
// You are 25 years old.
// This is a multi-line string.
```

### 实际应用

```php
<?php
declare(strict_types=1);

function generateEmail(string $name, string $message): string
{
    return <<<EMAIL
Dear {$name},

Thank you for your message:
{$message}

Best regards,
Support Team
EMAIL;
}

echo generateEmail('Alice', 'I need help with my account.');
```

### PHP 7.3+ 改进

PHP 7.3+ 允许标识符有缩进，使用 `<<<` 后的空格控制：

```php
<?php
declare(strict_types=1);

// PHP 7.3+：缩进的 Heredoc
$text = <<<'EOT'
    This text is indented.
    The closing identifier can also be indented.
    EOT;

echo $text;
```

## Nowdoc 语法

### 基本语法

```php
$string = <<<'IDENTIFIER'
多行字符串内容
不解析变量
IDENTIFIER;
```

### 特点

- **多行字符串**：可以包含换行符
- **不解析变量**：类似单引号，变量不会被替换
- **不解析转义序列**：除了 `\'` 和 `\\` 外，其他转义序列不会被解析
- **标识符规则**：标识符必须用单引号括起来

### 示例

```php
<?php
declare(strict_types=1);

$name = 'Alice';

$text = <<<'EOT'
Hello, $name!
This is a nowdoc string.
Variables like {$name} won't be parsed.
EOT;

echo $text;
// 输出：
// Hello, $name!
// This is a nowdoc string.
// Variables like {$name} won't be parsed.
```

### 实际应用

```php
<?php
declare(strict_types=1);

// 用于存储代码模板、SQL 查询等
$sql = <<<'SQL'
SELECT * FROM users 
WHERE status = 'active' 
AND created_at > NOW() - INTERVAL 30 DAY
SQL;

// 用于存储配置模板
$config = <<<'CONFIG'
{
    "app_name": "My Application",
    "version": "1.0.0"
}
CONFIG;
```

## 选择建议

| 方式     | 适用场景                           | 性能 |
| :------- | :--------------------------------- | :--- |
| 单引号   | 简单字符串，不需要变量插值         | 最快 |
| 双引号   | 需要变量插值或转义序列             | 较快 |
| Heredoc  | 多行字符串，需要变量插值           | 较慢 |
| Nowdoc   | 多行字符串，不需要变量插值（模板） | 较慢 |

## 完整示例

```php
<?php
declare(strict_types=1);

class StringExamples
{
    public static function demonstrate(): void
    {
        $name = 'Alice';
        $age = 25;
        
        echo "=== 单引号 ===\n";
        echo 'Hello, $name\n';  // 不解析变量和转义
        
        echo "\n=== 双引号 ===\n";
        echo "Hello, {$name}\n";  // 解析变量和转义
        echo "Age: {$age}\n";
        
        echo "\n=== Heredoc ===\n";
        $heredoc = <<<EOT
Multi-line string
Name: {$name}
Age: {$age}
EOT;
        echo $heredoc;
        
        echo "\n=== Nowdoc ===\n";
        $nowdoc = <<<'EOT'
Multi-line string
Name: $name
Age: $age
EOT;
        echo $nowdoc;
    }
}

StringExamples::demonstrate();
```

## 注意事项

1. **性能**：单引号性能最好，双引号次之，Heredoc/Nowdoc 最慢。

2. **可读性**：对于简单字符串，单引号或双引号即可；对于多行内容，使用 Heredoc/Nowdoc。

3. **安全性**：输出用户输入时，使用 `htmlspecialchars()` 转义。

4. **变量插值**：在双引号和 Heredoc 中，使用 `{$variable}` 语法更安全。

5. **标识符命名**：Heredoc/Nowdoc 的标识符应该使用有意义的名称。

## 练习

1. 创建一个函数，使用不同的字符串创建方式生成欢迎消息。

2. 编写一个函数，使用 Heredoc 生成 HTML 邮件模板。

3. 实现一个函数，使用 Nowdoc 存储 SQL 查询模板。

4. 创建一个函数，比较单引号和双引号的性能差异。

5. 编写一个函数，安全地处理用户输入并生成字符串。


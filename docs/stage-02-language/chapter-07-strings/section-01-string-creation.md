# 2.7.1 字符串创建方式

## 概述

PHP 提供了四种方式创建字符串：单引号、双引号、Heredoc、Nowdoc。每种方式有不同的特点和适用场景。本节详细介绍这四种方式的语法、特点、变量插值、转义序列及选择建议。

理解不同字符串创建方式的区别对于编写高效、可读的代码很重要。选择合适的创建方式可以提高代码性能和可维护性。

## 特性

- **单引号**：不解析转义（除 `\'`、`\\`），不解析变量，性能最好
- **双引号**：解析转义序列，解析变量插值，功能最全
- **Heredoc**：类似双引号，适合多行字符串，支持变量插值
- **Nowdoc**：类似单引号，适合多行字符串，不解析变量

## 语法/定义

### 单引号字符串

**语法**：`'string'`

**特点**：
- 不解析转义序列（除 `\'` 和 `\\`）
- 不解析变量
- 性能最好（无需解析）
- 适合简单字符串

**转义字符**：
- `\'`：单引号
- `\\`：反斜杠

### 双引号字符串

**语法**：`"string"`

**特点**：
- 解析转义序列
- 解析变量插值
- 功能最全
- 性能稍慢（需要解析）

**转义序列**：
- `\n`：换行符
- `\r`：回车符
- `\t`：制表符
- `\"`：双引号
- `\\`：反斜杠
- `\$`：美元符号
- `\0` 到 `\777`：八进制字符
- `\x[0-9A-Fa-f]{1,2}`：十六进制字符
- `\u{[0-9A-Fa-f]+}`：Unicode 字符（PHP 7.0+）

**变量插值**：
- 简单变量：`"Hello, $name"`
- 复杂表达式：`"Hello, {$name}"` 或 `"Hello, ${name}"`
- 数组/对象：`"Hello, {$user['name']}"` 或 `"Hello, {$user->name}"`

### Heredoc 语法

**语法**：
```php
<<<IDENTIFIER
string content
IDENTIFIER;
```

**特点**：
- 类似双引号字符串
- 解析转义序列
- 解析变量插值
- 适合多行字符串
- 结束标识符必须单独一行，不能有缩进

**要求**：
- 标识符必须遵循 PHP 变量命名规则
- 结束标识符必须单独一行
- 结束标识符前不能有空格或制表符
- 结束标识符后必须有分号

### Nowdoc 语法

**语法**：
```php
<<<'IDENTIFIER'
string content
IDENTIFIER;
```

**特点**：
- 类似单引号字符串
- 不解析转义序列（除 `\'` 和 `\\`）
- 不解析变量
- 适合多行字符串
- 结束标识符必须单独一行，不能有缩进

**要求**：
- 开始标识符必须用单引号括起来
- 其他要求与 Heredoc 相同

## 基本用法

### 示例 1：单引号字符串

```php
<?php
declare(strict_types=1);

// 基本使用
$str1 = 'Hello, World!';
echo $str1 . "\n";  // Hello, World!

// 转义单引号
$str2 = 'It\'s a string';
echo $str2 . "\n";  // It's a string

// 转义反斜杠
$str3 = 'Path: C:\\Windows\\System32';
echo $str3 . "\n";  // Path: C:\Windows\System32

// 不解析变量
$name = "Alice";
$str4 = 'Hello, $name';
echo $str4 . "\n";  // Hello, $name（不解析变量）

// 不解析转义序列
$str5 = 'Line 1\nLine 2';
echo $str5 . "\n";  // Line 1\nLine 2（不解析 \n）
```

**执行**：

```bash
php single-quote.php
```

**输出**：

```
Hello, World!
It's a string
Path: C:\Windows\System32
Hello, $name
Line 1\nLine 2
```

### 示例 2：双引号字符串

```php
<?php
declare(strict_types=1);

// 基本使用
$str1 = "Hello, World!";
echo $str1 . "\n";  // Hello, World!

// 变量插值
$name = "Alice";
$str2 = "Hello, $name";
echo $str2 . "\n";  // Hello, Alice

// 复杂变量插值
$user = ['name' => 'Bob', 'age' => 25];
$str3 = "User: {$user['name']}, Age: {$user['age']}";
echo $str3 . "\n";  // User: Bob, Age: 25

// 转义序列
$str4 = "Line 1\nLine 2\tTabbed";
echo $str4 . "\n";
// Line 1
// Line 2	Tabbed

// Unicode 字符（PHP 7.0+）
$str5 = "Unicode: \u{1F600}";
echo $str5 . "\n";  // Unicode: 😀
```

### 示例 3：Heredoc

```php
<?php
declare(strict_types=1);

$name = "Alice";
$age = 25;

// 基本 Heredoc
$str = <<<TEXT
Hello, {$name}!
You are {$age} years old.
This is a multi-line string.
TEXT;

echo $str . "\n";

// 在函数中使用
function generateEmail(string $name, string $message): string
{
    return <<<EMAIL
Dear {$name},

{$message}

Best regards,
System
EMAIL;
}

echo generateEmail("Bob", "Thank you for your registration.") . "\n";
```

### 示例 4：Nowdoc

```php
<?php
declare(strict_types=1);

$name = "Alice";  // 不会被解析

// 基本 Nowdoc
$str = <<<'TEXT'
Hello, World!
This is a multi-line string.
Variables like $name are not parsed.
TEXT;

echo $str . "\n";

// 用于模板或代码生成
$template = <<<'TEMPLATE'
class {ClassName}
{
    public function __construct()
    {
        // Constructor
    }
}
TEMPLATE;

echo $template . "\n";
```

## 完整代码示例

### 示例 1：字符串创建工具类

```php
<?php
declare(strict_types=1);

class StringBuilder
{
    private string $content = '';
    
    public function append(string $str): self
    {
        $this->content .= $str;
        return $this;
    }
    
    public function appendLine(string $str = ''): self
    {
        $this->content .= $str . "\n";
        return $this;
    }
    
    public function build(): string
    {
        return $this->content;
    }
    
    public function clear(): void
    {
        $this->content = '';
    }
}

// 使用
$builder = new StringBuilder();
$builder->append('Hello, ')
        ->append('World!')
        ->appendLine()
        ->appendLine('This is a new line.');

echo $builder->build();
```

### 示例 2：模板生成

```php
<?php
declare(strict_types=1);

function generateHtmlTemplate(string $title, string $content): string
{
    return <<<HTML
<!DOCTYPE html>
<html>
<head>
    <title>{$title}</title>
</head>
<body>
    <h1>{$title}</h1>
    <div>{$content}</div>
</body>
</html>
HTML;
}

function generateSqlQuery(string $table, array $columns): string
{
    $columnList = implode(', ', $columns);
    return <<<SQL
SELECT {$columnList}
FROM {$table}
WHERE id = :id
SQL;
}

// 使用
$html = generateHtmlTemplate("My Page", "Welcome to my page!");
echo $html . "\n";

$sql = generateSqlQuery("users", ["id", "name", "email"]);
echo $sql . "\n";
```

### 示例 3：变量插值对比

```php
<?php
declare(strict_types=1);

$name = "Alice";
$age = 25;

// 单引号：不解析变量
$str1 = 'Name: $name, Age: $age';
echo "Single quote: {$str1}\n";  // Name: $name, Age: $age

// 双引号：解析变量
$str2 = "Name: $name, Age: $age";
echo "Double quote: {$str2}\n";  // Name: Alice, Age: 25

// 复杂变量插值
$user = ['name' => 'Bob', 'age' => 30];
$str3 = "User: {$user['name']}, Age: {$user['age']}";
echo "Complex: {$str3}\n";  // User: Bob, Age: 30

// 表达式插值
$str4 = "Result: " . ($age * 2);
echo "Expression: {$str4}\n";  // Result: 50

// 方法调用插值
class User
{
    public function getName(): string
    {
        return "Charlie";
    }
}

$userObj = new User();
$str5 = "User: {$userObj->getName()}";
echo "Method: {$str5}\n";  // User: Charlie
```

## 使用场景

### 单引号

- **简单字符串**：不需要变量插值的简单字符串
- **性能敏感**：性能敏感的场景
- **字面量**：字符串字面量

### 双引号

- **变量插值**：需要变量插值的字符串
- **转义序列**：需要使用转义序列
- **动态内容**：动态生成的字符串

### Heredoc

- **多行字符串**：需要多行字符串
- **模板生成**：生成 HTML、SQL 等模板
- **文档字符串**：长文档字符串
- **需要变量插值**：多行且需要变量插值

### Nowdoc

- **多行字符串**：需要多行字符串
- **模板字面量**：不需要变量插值的模板
- **代码生成**：生成代码字符串
- **不需要变量插值**：多行但不需要变量插值

## 注意事项

### 性能考虑

- **单引号最快**：单引号性能最好，无需解析
- **双引号稍慢**：双引号需要解析转义和变量
- **Heredoc/Nowdoc**：性能与双引号/单引号类似
- **选择建议**：简单字符串优先使用单引号

### 转义规则

- **单引号**：只转义 `\'` 和 `\\`
- **双引号**：转义所有转义序列
- **Heredoc**：与双引号相同
- **Nowdoc**：与单引号相同

### 变量插值

- **单引号/Nowdoc**：不解析变量
- **双引号/Heredoc**：解析变量
- **复杂表达式**：使用 `{}` 括起来
- **数组/对象**：必须使用 `{}` 括起来

### Heredoc/Nowdoc 标识符

- **命名规则**：遵循 PHP 变量命名规则
- **结束标识符**：必须单独一行，不能有缩进
- **分号**：结束标识符后必须有分号
- **空格**：结束标识符前不能有空格或制表符

## 常见问题

### 问题 1：单引号内变量不解析

**症状**：单引号字符串中的变量不被解析

**原因**：单引号不解析变量

**错误示例**：

```php
<?php
declare(strict_types=1);

$name = "Alice";
$str = 'Hello, $name';  // 变量不解析
echo $str . "\n";  // Hello, $name
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$name = "Alice";

// 方法1：使用双引号
$str1 = "Hello, $name";
echo $str1 . "\n";  // Hello, Alice

// 方法2：使用字符串连接
$str2 = 'Hello, ' . $name;
echo $str2 . "\n";  // Hello, Alice

// 方法3：使用 Heredoc
$str3 = <<<TEXT
Hello, {$name}
TEXT;
echo $str3 . "\n";  // Hello, Alice
```

### 问题 2：Heredoc 结束标识符错误

**症状**：Heredoc 语法错误

**原因**：结束标识符格式不正确

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误1：结束标识符有缩进
$str = <<<TEXT
Hello, World!
    TEXT;  // 错误：有缩进

// 错误2：结束标识符前有空格
$str = <<<TEXT
Hello, World!
 TEXT;  // 错误：有空格

// 错误3：结束标识符后没有分号
$str = <<<TEXT
Hello, World!
TEXT  // 错误：没有分号
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 正确：结束标识符单独一行，无缩进，有分号
$str = <<<TEXT
Hello, World!
TEXT;

echo $str . "\n";
```

### 问题 3：复杂变量插值错误

**症状**：复杂变量插值失败

**原因**：没有使用 `{}` 括起来

**错误示例**：

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice'];

// 错误：数组访问需要 {}
$str = "User: $user['name']";  // 语法错误
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice'];

// 正确：使用 {} 括起来
$str = "User: {$user['name']}";
echo $str . "\n";  // User: Alice

// 或使用连接
$str = "User: " . $user['name'];
echo $str . "\n";  // User: Alice
```

## 最佳实践

### 选择创建方式

- **简单字符串**：使用单引号
- **需要变量插值**：使用双引号
- **多行且需要插值**：使用 Heredoc
- **多行且不需要插值**：使用 Nowdoc

### 性能优化

- **优先单引号**：简单字符串优先使用单引号
- **避免过度插值**：避免过度使用变量插值
- **缓存字符串**：重复使用的字符串进行缓存

### 代码可读性

- **明确意图**：选择合适的创建方式明确代码意图
- **使用 Heredoc/Nowdoc**：长字符串使用 Heredoc/Nowdoc
- **格式化代码**：保持代码格式整洁

## 对比分析

### 单引号 vs 双引号

| 特性 | 单引号 | 双引号 |
|:-----|:-------|:-------|
| 转义解析 | 否（除 `\'`、`\\`） | 是 |
| 变量插值 | 否 | 是 |
| 性能 | 最好 | 稍慢 |
| 推荐度 | 推荐（简单） | 推荐（需要插值） |

**选择建议**：
- **简单字符串**：使用单引号
- **需要插值**：使用双引号

### Heredoc vs Nowdoc

| 特性 | Heredoc | Nowdoc |
|:-----|:--------|:-------|
| 转义解析 | 是 | 否（除 `\'`、`\\`） |
| 变量插值 | 是 | 否 |
| 适用场景 | 多行+插值 | 多行+不插值 |
| 推荐度 | 推荐（多行+插值） | 推荐（多行+不插值） |

**选择建议**：
- **多行且需要插值**：使用 Heredoc
- **多行且不需要插值**：使用 Nowdoc

## 相关章节

- **2.4.1 标量类型**：了解字符串类型
- **2.6.2 赋值运算符**：了解字符串连接赋值
- **2.7.2 常用字符串函数**：了解字符串处理函数

## 练习任务

1. **字符串创建练习**：
   - 练习使用四种字符串创建方式
   - 理解不同方式的区别
   - 测试变量插值和转义序列
   - 观察性能差异

2. **变量插值练习**：
   - 练习简单变量插值
   - 练习复杂变量插值（数组、对象）
   - 练习表达式插值
   - 测试各种插值场景

3. **Heredoc/Nowdoc 练习**：
   - 练习使用 Heredoc 创建多行字符串
   - 练习使用 Nowdoc 创建多行字符串
   - 实现模板生成功能
   - 测试各种场景

4. **实际应用练习**：
   - 实现 HTML 模板生成
   - 实现 SQL 查询生成
   - 实现邮件模板生成
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个字符串工具类
   - 实现各种字符串创建方法
   - 优化性能和可读性
   - 进行代码审查，确保正确性

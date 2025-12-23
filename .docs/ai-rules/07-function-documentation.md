# 函数文档格式标准

## 概述

本文档定义了所有函数、方法、API 的文档格式标准。**所有函数文档必须严格遵守此标准。**

## 必需格式

每个函数文档必须包含以下三个部分，按顺序排列：

### 1. 语法（必需）

**格式**：
```markdown
**语法**：`functionName(type $param1, type $param2 = default): returnType`
```

**要求**：
- 使用反引号包裹完整函数签名
- 包含所有参数及其类型
- 包含返回值类型
- 参数默认值必须明确标注

### 2. 参数（必需）

**格式**：
```markdown
**参数**：
- `$param1`：参数说明
- `$param2`：可选，参数说明（如果是可选参数，必须标注"可选"）
```

**要求**：
- 每个参数必须单独列出
- 参数名使用反引号包裹
- 必须说明参数的作用和含义
- 可选参数必须标注"可选"或"默认值"
- 复杂参数需要详细说明

### 3. 返回值（必需）

**格式**：
```markdown
**返回值**：返回值的详细说明。
```

**要求**：
- 必须说明返回值的类型
- 必须说明返回值的含义
- 特殊情况必须说明（如返回 `false` 表示失败）
- 如果可能返回多种类型，必须说明所有情况

## 完整示例

```markdown
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
```

## 多参数函数示例

```markdown
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
```

## 可能返回多种类型的函数示例

```markdown
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
```
```

## 检查清单

在编写或审查函数文档时，必须确认：

- [ ] **语法**：包含完整函数签名，所有参数和返回值类型
- [ ] **参数**：每个参数都有说明，可选参数标注"可选"
- [ ] **返回值**：详细说明返回值类型和含义
- [ ] **格式一致**：所有函数使用相同的格式
- [ ] **内容完整**：没有遗漏任何参数

## 常见错误

### ❌ 错误示例 1：缺少参数说明

```markdown
### number_format() - 数字格式化

**语法**：`number_format(float $number, int $decimals = 0, ?string $decimal_separator = ".", ?string $thousands_separator = ","): string`

```php
// 缺少参数和返回值说明
```
```

### ❌ 错误示例 2：参数说明不完整

```markdown
**参数**：
- `$number`：要格式化的数字
// 缺少其他参数的说明
```

### ❌ 错误示例 3：缺少返回值说明

```markdown
**语法**：`strlen(string $string): int`

**参数**：
- `$string`：要计算长度的字符串

// 缺少返回值说明
```

## 适用范围

此标准适用于：

- PHP 内置函数
- 自定义函数
- 类方法
- API 方法
- 任何需要文档化的函数/方法

---

**最后更新**：2025-12-22

**重要提示**：所有函数文档必须包含语法、参数、返回值三个部分，缺一不可。

# 5.4.2 URL 编码与解码

## 概述

URL 编码是将特殊字符转换为 URL 安全格式的过程。由于 URL 中某些字符具有特殊含义（如 `?`、`&`、`=` 等），或者某些字符不能直接在 URL 中使用（如空格、中文字符等），需要进行编码处理。理解 URL 编码的概念、掌握编码和解码的方法，对于正确处理 URL 参数、构建安全的 URL、处理特殊字符等场景至关重要。本节详细介绍 URL 编码的概念、`urlencode()`、`urldecode()`、`rawurlencode()`、`rawurldecode()` 等函数的使用方法，帮助零基础学员掌握 URL 编码技术。

URL 编码遵循 RFC 3986 标准，将特殊字符转换为百分号编码（Percent-Encoding）格式，即 `%` 后跟两个十六进制数字。例如，空格可以编码为 `+`（在查询字符串中）或 `%20`（在路径中）。

**主要内容**：
- URL 编码的概念和必要性
- `urlencode()` 函数的语法、参数和使用方法
- `urldecode()` 函数的语法和使用方法
- `rawurlencode()` 函数与 `urlencode()` 的区别
- `rawurldecode()` 函数的使用方法
- 编码规则和字符分类
- 编码位置（查询字符串 vs 路径）
- 双重编码问题和解决方案
- 实际应用示例和最佳实践

## 特性

- **字符转换**：将特殊字符转换为 URL 安全格式
- **可逆操作**：编码和解码是可逆的
- **标准兼容**：遵循 RFC 3986 标准
- **位置相关**：查询字符串和路径使用不同的编码方式
- **字符集支持**：正确处理 UTF-8 字符

## URL 编码概念

### 为什么需要编码

URL 中某些字符具有特殊含义，不能直接使用：

| 字符 | 含义 | 是否需要编码 |
|:-----|:-----|:-------------|
| `?` | 查询字符串开始 | 在查询字符串中需要编码 |
| `&` | 参数分隔符 | 在查询字符串中需要编码 |
| `=` | 键值分隔符 | 在查询字符串中需要编码 |
| `#` | 片段标识符 | 需要编码 |
| `%` | 编码标识符 | 需要编码 |
| ` `（空格） | 空白字符 | 需要编码 |
| `/` | 路径分隔符 | 在路径中不需要编码，在查询字符串中需要编码 |

### 编码格式

URL 编码使用百分号编码（Percent-Encoding）格式：

- **格式**：`%XX`（XX 是两个十六进制数字）
- **示例**：空格编码为 `%20`，中文字符"中"编码为 `%E4%B8%AD`

### 字符分类

根据 RFC 3986，URL 字符分为：

1. **未保留字符**（Unreserved Characters）：不需要编码
   - 字母：`A-Z`、`a-z`
   - 数字：`0-9`
   - 特殊字符：`-`、`.`、`_`、`~`

2. **保留字符**（Reserved Characters）：在特定位置有特殊含义
   - `:`、`/`、`?`、`#`、`[`、`]`、`@`、`!`、`$`、`&`、`'`、`(`、`)`、`*`、`+`、`,`、`;`、`=`

3. **其他字符**：需要编码
   - 空格、中文字符、特殊符号等

## urlencode() 函数

### 语法

**语法**：`urlencode(string $string): string`

### 参数

- `$string`：要编码的字符串

### 返回值

返回编码后的字符串。

### 编码规则

- **空格**：编码为 `+`（在查询字符串中）
- **特殊字符**：编码为 `%XX` 格式
- **字母和数字**：不编码
- **未保留字符**：不编码

### 基本用法

**示例 1：基本编码**
```php
<?php
declare(strict_types=1);

$string = 'hello world';
$encoded = urlencode($string);
echo $encoded;  // hello+world
```

**示例 2：特殊字符编码**
```php
<?php
declare(strict_types=1);

$string = 'key=value&name=John Doe';
$encoded = urlencode($string);
echo $encoded;  // key%3Dvalue%26name%3DJohn+Doe
```

**示例 3：中文字符编码**
```php
<?php
declare(strict_types=1);

$string = '搜索 PHP 教程';
$encoded = urlencode($string);
echo $encoded;  // %E6%90%9C%E7%B4%A2+PHP+%E6%95%99%E7%A8%8B
```

### 使用场景

**查询字符串编码**：
```php
<?php
declare(strict_types=1);

$params = [
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'message' => 'Hello, world!',
];

$queryString = http_build_query($params);
// 或者手动编码
$queryString = 'name=' . urlencode($params['name']) . 
               '&email=' . urlencode($params['email']) . 
               '&message=' . urlencode($params['message']);

echo $queryString;
// name=John+Doe&email=john%40example.com&message=Hello%2C+world%21
```

## urldecode() 函数

### 语法

**语法**：`urldecode(string $string): string`

### 参数

- `$string`：要解码的字符串

### 返回值

返回解码后的字符串。

### 解码规则

- **`+`**：解码为空格
- **`%XX`**：解码为对应的字符

### 基本用法

**示例 1：基本解码**
```php
<?php
declare(strict_types=1);

$encoded = 'hello+world';
$decoded = urldecode($encoded);
echo $decoded;  // hello world
```

**示例 2：特殊字符解码**
```php
<?php
declare(strict_types=1);

$encoded = 'key%3Dvalue%26name%3DJohn+Doe';
$decoded = urldecode($encoded);
echo $decoded;  // key=value&name=John Doe
```

**示例 3：中文字符解码**
```php
<?php
declare(strict_types=1);

$encoded = '%E6%90%9C%E7%B4%A2+PHP+%E6%95%99%E7%A8%8B';
$decoded = urldecode($encoded);
echo $decoded;  // 搜索 PHP 教程
```

### 与 urlencode() 的配对使用

**示例**：
```php
<?php
declare(strict_types=1);

$original = 'Hello, World!';
$encoded = urlencode($original);
$decoded = urldecode($encoded);

echo "原始: {$original}\n";
echo "编码: {$encoded}\n";
echo "解码: {$decoded}\n";
```

**输出**：
```
原始: Hello, World!
编码: Hello%2C+World%21
解码: Hello, World!
```

## rawurlencode() 函数

### 语法

**语法**：`rawurlencode(string $string): string`

### 参数

- `$string`：要编码的字符串

### 返回值

返回编码后的字符串。

### 与 urlencode() 的区别

| 特性 | urlencode() | rawurlencode() |
|:-----|:------------|:---------------|
| 空格编码 | `+` | `%20` |
| 标准 | 遵循 application/x-www-form-urlencoded | 遵循 RFC 3986 |
| 使用场景 | 查询字符串 | URL 路径 |
| 兼容性 | 与 HTML 表单兼容 | 更标准 |

### 基本用法

**示例 1：空格编码差异**
```php
<?php
declare(strict_types=1);

$string = 'hello world';

$urlencoded = urlencode($string);
echo "urlencode: {$urlencoded}\n";  // hello+world

$rawEncoded = rawurlencode($string);
echo "rawurlencode: {$rawEncoded}\n";  // hello%20world
```

**示例 2：URL 路径编码**
```php
<?php
declare(strict_types=1);

$path = '/path/to/file with spaces.txt';
$encodedPath = rawurlencode($path);
echo $encodedPath;  // %2Fpath%2Fto%2Ffile%20with%20spaces.txt
```

**注意**：`rawurlencode()` 会编码 `/`，这在路径中通常不需要。对于路径编码，应该只编码路径段，而不是整个路径。

**正确的路径编码**：
```php
<?php
declare(strict_types=1);

function encodePath(string $path): string
{
    $segments = explode('/', $path);
    $encodedSegments = array_map('rawurlencode', $segments);
    return implode('/', $encodedSegments);
}

$path = '/path/to/file with spaces.txt';
$encodedPath = encodePath($path);
echo $encodedPath;  // /path/to/file%20with%20spaces.txt
```

## rawurldecode() 函数

### 语法

**语法**：`rawurldecode(string $string): string`

### 参数

- `$string`：要解码的字符串

### 返回值

返回解码后的字符串。

### 与 urldecode() 的区别

| 特性 | urldecode() | rawurldecode() |
|:-----|:------------|:----------------|
| `+` 处理 | 解码为空格 | 不处理（保持 `+`） |
| 使用场景 | 查询字符串 | URL 路径 |

### 基本用法

**示例**：
```php
<?php
declare(strict_types=1);

$encoded = 'hello%20world';
$decoded = rawurldecode($encoded);
echo $decoded;  // hello world

// + 不会被解码
$encoded2 = 'hello+world';
$decoded2 = rawurldecode($encoded2);
echo $decoded2;  // hello+world（+ 保持不变）
```

## 编码规则

### 查询字符串编码

**规则**：
- 使用 `urlencode()` 或 `http_build_query()`
- 空格编码为 `+`
- 与 HTML 表单兼容

**示例**：
```php
<?php
declare(strict_types=1);

$params = [
    'name' => 'John Doe',
    'email' => 'john@example.com',
];

// 方法 1：使用 http_build_query()（推荐）
$queryString = http_build_query($params);
echo $queryString;  // name=John+Doe&email=john%40example.com

// 方法 2：手动编码
$queryString = 'name=' . urlencode($params['name']) . 
               '&email=' . urlencode($params['email']);
echo $queryString;  // name=John+Doe&email=john%40example.com
```

### URL 路径编码

**规则**：
- 使用 `rawurlencode()` 编码路径段
- 空格编码为 `%20`
- 不编码路径分隔符 `/`

**示例**：
```php
<?php
declare(strict_types=1);

function encodeUrlPath(string $path): string
{
    $segments = explode('/', trim($path, '/'));
    $encodedSegments = array_map('rawurlencode', $segments);
    return '/' . implode('/', $encodedSegments);
}

$path = '/path/to/file with spaces.txt';
$encodedPath = encodeUrlPath($path);
echo $encodedPath;  // /path/to/file%20with%20spaces.txt
```

### 完整 URL 编码

**示例**：
```php
<?php
declare(strict_types=1);

function encodeUrl(string $url): string
{
    $parts = parse_url($url);
    
    if ($parts === false) {
        throw new InvalidArgumentException('Invalid URL');
    }
    
    // 编码路径段
    if (isset($parts['path'])) {
        $segments = explode('/', trim($parts['path'], '/'));
        $encodedSegments = array_map('rawurlencode', $segments);
        $parts['path'] = '/' . implode('/', $encodedSegments);
    }
    
    // 编码查询字符串
    if (isset($parts['query'])) {
        parse_str($parts['query'], $queryParams);
        $parts['query'] = http_build_query($queryParams);
    }
    
    // 构建 URL
    return buildUrl($parts);
}

// 使用
$url = 'https://example.com/path with spaces?name=John Doe';
$encodedUrl = encodeUrl($url);
echo $encodedUrl;  // https://example.com/path%20with%20spaces?name=John+Doe
```

## 双重编码问题

### 问题描述

对已经编码的字符串再次编码，会导致双重编码问题。

**示例**：
```php
<?php
declare(strict_types=1);

$original = 'Hello, World!';
$encoded1 = urlencode($original);
echo "第一次编码: {$encoded1}\n";  // Hello%2C+World%21

$encoded2 = urlencode($encoded1);
echo "第二次编码: {$encoded2}\n";  // Hello%252C%2BWorld%2521（错误！）
```

### 解决方案

**方法 1：检查是否已编码**
```php
<?php
declare(strict_types=1);

function safeUrlencode(string $string): string
{
    // 检查是否已经编码（包含 %XX 格式）
    if (preg_match('/%[0-9A-Fa-f]{2}/', $string)) {
        return $string;  // 已经编码，直接返回
    }
    
    return urlencode($string);
}

$original = 'Hello, World!';
$encoded1 = safeUrlencode($original);
$encoded2 = safeUrlencode($encoded1);  // 不会重复编码

echo "原始: {$original}\n";
echo "编码 1: {$encoded1}\n";
echo "编码 2: {$encoded2}\n";
```

**方法 2：先解码再编码**
```php
<?php
declare(strict_types=1);

function normalizeUrlencode(string $string): string
{
    // 先解码（如果已编码），再编码
    $decoded = urldecode($string);
    return urlencode($decoded);
}

$original = 'Hello, World!';
$encoded1 = urlencode($original);
$normalized = normalizeUrlencode($encoded1);

echo "原始: {$original}\n";
echo "编码: {$encoded1}\n";
echo "规范化: {$normalized}\n";
```

## 使用场景

### 查询字符串编码

- 构建 URL 查询参数
- 表单数据提交
- API 参数传递

### URL 路径编码

- 处理包含特殊字符的文件名
- 构建包含中文的 URL 路径
- SEO 友好的 URL

### 参数传递

- GET 请求参数编码
- POST 请求参数编码
- Cookie 值编码

### API 调用

- 构建 API 请求 URL
- 编码 API 参数
- 处理特殊字符

## 注意事项

### 编码一致性

- **查询字符串**：使用 `urlencode()` 或 `http_build_query()`
- **URL 路径**：使用 `rawurlencode()` 编码路径段
- **统一标准**：在项目中统一编码标准

### 双重编码问题

- **检查已编码**：避免对已编码的字符串再次编码
- **先解码再编码**：如果需要重新编码，先解码再编码
- **使用标准函数**：使用 `http_build_query()` 避免手动编码

### 字符集问题

- **UTF-8 编码**：确保字符串使用 UTF-8 编码
- **字符集转换**：必要时进行字符集转换
- **多字节字符**：正确处理多字节字符（如中文）

### 编码位置

- **查询字符串**：使用 `urlencode()`（空格变为 `+`）
- **URL 路径**：使用 `rawurlencode()`（空格变为 `%20`）
- **不要混用**：不要在查询字符串中使用 `rawurlencode()`

## 常见问题

### urlencode() 和 rawurlencode() 的区别？

| 特性 | urlencode() | rawurlencode() |
|:-----|:------------|:---------------|
| 空格编码 | `+` | `%20` |
| 标准 | application/x-www-form-urlencoded | RFC 3986 |
| 使用场景 | 查询字符串 | URL 路径 |
| `+` 处理 | `+` 编码为 `%2B` | `+` 编码为 `%2B` |

### 什么时候需要编码？

- **查询参数值**：包含特殊字符的参数值需要编码
- **URL 路径**：包含特殊字符的路径段需要编码
- **Cookie 值**：包含特殊字符的 Cookie 值需要编码
- **表单数据**：表单提交的数据会自动编码

### 如何避免双重编码？

1. **使用标准函数**：使用 `http_build_query()` 构建查询字符串
2. **检查已编码**：检查字符串是否已包含 `%XX` 格式
3. **先解码再编码**：如果需要重新编码，先解码再编码

### 编码后的 URL 如何解码？

- **查询字符串**：使用 `urldecode()` 解码
- **URL 路径**：使用 `rawurldecode()` 解码
- **自动解码**：PHP 会自动解码 `$_GET`、`$_POST` 等超全局变量

## 最佳实践

### 查询参数使用 urlencode()

- 使用 `http_build_query()` 构建查询字符串（推荐）
- 或使用 `urlencode()` 手动编码参数值
- 确保空格编码为 `+`

### URL 路径使用 rawurlencode()

- 使用 `rawurlencode()` 编码路径段
- 不编码路径分隔符 `/`
- 确保空格编码为 `%20`

### 避免双重编码

- 使用 `http_build_query()` 避免手动编码
- 检查字符串是否已编码
- 统一使用标准函数

### 统一编码标准

- 在项目中统一编码标准
- 查询字符串使用 `urlencode()`
- URL 路径使用 `rawurlencode()`
- 使用 `http_build_query()` 构建查询字符串

## 相关章节

- **[5.4.1 URL 解析与构建](section-01-url-parsing.md)**：了解 URL 解析的使用方法
- **[5.4.3 查询字符串处理](section-03-query-string.md)**：了解查询字符串的处理方法
- **[5.3.1 $_GET、$_POST 与 $_REQUEST](../chapter-03-superglobals/section-01-get-post-request.md)**：了解 URL 参数的处理

## 练习任务

1. **实现 URL 编码函数**
   - 创建函数，根据位置（查询字符串/路径）选择编码方式
   - 处理中文字符编码
   - 避免双重编码

2. **实现 URL 解码函数**
   - 创建函数，安全地解码 URL
   - 处理解码错误
   - 支持查询字符串和路径解码

3. **实现查询字符串构建**
   - 使用 `http_build_query()` 构建查询字符串
   - 处理嵌套数组参数
   - 处理特殊字符

4. **实现路径编码函数**
   - 编码 URL 路径段
   - 保持路径分隔符不变
   - 处理包含空格的路径

5. **实现完整的 URL 编码工具**
   - 解析 URL
   - 编码路径和查询字符串
   - 重新构建 URL

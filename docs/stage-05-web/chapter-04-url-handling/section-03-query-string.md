# 5.4.3 查询字符串处理

## 概述

查询字符串（Query String）是 URL 中用于传递参数的部分，通常位于 `?` 之后，多个参数之间使用 `&` 分隔。查询字符串处理是 Web 开发中的常见需求，包括构建查询字符串、解析查询字符串、处理数组参数等。理解查询字符串的格式、掌握构建和解析的方法，对于处理 URL 参数、构建 API 请求、实现分页等功能至关重要。本节详细介绍查询字符串的概念、`http_build_query()` 和 `parse_str()` 函数的使用方法，帮助零基础学员掌握查询字符串处理技术。

查询字符串的格式为 `key1=value1&key2=value2&key3=value3`，其中键值对使用 `=` 连接，多个键值对使用 `&` 分隔。PHP 提供了 `http_build_query()` 函数来构建查询字符串，`parse_str()` 函数来解析查询字符串。

**主要内容**：
- 查询字符串的概念和格式
- `http_build_query()` 函数的语法、参数和使用方法
- `parse_str()` 函数的语法、参数和使用方法
- 参数构建和解析
- 数组参数的处理
- 嵌套数组的处理
- 特殊字符的处理
- 实际应用示例和最佳实践

## 特性

- **自动编码**：`http_build_query()` 自动进行 URL 编码
- **数组支持**：支持一维和多维数组
- **类型保持**：解析时保持数据类型
- **灵活配置**：支持多种配置选项
- **安全解析**：可以解析到数组而不是创建变量

## 查询字符串概念

### 查询字符串格式

查询字符串的基本格式：

```
?key1=value1&key2=value2&key3=value3
```

**组成部分**：
- `?`：查询字符串开始标识符
- `key=value`：键值对
- `&`：参数分隔符

### 示例

**示例 1：简单查询字符串**
```
?name=John&age=30
```

**示例 2：包含特殊字符**
```
?search=hello+world&category=electronics
```

**示例 3：数组参数**
```
?tags[]=php&tags[]=web&tags[]=development
```

**示例 4：关联数组参数**
```
?user[name]=John&user[email]=john@example.com
```

## http_build_query() 函数

### 语法

**语法**：`http_build_query(array|object $data, string $numeric_prefix = "", ?string $arg_separator = null, int $encoding_type = PHP_QUERY_RFC1738): string`

### 参数

- `$data`：要构建查询字符串的数据（数组或对象）
- `$numeric_prefix`：可选，数字键的前缀
- `$arg_separator`：可选，参数分隔符（默认 `&`）
- `$encoding_type`：可选，编码类型（`PHP_QUERY_RFC1738` 或 `PHP_QUERY_RFC3986`）

### 返回值

返回构建的查询字符串（不包含 `?`）。

### 基本用法

**示例 1：简单数组**
```php
<?php
declare(strict_types=1);

$params = [
    'name' => 'John',
    'age' => 30,
    'city' => 'New York',
];

$queryString = http_build_query($params);
echo $queryString;
// 输出: name=John&age=30&city=New+York
```

**示例 2：包含特殊字符**
```php
<?php
declare(strict_types=1);

$params = [
    'search' => 'hello world',
    'email' => 'john@example.com',
    'message' => 'Hello, World!',
];

$queryString = http_build_query($params);
echo $queryString;
// 输出: search=hello+world&email=john%40example.com&message=Hello%2C+World%21
```

**示例 3：数字键前缀**
```php
<?php
declare(strict_types=1);

$params = [
    0 => 'first',
    1 => 'second',
    'key' => 'value',
];

$queryString = http_build_query($params, 'prefix_');
echo $queryString;
// 输出: prefix_0=first&prefix_1=second&key=value
```

### 数组参数处理

**示例 1：索引数组**
```php
<?php
declare(strict_types=1);

$params = [
    'tags' => ['php', 'web', 'development'],
];

$queryString = http_build_query($params);
echo $queryString;
// 输出: tags%5B0%5D=php&tags%5B1%5D=web&tags%5B2%5D=development
// 解码后: tags[0]=php&tags[1]=web&tags[2]=development
```

**示例 2：关联数组**
```php
<?php
declare(strict_types=1);

$params = [
    'user' => [
        'name' => 'John',
        'email' => 'john@example.com',
    ],
];

$queryString = http_build_query($params);
echo $queryString;
// 输出: user%5Bname%5D=John&user%5Bemail%5D=john%40example.com
// 解码后: user[name]=John&user[email]=john@example.com
```

**示例 3：多维数组**
```php
<?php
declare(strict_types=1);

$params = [
    'filters' => [
        'category' => ['electronics', 'computers'],
        'price' => [
            'min' => 100,
            'max' => 1000,
        ],
    ],
];

$queryString = http_build_query($params);
echo $queryString;
// 输出: filters[category][0]=electronics&filters[category][1]=computers&filters[price][min]=100&filters[price][max]=1000
```

### 编码类型

**PHP_QUERY_RFC1738**（默认）：
- 空格编码为 `+`
- 与 HTML 表单兼容

**PHP_QUERY_RFC3986**：
- 空格编码为 `%20`
- 遵循 RFC 3986 标准

**示例**：
```php
<?php
declare(strict_types=1);

$params = ['search' => 'hello world'];

$rfc1738 = http_build_query($params, '', '&', PHP_QUERY_RFC1738);
echo "RFC1738: {$rfc1738}\n";  // search=hello+world

$rfc3986 = http_build_query($params, '', '&', PHP_QUERY_RFC3986);
echo "RFC3986: {$rfc3986}\n";  // search=hello%20world
```

### 自定义分隔符

**示例**：
```php
<?php
declare(strict_types=1);

$params = ['name' => 'John', 'age' => 30];

// 使用 &amp; 作为分隔符（HTML 实体）
$queryString = http_build_query($params, '', '&amp;');
echo $queryString;  // name=John&amp;age=30
```

## parse_str() 函数

### 语法

**语法**：`parse_str(string $string, array &$result): void`

### 参数

- `$string`：要解析的查询字符串
- `$result`：可选，用于存储解析结果的数组（推荐使用）

### 返回值

无返回值（`void`）。如果提供了 `$result` 参数，解析结果存储在数组中；否则，解析结果作为变量创建在当前作用域。

### 基本用法

**示例 1：解析到数组（推荐）**
```php
<?php
declare(strict_types=1);

$queryString = 'name=John&age=30&city=New+York';
parse_str($queryString, $result);

print_r($result);
```

**输出**：
```
Array
(
    [name] => John
    [age] => 30
    [city] => New York
)
```

**示例 2：创建变量（不推荐）**
```php
<?php
declare(strict_types=1);

$queryString = 'name=John&age=30';
parse_str($queryString);  // 不提供 $result 参数

echo $name;  // John
echo $age;   // 30
```

**注意**：不推荐使用这种方式，因为会污染当前作用域的变量。

### 数组参数解析

**示例 1：索引数组**
```php
<?php
declare(strict_types=1);

$queryString = 'tags[0]=php&tags[1]=web&tags[2]=development';
parse_str($queryString, $result);

print_r($result);
```

**输出**：
```
Array
(
    [tags] => Array
        (
            [0] => php
            [1] => web
            [2] => development
        )
)
```

**示例 2：关联数组**
```php
<?php
declare(strict_types=1);

$queryString = 'user[name]=John&user[email]=john@example.com';
parse_str($queryString, $result);

print_r($result);
```

**输出**：
```
Array
(
    [user] => Array
        (
            [name] => John
            [email] => john@example.com
        )
)
```

**示例 3：多维数组**
```php
<?php
declare(strict_types=1);

$queryString = 'filters[category][0]=electronics&filters[category][1]=computers&filters[price][min]=100&filters[price][max]=1000';
parse_str($queryString, $result);

print_r($result);
```

**输出**：
```
Array
(
    [filters] => Array
        (
            [category] => Array
                (
                    [0] => electronics
                    [1] => computers
                )
            [price] => Array
                (
                    [min] => 100
                    [max] => 1000
                )
        )
)
```

### 安全使用

**推荐方式**（解析到数组）：
```php
<?php
declare(strict_types=1);

$queryString = $_SERVER['QUERY_STRING'] ?? '';
parse_str($queryString, $params);

// 安全访问参数
$name = $params['name'] ?? '';
$age = (int) ($params['age'] ?? 0);
```

**不推荐方式**（创建变量）：
```php
<?php
// 危险：会污染作用域，可能覆盖现有变量
parse_str($_SERVER['QUERY_STRING']);
```

## 参数构建和解析

### 构建查询字符串

**示例**：
```php
<?php
declare(strict_types=1);

class QueryStringBuilder
{
    private array $params = [];

    public function add(string $key, mixed $value): self
    {
        $this->params[$key] = $value;
        return $this;
    }

    public function remove(string $key): self
    {
        unset($this->params[$key]);
        return $this;
    }

    public function build(): string
    {
        return http_build_query($this->params);
    }

    public function buildUrl(string $baseUrl): string
    {
        $queryString = $this->build();
        return $baseUrl . '?' . $queryString;
    }
}

// 使用
$builder = new QueryStringBuilder();
$builder->add('page', 1)
        ->add('limit', 10)
        ->add('search', 'php');

$url = $builder->buildUrl('https://example.com/api/users');
echo $url;  // https://example.com/api/users?page=1&limit=10&search=php
```

### 解析查询字符串

**示例**：
```php
<?php
declare(strict_types=1);

class QueryStringParser
{
    public static function parse(string $queryString): array
    {
        $params = [];
        parse_str($queryString, $params);
        return $params;
    }

    public static function parseUrl(string $url): array
    {
        $parts = parse_url($url);
        if ($parts === false || !isset($parts['query'])) {
            return [];
        }
        return self::parse($parts['query']);
    }

    public static function get(string $queryString, string $key, mixed $default = null): mixed
    {
        $params = self::parse($queryString);
        return $params[$key] ?? $default;
    }
}

// 使用
$queryString = 'name=John&age=30';
$params = QueryStringParser::parse($queryString);
print_r($params);

$name = QueryStringParser::get($queryString, 'name', 'Unknown');
echo $name;  // John
```

### 合并查询参数

**示例**：
```php
<?php
declare(strict_types=1);

function mergeQueryParams(string $baseQuery, array $newParams): string
{
    // 解析现有查询字符串
    parse_str($baseQuery, $existingParams);
    
    // 合并新参数
    $mergedParams = array_merge($existingParams, $newParams);
    
    // 构建新的查询字符串
    return http_build_query($mergedParams);
}

// 使用
$baseQuery = 'page=1&limit=10';
$newParams = ['search' => 'php', 'sort' => 'name'];
$mergedQuery = mergeQueryParams($baseQuery, $newParams);
echo $mergedQuery;  // page=1&limit=10&search=php&sort=name
```

## 使用场景

### URL 参数构建

- 构建分页 URL
- 构建搜索 URL
- 构建筛选 URL

### 表单数据处理

- 处理 GET 表单数据
- 构建表单提交 URL
- 处理表单数组字段

### API 参数传递

- 构建 API 请求 URL
- 处理 API 查询参数
- 传递复杂参数结构

### 分页参数处理

- 构建分页链接
- 处理分页参数
- 保持其他参数

## 注意事项

### 参数编码

- **自动编码**：`http_build_query()` 自动进行 URL 编码
- **特殊字符**：特殊字符会被正确编码
- **编码一致性**：确保编码和解码使用相同的标准

### 数组参数格式

- **索引数组**：`tags[0]=value1&tags[1]=value2`
- **关联数组**：`user[name]=John&user[email]=john@example.com`
- **多维数组**：支持任意深度的嵌套

### 特殊字符处理

- **空格**：编码为 `+`（RFC1738）或 `%20`（RFC3986）
- **特殊字符**：自动编码为 `%XX` 格式
- **保留字符**：`&`、`=` 等会被编码

### 安全性考虑

- **使用数组参数**：`parse_str()` 时始终提供 `$result` 参数
- **验证参数**：解析后验证参数类型和值
- **防止变量污染**：避免使用 `parse_str()` 创建变量

## 常见问题

### 如何构建查询字符串？

使用 `http_build_query()` 函数：

```php
<?php
declare(strict_types=1);

$params = ['name' => 'John', 'age' => 30];
$queryString = http_build_query($params);
echo $queryString;  // name=John&age=30
```

### 如何解析查询字符串？

使用 `parse_str()` 函数，并提供 `$result` 参数：

```php
<?php
declare(strict_types=1);

$queryString = 'name=John&age=30';
parse_str($queryString, $params);
print_r($params);
```

### 如何处理数组参数？

`http_build_query()` 和 `parse_str()` 都支持数组参数：

```php
<?php
declare(strict_types=1);

// 构建
$params = ['tags' => ['php', 'web']];
$queryString = http_build_query($params);
// tags[0]=php&tags[1]=web

// 解析
parse_str($queryString, $result);
print_r($result);
```

### parse_str() 的安全问题？

`parse_str()` 如果不提供 `$result` 参数，会在当前作用域创建变量，可能：
- 污染作用域
- 覆盖现有变量
- 导致安全问题

**解决方案**：始终提供 `$result` 参数。

## 最佳实践

### 使用 http_build_query() 构建

- 使用 `http_build_query()` 构建查询字符串
- 自动处理编码和数组
- 支持多种配置选项

### 使用 parse_str() 解析到数组

- 始终提供 `$result` 参数
- 避免创建变量污染作用域
- 解析后验证参数

### 注意参数编码

- 使用默认编码（RFC1738）与 HTML 表单兼容
- 需要时使用 RFC3986 编码
- 确保编码和解码使用相同标准

### 验证解析结果

- 验证参数类型
- 验证参数值
- 提供默认值

## 相关章节

- **[5.4.1 URL 解析与构建](section-01-url-parsing.md)**：了解 URL 解析的使用方法
- **[5.4.2 URL 编码与解码](section-02-url-encoding.md)**：了解 URL 编码的使用方法
- **[5.3.1 $_GET、$_POST 与 $_REQUEST](../chapter-03-superglobals/section-01-get-post-request.md)**：了解 URL 参数的处理

## 练习任务

1. **实现查询字符串构建器**
   - 创建类，支持添加、删除参数
   - 支持数组参数
   - 构建完整 URL

2. **实现查询字符串解析器**
   - 解析查询字符串到数组
   - 支持数组参数解析
   - 提供安全的参数访问方法

3. **实现参数合并功能**
   - 合并多个查询字符串
   - 处理参数冲突
   - 保持参数顺序

4. **实现分页 URL 构建**
   - 构建分页链接
   - 保持其他参数
   - 处理当前页码

5. **实现查询参数验证**
   - 验证参数类型
   - 验证参数值
   - 提供默认值

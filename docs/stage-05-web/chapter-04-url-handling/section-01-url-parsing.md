# 5.4.1 URL 解析与构建

## 概述

URL（Uniform Resource Locator，统一资源定位符）是 Web 开发中最重要的概念之一。URL 解析是将 URL 分解为各个组成部分的过程，URL 构建则是将各个组件组合成完整 URL 的过程。理解 URL 的结构、掌握 URL 解析和构建的方法，对于处理 Web 请求、实现路由、构建 API 等场景至关重要。本节详细介绍 URL 的结构、`parse_url()` 函数的使用、URL 组件的获取、URL 构建方法等内容，帮助零基础学员掌握 URL 处理技术。

在 Web 开发中，我们经常需要解析 URL 获取其中的各个组件（如协议、主机、路径、查询参数等），或者根据各个组件构建完整的 URL。PHP 提供了 `parse_url()` 函数来解析 URL，也提供了多种方法来构建 URL。

**主要内容**：
- URL 结构概述（scheme、host、path、query、fragment 等）
- `parse_url()` 函数的语法、参数和返回值
- URL 组件的获取和访问
- URL 构建方法（手动构建、使用组件构建）
- 相对 URL 和绝对 URL 的处理
- URL 验证和错误处理
- 实际应用示例和最佳实践

## 特性

- **组件解析**：可以将 URL 分解为各个组成部分
- **灵活构建**：可以根据组件灵活构建 URL
- **类型安全**：返回关联数组，类型明确
- **错误处理**：解析失败时返回 `false`
- **编码支持**：正确处理 URL 编码的字符

## URL 结构

### URL 组成部分

一个完整的 URL 由以下部分组成：

```
https://user:pass@example.com:8080/path/to/resource?key1=value1&key2=value2#fragment
│     │   │    │   │            │    │                    │                    │
│     │   │    │   │            │    │                    │                    └─ fragment（片段）
│     │   │    │   │            │    │                    └─ query（查询字符串）
│     │   │    │   │            │    └─ path（路径）
│     │   │    │   │            └─ port（端口）
│     │   │    │   └─ host（主机）
│     │   │    └─ password（密码）
│     │   └─ user（用户名）
│     └─ scheme（协议）
```

### URL 组件说明

| 组件 | 说明 | 示例 | 是否必需 |
|:-----|:-----|:-----|:---------|
| `scheme` | 协议（http、https、ftp 等） | `https` | 通常必需 |
| `user` | 用户名 | `admin` | 可选 |
| `pass` | 密码 | `secret` | 可选 |
| `host` | 主机名或 IP 地址 | `example.com` | 必需 |
| `port` | 端口号 | `8080` | 可选 |
| `path` | 路径 | `/path/to/resource` | 可选 |
| `query` | 查询字符串 | `key1=value1&key2=value2` | 可选 |
| `fragment` | 片段（锚点） | `section1` | 可选 |

### 常见 URL 示例

**示例 1：简单 URL**
```
https://example.com
```
- scheme: `https`
- host: `example.com`

**示例 2：带路径的 URL**
```
https://example.com/path/to/page
```
- scheme: `https`
- host: `example.com`
- path: `/path/to/page`

**示例 3：带查询参数的 URL**
```
https://example.com/search?q=php&page=1
```
- scheme: `https`
- host: `example.com`
- path: `/search`
- query: `q=php&page=1`

**示例 4：完整 URL**
```
https://user:pass@example.com:8080/path?key=value#fragment
```
- scheme: `https`
- user: `user`
- pass: `pass`
- host: `example.com`
- port: `8080`
- path: `/path`
- query: `key=value`
- fragment: `fragment`

## parse_url() 函数

### 语法

**语法**：`parse_url(string $url, int $component = -1): array|string|int|null|false`

### 参数

- `$url`：要解析的 URL 字符串
- `$component`：可选，指定要返回的组件（`PHP_URL_SCHEME`、`PHP_URL_HOST`、`PHP_URL_PATH` 等）。如果指定，返回该组件的值（字符串或整数）；如果未指定（默认 -1），返回包含所有组件的关联数组

### 返回值

- **指定组件**：返回该组件的值（字符串、整数或 `null`）
- **未指定组件**：返回关联数组，包含所有解析出的组件
- **解析失败**：返回 `false`

### 基本用法

**示例 1：解析完整 URL**
```php
<?php
declare(strict_types=1);

$url = 'https://user:pass@example.com:8080/path/to/resource?key1=value1&key2=value2#fragment';
$parts = parse_url($url);

print_r($parts);
```

**输出**：
```
Array
(
    [scheme] => https
    [host] => example.com
    [port] => 8080
    [user] => user
    [pass] => pass
    [path] => /path/to/resource
    [query] => key1=value1&key2=value2
    [fragment] => fragment
)
```

**示例 2：解析简单 URL**
```php
<?php
declare(strict_types=1);

$url = 'https://example.com/path';
$parts = parse_url($url);

print_r($parts);
```

**输出**：
```
Array
(
    [scheme] => https
    [host] => example.com
    [path] => /path
)
```

**示例 3：获取特定组件**
```php
<?php
declare(strict_types=1);

$url = 'https://example.com/path?key=value';

// 获取协议
$scheme = parse_url($url, PHP_URL_SCHEME);
echo "协议: {$scheme}\n";  // https

// 获取主机
$host = parse_url($url, PHP_URL_HOST);
echo "主机: {$host}\n";  // example.com

// 获取路径
$path = parse_url($url, PHP_URL_PATH);
echo "路径: {$path}\n";  // /path

// 获取查询字符串
$query = parse_url($url, PHP_URL_QUERY);
echo "查询: {$query}\n";  // key=value

// 获取端口
$port = parse_url($url, PHP_URL_PORT);
echo "端口: " . ($port ?? '默认') . "\n";  // 默认
```

### 组件常量

| 常量 | 值 | 说明 |
|:-----|:---|:-----|
| `PHP_URL_SCHEME` | 1 | 协议 |
| `PHP_URL_HOST` | 2 | 主机 |
| `PHP_URL_PORT` | 3 | 端口 |
| `PHP_URL_USER` | 4 | 用户名 |
| `PHP_URL_PASS` | 5 | 密码 |
| `PHP_URL_PATH` | 6 | 路径 |
| `PHP_URL_QUERY` | 7 | 查询字符串 |
| `PHP_URL_FRAGMENT` | 8 | 片段 |

### 访问组件

**示例**：
```php
<?php
declare(strict_types=1);

$url = 'https://example.com/path?key=value#fragment';
$parts = parse_url($url);

// 安全访问组件（使用 ?? 提供默认值）
$scheme = $parts['scheme'] ?? 'http';
$host = $parts['host'] ?? '';
$path = $parts['path'] ?? '/';
$query = $parts['query'] ?? '';
$fragment = $parts['fragment'] ?? '';

echo "协议: {$scheme}\n";
echo "主机: {$host}\n";
echo "路径: {$path}\n";
echo "查询: {$query}\n";
echo "片段: {$fragment}\n";
```

### 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

function parseUrlSafe(string $url): ?array
{
    $parts = parse_url($url);
    
    if ($parts === false) {
        return null;  // 解析失败
    }
    
    return $parts;
}

$url = 'not a valid url';
$parts = parseUrlSafe($url);

if ($parts === null) {
    echo "URL 解析失败\n";
} else {
    print_r($parts);
}
```

## URL 构建

### 方法一：手动构建

根据各个组件手动拼接 URL。

**示例**：
```php
<?php
declare(strict_types=1);

function buildUrl(array $parts): string
{
    $url = '';
    
    // 协议
    if (isset($parts['scheme'])) {
        $url .= $parts['scheme'] . '://';
    }
    
    // 用户和密码
    if (isset($parts['user'])) {
        $url .= $parts['user'];
        if (isset($parts['pass'])) {
            $url .= ':' . $parts['pass'];
        }
        $url .= '@';
    }
    
    // 主机
    if (isset($parts['host'])) {
        $url .= $parts['host'];
    }
    
    // 端口
    if (isset($parts['port'])) {
        $url .= ':' . $parts['port'];
    }
    
    // 路径
    if (isset($parts['path'])) {
        $url .= $parts['path'];
    }
    
    // 查询字符串
    if (isset($parts['query'])) {
        $url .= '?' . $parts['query'];
    }
    
    // 片段
    if (isset($parts['fragment'])) {
        $url .= '#' . $parts['fragment'];
    }
    
    return $url;
}

// 使用
$parts = [
    'scheme' => 'https',
    'host' => 'example.com',
    'path' => '/path',
    'query' => 'key=value',
];

$url = buildUrl($parts);
echo $url;  // https://example.com/path?key=value
```

### 方法二：使用现有组件修改

解析现有 URL，修改组件后重新构建。

**示例**：
```php
<?php
declare(strict_types=1);

function modifyUrl(string $url, array $modifications): string
{
    $parts = parse_url($url);
    
    if ($parts === false) {
        throw new InvalidArgumentException('Invalid URL');
    }
    
    // 应用修改
    foreach ($modifications as $key => $value) {
        if ($value === null) {
            unset($parts[$key]);
        } else {
            $parts[$key] = $value;
        }
    }
    
    // 重新构建 URL
    return buildUrl($parts);
}

// 使用
$originalUrl = 'https://example.com/path?old=value';
$newUrl = modifyUrl($originalUrl, [
    'path' => '/new/path',
    'query' => 'new=value',
]);

echo $newUrl;  // https://example.com/new/path?new=value
```

### 方法三：使用 http_build_url()（需要 PECL pecl_http）

如果安装了 `pecl_http` 扩展，可以使用 `http_build_url()` 函数。

**示例**：
```php
<?php
// 需要安装 pecl_http 扩展
if (function_exists('http_build_url')) {
    $url = http_build_url([
        'scheme' => 'https',
        'host' => 'example.com',
        'path' => '/path',
        'query' => 'key=value',
    ]);
}
```

## 相对 URL 和绝对 URL

### 相对 URL

相对 URL 不包含协议和主机，相对于当前页面。

**示例**：
```php
<?php
declare(strict_types=1);

// 相对 URL
$relativeUrl = '/path/to/page';
$parts = parse_url($relativeUrl);

print_r($parts);
```

**输出**：
```
Array
(
    [path] => /path/to/page
)
```

### 绝对 URL

绝对 URL 包含完整的协议和主机信息。

**示例**：
```php
<?php
declare(strict_types=1);

// 绝对 URL
$absoluteUrl = 'https://example.com/path/to/page';
$parts = parse_url($absoluteUrl);

print_r($parts);
```

**输出**：
```
Array
(
    [scheme] => https
    [host] => example.com
    [path] => /path/to/page
)
```

### 相对 URL 转绝对 URL

**示例**：
```php
<?php
declare(strict_types=1);

function resolveUrl(string $baseUrl, string $relativeUrl): string
{
    // 如果已经是绝对 URL，直接返回
    if (parse_url($relativeUrl, PHP_URL_SCHEME) !== null) {
        return $relativeUrl;
    }
    
    $baseParts = parse_url($baseUrl);
    if ($baseParts === false) {
        throw new InvalidArgumentException('Invalid base URL');
    }
    
    $relativeParts = parse_url($relativeUrl);
    if ($relativeParts === false) {
        throw new InvalidArgumentException('Invalid relative URL');
    }
    
    // 合并组件
    $resolvedParts = $baseParts;
    
    if (isset($relativeParts['path'])) {
        if (str_starts_with($relativeParts['path'], '/')) {
            // 绝对路径
            $resolvedParts['path'] = $relativeParts['path'];
        } else {
            // 相对路径，需要合并
            $basePath = dirname($baseParts['path'] ?? '/');
            $resolvedParts['path'] = rtrim($basePath, '/') . '/' . $relativeParts['path'];
        }
    }
    
    if (isset($relativeParts['query'])) {
        $resolvedParts['query'] = $relativeParts['query'];
    }
    
    if (isset($relativeParts['fragment'])) {
        $resolvedParts['fragment'] = $relativeParts['fragment'];
    }
    
    return buildUrl($resolvedParts);
}

// 使用
$baseUrl = 'https://example.com/path/to/page';
$relativeUrl = '../other/page';
$absoluteUrl = resolveUrl($baseUrl, $relativeUrl);
echo $absoluteUrl;  // https://example.com/path/other/page
```

## 使用场景

### URL 验证

验证 URL 格式是否正确。

**示例**：
```php
<?php
declare(strict_types=1);

function isValidUrl(string $url): bool
{
    $parts = parse_url($url);
    
    if ($parts === false) {
        return false;
    }
    
    // 检查必需组件
    if (!isset($parts['scheme']) || !isset($parts['host'])) {
        return false;
    }
    
    // 检查协议
    $validSchemes = ['http', 'https', 'ftp', 'ftps'];
    if (!in_array($parts['scheme'], $validSchemes, true)) {
        return false;
    }
    
    return true;
}

// 使用
$url = 'https://example.com';
echo isValidUrl($url) ? '有效' : '无效';  // 有效
```

### URL 重定向

构建重定向 URL。

**示例**：
```php
<?php
declare(strict_types=1);

function buildRedirectUrl(string $baseUrl, string $path, array $params = []): string
{
    $parts = parse_url($baseUrl);
    
    if ($parts === false) {
        throw new InvalidArgumentException('Invalid base URL');
    }
    
    $parts['path'] = $path;
    
    if (!empty($params)) {
        $parts['query'] = http_build_query($params);
    }
    
    return buildUrl($parts);
}

// 使用
$baseUrl = 'https://example.com';
$redirectUrl = buildRedirectUrl($baseUrl, '/login', ['redirect' => '/dashboard']);
echo $redirectUrl;  // https://example.com/login?redirect=%2Fdashboard
```

### API 调用

构建 API 请求 URL。

**示例**：
```php
<?php
declare(strict_types=1);

class ApiUrlBuilder
{
    public function __construct(
        private string $baseUrl
    ) {}
    
    public function build(string $endpoint, array $params = []): string
    {
        $parts = parse_url($this->baseUrl);
        
        if ($parts === false) {
            throw new InvalidArgumentException('Invalid base URL');
        }
        
        // 合并路径
        $basePath = rtrim($parts['path'] ?? '', '/');
        $endpoint = ltrim($endpoint, '/');
        $parts['path'] = $basePath . '/' . $endpoint;
        
        // 添加查询参数
        if (!empty($params)) {
            $existingQuery = $parts['query'] ?? '';
            $newQuery = http_build_query($params);
            $parts['query'] = $existingQuery ? $existingQuery . '&' . $newQuery : $newQuery;
        }
        
        return buildUrl($parts);
    }
}

// 使用
$builder = new ApiUrlBuilder('https://api.example.com/v1');
$url = $builder->build('users', ['page' => 1, 'limit' => 10]);
echo $url;  // https://api.example.com/v1/users?page=1&limit=10
```

### 链接生成

生成页面链接。

**示例**：
```php
<?php
declare(strict_types=1);

function generateLink(string $baseUrl, string $path, array $query = []): string
{
    $parts = parse_url($baseUrl);
    
    if ($parts === false) {
        throw new InvalidArgumentException('Invalid base URL');
    }
    
    $parts['path'] = $path;
    
    if (!empty($query)) {
        $parts['query'] = http_build_query($query);
    }
    
    return buildUrl($parts);
}

// 使用
$baseUrl = 'https://example.com';
$link = generateLink($baseUrl, '/products', ['category' => 'electronics', 'page' => 1]);
echo $link;  // https://example.com/products?category=electronics&page=1
```

## 注意事项

### URL 格式验证

- **验证解析结果**：检查 `parse_url()` 是否返回 `false`
- **检查必需组件**：确保 URL 包含必需的组件（如 scheme、host）
- **验证协议**：确保协议是允许的（http、https 等）

### 组件缺失处理

- **使用默认值**：使用 `??` 运算符提供默认值
- **检查存在性**：使用 `isset()` 检查组件是否存在
- **处理 null 值**：某些组件可能为 `null`

### 编码问题

- **URL 编码**：查询字符串和路径中的特殊字符需要编码
- **字符集**：确保使用正确的字符集（UTF-8）
- **双重编码**：避免对已编码的 URL 再次编码

### 安全性检查

- **协议验证**：只允许安全的协议（https）
- **主机验证**：验证主机名是否可信
- **路径验证**：防止路径遍历攻击（`../`）

## 常见问题

### parse_url() 返回什么？

`parse_url()` 返回关联数组，包含解析出的 URL 组件。如果指定了 `$component` 参数，返回该组件的值。如果解析失败，返回 `false`。

### 如何构建 URL？

可以使用以下方法：
1. **手动构建**：根据组件手动拼接
2. **修改现有 URL**：解析现有 URL，修改组件后重新构建
3. **使用扩展**：使用 `pecl_http` 扩展的 `http_build_url()` 函数

### 如何处理相对 URL？

相对 URL 不包含协议和主机。可以使用 `resolveUrl()` 函数将相对 URL 转换为绝对 URL，或者根据当前页面的 URL 解析相对 URL。

### URL 解析失败怎么办？

检查 `parse_url()` 的返回值：
- 如果返回 `false`，说明 URL 格式无效
- 检查 URL 格式是否符合标准
- 验证 URL 是否包含必需的组件

## 最佳实践

### 验证 URL 格式

- 使用 `parse_url()` 验证 URL 格式
- 检查必需组件是否存在
- 验证协议和主机

### 处理缺失组件

- 使用 `??` 运算符提供默认值
- 检查组件是否存在
- 处理 `null` 值

### 使用安全的 URL 构建方法

- 使用 `http_build_query()` 构建查询字符串
- 使用 `rawurlencode()` 编码路径
- 避免手动拼接 URL

### 注意编码问题

- 查询参数使用 `urlencode()` 或 `http_build_query()`
- 路径使用 `rawurlencode()`
- 避免双重编码

## 相关章节

- **[5.4.2 URL 编码与解码](section-02-url-encoding.md)**：了解 URL 编码的使用方法
- **[5.4.3 查询字符串处理](section-03-query-string.md)**：了解查询字符串的处理方法
- **[5.3.1 $_GET、$_POST 与 $_REQUEST](../chapter-03-superglobals/section-01-get-post-request.md)**：了解 URL 参数的处理

## 练习任务

1. **实现 URL 解析函数**
   - 创建一个函数，解析 URL 并返回所有组件
   - 处理解析失败的情况
   - 提供默认值

2. **实现 URL 构建函数**
   - 创建一个函数，根据组件构建 URL
   - 支持所有 URL 组件
   - 处理缺失组件

3. **实现 URL 验证函数**
- 验证 URL 格式
   - 检查必需组件
   - 验证协议和主机

4. **实现相对 URL 解析**
   - 将相对 URL 转换为绝对 URL
   - 处理路径合并
   - 处理查询参数合并

5. **实现 API URL 构建器**
   - 创建 API URL 构建类
   - 支持端点路径和查询参数
   - 支持版本控制

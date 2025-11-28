# 2.7 字符串操作

## 目标

- 掌握 PHP 字符串的创建、插值、转义与多字节处理。
- 熟悉常用字符串函数及其语法、参数与返回值。
- 能在处理 UTF-8、JSON、URL、模板时避免编码错误。

## 字符串创建方式

| 形式     | 示例                         | 特性 / 说明                  |
| :------- | :--------------------------- | :--------------------------- |
| 单引号   | `'Hello\n'`                  | 不解析转义，`\'`、`\\` 例外 |
| 双引号   | `"Hello\n{$name}"`          | 支持转义与变量插值          |
| Heredoc  | `<<<TEXT ... TEXT;`         | 类似双引号，可插值          |
| Nowdoc   | `<<<'TEXT' ... TEXT;`       | 类似单引号，不解析变量      |

示例：

```php
$name = "PHP";
echo "Hello, {$name}\n";
```

## 转义与安全

- `addslashes()`：添加反斜线（避免 SQL 注入需使用 PDO 预处理替代）。
- `htmlspecialchars($str, ENT_QUOTES, 'UTF-8')`：输出 HTML 前进行转义。
- `preg_quote($pattern)`：在正则表达式中转义特殊字符。

## 多字节字符串

- 启用 `mbstring` 扩展，设置默认编码：`mb_internal_encoding('UTF-8');`
- 常用函数：
  - `mb_strlen($str)`：字符长度。
  - `mb_substr($str, $start, $length)`：截取子串。
  - `mb_strpos($haystack, $needle)`：查找位置。

示例：

```php
mb_internal_encoding('UTF-8');
$title = '全栈实战';
echo mb_substr($title, 0, 2); // 全栈
```

## 常用字符串函数

| 函数        | 语法                                                       | 说明                            |
| :---------- | :--------------------------------------------------------- | :------------------------------ |
| `strlen`    | `strlen(string $string): int`                              | 返回字节长度（非字符数）        |
| `trim`      | `trim(string $string, string $characters = " \n\r\t\v\0"): string` | 去掉首尾空白或指定字符 |
| `strpos`    | `strpos(string $haystack, string $needle, int $offset = 0): int|false` | 查找子串起始位置 |
| `str_replace` | `str_replace(mixed $search, mixed $replace, mixed $subject, int &$count = null): mixed` | 字符串替换 |
| `substr`    | `substr(string $string, int $offset, ?int $length = null): string` | 截取子串       |
| `sprintf`   | `sprintf(string $format, mixed ...$values): string`        | 格式化输出字符串               |
| `number_format` | `number_format(float $number, int $decimals = 0, string $decimal_separator = ".", string $thousands_separator = ","): string` | 金额格式 |

## 模板与输出缓冲

- 利用 `ob_start()` 与 `ob_get_clean()` 可捕获模板输出至字符串。

```php
ob_start();
include __DIR__ . '/templates/email.php';
$html = ob_get_clean();
```

## 分割与拼接

- `explode(',', 'a,b,c')` → `['a', 'b', 'c']`
- `implode('-', ['2025', '11', '28'])` → `'2025-11-28'`

## JSON 与 URL 编码

- `json_encode($data, JSON_UNESCAPED_UNICODE)` 防止中文被转义。
- `urlencode($str)` 与 `urldecode($str)` 处理查询字符串。
- `http_build_query($params)` 快速构建 URL。

## 正则表达式

- `preg_match($pattern, $subject)`：匹配一次。
- `preg_replace($pattern, $replacement, $subject)`：替换。
- `preg_split($pattern, $subject)`：按正则分割。

示例：

```php
$email = 'user@example.com';
if (!preg_match('/^[\w\-.]+@[\w\-]+\.[A-Za-z]{2,}$/', $email)) {
    throw new InvalidArgumentException('Invalid email');
}
```

## 字符串比较

- `strcmp($a, $b)`：区分大小写。
- `strcasecmp($a, $b)`：忽略大小写。
- `strncasecmp($a, $b, $len)`：比较前 `len` 个字符。
- `similar_text($a, $b, &$percent)`：计算相似度。

## 国际化与本地化

- 利用 `intl` 扩展：`Collator`、`NumberFormatter`、`IntlDateFormatter`。
- 示例：按中文规则排序
  ```php
  $collator = new Collator('zh_CN');
  $collator->sort($names);
  ```

## 练习

1. 编写 `slugify(string $title): string`，将任意 UTF-8 标题转换为 URL 友好的 slug。
2. 实现一个模板渲染函数：传入路径与变量数组，返回渲染后的字符串，并确保输出安全。
3. 编写脚本统计文本中出现频率最高的 10 个词，忽略大小写与标点。

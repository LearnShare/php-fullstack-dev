# 4.2.2 输出缓冲与控制

## 概述

输出缓冲（Output Buffering）是 PHP 中控制输出时机和内容的重要机制。本节详细介绍输出缓冲的基础用法、嵌套缓冲、实际应用场景，以及性能优化技巧。

## 输出缓冲基础

### 核心函数

| 函数 | 说明 |
| :--- | :--- |
| `ob_start()` | 开始输出缓冲 |
| `ob_get_contents()` | 获取缓冲区内容（不清空） |
| `ob_get_clean()` | 获取缓冲区内容并清空 |
| `ob_end_flush()` | 输出缓冲区内容并关闭缓冲 |
| `ob_end_clean()` | 清空缓冲区并关闭缓冲 |
| `ob_get_level()` | 获取当前缓冲层级 |

### 基本用法

```php
<?php
declare(strict_types=1);

// 开始缓冲
ob_start();

echo "Hello ";
echo "World ";

// 获取缓冲区内容（不清空）
$content = ob_get_contents();
echo $content; // Hello World Hello World

// 清空缓冲区
ob_clean();

echo "New content";

// 输出并关闭缓冲
ob_end_flush(); // 输出: New content
```

### 缓冲层级

```php
<?php
ob_start(); // 层级 1
echo "Level 1 ";

ob_start(); // 层级 2
echo "Level 2 ";

ob_start(); // 层级 3
echo "Level 3 ";

echo ob_get_level(); // 3

$level3 = ob_get_clean(); // 关闭层级 3
$level2 = ob_get_clean(); // 关闭层级 2
$level1 = ob_get_clean(); // 关闭层级 1

echo $level1 . $level2 . $level3; // Level 1 Level 2 Level 3
```

## 实际应用场景

### 1. 模板渲染

```php
<?php
declare(strict_types=1);

function renderTemplate(string $template, array $data = []): string
{
    extract($data, EXTR_SKIP);
    
    ob_start();
    include __DIR__ . "/templates/{$template}.php";
    return ob_get_clean();
}

// 使用
$html = renderTemplate('user-list', ['users' => $users]);
echo $html;
```

### 2. 邮件内容生成

```php
<?php
declare(strict_types=1);

function generateEmailContent(string $template, array $data = []): string
{
    extract($data, EXTR_SKIP);
    
    ob_start();
    include __DIR__ . "/emails/{$template}.php";
    $content = ob_get_clean();
    
    return "<html><body>{$content}</body></html>";
}

$emailBody = generateEmailContent('welcome', ['name' => 'Alice']);
mail('alice@example.com', 'Welcome', $emailBody);
```

### 3. 测试输出

```php
<?php
declare(strict_types=1);

function captureOutput(callable $callback): string
{
    ob_start();
    $callback();
    return ob_get_clean();
}

// 测试函数输出
$output = captureOutput(function () {
    echo "Test output";
    var_dump(['key' => 'value']);
});

assert($output !== '');
```

### 4. 条件输出

```php
<?php
ob_start();

if ($error) {
    echo "Error: {$error}";
}

if ($success) {
    echo "Success!";
}

$message = ob_get_clean();

// 只在有内容时输出
if (!empty($message)) {
    echo "<div class='message'>{$message}</div>";
}
```

### 5. 错误处理

```php
<?php
declare(strict_types=1);

ob_start();

try {
    // 可能产生输出的代码
    echo "Processing...";
    riskyOperation();
    echo "Done!";
} catch (Exception $e) {
    ob_clean(); // 清空之前的输出
    echo "Error: " . $e->getMessage();
}

$output = ob_get_clean();
echo $output;
```

## 性能优化

### 1. 输出压缩

```php
<?php
// 启用 Gzip 压缩
ob_start('ob_gzhandler');
echo "Content...";
ob_end_flush();
```

### 2. 输出缓存

```php
<?php
declare(strict_types=1);

function renderWithCache(string $template, array $data, int $ttl = 3600): string
{
    $cacheKey = 'template:' . md5($template . serialize($data));
    
    // 尝试从缓存获取
    $cached = apc_fetch($cacheKey);
    if ($cached !== false) {
        return $cached;
    }
    
    // 渲染模板
    $content = renderTemplate($template, $data);
    
    // 缓存结果
    apc_store($cacheKey, $content, $ttl);
    
    return $content;
}
```

### 3. 延迟输出

```php
<?php
declare(strict_types=1);

// 先处理业务逻辑，最后输出
ob_start();

// 执行数据库查询、计算等
$users = fetchUsers();
$stats = calculateStats();

// 最后输出
ob_end_flush();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class OutputBuffer
{
    private array $buffers = [];

    public function start(string $name = 'default'): void
    {
        $this->buffers[$name] = ob_get_level();
        ob_start();
    }

    public function get(string $name = 'default'): ?string
    {
        if (!isset($this->buffers[$name])) {
            return null;
        }

        $level = $this->buffers[$name];
        while (ob_get_level() > $level) {
            ob_end_clean();
        }

        return ob_get_clean();
    }

    public function flush(string $name = 'default'): void
    {
        $content = $this->get($name);
        if ($content !== null) {
            echo $content;
        }
    }
}

// 使用
$buffer = new OutputBuffer();

$buffer->start('header');
echo "<header>Header</header>";

$buffer->start('content');
echo "<main>Content</main>";

$buffer->start('footer');
echo "<footer>Footer</footer>";

// 按顺序输出
$buffer->flush('header');
$buffer->flush('content');
$buffer->flush('footer');
```

## 注意事项

1. **缓冲层级**：注意嵌套缓冲的层级，确保正确关闭
2. **内存使用**：大内容缓冲会占用内存，注意及时清理
3. **错误处理**：在缓冲中发生错误时，记得清理缓冲区
4. **性能影响**：输出缓冲有轻微性能开销，合理使用

## 练习

1. 实现一个模板渲染函数，使用输出缓冲生成 HTML 内容。

2. 创建一个邮件模板系统，使用输出缓冲生成 HTML 邮件内容，支持变量替换。

3. 实现一个输出捕获工具，用于测试函数的输出内容。

4. 创建一个条件输出系统，根据条件决定是否输出内容。

5. 实现一个输出压缩功能，自动压缩 HTML 输出内容。

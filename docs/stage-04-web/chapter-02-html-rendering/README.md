# 4.2 HTML 渲染与输出控制

## 目标

- 掌握 PHP 的三种页面渲染方式：纯 PHP 输出、PHP 模板嵌入、模板引擎。
- 理解输出缓冲（Output Buffering）的作用与使用场景。
- 熟悉模板引擎的工作原理，理解为什么模板引擎能提高安全性。
- 能够在实际项目中选择合适的渲染方式。

## PHP 页面渲染方式

### 方式一：纯 PHP 输出 HTML

- 直接在 PHP 代码中使用 `echo` 或 `print` 输出 HTML。
- 适合简单的页面或 API 响应。

```php
<?php
declare(strict_types=1);

header('Content-Type: text/html; charset=UTF-8');
?>
<!DOCTYPE html>
<html>
<head>
    <title>用户列表</title>
</head>
<body>
    <h1>用户列表</h1>
    <ul>
        <?php
        $users = [
            ['id' => 1, 'name' => 'Alice'],
            ['id' => 2, 'name' => 'Bob'],
        ];
        
        foreach ($users as $user) {
            echo "<li>{$user['name']}</li>";
        }
        ?>
    </ul>
</body>
</html>
```

**优点：**
- 简单直接，无需额外依赖。
- 性能好，无模板解析开销。

**缺点：**
- HTML 与 PHP 代码混合，可读性差。
- 难以维护，修改 HTML 需要修改 PHP 代码。
- 安全性低，容易忘记转义输出。

### 方式二：PHP 模板嵌入（最常见）

- 将 HTML 模板文件与 PHP 代码分离。
- 使用 `include` 或 `require` 引入模板文件。
- 通过变量传递数据给模板。

**控制器文件（controller.php）：**

```php
<?php
declare(strict_types=1);

// 业务逻辑
$users = [
    ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'],
    ['id' => 2, 'name' => 'Bob', 'email' => 'bob@example.com'],
];

// 传递数据给模板
$title = '用户列表';
$pageTitle = '用户管理';

// 包含模板
include __DIR__ . '/views/user-list.php';
```

**模板文件（views/user-list.php）：**

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><?= htmlspecialchars($title, ENT_QUOTES, 'UTF-8') ?></title>
</head>
<body>
    <h1><?= htmlspecialchars($pageTitle, ENT_QUOTES, 'UTF-8') ?></h1>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>姓名</th>
                <th>邮箱</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach ($users as $user): ?>
            <tr>
                <td><?= htmlspecialchars((string) $user['id'], ENT_QUOTES, 'UTF-8') ?></td>
                <td><?= htmlspecialchars($user['name'], ENT_QUOTES, 'UTF-8') ?></td>
                <td><?= htmlspecialchars($user['email'], ENT_QUOTES, 'UTF-8') ?></td>
            </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
</body>
</html>
```

**改进版本：使用函数封装模板渲染**

```php
<?php
declare(strict_types=1);

function render(string $template, array $data = []): void
{
    // 将数组键作为变量
    extract($data, EXTR_SKIP);
    
    // 开始输出缓冲
    ob_start();
    include __DIR__ . "/views/{$template}.php";
    $content = ob_get_clean();
    
    // 包含布局
    include __DIR__ . '/views/layout.php';
}

// 使用
render('user-list', [
    'title' => '用户列表',
    'users' => $users,
]);
```

**布局文件（views/layout.php）：**

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><?= htmlspecialchars($title ?? '页面', ENT_QUOTES, 'UTF-8') ?></title>
</head>
<body>
    <header>
        <nav>
            <a href="/">首页</a>
            <a href="/users">用户</a>
        </nav>
    </header>
    <main>
        <?= $content ?? '' ?>
    </main>
    <footer>
        <p>&copy; 2025 My App</p>
    </footer>
</body>
</html>
```

### 方式三：模板引擎（Twig / Blade）

- 使用专门的模板引擎，提供更强大的功能和更好的安全性。
- 模板语法与 PHP 代码完全分离。

**Twig 示例：**

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Twig\Environment;
use Twig\Loader\FilesystemLoader;

// 初始化 Twig
$loader = new FilesystemLoader(__DIR__ . '/templates');
$twig = new Environment($loader, [
    'cache' => __DIR__ . '/cache',
    'auto_reload' => true,
]);

// 渲染模板
echo $twig->render('user-list.twig', [
    'title' => '用户列表',
    'users' => [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob'],
    ],
]);
```

**Twig 模板文件（templates/user-list.twig）：**

```twig
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <h1>用户列表</h1>
    <ul>
        {% for user in users %}
        <li>{{ user.name }}</li>
        {% endfor %}
    </ul>
</body>
</html>
```

**Blade 示例（Laravel）：**

```php
return view('user-list', [
    'title' => '用户列表',
    'users' => $users,
]);
```

```blade
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title }}</title>
</head>
<body>
    <h1>用户列表</h1>
    <ul>
        @foreach($users as $user)
        <li>{{ $user->name }}</li>
        @endforeach
    </ul>
</body>
</html>
```

## 输出缓冲（Output Buffering）

### 基础用法

- **`ob_start()`**：开始输出缓冲。
- **`ob_get_contents()`**：获取缓冲区内容（不清空）。
- **`ob_get_clean()`**：获取缓冲区内容并清空。
- **`ob_end_flush()`**：输出缓冲区内容并关闭缓冲。

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

### 实际应用场景

#### 1. 模板渲染

```php
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

#### 2. 邮件内容生成

```php
function generateEmailContent(string $template, array $data = []): string
{
    extract($data, EXTR_SKIP);
    
    ob_start();
    include __DIR__ . "/emails/{$template}.php";
    $content = ob_get_clean();
    
    // 可以进一步处理内容（如添加 HTML 包装）
    return "<html><body>{$content}</body></html>";
}

$emailBody = generateEmailContent('welcome', ['name' => 'Alice']);
mail('alice@example.com', 'Welcome', $emailBody);
```

#### 3. 测试输出

```php
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

#### 4. 条件输出

```php
ob_start();

// 可能输出错误信息
if ($error) {
    echo "Error: {$error}";
}

// 可能输出成功信息
if ($success) {
    echo "Success!";
}

$message = ob_get_clean();

// 只在有内容时输出
if (!empty($message)) {
    echo "<div class='message'>{$message}</div>";
}
```

### 嵌套缓冲

```php
ob_start(); // 外层缓冲
echo "Outer ";

ob_start(); // 内层缓冲
echo "Inner ";
$inner = ob_get_clean();

echo $inner;
$outer = ob_get_clean();

echo $outer; // Outer Inner
```

## 模板引擎的安全性

### XSS 攻击风险

**不安全的输出：**

```php
<?php
$userInput = '<script>alert("XSS")</script>';
?>
<div><?= $userInput ?></div> <!-- 危险！ -->
```

**安全的输出（手动转义）：**

```php
<?php
$userInput = '<script>alert("XSS")</script>';
?>
<div><?= htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8') ?></div>
```

### 模板引擎自动转义

**Twig 自动转义：**

```twig
{# Twig 默认自动转义所有输出 #}
<div>{{ userInput }}</div> {# 安全：自动转义 #}

{# 需要输出 HTML 时使用 raw 过滤器 #}
<div>{{ trustedHtml|raw }}</div> {# 注意：仅在确定安全时使用 #}
```

**Blade 自动转义：**

```blade
{{-- Blade 默认自动转义 --}}
<div>{{ $userInput }}</div> {{-- 安全：自动转义 --}}

{{-- 需要输出 HTML 时使用 {!! !!} --}}
<div>{!! $trustedHtml !!}</div> {{-- 注意：仅在确定安全时使用 --}}
```

### 转义函数对比

| 函数                    | 说明                           | 示例                           |
| :---------------------- | :----------------------------- | :----------------------------- |
| `htmlspecialchars()`    | 转义 HTML 特殊字符            | `&lt;script&gt;` → `&lt;script&gt;` |
| `htmlentities()`        | 转义所有 HTML 实体             | `&` → `&amp;`                 |
| `strip_tags()`          | 删除 HTML 标签                 | `<script>` → ``                |
| `filter_var()`          | 使用过滤器清理                 | `FILTER_SANITIZE_STRING`       |

**安全输出函数封装：**

```php
function e(string $string): string
{
    return htmlspecialchars($string, ENT_QUOTES, 'UTF-8');
}

// 使用
<div><?= e($userInput) ?></div>
```

## 模板继承与组件

### 模板继承（Twig）

**基础模板（base.twig）：**

```twig
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}默认标题{% endblock %}</title>
</head>
<body>
    <header>
        {% block header %}{% endblock %}
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        {% block footer %}{% endblock %}
    </footer>
</body>
</html>
```

**子模板（user-list.twig）：**

```twig
{% extends "base.twig" %}

{% block title %}用户列表{% endblock %}

{% block content %}
<h1>用户列表</h1>
<ul>
    {% for user in users %}
    <li>{{ user.name }}</li>
    {% endfor %}
</ul>
{% endblock %}
```

### 组件化（Include）

```php
<!-- components/header.php -->
<header>
    <nav>
        <a href="/">首页</a>
        <a href="/users">用户</a>
    </nav>
</header>

<!-- components/footer.php -->
<footer>
    <p>&copy; 2025 My App</p>
</footer>

<!-- page.php -->
<?php include 'components/header.php'; ?>
<main>内容</main>
<?php include 'components/footer.php'; ?>
```

## 性能考虑

### 模板缓存

**Twig 缓存配置：**

```php
$twig = new Environment($loader, [
    'cache' => __DIR__ . '/cache', // 启用缓存
    'auto_reload' => true, // 开发时自动重新加载
]);
```

### 输出压缩

```php
ob_start('ob_gzhandler'); // 启用 Gzip 压缩
echo "Content...";
ob_end_flush();
```

## 最佳实践

### 1. 始终转义用户输入

```php
// 不推荐
echo $userInput;

// 推荐
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
```

### 2. 使用模板引擎

- 对于复杂项目，使用模板引擎（Twig、Blade）。
- 自动转义、模板继承等功能提高开发效率。

### 3. 分离关注点

- 控制器处理业务逻辑。
- 模板只负责展示。
- 避免在模板中编写复杂业务逻辑。

### 4. 使用布局系统

- 定义基础布局。
- 子模板继承布局。
- 减少重复代码。

## 练习

1. 创建一个简单的模板系统，使用 `include` 和输出缓冲实现布局和内容分离。

2. 实现一个 `render()` 函数，接受模板名称和数据数组，返回渲染后的 HTML 字符串。

3. 编写一个安全的输出函数 `e()`，自动转义 HTML 特殊字符，并在模板中使用。

4. 使用 Twig 创建一个博客系统，包含文章列表页和文章详情页，使用模板继承。

5. 实现一个邮件模板系统，使用输出缓冲生成 HTML 邮件内容，支持变量替换。

6. 创建一个组件系统，将页面的 header、footer、sidebar 等部分拆分为独立组件，可以复用。

# 4.2.1 HTML 渲染与模板引擎

## 概述

HTML 渲染是 Web 开发的核心。本节详细介绍 PHP 的三种渲染方式、模板引擎的使用、模板继承与组件化，以及 XSS 防护的最佳实践。

## PHP 页面渲染方式

### 方式一：纯 PHP 输出 HTML

直接在 PHP 代码中使用 `echo` 或 `print` 输出 HTML。

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

**优点**：
- 简单直接，无需额外依赖
- 性能好，无模板解析开销

**缺点**：
- HTML 与 PHP 代码混合，可读性差
- 难以维护，修改 HTML 需要修改 PHP 代码
- 安全性低，容易忘记转义输出

### 方式二：PHP 模板嵌入（最常见）

将 HTML 模板文件与 PHP 代码分离，使用 `include` 或 `require` 引入模板文件。

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
    extract($data, EXTR_SKIP);
    
    ob_start();
    include __DIR__ . "/views/{$template}.php";
    $content = ob_get_clean();
    
    include __DIR__ . '/views/layout.php';
}

// 使用
render('user-list', [
    'title' => '用户列表',
    'users' => $users,
]);
```

### 方式三：模板引擎（Twig / Blade）

使用专门的模板引擎，提供更强大的功能和更好的安全性。

**Twig 示例：**

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Twig\Environment;
use Twig\Loader\FilesystemLoader;

$loader = new FilesystemLoader(__DIR__ . '/templates');
$twig = new Environment($loader, [
    'cache' => __DIR__ . '/cache',
    'auto_reload' => true,
]);

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

## 安全性：XSS 防护

### XSS 攻击风险

**不安全的输出：**

```php
<?php
$userInput = '<script>alert("XSS")</script>';
?>
<div><?= $userInput ?></div> <!-- 危险！ -->
```

### 安全输出

**手动转义：**

```php
<?php
$userInput = '<script>alert("XSS")</script>';
?>
<div><?= htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8') ?></div>
```

**模板引擎自动转义：**

```twig
{# Twig 默认自动转义所有输出 #}
<div>{{ userInput }}</div> {# 安全：自动转义 #}

{# 需要输出 HTML 时使用 raw 过滤器 #}
<div>{{ trustedHtml|raw }}</div> {# 注意：仅在确定安全时使用 #}
```

### 转义函数对比

| 函数 | 说明 | 示例 |
| :--- | :--- | :--- |
| `htmlspecialchars()` | 转义 HTML 特殊字符 | `&lt;script&gt;` |
| `htmlentities()` | 转义所有 HTML 实体 | `&` → `&amp;` |
| `strip_tags()` | 删除 HTML 标签 | `<script>` → `` |

**安全输出函数封装：**

```php
function e(string $string): string
{
    return htmlspecialchars($string, ENT_QUOTES, 'UTF-8');
}

// 使用
<div><?= e($userInput) ?></div>
```

## 完整示例

```php
<?php
declare(strict_types=1);

class TemplateRenderer
{
    public function __construct(
        private string $templateDir,
        private string $layoutFile = 'layout.php'
    ) {}

    public function render(string $template, array $data = []): string
    {
        extract($data, EXTR_SKIP);
        
        ob_start();
        include $this->templateDir . "/{$template}.php";
        $content = ob_get_clean();
        
        ob_start();
        include $this->templateDir . "/{$this->layoutFile}.php";
        return ob_get_clean();
    }
}

// 使用
$renderer = new TemplateRenderer(__DIR__ . '/views');
echo $renderer->render('user-list', [
    'title' => '用户列表',
    'users' => $users,
]);
```

## 注意事项

1. **始终转义用户输入**：防止 XSS 攻击
2. **使用模板引擎**：复杂项目推荐使用模板引擎
3. **分离关注点**：控制器处理逻辑，模板只负责展示
4. **使用布局系统**：减少重复代码

## 练习

1. 创建一个简单的模板系统，使用 `include` 实现布局和内容分离。

2. 实现一个 `render()` 函数，接受模板名称和数据数组，返回渲染后的 HTML 字符串。

3. 编写一个安全的输出函数 `e()`，自动转义 HTML 特殊字符。

4. 使用 Twig 创建一个博客系统，包含文章列表页和文章详情页，使用模板继承。

5. 创建一个组件系统，将页面的 header、footer、sidebar 等部分拆分为独立组件。

# 5.2.1 HTML 渲染与模板引擎

## 概述

HTML 渲染是 Web 应用的核心功能，将数据转换为用户可见的 HTML 页面。PHP 提供了多种 HTML 渲染方式，从简单的字符串拼接到功能强大的模板引擎。理解不同的渲染方式及其适用场景，对于构建可维护、安全的 Web 应用至关重要。本节详细介绍 HTML 渲染的方法、模板引擎的概念和使用，包括原生 PHP 模板、Twig、Blade 等主流模板引擎，帮助零基础学员掌握模板渲染技术。

在 Web 开发中，将业务逻辑和视图分离是重要的设计原则。模板引擎提供了这种分离的实现方式，不仅提高了代码的可维护性，还增强了安全性（自动转义输出，防止 XSS 攻击）。选择合适的渲染方式需要根据项目规模、团队经验、性能要求等因素综合考虑。

**主要内容**：
- HTML 渲染方法概述（直接输出、字符串拼接、模板文件、模板引擎）
- 原生 PHP 模板的使用方法
- 模板引擎的概念和优势
- Twig 模板引擎的安装、配置和使用
- Blade 模板引擎的语法和特性
- 模板继承和组件化
- 变量转义和 XSS 防护
- 模板缓存和性能优化
- 实际应用示例和最佳实践

## 特性

- **多种渲染方式**：支持直接输出、模板文件、模板引擎等多种方式
- **视图分离**：模板引擎实现业务逻辑和视图的分离
- **自动转义**：模板引擎默认自动转义输出，防止 XSS 攻击
- **模板继承**：支持模板继承和组件化，减少重复代码
- **性能优化**：支持模板编译和缓存，提高渲染性能
- **灵活扩展**：支持自定义函数、过滤器、标签等扩展

## HTML 渲染方法

### 方法一：直接输出 HTML

直接在 PHP 代码中使用 `echo` 或 `print` 输出 HTML。

**示例**：
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
- 适合简单的页面

**缺点**：
- HTML 与 PHP 代码混合，可读性差
- 难以维护，修改 HTML 需要修改 PHP 代码
- 安全性低，容易忘记转义输出
- 不适合复杂项目

### 方法二：字符串拼接

使用字符串拼接方式生成 HTML。

**示例**：
```php
<?php
declare(strict_types=1);

$users = [
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
];

$html = '<!DOCTYPE html><html><head><title>用户列表</title></head><body>';
$html .= '<h1>用户列表</h1><ul>';

foreach ($users as $user) {
    $html .= "<li>{$user['name']}</li>";
}

$html .= '</ul></body></html>';

echo $html;
```

**优点**：
- 逻辑和视图完全分离
- 可以动态生成 HTML

**缺点**：
- 代码可读性差
- 难以维护
- 容易出错（引号、转义等）
- 不适合复杂 HTML

### 方法三：PHP 模板文件

将 HTML 模板文件与 PHP 代码分离，使用 `include` 或 `require` 引入模板文件。

**控制器文件（controller.php）**：
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

**模板文件（views/user-list.php）**：
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

**优点**：
- HTML 和 PHP 代码分离
- 可读性好
- 易于维护
- 性能好（无模板解析开销）

**缺点**：
- 需要手动转义输出（容易忘记）
- 模板语法不够友好
- 缺少高级功能（继承、组件等）

### 方法四：模板引擎

使用专门的模板引擎，提供更强大的功能和更好的安全性。

**优势**：
- 自动转义输出，防止 XSS 攻击
- 友好的模板语法
- 支持模板继承和组件化
- 支持自定义函数和过滤器
- 模板编译和缓存

**主流模板引擎**：
- **Twig**：Symfony 框架使用的模板引擎
- **Blade**：Laravel 框架使用的模板引擎
- **Smarty**：老牌模板引擎
- **Plates**：轻量级模板引擎

## 原生 PHP 模板

### 基本用法

使用 `include` 或 `require` 引入模板文件，通过变量传递数据。

**示例**：
```php
<?php
declare(strict_types=1);

// 准备数据
$data = [
    'title' => '欢迎页面',
    'user' => ['name' => 'John Doe', 'email' => 'john@example.com'],
];

// 提取变量（使数组键成为变量）
extract($data, EXTR_SKIP);

// 包含模板
include __DIR__ . '/templates/welcome.php';
```

**模板文件（templates/welcome.php）**：
```php
<!DOCTYPE html>
<html>
<head>
    <title><?= htmlspecialchars($title, ENT_QUOTES, 'UTF-8') ?></title>
</head>
<body>
    <h1>欢迎，<?= htmlspecialchars($user['name'], ENT_QUOTES, 'UTF-8') ?>！</h1>
    <p>邮箱：<?= htmlspecialchars($user['email'], ENT_QUOTES, 'UTF-8') ?></p>
</body>
</html>
```

### 模板渲染函数

封装模板渲染函数，提高代码复用性。

**示例**：
```php
<?php
declare(strict_types=1);

function render(string $template, array $data = []): string
{
    extract($data, EXTR_SKIP);
    
    ob_start();
    include __DIR__ . "/views/{$template}.php";
    return ob_get_clean();
}

// 使用
$html = render('user-list', [
    'title' => '用户列表',
    'users' => $users,
]);

echo $html;
```

### 布局系统

实现简单的布局系统，减少重复代码。

**布局文件（views/layout.php）**：
```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><?= htmlspecialchars($title ?? '默认标题', ENT_QUOTES, 'UTF-8') ?></title>
</head>
<body>
    <header>
        <nav>导航栏</nav>
    </header>
    <main>
        <?= $content ?? '' ?>
    </main>
    <footer>
        <p>版权所有</p>
    </footer>
</body>
</html>
```

**使用布局**：
```php
<?php
declare(strict_types=1);

function renderWithLayout(string $template, array $data = []): void
{
    extract($data, EXTR_SKIP);
    
    // 渲染内容
    ob_start();
    include __DIR__ . "/views/{$template}.php";
    $content = ob_get_clean();
    
    // 渲染布局
    include __DIR__ . '/views/layout.php';
}

// 使用
renderWithLayout('user-list', [
    'title' => '用户列表',
    'users' => $users,
]);
```

## 模板引擎概念

### 为什么需要模板引擎

**问题**：
- 原生 PHP 模板需要手动转义，容易忘记
- 模板语法不够友好
- 缺少高级功能（继承、组件等）
- 业务逻辑和视图容易混合

**解决方案**：
- 模板引擎自动转义输出
- 提供友好的模板语法
- 支持模板继承和组件化
- 强制分离业务逻辑和视图

### 模板引擎的功能

1. **变量输出**：安全地输出变量
2. **控制结构**：支持条件、循环等控制结构
3. **模板继承**：支持模板继承，减少重复代码
4. **组件化**：支持组件和包含
5. **过滤器**：支持数据过滤和格式化
6. **函数**：支持自定义函数
7. **自动转义**：默认自动转义输出，防止 XSS

## Twig 模板引擎

Twig 是 Symfony 框架使用的模板引擎，语法简洁，功能强大。

### 安装

使用 Composer 安装：

```bash
composer require twig/twig
```

### 基本配置

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Twig\Environment;
use Twig\Loader\FilesystemLoader;

// 配置模板加载器
$loader = new FilesystemLoader(__DIR__ . '/templates');

// 配置 Twig 环境
$twig = new Environment($loader, [
    'cache' => __DIR__ . '/cache',  // 模板缓存目录
    'auto_reload' => true,          // 开发环境自动重新加载
    'debug' => true,                // 开发环境启用调试
]);
```

### 基本语法

#### 变量输出

**Twig 模板**：
```twig
<h1>{{ title }}</h1>
<p>用户：{{ user.name }}</p>
<p>邮箱：{{ user.email }}</p>
```

**PHP 代码**：
```php
<?php
declare(strict_types=1);

echo $twig->render('user.twig', [
    'title' => '用户信息',
    'user' => [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ],
]);
```

**输出**：
```html
<h1>用户信息</h1>
<p>用户：John Doe</p>
<p>邮箱：john@example.com</p>
```

#### 控制结构

**条件语句**：
```twig
{% if user.isActive %}
    <p>用户已激活</p>
{% else %}
    <p>用户未激活</p>
{% endif %}
```

**循环语句**：
```twig
<ul>
    {% for user in users %}
        <li>{{ user.name }}</li>
    {% endfor %}
</ul>
```

**循环变量**：
```twig
<ul>
    {% for user in users %}
        <li>
            {{ loop.index }}. {{ user.name }}
            {% if loop.first %}（第一个）{% endif %}
            {% if loop.last %}（最后一个）{% endif %}
        </li>
    {% endfor %}
</ul>
```

#### 过滤器

**常用过滤器**：
```twig
{# 转大写 #}
{{ name|upper }}

{# 转小写 #}
{{ name|lower }}

{# 首字母大写 #}
{{ name|capitalize }}

{# 日期格式化 #}
{{ date|date('Y-m-d H:i:s') }}

{# 默认值 #}
{{ name|default('未知') }}

{# 转义（默认自动转义） #}
{{ content|escape }}

{# 不转义（谨慎使用） #}
{{ trustedHtml|raw }}
```

#### 函数

**常用函数**：
```twig
{# 包含其他模板 #}
{% include 'header.twig' %}

{# 继承模板 #}
{% extends 'base.twig' %}

{# 定义块 #}
{% block content %}
    内容
{% endblock %}

{# 输出块 #}
{{ block('content') }}
```

### 模板继承

**基础模板（templates/base.twig）**：
```twig
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{% block title %}默认标题{% endblock %}</title>
</head>
<body>
    <header>
        {% block header %}
            <nav>导航栏</nav>
        {% endblock %}
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        {% block footer %}
            <p>版权所有</p>
        {% endblock %}
    </footer>
</body>
</html>
```

**子模板（templates/user-list.twig）**：
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

### 完整示例

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Twig\Environment;
use Twig\Loader\FilesystemLoader;

// 配置 Twig
$loader = new FilesystemLoader(__DIR__ . '/templates');
$twig = new Environment($loader, [
    'cache' => __DIR__ . '/cache',
    'auto_reload' => true,
]);

// 准备数据
$data = [
    'title' => '用户列表',
    'users' => [
        ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'],
        ['id' => 2, 'name' => 'Bob', 'email' => 'bob@example.com'],
    ],
];

// 渲染模板
echo $twig->render('user-list.twig', $data);
```

## Blade 模板引擎

Blade 是 Laravel 框架使用的模板引擎，语法简洁，功能强大。

### 基本语法

#### 变量输出

**Blade 模板**：
```blade
<h1>{{ $title }}</h1>
<p>用户：{{ $user->name }}</p>
```

#### 控制结构

**条件语句**：
```blade
@if($user->isActive)
    <p>用户已激活</p>
@else
    <p>用户未激活</p>
@endif
```

**循环语句**：
```blade
<ul>
    @foreach($users as $user)
        <li>{{ $user->name }}</li>
    @endforeach
</ul>
```

#### 模板继承

**基础模板（layouts/app.blade.php）**：
```blade
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', '默认标题')</title>
</head>
<body>
    @include('partials.header')
    
    <main>
        @yield('content')
    </main>
    
    @include('partials.footer')
</body>
</html>
```

**子模板**：
```blade
@extends('layouts.app')

@section('title', '用户列表')

@section('content')
    <h1>用户列表</h1>
    <ul>
        @foreach($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endsection
```

## 变量转义和 XSS 防护

### XSS 攻击风险

XSS（Cross-Site Scripting，跨站脚本攻击）是通过在网页中注入恶意脚本实现的攻击。

**不安全的输出**：
```php
<?php
$userInput = '<script>alert("XSS")</script>';
?>
<div><?= $userInput ?></div> <!-- 危险！会执行脚本 -->
```

### 安全输出方法

#### 手动转义

使用 `htmlspecialchars()` 函数转义输出：

```php
<?php
declare(strict_types=1);

$userInput = '<script>alert("XSS")</script>';
?>
<div><?= htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8') ?></div>
```

**输出**：
```html
<div>&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</div>
```

#### 转义函数封装

封装转义函数，简化使用：

```php
<?php
declare(strict_types=1);

function e(string $string): string
{
    return htmlspecialchars($string, ENT_QUOTES, 'UTF-8');
}

// 使用
<div><?= e($userInput) ?></div>
```

#### 模板引擎自动转义

**Twig**（默认自动转义）：
```twig
{# 自动转义，安全 #}
<div>{{ userInput }}</div>

{# 需要输出 HTML 时使用 raw 过滤器（谨慎使用） #}
<div>{{ trustedHtml|raw }}</div>
```

**Blade**（默认自动转义）：
```blade
{{-- 自动转义，安全 --}}
<div>{{ $userInput }}</div>

{{-- 需要输出 HTML 时使用 {!! !!}（谨慎使用） --}}
<div>{!! $trustedHtml !!}</div>
```

### 转义函数对比

| 函数 | 说明 | 示例 |
|:-----|:-----|:-----|
| `htmlspecialchars()` | 转义 HTML 特殊字符 | `<script>` → `&lt;script&gt;` |
| `htmlentities()` | 转义所有 HTML 实体 | `&` → `&amp;` |
| `strip_tags()` | 删除 HTML 标签 | `<script>alert()</script>` → `alert()` |

## 使用场景

### Web 页面渲染

- 渲染 HTML 页面
- 生成动态内容
- 实现页面布局

### 邮件模板

- 生成邮件内容
- 使用模板引擎渲染邮件模板
- 支持 HTML 和纯文本格式

### 报表生成

- 生成 PDF 报表
- 生成 Excel 报表
- 使用模板定义报表格式

### 视图分离

- 实现 MVC 架构
- 分离业务逻辑和视图
- 提高代码可维护性

## 注意事项

### 变量转义

- **始终转义用户输入**：防止 XSS 攻击
- **使用模板引擎**：模板引擎默认自动转义
- **谨慎使用 raw**：仅在确定安全时输出原始 HTML

### XSS 防护

- **输入验证**：验证和过滤用户输入
- **输出转义**：始终转义输出内容
- **Content Security Policy**：使用 CSP 进一步防护

### 模板缓存

- **开发环境**：禁用缓存，启用自动重新加载
- **生产环境**：启用缓存，提高性能
- **缓存清理**：更新模板后清理缓存

### 性能考虑

- **模板编译**：模板引擎会编译模板，提高性能
- **缓存策略**：合理使用模板缓存
- **避免过度嵌套**：避免过深的模板嵌套

## 常见问题

### 为什么使用模板引擎？

1. **安全性**：自动转义输出，防止 XSS 攻击
2. **可维护性**：分离业务逻辑和视图，提高可维护性
3. **功能强大**：支持模板继承、组件化等高级功能
4. **语法友好**：提供友好的模板语法

### Twig 和 Blade 的区别？

| 特性 | Twig | Blade |
|:-----|:-----|:------|
| 框架 | Symfony | Laravel |
| 语法 | `{{ }}`、`{% %}` | `{{ }}`、`@` 指令 |
| 自动转义 | 默认开启 | 默认开启 |
| 模板继承 | 支持 | 支持 |
| 组件系统 | 支持 | 支持（更强大） |

### 如何防止 XSS 攻击？

1. **使用模板引擎**：模板引擎默认自动转义
2. **手动转义**：使用 `htmlspecialchars()` 转义输出
3. **输入验证**：验证和过滤用户输入
4. **Content Security Policy**：使用 CSP 进一步防护

### 模板引擎的性能影响？

- **编译开销**：首次编译有开销，后续使用编译后的模板
- **缓存优化**：使用模板缓存可以显著提高性能
- **实际影响**：对于大多数应用，性能影响可以忽略

## 最佳实践

### 使用模板引擎分离视图

- 使用模板引擎实现视图分离
- 避免在模板中编写业务逻辑
- 保持模板简洁清晰

### 始终转义输出

- 使用模板引擎的自动转义功能
- 手动转义时使用 `htmlspecialchars()`
- 谨慎使用 raw 输出原始 HTML

### 使用模板继承

- 使用模板继承减少重复代码
- 定义基础模板和布局
- 子模板继承基础模板

### 缓存编译后的模板

- 生产环境启用模板缓存
- 开发环境禁用缓存
- 更新模板后清理缓存

### 合理选择渲染方式

- **简单页面**：使用原生 PHP 模板
- **中小型项目**：使用原生 PHP 模板或轻量级模板引擎
- **大型项目**：使用功能强大的模板引擎（Twig、Blade）

## 相关章节

- **[5.2.2 输出缓冲与控制](section-02-output-buffering.md)**：了解输出缓冲的使用方法
- **[5.1.1 HTTP 请求响应流程](../chapter-01-request-response/section-01-http-flow.md)**：了解 HTTP 响应中的 HTML 输出
- **[9.3 Laravel](../stage-09-frameworks/chapter-03-laravel/readme.md)**：了解 Laravel 框架中的 Blade 模板系统

## 练习任务

1. **创建简单的模板系统**
   - 使用原生 PHP 模板实现布局系统
   - 实现模板渲染函数
   - 测试模板继承功能

2. **实现安全的输出函数**
   - 创建 `e()` 函数，自动转义 HTML 特殊字符
   - 测试 XSS 防护效果
   - 对比转义前后的输出

3. **使用 Twig 模板引擎**
   - 安装和配置 Twig
   - 创建基础模板和子模板
   - 实现用户列表页面

4. **实现模板组件系统**
   - 将页面的 header、footer、sidebar 等部分拆分为独立组件
   - 使用模板包含功能组合组件
   - 实现可复用的组件库

5. **性能优化实践**
   - 对比原生 PHP 模板和模板引擎的性能
   - 测试模板缓存的效果
   - 优化模板渲染性能

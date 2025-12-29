# 5.4.4 URL 路由与重写

## 概述

URL 路由和重写用于创建友好的 URL 结构，将用户友好的 URL 映射到实际的处理程序。理解 URL 路由的概念、掌握路由实现方法、了解 URL 重写配置，对于构建现代化的 Web 应用、实现 RESTful API、优化 SEO 等场景至关重要。本节详细介绍 URL 路由的概念、路由规则设计、URL 重写实现、Apache 和 Nginx 配置等内容，帮助零基础学员理解路由机制。

传统的 PHP 应用使用文件路径来访问页面（如 `index.php`、`about.php`），而现代 Web 应用使用友好的 URL（如 `/about`、`/users/123`）。URL 路由和重写机制实现了这种映射，提高了 URL 的可读性和 SEO 友好性。

**主要内容**：
- URL 路由的概念和作用
- 路由规则设计（静态路由、动态路由、参数路由）
- URL 重写实现方法
- Apache `.htaccess` 配置
- Nginx `rewrite` 配置
- 路由解析和参数提取
- 路由匹配算法
- 实际应用示例和最佳实践

## 特性

- **友好 URL**：创建可读性强的 URL
- **参数提取**：从 URL 中提取参数
- **灵活映射**：支持多种路由模式
- **SEO 优化**：提高搜索引擎优化
- **框架集成**：现代框架都提供路由功能

## URL 路由概念

### 什么是路由

路由（Routing）是将 URL 映射到处理程序（控制器、函数等）的机制。

**传统方式**：
```
https://example.com/user.php?id=123
```

**路由方式**：
```
https://example.com/users/123
```

### 友好 URL 的优势

1. **可读性强**：URL 更易理解和记忆
2. **SEO 友好**：搜索引擎更容易索引
3. **RESTful**：符合 REST 架构风格
4. **隐藏技术细节**：不暴露文件结构和技术实现

### 路由的作用

1. **URL 映射**：将友好 URL 映射到实际处理程序
2. **参数提取**：从 URL 中提取参数
3. **请求分发**：根据 URL 分发到不同的处理程序
4. **中间件支持**：支持中间件处理

## 路由规则设计

### 静态路由

静态路由是固定的 URL 路径，不包含参数。

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    '/' => 'HomeController@index',
    '/about' => 'PageController@about',
    '/contact' => 'PageController@contact',
];
```

### 动态路由

动态路由包含参数，可以从 URL 中提取参数值。

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    '/users/:id' => 'UserController@show',
    '/posts/:slug' => 'PostController@show',
    '/categories/:category/posts' => 'PostController@byCategory',
];
```

### 参数路由

参数路由使用占位符表示参数。

**常见占位符格式**：
- `:id`：命名参数（如 `/users/:id`）
- `{id}`：花括号参数（如 `/users/{id}`）
- `[0-9]+`：正则表达式（如 `/users/[0-9]+`）

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    '/users/:id' => 'UserController@show',
    '/users/:id/posts' => 'UserController@posts',
    '/posts/:year/:month/:slug' => 'PostController@show',
];
```

### 路由优先级

路由匹配通常按照定义顺序进行，更具体的路由应该放在前面。

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    '/users/new' => 'UserController@create',      // 更具体，放在前面
    '/users/:id' => 'UserController@show',        // 更通用，放在后面
];
```

## URL 重写实现

### Apache mod_rewrite

Apache 使用 `mod_rewrite` 模块实现 URL 重写。

#### .htaccess 配置

**基本配置**：
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?route=$1 [L,QSA]
```

**配置说明**：
- `RewriteEngine On`：启用重写引擎
- `RewriteCond %{REQUEST_FILENAME} !-f`：如果请求的文件不存在
- `RewriteCond %{REQUEST_FILENAME} !-d`：如果请求的目录不存在
- `RewriteRule ^(.*)$ index.php?route=$1`：将所有请求重写到 `index.php`
- `[L,QSA]`：`L` 表示最后一条规则，`QSA` 表示追加查询字符串

#### 完整示例

**.htaccess**：
```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /

    # 处理前端控制器
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ index.php?route=$1 [L,QSA]

    # 强制 HTTPS（可选）
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

    # 移除 www（可选）
    RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
    RewriteRule ^(.*)$ https://%1/$1 [R=301,L]
</IfModule>
```

### Nginx rewrite

Nginx 使用 `rewrite` 指令实现 URL 重写。

#### 基本配置

**nginx.conf**：
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### 完整示例

**nginx.conf**：
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php;

    # URL 重写
    location / {
        try_files $uri $uri/ /index.php?route=$uri&$args;
    }

    # PHP 处理
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

## 路由解析

### 基本路由解析

**示例**：
```php
<?php
declare(strict_types=1);

class SimpleRouter
{
    private array $routes = [];

    public function add(string $pattern, callable $handler): void
    {
        $this->routes[$pattern] = $handler;
    }

    public function match(string $path): ?callable
    {
        foreach ($this->routes as $pattern => $handler) {
            if ($this->matchPattern($pattern, $path)) {
                return $handler;
            }
        }
        return null;
    }

    private function matchPattern(string $pattern, string $path): bool
    {
        // 将 :id 转换为正则表达式
        $regex = preg_replace('/:(\w+)/', '([^/]+)', $pattern);
        $regex = '#^' . $regex . '$#';
        
        return preg_match($regex, $path) === 1;
    }
}

// 使用
$router = new SimpleRouter();
$router->add('/users/:id', function (string $id) {
    echo "用户 ID: {$id}\n";
});

$handler = $router->match('/users/123');
if ($handler !== null) {
    $handler('123');
}
```

### 参数提取

**示例**：
```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];

    public function add(string $pattern, callable $handler): void
    {
        $this->routes[] = [
            'pattern' => $pattern,
            'handler' => $handler,
        ];
    }

    public function match(string $path): ?array
    {
        foreach ($this->routes as $route) {
            $result = $this->matchPattern($route['pattern'], $path);
            if ($result !== null) {
                return [
                    'handler' => $route['handler'],
                    'params' => $result,
                ];
            }
        }
        return null;
    }

    private function matchPattern(string $pattern, string $path): ?array
    {
        // 提取参数名
        preg_match_all('/:(\w+)/', $pattern, $paramNames);
        $paramNames = $paramNames[1];

        // 转换为正则表达式
        $regex = preg_replace('/:(\w+)/', '([^/]+)', $pattern);
        $regex = '#^' . $regex . '$#';

        // 匹配并提取参数值
        if (preg_match($regex, $path, $matches)) {
            array_shift($matches);  // 移除完整匹配
            return array_combine($paramNames, $matches);
        }

        return null;
    }
}

// 使用
$router = new Router();
$router->add('/users/:id', function (array $params) {
    echo "用户 ID: {$params['id']}\n";
});

$result = $router->match('/users/123');
if ($result !== null) {
    $result['handler']($result['params']);
}
```

### 路由匹配算法

**示例**：
```php
<?php
declare(strict_types=1);

class AdvancedRouter
{
    private array $routes = [];

    public function add(string $method, string $pattern, callable $handler): void
    {
        $this->routes[] = [
            'method' => $method,
            'pattern' => $pattern,
            'handler' => $handler,
            'regex' => $this->patternToRegex($pattern),
            'paramNames' => $this->extractParamNames($pattern),
        ];
    }

    public function match(string $method, string $path): ?array
    {
        foreach ($this->routes as $route) {
            if ($route['method'] !== $method) {
                continue;
            }

            $params = $this->matchRegex($route['regex'], $path, $route['paramNames']);
            if ($params !== null) {
                return [
                    'handler' => $route['handler'],
                    'params' => $params,
                ];
            }
        }
        return null;
    }

    private function patternToRegex(string $pattern): string
    {
        $regex = preg_replace('/:(\w+)/', '([^/]+)', $pattern);
        return '#^' . $regex . '$#';
    }

    private function extractParamNames(string $pattern): array
    {
        preg_match_all('/:(\w+)/', $pattern, $matches);
        return $matches[1];
    }

    private function matchRegex(string $regex, string $path, array $paramNames): ?array
    {
        if (preg_match($regex, $path, $matches)) {
            array_shift($matches);
            return array_combine($paramNames, $matches);
        }
        return null;
    }
}

// 使用
$router = new AdvancedRouter();
$router->add('GET', '/users/:id', function (array $params) {
    echo "GET 用户 ID: {$params['id']}\n";
});

$router->add('POST', '/users', function () {
    echo "创建用户\n";
});

$result = $router->match('GET', '/users/123');
if ($result !== null) {
    $result['handler']($result['params']);
}
```

## 使用场景

### 友好 URL 实现

- 将 `/page.php?id=123` 转换为 `/pages/123`
- 将 `/user.php?name=john` 转换为 `/users/john`
- 提高 URL 可读性

### RESTful API

- 实现 RESTful 风格的 API
- 使用 HTTP 方法区分操作
- 支持资源嵌套

### MVC 框架

- 现代 MVC 框架都提供路由功能
- 将 URL 映射到控制器和方法
- 支持中间件和参数验证

### SEO 优化

- 创建 SEO 友好的 URL
- 包含关键词的 URL
- 清晰的 URL 结构

## 注意事项

### 路由性能

- **路由缓存**：缓存编译后的路由规则
- **匹配顺序**：更具体的路由放在前面
- **正则优化**：优化正则表达式性能

### 路由冲突

- **优先级**：定义路由的优先级
- **冲突检测**：检测路由冲突
- **测试覆盖**：测试所有路由

### 参数验证

- **类型验证**：验证参数类型
- **范围验证**：验证参数范围
- **格式验证**：验证参数格式

### 安全性考虑

- **路径遍历防护**：防止 `../` 攻击
- **参数验证**：验证所有参数
- **权限检查**：检查路由权限

## 常见问题

### 如何实现 URL 路由？

1. **配置 Web 服务器**：配置 Apache 或 Nginx 重写规则
2. **创建路由表**：定义路由规则和处理程序
3. **实现路由匹配**：实现路由匹配算法
4. **提取参数**：从 URL 中提取参数
5. **调用处理程序**：调用对应的处理程序

### .htaccess 如何配置？

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?route=$1 [L,QSA]
```

### 如何提取路由参数？

使用正则表达式匹配 URL，提取参数值：

```php
<?php
declare(strict_types=1);

$pattern = '/users/:id';
$path = '/users/123';

// 转换为正则表达式
$regex = preg_replace('/:(\w+)/', '([^/]+)', $pattern);
$regex = '#^' . $regex . '$#';

// 匹配并提取
if (preg_match($regex, $path, $matches)) {
    $id = $matches[1];
    echo "ID: {$id}\n";
}
```

### 路由和重写的区别？

- **URL 重写**：Web 服务器层面的 URL 转换（Apache、Nginx）
- **路由**：应用层面的 URL 映射（PHP 代码）
- **关系**：URL 重写将请求转发到路由系统

## 最佳实践

### 设计清晰的路由规则

- 使用 RESTful 风格
- 保持 URL 简洁
- 使用有意义的路径

### 使用框架的路由系统

- 使用 Laravel、Symfony 等框架的路由系统
- 利用框架提供的功能（中间件、验证等）
- 遵循框架的最佳实践

### 优化路由匹配性能

- 缓存路由规则
- 优化匹配顺序
- 使用高效的正则表达式

### 提供路由文档

- 文档化所有路由
- 说明参数和返回值
- 提供使用示例

## 相关章节

- **[5.14 路由设计](../chapter-14-routing/readme.md)**：了解路由设计的深入内容
- **[5.7 RESTful API 设计](../chapter-07-restful-api/readme.md)**：了解 RESTful API 的路由设计
- **[5.4.1 URL 解析与构建](section-01-url-parsing.md)**：了解 URL 解析的使用方法

## 练习任务

1. **实现简单路由系统**
   - 创建路由类
   - 实现路由匹配
   - 支持参数提取

2. **配置 URL 重写**
   - 配置 Apache .htaccess
   - 或配置 Nginx rewrite
   - 测试重写规则

3. **实现 RESTful 路由**
   - 支持 HTTP 方法
   - 实现资源路由
   - 支持嵌套资源

4. **实现路由中间件**
   - 支持路由中间件
   - 实现认证检查
   - 实现权限验证

5. **实现路由缓存**
   - 缓存路由规则
   - 优化匹配性能
   - 支持路由热重载

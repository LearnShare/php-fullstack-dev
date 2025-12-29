# 5.14.2 路由组织与模块化

## 概述

路由组织是管理大量路由的关键。理解路由组织方法、掌握模块化策略、实现路由分组和前缀，对于构建清晰、可维护的大型 Web 应用至关重要。本节详细介绍路由组织概述、路由分组、路由前缀、模块化路由、路由文件组织、命名空间路由等内容，帮助零基础学员设计清晰的路由结构。

随着应用规模的增长，路由数量会急剧增加。合理的路由组织可以提高代码的可维护性和可读性。理解路由组织方法对于管理大型应用的路由至关重要。

**主要内容**：
- 路由组织概述（为什么需要组织、组织原则、组织方法）
- 路由分组（分组概念、分组规则、共享中间件、共享前缀）
- 路由前缀（前缀作用、API 版本前缀、模块前缀、前缀配置）
- 模块化路由（按模块组织、路由文件分离、自动加载、模块注册）
- 路由文件组织（文件结构、文件命名、文件加载、文件管理）
- 命名空间路由（命名空间概念、命名空间路由、控制器命名空间）
- 实际应用示例和最佳实践

## 特性

- **清晰组织**：清晰的路由组织结构
- **易于维护**：易于维护和扩展
- **模块化**：支持模块化设计
- **可重用**：可重用的路由配置
- **可扩展**：易于扩展新功能

## 路由组织概述

### 为什么需要组织

1. **可维护性**：大量路由难以维护
2. **可读性**：清晰的组织提高可读性
3. **模块化**：支持模块化设计
4. **团队协作**：便于团队协作
5. **代码复用**：提高代码复用

### 组织原则

1. **按功能分组**：按功能模块分组
2. **按资源分组**：按资源类型分组
3. **按版本分组**：按 API 版本分组
4. **层次清晰**：保持层次清晰
5. **易于查找**：易于查找和维护

### 组织方法

**主要方法**：
- **路由分组**：将相关路由分组
- **路由前缀**：使用前缀区分路由
- **模块化**：按模块组织路由
- **文件分离**：分离路由文件

## 路由分组

### 分组概念

路由分组是将相关的路由组织在一起，共享配置和中间件。

### 分组规则

**示例**：
```php
<?php
declare(strict_types=1);

// 路由分组
Route::group(['prefix' => 'api', 'middleware' => 'auth'], function() {
    Route::get('/users', 'UserController@index');
    Route::get('/users/{id}', 'UserController@show');
    Route::post('/users', 'UserController@store');
});
```

### 共享中间件

**示例**：
```php
<?php
declare(strict_types=1);

// 共享中间件
Route::group(['middleware' => ['auth', 'throttle']], function() {
    Route::get('/dashboard', 'DashboardController@index');
    Route::get('/profile', 'ProfileController@index');
});
```

### 共享前缀

**示例**：
```php
<?php
declare(strict_types=1);

// 共享前缀
Route::group(['prefix' => 'admin'], function() {
    Route::get('/users', 'Admin\UserController@index');
    Route::get('/posts', 'Admin\PostController@index');
});
```

## 路由前缀

### 前缀作用

路由前缀用于区分不同模块或版本的路由。

### API 版本前缀

**示例**：
```php
<?php
declare(strict_types=1);

// API 版本前缀
Route::group(['prefix' => 'api/v1'], function() {
    Route::get('/users', 'Api\V1\UserController@index');
});

Route::group(['prefix' => 'api/v2'], function() {
    Route::get('/users', 'Api\V2\UserController@index');
});
```

### 模块前缀

**示例**：
```php
<?php
declare(strict_types=1);

// 模块前缀
Route::group(['prefix' => 'admin'], function() {
    Route::get('/users', 'Admin\UserController@index');
    Route::get('/posts', 'Admin\PostController@index');
});

Route::group(['prefix' => 'api'], function() {
    Route::get('/users', 'Api\UserController@index');
    Route::get('/posts', 'Api\PostController@index');
});
```

### 前缀配置

**示例**：
```php
<?php
declare(strict_types=1);

class RouteGroup
{
    private string $prefix = '';
    private array $middlewares = [];
    private array $routes = [];

    public function prefix(string $prefix): self
    {
        $this->prefix = $prefix;
        return $this;
    }

    public function middleware(array $middlewares): self
    {
        $this->middlewares = $middlewares;
        return $this;
    }

    public function get(string $path, callable $handler): self
    {
        $this->routes[] = [
            'method' => 'GET',
            'path' => $this->prefix . $path,
            'handler' => $handler,
            'middlewares' => $this->middlewares,
        ];
        return $this;
    }
}
```

## 模块化路由

### 按模块组织

**目录结构**：
```
routes/
  web.php          # Web 路由
  api.php          # API 路由
  admin.php        # 管理路由
  modules/
    users.php      # 用户模块路由
    posts.php      # 文章模块路由
    comments.php   # 评论模块路由
```

### 路由文件分离

**示例**：
```php
<?php
declare(strict_types=1);

// routes/web.php
Route::get('/', 'HomeController@index');
Route::get('/about', 'AboutController@index');

// routes/api.php
Route::group(['prefix' => 'api'], function() {
    Route::get('/users', 'Api\UserController@index');
    Route::get('/posts', 'Api\PostController@index');
});

// routes/admin.php
Route::group(['prefix' => 'admin', 'middleware' => 'auth'], function() {
    Route::get('/dashboard', 'Admin\DashboardController@index');
    Route::get('/users', 'Admin\UserController@index');
});
```

### 自动加载

**示例**：
```php
<?php
declare(strict_types=1);

function loadRoutes(string $directory): void
{
    $files = glob($directory . '/*.php');
    
    foreach ($files as $file) {
        require $file;
    }
}

// 加载路由
loadRoutes(__DIR__ . '/routes');
```

### 模块注册

**示例**：
```php
<?php
declare(strict_types=1);

class RouteManager
{
    private array $modules = [];

    public function registerModule(string $name, callable $routes): void
    {
        $this->modules[$name] = $routes;
    }

    public function loadModule(string $name): void
    {
        if (isset($this->modules[$name])) {
            $this->modules[$name]();
        }
    }

    public function loadAllModules(): void
    {
        foreach ($this->modules as $routes) {
            $routes();
        }
    }
}

// 使用
$routeManager = new RouteManager();

$routeManager->registerModule('users', function() {
    Route::get('/users', 'UserController@index');
    Route::get('/users/{id}', 'UserController@show');
});

$routeManager->loadAllModules();
```

## 路由文件组织

### 文件结构

**推荐结构**：
```
routes/
  web.php
  api.php
  admin.php
  modules/
    users.php
    posts.php
    comments.php
```

### 文件命名

**命名规范**：
- 使用小写字母
- 使用下划线分隔
- 使用描述性名称

**示例**：
```
routes/
  web.php
  api_v1.php
  api_v2.php
  admin_users.php
  admin_posts.php
```

### 文件加载

**示例**：
```php
<?php
declare(strict_types=1);

class RouteLoader
{
    public function load(string $directory): void
    {
        $files = $this->getRouteFiles($directory);
        
        foreach ($files as $file) {
            $this->loadFile($file);
        }
    }

    private function getRouteFiles(string $directory): array
    {
        $files = [];
        $iterator = new \RecursiveIteratorIterator(
            new \RecursiveDirectoryIterator($directory)
        );
        
        foreach ($iterator as $file) {
            if ($file->isFile() && $file->getExtension() === 'php') {
                $files[] = $file->getPathname();
            }
        }
        
        return $files;
    }

    private function loadFile(string $file): void
    {
        require $file;
    }
}
```

### 文件管理

**示例**：
```php
<?php
declare(strict_types=1);

class RouteFileManager
{
    private array $loadedFiles = [];

    public function load(string $file): void
    {
        if (in_array($file, $this->loadedFiles, true)) {
            return;  // 避免重复加载
        }
        
        require $file;
        $this->loadedFiles[] = $file;
    }

    public function reload(string $file): void
    {
        $key = array_search($file, $this->loadedFiles, true);
        if ($key !== false) {
            unset($this->loadedFiles[$key]);
        }
        
        $this->load($file);
    }
}
```

## 命名空间路由

### 命名空间概念

命名空间路由使用命名空间组织控制器，避免控制器名称冲突。

### 命名空间路由

**示例**：
```php
<?php
declare(strict_types=1);

// 使用命名空间
Route::group(['namespace' => 'Admin'], function() {
    Route::get('/users', 'UserController@index');  // Admin\UserController
    Route::get('/posts', 'PostController@index');  // Admin\PostController
});

Route::group(['namespace' => 'Api\V1'], function() {
    Route::get('/users', 'UserController@index');  // Api\V1\UserController
    Route::get('/posts', 'PostController@index');  // Api\V1\PostController
});
```

### 控制器命名空间

**示例**：
```php
<?php
declare(strict_types=1);

namespace App\Controllers\Admin;

class UserController
{
    public function index()
    {
        return new Response('Admin Users');
    }
}

namespace App\Controllers\Api\V1;

class UserController
{
    public function index()
    {
        return new Response('API V1 Users');
    }
}
```

## 使用场景

### 大型应用

- 大量路由管理
- 模块化设计
- 团队协作

### 模块化应用

- 功能模块分离
- 独立模块开发
- 模块复用

### API 版本管理

- API 版本控制
- 版本兼容
- 版本迁移

### 团队协作

- 分工开发
- 代码组织
- 易于维护

## 注意事项

### 组织清晰

- **层次清晰**：保持层次清晰
- **命名规范**：遵循命名规范
- **易于查找**：易于查找和维护

### 避免重复

- **避免重复路由**：避免定义重复的路由
- **路由冲突检测**：检测路由冲突
- **统一管理**：统一管理路由

### 易于维护

- **文档化**：文档化路由组织
- **注释说明**：添加注释说明
- **版本控制**：使用版本控制

### 性能考虑

- **加载优化**：优化路由文件加载
- **缓存使用**：使用路由缓存
- **延迟加载**：延迟加载路由

## 常见问题

### 如何组织路由？

按功能、资源、版本等分组，使用路由分组、前缀、模块化等方法。

### 如何实现路由分组？

使用路由分组功能，共享配置和中间件。

### 如何管理模块路由？

按模块组织路由文件，使用自动加载和模块注册。

### 路由组织的原则？

按功能分组、保持层次清晰、易于查找和维护。

## 最佳实践

### 按模块组织路由

- 按功能模块组织
- 使用路由文件分离
- 支持模块独立开发

### 使用路由分组

- 共享配置和中间件
- 使用路由前缀
- 清晰的层次结构

### 统一路由前缀

- API 版本前缀
- 模块前缀
- 统一前缀规范

### 分离路由文件

- 按功能分离文件
- 清晰的文件结构
- 易于维护和扩展

## 相关章节

- **[5.14.1 路由设计概述](section-01-overview.md)**：了解路由设计概述的详细内容
- **[5.14.3 动态路由与参数](section-03-dynamic.md)**：了解动态路由的详细内容
- **[5.14.4 路由守卫与权限控制](section-04-guards.md)**：了解路由守卫的详细内容

## 练习任务

1. **实现路由分组**
   - 路由分组功能
   - 共享中间件
   - 共享前缀

2. **实现模块化路由**
   - 按模块组织路由
   - 路由文件分离
   - 自动加载

3. **实现路由文件管理**
   - 文件结构设计
   - 文件加载机制
   - 文件管理工具

4. **实现命名空间路由**
   - 命名空间支持
   - 控制器命名空间
   - 命名空间路由匹配

5. **实现完整的路由组织系统**
   - 路由分组和模块化
   - 路由文件管理
   - 路由文档和测试

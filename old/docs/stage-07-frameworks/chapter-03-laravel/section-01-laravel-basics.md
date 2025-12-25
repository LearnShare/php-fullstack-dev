# 7.3.1 Laravel 基础

## 概述

Laravel 是现代 PHP 框架的代表。本节介绍 Laravel 的架构、服务容器、路由系统、中间件等核心概念。

## Laravel 架构

### MVC 架构

```
Request → Middleware → Route → Controller → Model → View → Response
```

### 目录结构

```
app/
  Http/
    Controllers/
    Middleware/
  Models/
  Services/
bootstrap/
config/
database/
  migrations/
  seeders/
public/
resources/
  views/
routes/
  web.php
  api.php
storage/
tests/
```

## 服务容器

### Laravel 容器使用

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Illuminate\Support\Facades\App;

// 绑定服务
App::bind(EmailServiceInterface::class, SmtpEmailService::class);

// 单例绑定
App::singleton(EmailServiceInterface::class, SmtpEmailService::class);

// 解析服务
$emailService = App::make(EmailServiceInterface::class);

// 使用依赖注入
class UserService
{
    public function __construct(
        private EmailServiceInterface $emailService
    ) {}
}
```

## 路由系统

### 基础路由

```php
<?php
declare(strict_types=1);

// routes/web.php
use Illuminate\Support\Facades\Route;

Route::get('/users', function () {
    return 'User list';
});

Route::post('/users', function () {
    return 'Create user';
});

Route::get('/users/{id}', function ($id) {
    return "User ID: {$id}";
});
```

### 控制器路由

```php
<?php
declare(strict_types=1);

// routes/web.php
use App\Http\Controllers\UserController;

Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::get('/users/{id}', [UserController::class, 'show']);
Route::put('/users/{id}', [UserController::class, 'update']);
Route::delete('/users/{id}', [UserController::class, 'destroy']);

// 资源路由
Route::resource('users', UserController::class);
```

### 路由组

```php
<?php
declare(strict_types=1);

Route::prefix('api')->middleware('auth')->group(function () {
    Route::get('/users', [UserController::class, 'index']);
    Route::post('/users', [UserController::class, 'store']);
});
```

## 中间件

### 创建中间件

```php
<?php
declare(strict_types=1);

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class AuthMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        if (!$request->user()) {
            return response('Unauthorized', 401);
        }
        
        return $next($request);
    }
}
```

### 注册中间件

```php
<?php
declare(strict_types=1);

// app/Http/Kernel.php
protected $middleware = [
    // 全局中间件
];

protected $middlewareGroups = [
    'web' => [
        // Web 中间件组
    ],
    'api' => [
        // API 中间件组
    ],
];

protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

## 完整示例

```php
<?php
declare(strict_types=1);

// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        return response()->json($users);
    }
    
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string',
            'email' => 'required|email|unique:users',
        ]);
        
        $user = User::create($validated);
        return response()->json($user, 201);
    }
    
    public function show($id)
    {
        $user = User::findOrFail($id);
        return response()->json($user);
    }
}

// routes/api.php
use App\Http\Controllers\UserController;

Route::middleware('auth:api')->group(function () {
    Route::apiResource('users', UserController::class);
});
```

## 最佳实践

1. **使用资源控制器**：遵循 RESTful 设计
2. **路由缓存**：生产环境使用路由缓存
3. **中间件**：合理使用中间件
4. **服务容器**：使用依赖注入

## 注意事项

1. 遵循 Laravel 的命名约定
2. 使用 Eloquent ORM 进行数据库操作
3. 合理使用中间件
4. 注意路由顺序

## 练习

1. 创建一个 Laravel 项目，配置路由和控制器。

2. 实现认证中间件，保护路由。

3. 使用服务容器注册和解析服务。

4. 创建一个完整的 RESTful API。

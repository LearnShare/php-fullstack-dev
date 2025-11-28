# 7.3 Laravel（2025 最新特性）

## 目标

- 了解 Laravel 的核心架构和设计思想。
- 掌握 Livewire、Inertia 等现代前端集成方案。
- 熟悉 Octane 的性能优化能力。
- 了解 Reverb、Horizon、Telescope 等扩展工具。
- 了解 Laravel 11 的新特性和改进。

## Laravel 核心

### 服务容器

```php
<?php
// 绑定
app()->bind(UserRepository::class, DatabaseUserRepository::class);

// 单例
app()->singleton(EmailService::class, SmtpEmailService::class);

// 解析
$service = app(UserService::class);
```

### 路由

```php
<?php
// web.php
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::get('/users/{user}', [UserController::class, 'show']);

// API 路由
Route::apiResource('users', UserController::class);
```

### Eloquent ORM

```php
<?php
// 模型
class User extends Model
{
    protected $fillable = ['name', 'email'];
    
    public function orders()
    {
        return $this->hasMany(Order::class);
    }
}

// 使用
$user = User::find(1);
$orders = $user->orders;
```

## Livewire

### 组件

```php
<?php
namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;
    
    public function increment()
    {
        $this->count++;
    }
    
    public function render()
    {
        return view('livewire.counter');
    }
}
```

## Inertia

### 控制器

```php
<?php
use Inertia\Inertia;

class UserController extends Controller
{
    public function index()
    {
        return Inertia::render('Users/Index', [
            'users' => User::all(),
        ]);
    }
}
```

## Octane

### 配置

```bash
# 安装 Octane
composer require laravel/octane

# 启动 Octane
php artisan octane:start --server=swoole
```

## Laravel 11 新特性

### 简化的目录结构

```php
// Laravel 11 简化了目录结构
// 不再需要 app/Http/Kernel.php
// 不再需要 bootstrap/app.php（简化版）

// 新的应用结构
app/
├── Http/
│   └── Controllers/
├── Models/
└── Services/
```

### 新的配置方式

```php
// config/app.php 简化
// 使用环境变量替代大量配置项

// .env
APP_NAME="My App"
APP_ENV=local
```

### 改进的路由

```php
<?php
// routes/web.php
// Laravel 11 支持更简洁的路由定义

use App\Http\Controllers\UserController;

Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);

// 资源路由
Route::apiResource('users', UserController::class);
```

### 新的中间件语法

```php
<?php
// Laravel 11 支持更灵活的中间件定义
Route::middleware(['auth', 'throttle:60,1'])->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
});
```

## Livewire 3

### 新特性

- 更快的性能
- 改进的组件系统
- 更好的 TypeScript 支持

```php
<?php
namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public int $count = 0;
    
    public function increment(): void
    {
        $this->count++;
    }
    
    public function render()
    {
        return view('livewire.counter');
    }
}
```

## Inertia.js 详细使用

### 安装和配置

```bash
composer require inertiajs/inertia-laravel

php artisan inertia:middleware
```

### 控制器使用

```php
<?php
namespace App\Http\Controllers;

use Inertia\Inertia;

class UserController extends Controller
{
    public function index()
    {
        return Inertia::render('Users/Index', [
            'users' => User::all(),
        ]);
    }
    
    public function show(User $user)
    {
        return Inertia::render('Users/Show', [
            'user' => $user,
        ]);
    }
}
```

### 前端集成（Vue/React）

```javascript
// resources/js/Pages/Users/Index.vue
<template>
  <div>
    <h1>Users</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
defineProps({
  users: Array
})
</script>
```

## 练习

1. 创建一个 Laravel 11 项目，实现用户管理的 CRUD 功能。

2. 使用 Livewire 3 创建一个实时计数器组件。

3. 配置 Octane，对比性能提升。

4. 集成 Horizon 监控队列任务。

5. 使用 Telescope 调试应用。

6. 使用 Inertia.js 创建一个 SPA 应用。

7. 实现一个完整的 Laravel 应用，包含认证、授权、API 等功能。

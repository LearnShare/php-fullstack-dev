# 7.3.5 2025 最新特性

## 概述

Laravel 持续演进，2025 年引入了许多新特性。本节介绍 Laravel 11 新特性、Livewire、Inertia、Octane、Reverb 等最新功能。

## Laravel 11 新特性

### 简化的应用结构

```php
<?php
// Laravel 11 简化了应用结构
// 不再需要 app/Http/Kernel.php
// 中间件直接在 bootstrap/app.php 配置

// bootstrap/app.php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->web(append: [
            \App\Http\Middleware\HandleCors::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

### 模型类型提示

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    // Laravel 11 支持模型属性类型提示
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
    
    // 类型提示属性
    public function getEmailAttribute(): string
    {
        return $this->attributes['email'];
    }
}
```

## Livewire

### 组件创建

```php
<?php
declare(strict_types=1);

namespace App\Livewire;

use Livewire\Component;
use App\Models\User;

class UserList extends Component
{
    public $users = [];
    
    public function mount()
    {
        $this->users = User::all();
    }
    
    public function delete($id)
    {
        User::find($id)->delete();
        $this->users = User::all();
    }
    
    public function render()
    {
        return view('livewire.user-list');
    }
}
```

### 视图

```blade
{{-- resources/views/livewire/user-list.blade.php --}}
<div>
    <h1>Users</h1>
    <ul>
        @foreach($users as $user)
            <li>
                {{ $user->name }}
                <button wire:click="delete({{ $user->id }})">Delete</button>
            </li>
        @endforeach
    </ul>
</div>
```

## Inertia

### 安装配置

```bash
composer require inertiajs/inertia-laravel
php artisan inertia:middleware
```

### 使用 Inertia

```php
<?php
declare(strict_types=1);

namespace App\Http\Controllers;

use Inertia\Inertia;
use App\Models\User;

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

### 安装配置

```bash
composer require laravel/octane
php artisan octane:install
php artisan octane:start
```

### 性能优化

```php
<?php
declare(strict_types=1);

// Octane 支持长生命周期应用
// 可以在请求之间共享状态

use Laravel\Octane\Facades\Octane;

Octane::concurrently([
    fn() => Http::get('https://api.example.com/users'),
    fn() => Http::get('https://api.example.com/posts'),
]);
```

## Reverb

### WebSocket 服务器

```bash
composer require laravel/reverb
php artisan reverb:install
php artisan reverb:start
```

### 广播事件

```php
<?php
declare(strict_types=1);

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class UserCreated implements ShouldBroadcast
{
    public function __construct(
        public User $user
    ) {}
    
    public function broadcastOn(): Channel
    {
        return new Channel('users');
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// Laravel 11 + Livewire + Inertia 示例
class UserController extends Controller
{
    public function index()
    {
        // 使用 Inertia 渲染
        return Inertia::render('Users/Index', [
            'users' => User::all(),
        ]);
    }
}

// Livewire 组件
class UserForm extends Component
{
    public $name = '';
    public $email = '';
    
    public function save()
    {
        User::create([
            'name' => $this->name,
            'email' => $this->email,
        ]);
        
        // 广播事件
        broadcast(new UserCreated(User::latest()->first()));
        
        session()->flash('message', 'User created!');
    }
    
    public function render()
    {
        return view('livewire.user-form');
    }
}
```

## 最佳实践

1. **Livewire**：适合交互式组件
2. **Inertia**：适合 SPA 应用
3. **Octane**：提升性能
4. **Reverb**：实时功能

## 注意事项

1. Livewire 适合简单交互，复杂逻辑使用 API
2. Inertia 需要前端框架配合
3. Octane 需要特殊配置
4. Reverb 需要 WebSocket 支持

## 练习

1. 使用 Livewire 创建交互式组件。

2. 配置 Inertia，创建 SPA 应用。

3. 安装 Octane，提升应用性能。

4. 使用 Reverb 实现实时功能。

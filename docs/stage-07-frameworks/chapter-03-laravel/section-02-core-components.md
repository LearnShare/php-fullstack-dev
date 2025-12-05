# 7.3.2 核心组件

## 概述

Laravel 的核心组件包括 Eloquent ORM、Blade 模板、队列系统、事件系统等。本节详细介绍这些核心组件。

## Eloquent ORM

### 模型定义

```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'users';
    protected $fillable = ['name', 'email', 'password'];
    protected $hidden = ['password'];
    
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

### 查询操作

```php
<?php
declare(strict_types=1);

// 查询
$users = User::all();
$user = User::find(1);
$user = User::where('email', 'test@example.com')->first();

// 创建
$user = User::create([
    'name' => 'John',
    'email' => 'john@example.com',
]);

// 更新
$user->update(['name' => 'Jane']);

// 删除
$user->delete();
```

## Blade 模板

### 基础语法

```blade
{{-- resources/views/users/index.blade.php --}}
@extends('layouts.app')

@section('content')
    <h1>Users</h1>
    <ul>
        @foreach($users as $user)
            <li>{{ $user->name }} - {{ $user->email }}</li>
        @endforeach
    </ul>
@endsection
```

### 组件

```blade
{{-- resources/views/components/alert.blade.php --}}
<div class="alert alert-{{ $type }}">
    {{ $slot }}
</div>

{{-- 使用 --}}
<x-alert type="success">
    User created successfully!
</x-alert>
```

## 队列系统

### 创建任务

```php
<?php
declare(strict_types=1);

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public function __construct(
        private User $user
    ) {}
    
    public function handle(): void
    {
        Mail::to($this->user->email)->send(new WelcomeMail($this->user));
    }
}

// 分发任务
SendWelcomeEmail::dispatch($user);
```

## 事件系统

### 定义事件

```php
<?php
declare(strict_types=1);

namespace App\Events;

use App\Models\User;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class UserCreated
{
    use Dispatchable, SerializesModels;
    
    public function __construct(
        public User $user
    ) {}
}
```

### 事件监听器

```php
<?php
declare(strict_types=1);

namespace App\Listeners;

use App\Events\UserCreated;

class SendWelcomeEmail
{
    public function handle(UserCreated $event): void
    {
        Mail::to($event->user->email)->send(new WelcomeMail($event->user));
    }
}

// 注册监听器
// app/Providers/EventServiceProvider.php
protected $listen = [
    UserCreated::class => [
        SendWelcomeEmail::class,
    ],
];
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 控制器
class UserController extends Controller
{
    public function store(Request $request)
    {
        $user = User::create($request->validated());
        
        // 触发事件
        event(new UserCreated($user));
        
        return response()->json($user, 201);
    }
}

// 模型
class User extends Model
{
    protected static function booted()
    {
        static::created(function ($user) {
            // 自动发送欢迎邮件
            SendWelcomeEmail::dispatch($user);
        });
    }
}
```

## 最佳实践

1. **Eloquent**：使用 Eloquent 进行数据库操作
2. **Blade**：使用 Blade 模板渲染视图
3. **队列**：耗时操作使用队列
4. **事件**：使用事件解耦代码

## 注意事项

1. Eloquent 查询应该优化，避免 N+1 问题
2. Blade 模板应该保持简洁
3. 队列任务应该可重试
4. 事件应该轻量，避免耗时操作

## 练习

1. 使用 Eloquent 创建模型和关系。

2. 创建 Blade 模板，渲染用户列表。

3. 实现队列任务，发送邮件。

4. 使用事件系统，解耦业务逻辑。

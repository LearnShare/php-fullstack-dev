# 7.3.3 数据库与 ORM

## 概述

Laravel 的数据库操作主要通过 Eloquent ORM 完成。本节介绍数据库配置、Eloquent 使用、关系映射、查询优化等内容。

## 数据库配置

### 配置文件

```php
<?php
// config/database.php
return [
    'default' => env('DB_CONNECTION', 'mysql'),
    
    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
        ],
    ],
];
```

## Eloquent 使用

### 基础查询

```php
<?php
declare(strict_types=1);

// 查询所有
$users = User::all();

// 条件查询
$users = User::where('active', true)->get();
$user = User::where('email', 'test@example.com')->first();

// 分页
$users = User::paginate(15);

// 排序
$users = User::orderBy('created_at', 'desc')->get();
```

### 关系查询

```php
<?php
declare(strict_types=1);

// 预加载关系
$users = User::with('posts')->get();

// 条件预加载
$users = User::with(['posts' => function($query) {
    $query->where('published', true);
}])->get();

// 延迟加载
$user = User::find(1);
$posts = $user->posts; // 延迟加载
```

## 关系映射

### 一对一

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}

class Profile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

### 一对多

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

### 多对多

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}

class Role extends Model
{
    public function users()
    {
        return $this->belongsToMany(User::class);
    }
}

// 使用
$user->roles()->attach($roleId);
$user->roles()->detach($roleId);
$user->roles()->sync([$roleId1, $roleId2]);
```

## 查询优化

### 避免 N+1 问题

```php
<?php
declare(strict_types=1);

// 不推荐：N+1 查询
$users = User::all();
foreach ($users as $user) {
    $posts = $user->posts; // 每次循环都查询
}

// 推荐：预加载
$users = User::with('posts')->get();
foreach ($users as $user) {
    $posts = $user->posts; // 已预加载，不查询
}
```

### 查询作用域

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function scopeActive($query)
    {
        return $query->where('active', true);
    }
    
    public function scopeEmail($query, string $email)
    {
        return $query->where('email', $email);
    }
}

// 使用
$activeUsers = User::active()->get();
$user = User::active()->email('test@example.com')->first();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class UserController extends Controller
{
    public function index()
    {
        $users = User::with(['posts', 'profile'])
            ->active()
            ->orderBy('created_at', 'desc')
            ->paginate(15);
        
        return view('users.index', compact('users'));
    }
    
    public function store(Request $request)
    {
        $user = User::create($request->validated());
        
        // 创建关联数据
        $user->profile()->create($request->input('profile'));
        
        return response()->json($user->load('profile'), 201);
    }
}
```

## 最佳实践

1. **预加载关系**：使用 `with()` 避免 N+1 问题
2. **查询作用域**：使用作用域封装常用查询
3. **批量操作**：使用批量插入和更新
4. **索引优化**：为常用查询字段创建索引

## 注意事项

1. 避免在循环中查询数据库
2. 使用预加载优化关系查询
3. 合理使用分页
4. 注意查询性能

## 练习

1. 创建 Eloquent 模型，定义关系。

2. 实现查询作用域，封装常用查询。

3. 优化查询，避免 N+1 问题。

4. 实现复杂的关联查询。

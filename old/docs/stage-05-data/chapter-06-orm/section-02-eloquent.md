# 5.6.2 Eloquent 使用

## 概述

Eloquent 是 Laravel 的 ORM 系统，提供了简洁优雅的数据库操作接口。本节详细介绍 Eloquent 基础、CRUD 操作、查询构建器，以及关联关系。

## Eloquent 基础

### 基础模型

```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'users';
    protected $primaryKey = 'id';
    public $timestamps = true;
    
    protected $fillable = [
        'username',
        'email',
        'password_hash',
    ];
    
    protected $hidden = [
        'password_hash',
    ];
    
    protected $casts = [
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];
}
```

## CRUD 操作

### Create

```php
<?php
declare(strict_types=1);

use App\Models\User;

// 方式一：创建实例后保存
$user = new User();
$user->username = 'alice';
$user->email = 'alice@example.com';
$user->password_hash = password_hash('password', PASSWORD_DEFAULT);
$user->save();

// 方式二：使用 create
$user = User::create([
    'username' => 'bob',
    'email' => 'bob@example.com',
    'password_hash' => password_hash('password', PASSWORD_DEFAULT),
]);
```

### Read

```php
<?php
declare(strict_types=1);

// 查找单个记录
$user = User::find(1);
$user = User::where('email', 'alice@example.com')->first();

// 查找多条记录
$users = User::where('status', 'active')->get();

// 条件查询
$users = User::where('age', '>', 18)
    ->where('status', 'active')
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

### Update

```php
<?php
declare(strict_types=1);

// 方式一：查找后更新
$user = User::find(1);
$user->email = 'newemail@example.com';
$user->save();

// 方式二：批量更新
User::where('id', 1)->update(['email' => 'newemail@example.com']);
```

### Delete

```php
<?php
declare(strict_types=1);

// 方式一：查找后删除
$user = User::find(1);
$user->delete();

// 方式二：直接删除
User::destroy(1);

// 软删除
$user->delete();  // 设置 deleted_at
```

## 查询构建器

```php
<?php
declare(strict_types=1);

// 条件查询
$users = User::where('status', 'active')
    ->where('age', '>', 18)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();

// 关联查询
$users = User::whereHas('orders', function ($query) {
    $query->where('total', '>', 100);
})->get();

// 聚合查询
$count = User::where('status', 'active')->count();
$avgAge = User::avg('age');
$maxAge = User::max('age');
```

## 关联关系

### 一对多

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function orders()
    {
        return $this->hasMany(Order::class);
    }
}

class Order extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}

// 使用
$user = User::find(1);
$orders = $user->orders;  // 获取用户的所有订单

$order = Order::find(1);
$user = $order->user;  // 获取订单的用户
```

### 多对多

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function roles()
    {
        return $this->belongsToMany(Role::class, 'user_roles');
    }
}

// 使用
$user = User::find(1);
$roles = $user->roles;  // 获取用户的所有角色
```

## 完整示例

```php
<?php
declare(strict_types=1);

class UserService
{
    public function createUser(array $data): User
    {
        return User::create([
            'username' => $data['username'],
            'email' => $data['email'],
            'password_hash' => password_hash($data['password'], PASSWORD_DEFAULT),
        ]);
    }

    public function getUserWithOrders(int $userId): ?User
    {
        return User::with('orders')->find($userId);
    }

    public function getActiveUsers(int $limit = 10): \Illuminate\Database\Eloquent\Collection
    {
        return User::where('status', 'active')
            ->orderBy('created_at', 'desc')
            ->limit($limit)
            ->get();
    }
}
```

## 注意事项

1. **批量赋值**：使用 `$fillable` 或 `$guarded` 保护字段
2. **关联预加载**：使用 `with()` 避免 N+1 问题
3. **查询优化**：合理使用索引和查询优化
4. **性能考虑**：复杂查询可能需要使用原生 SQL

## 练习

1. 创建一个完整的 Eloquent 模型，包含 CRUD 操作。

2. 实现关联关系查询，使用预加载优化性能。

3. 编写一个查询构建器，支持复杂的条件查询。

4. 实现一个数据访问层，封装 Eloquent 操作。

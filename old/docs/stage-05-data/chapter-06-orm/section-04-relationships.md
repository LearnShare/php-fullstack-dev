# 5.6.4 关系映射

## 概述

ORM 的关系映射功能可以自动处理表之间的关联关系。本节详细介绍一对一、一对多、多对多关系，关联查询，以及关联操作。

## 一对一

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function profile()
    {
        return $this->hasOne(UserProfile::class);
    }
}

class UserProfile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}

// 使用
$user = User::find(1);
$profile = $user->profile;
```

## 一对多

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
    
    public function items()
    {
        return $this->hasMany(OrderItem::class);
    }
}

// 使用
$user = User::find(1);
$orders = $user->orders;  // 获取用户的所有订单

$order = Order::find(1);
$user = $order->user;  // 获取订单的用户
$items = $order->items;  // 获取订单的所有项
```

## 多对多

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

class Role extends Model
{
    public function users()
    {
        return $this->belongsToMany(User::class, 'user_roles');
    }
}

// 使用
$user = User::find(1);
$roles = $user->roles;  // 获取用户的所有角色

// 附加关系
$user->roles()->attach($roleId);
$user->roles()->detach($roleId);
$user->roles()->sync([$roleId1, $roleId2]);
```

## 关联查询

### 预加载（Eager Loading）

```php
<?php
declare(strict_types=1);

// 避免 N+1 问题
$users = User::with('orders')->get();

// 嵌套预加载
$users = User::with('orders.items')->get();

// 条件预加载
$users = User::with(['orders' => function ($query) {
    $query->where('status', 'paid');
}])->get();
```

### 关联查询

```php
<?php
declare(strict_types=1);

// 查询有订单的用户
$users = User::whereHas('orders', function ($query) {
    $query->where('total', '>', 100);
})->get();

// 查询订单数量
$users = User::withCount('orders')->get();
foreach ($users as $user) {
    echo $user->orders_count;
}
```

## 关联操作

```php
<?php
declare(strict_types=1);

$user = User::find(1);

// 创建关联记录
$order = $user->orders()->create([
    'total' => 99.99,
    'status' => 'pending',
]);

// 附加多对多关系
$user->roles()->attach([1, 2, 3]);

// 同步多对多关系
$user->roles()->sync([1, 2, 3]);

// 切换关系
$user->roles()->toggle([1, 2]);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class UserService
{
    public function getUserWithRelations(int $userId): ?User
    {
        return User::with(['orders.items', 'roles'])
            ->find($userId);
    }

    public function getUsersWithOrderCount(): \Illuminate\Database\Eloquent\Collection
    {
        return User::withCount('orders')
            ->having('orders_count', '>', 0)
            ->get();
    }

    public function assignRole(int $userId, int $roleId): void
    {
        $user = User::find($userId);
        $user->roles()->attach($roleId);
    }
}
```

## 注意事项

1. **N+1 问题**：使用 `with()` 预加载避免 N+1 问题
2. **关联查询**：合理使用 `whereHas` 和 `withCount`
3. **性能优化**：预加载关联数据提升性能
4. **关系维护**：正确维护关联关系的数据一致性

## 练习

1. 实现完整的关联关系，包含一对一、一对多、多对多。

2. 优化关联查询，避免 N+1 问题。

3. 编写关联操作函数，处理关联数据的增删改。

4. 实现关联查询优化，提升查询性能。

# 6.4.2 Eloquent 使用

## 概述

Eloquent 是 Laravel 框架的 ORM（Object-Relational Mapping），采用 Active Record 模式，提供了简洁优雅的 API 来操作数据库。Eloquent 将数据库表映射为模型类，将表中的行映射为模型实例，使开发者可以使用面向对象的方式操作数据库。

本节详细介绍 Eloquent 的使用方法，包括模型定义、查询构建、CRUD 操作、查询方法、批量操作、性能优化等，帮助零基础学员掌握 Eloquent 的使用。

**主要内容**：
- Eloquent 概述和特点
- 模型定义和约定
- 查询构建器和方法
- CRUD 操作详解
- 查询优化和性能考虑
- 完整示例和最佳实践

---

## 特性

- **Active Record 模式**：模型既表示数据，又包含操作方法
- **简洁 API**：提供简洁优雅的 API
- **链式查询**：支持链式查询构建
- **关系处理**：简化表关系的处理
- **查询优化**：提供多种查询优化方法

---

## Eloquent 概述

### Eloquent 的特点

Eloquent 的主要特点包括：

1. **Active Record 模式**：模型类既表示数据，又包含操作方法
2. **约定优于配置**：遵循约定，减少配置
3. **链式查询**：支持链式查询构建
4. **关系处理**：简化表关系的处理
5. **查询优化**：提供多种查询优化方法

### Active Record 模式

Eloquent 采用 Active Record 模式：

- **模型即数据**：模型类既表示数据，又包含操作方法
- **静态方法**：使用静态方法进行查询
- **实例方法**：使用实例方法进行操作

**示例**：

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    // 模型定义
}

// 静态方法查询
$user = User::find(1);

// 实例方法操作
$user->name = 'John';
$user->save();
```

### 模型约定

Eloquent 遵循约定优于配置的原则：

- **表名**：模型类名复数形式（User → users）
- **主键**：默认为 `id`
- **时间戳**：默认包含 `created_at` 和 `updated_at`

**示例**：

```php
<?php
declare(strict_types=1);

// 模型类：User
// 对应表：users（自动推断）
// 主键：id（默认）
// 时间戳：created_at, updated_at（默认）

class User extends Model
{
    // 如果表名不是 users，可以指定
    // protected $table = 'my_users';
    
    // 如果主键不是 id，可以指定
    // protected $primaryKey = 'user_id';
    
    // 如果不需要时间戳，可以禁用
    // public $timestamps = false;
}
```

---

## 模型定义

### 模型类创建

创建 Eloquent 模型类。

**语法**：`php artisan make:model ModelName`

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    // 模型定义
}
```

### 表名约定

Eloquent 自动推断表名，也可以手动指定。

**约定**：

- 模型类名复数形式：`User` → `users`
- 模型类名复数形式：`Post` → `posts`

**手动指定**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    protected $table = 'my_users';  // 手动指定表名
}
```

### 主键设置

Eloquent 默认主键为 `id`，也可以自定义。

**示例**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    protected $primaryKey = 'user_id';  // 自定义主键
    
    public $incrementing = false;  // 非自增主键
    
    protected $keyType = 'string';  // 主键类型
}
```

### 时间戳

Eloquent 默认包含 `created_at` 和 `updated_at` 时间戳。

**示例**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    // 禁用时间戳
    public $timestamps = false;
    
    // 自定义时间戳字段名
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'last_update';
    
    // 自定义时间戳格式
    protected $dateFormat = 'U';  // Unix 时间戳
}
```

### 可填充字段

使用 `$fillable` 或 `$guarded` 定义可填充字段。

**示例**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    // 白名单：只允许填充这些字段
    protected $fillable = [
        'name',
        'email',
        'password'
    ];
    
    // 黑名单：不允许填充这些字段
    protected $guarded = [
        'id',
        'is_admin'
    ];
    
    // 或者允许所有字段（不推荐）
    // protected $guarded = [];
}
```

### 隐藏字段

使用 `$hidden` 定义序列化时隐藏的字段。

**示例**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    protected $hidden = [
        'password',
        'remember_token'
    ];
    
    // 或者使用 $visible 定义可见字段
    // protected $visible = ['name', 'email'];
}
```

---

## 查询构建器

### 链式查询

Eloquent 支持链式查询构建。

**示例**：

```php
<?php
declare(strict_types=1);

// 链式查询
$users = User::where('status', 'active')
    ->where('age', '>', 18)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

### 条件查询

使用 `where` 方法进行条件查询。

**示例**：

```php
<?php
declare(strict_types=1);

// 等值查询
$users = User::where('status', 'active')->get();

// 比较查询
$users = User::where('age', '>', 18)->get();
$users = User::where('age', '>=', 18)->get();
$users = User::where('age', '<', 65)->get();
$users = User::where('age', '<=', 65)->get();
$users = User::where('age', '!=', 30)->get();

// LIKE 查询
$users = User::where('name', 'like', '%John%')->get();

// IN 查询
$users = User::whereIn('id', [1, 2, 3])->get();

// NULL 查询
$users = User::whereNull('deleted_at')->get();
$users = User::whereNotNull('email')->get();

// 多条件查询
$users = User::where('status', 'active')
    ->where('age', '>', 18)
    ->get();

// OR 查询
$users = User::where('status', 'active')
    ->orWhere('status', 'pending')
    ->get();

// 复杂条件
$users = User::where(function ($query) {
    $query->where('status', 'active')
        ->orWhere('status', 'pending');
})->where('age', '>', 18)
    ->get();
```

### 排序和分页

使用 `orderBy` 和 `limit` 进行排序和分页。

**示例**：

```php
<?php
declare(strict_types=1);

// 排序
$users = User::orderBy('created_at', 'desc')->get();
$users = User::orderBy('name', 'asc')->orderBy('age', 'desc')->get();

// 限制数量
$users = User::limit(10)->get();
$users = User::take(10)->get();

// 跳过记录
$users = User::skip(10)->take(10)->get();
$users = User::offset(10)->limit(10)->get();

// 分页
$users = User::paginate(15);  // 每页 15 条
$users = User::simplePaginate(15);  // 简单分页
```

### 聚合查询

使用聚合方法进行统计查询。

**示例**：

```php
<?php
declare(strict_types=1);

// 计数
$count = User::count();
$count = User::where('status', 'active')->count();

// 最大值
$maxAge = User::max('age');

// 最小值
$minAge = User::min('age');

// 平均值
$avgAge = User::avg('age');

// 求和
$total = Order::sum('total');

// 分组统计
$stats = User::selectRaw('status, count(*) as count')
    ->groupBy('status')
    ->get();
```

---

## CRUD 操作

### 创建操作

使用 `create` 或 `save` 方法创建记录。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 create（推荐）
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password')
]);

// 方法 2：使用 new 和 save
$user = new User();
$user->name = 'John Doe';
$user->email = 'john@example.com';
$user->password = bcrypt('password');
$user->save();

// 方法 3：使用 firstOrCreate（不存在则创建）
$user = User::firstOrCreate(
    ['email' => 'john@example.com'],
    ['name' => 'John Doe', 'password' => bcrypt('password')]
);

// 方法 4：使用 updateOrCreate（不存在则创建，存在则更新）
$user = User::updateOrCreate(
    ['email' => 'john@example.com'],
    ['name' => 'John Doe', 'password' => bcrypt('password')]
);
```

### 读取操作

使用 `find`、`get`、`first` 等方法读取记录。

**示例**：

```php
<?php
declare(strict_types=1);

// 根据主键查找
$user = User::find(1);
$user = User::findOrFail(1);  // 不存在则抛出异常

// 查找多个
$users = User::find([1, 2, 3]);

// 查找第一个
$user = User::where('email', 'john@example.com')->first();
$user = User::where('email', 'john@example.com')->firstOrFail();

// 查找所有
$users = User::all();
$users = User::get();

// 条件查询
$users = User::where('status', 'active')->get();

// 查找或创建
$user = User::firstOrNew(['email' => 'john@example.com']);
$user->name = 'John Doe';
$user->save();
```

### 更新操作

使用 `update` 或 `save` 方法更新记录。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 update（推荐）
$user = User::find(1);
$user->update([
    'name' => 'Jane Doe',
    'email' => 'jane@example.com'
]);

// 方法 2：使用 save
$user = User::find(1);
$user->name = 'Jane Doe';
$user->email = 'jane@example.com';
$user->save();

// 批量更新
User::where('status', 'inactive')
    ->update(['status' => 'active']);

// 使用 updateOrCreate
$user = User::updateOrCreate(
    ['email' => 'john@example.com'],
    ['name' => 'John Doe']
);
```

### 删除操作

使用 `delete` 或 `destroy` 方法删除记录。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 delete
$user = User::find(1);
$user->delete();

// 方法 2：使用 destroy（根据主键）
User::destroy(1);
User::destroy([1, 2, 3]);

// 批量删除
User::where('status', 'inactive')->delete();

// 软删除（如果模型支持）
$user = User::find(1);
$user->delete();  // 设置 deleted_at，不真正删除

// 强制删除（软删除模型）
$user = User::withTrashed()->find(1);
$user->forceDelete();  // 真正删除
```

---

## 查询方法

### find() 方法

`find()` 方法根据主键查找记录。

**语法**：`Model::find($id)`

**示例**：

```php
<?php
declare(strict_types=1);

$user = User::find(1);
// 等价于：SELECT * FROM users WHERE id = 1

$users = User::find([1, 2, 3]);
// 等价于：SELECT * FROM users WHERE id IN (1, 2, 3)
```

### where() 方法

`where()` 方法添加查询条件。

**语法**：`Model::where($column, $operator, $value)`

**示例**：

```php
<?php
declare(strict_types=1);

// 等值查询
$users = User::where('status', 'active')->get();

// 比较查询
$users = User::where('age', '>', 18)->get();

// 多条件
$users = User::where('status', 'active')
    ->where('age', '>', 18)
    ->get();
```

### get() 方法

`get()` 方法获取查询结果集合。

**语法**：`Model::get()`

**示例**：

```php
<?php
declare(strict_types=1);

$users = User::get();  // 获取所有记录
$users = User::where('status', 'active')->get();  // 获取符合条件的记录
```

### first() 方法

`first()` 方法获取查询结果的第一条记录。

**语法**：`Model::first()`

**示例**：

```php
<?php
declare(strict_types=1);

$user = User::where('email', 'john@example.com')->first();
// 返回 User 对象或 null
```

### all() 方法

`all()` 方法获取所有记录。

**语法**：`Model::all()`

**示例**：

```php
<?php
declare(strict_types=1);

$users = User::all();
// 等价于：SELECT * FROM users
```

---

## 批量操作

### 批量插入

使用 `insert` 方法批量插入。

**示例**：

```php
<?php
declare(strict_types=1);

User::insert([
    ['name' => 'John', 'email' => 'john@example.com'],
    ['name' => 'Jane', 'email' => 'jane@example.com'],
    ['name' => 'Bob', 'email' => 'bob@example.com']
]);
```

### 批量更新

使用 `update` 方法批量更新。

**示例**：

```php
<?php
declare(strict_types=1);

// 批量更新
User::where('status', 'inactive')
    ->update(['status' => 'active']);

// 使用 upsert（不存在则插入，存在则更新）
User::upsert(
    [
        ['id' => 1, 'name' => 'John', 'email' => 'john@example.com'],
        ['id' => 2, 'name' => 'Jane', 'email' => 'jane@example.com']
    ],
    ['id'],  // 唯一键
    ['name', 'email']  // 更新字段
);
```

### 批量删除

使用 `delete` 方法批量删除。

**示例**：

```php
<?php
declare(strict_types=1);

// 批量删除
User::where('status', 'inactive')->delete();

// 根据主键删除
User::destroy([1, 2, 3]);
```

---

## 查询优化

### 预加载（Eager Loading）

使用 `with` 方法预加载关联数据，避免 N+1 问题。

**示例**：

```php
<?php
declare(strict_types=1);

// N+1 问题
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count();  // 每个用户都执行一次查询
}
// 执行了 1 + N 次查询

// 使用预加载
$users = User::with('posts')->get();
foreach ($users as $user) {
    echo $user->posts->count();  // 只执行 2 次查询
}
// 执行了 2 次查询（1 次查询用户，1 次查询所有文章）

// 预加载多个关联
$users = User::with(['posts', 'profile'])->get();

// 条件预加载
$users = User::with(['posts' => function ($query) {
    $query->where('status', 'published');
}])->get();
```

### 选择字段

使用 `select` 方法选择需要的字段。

**示例**：

```php
<?php
declare(strict_types=1);

// 只选择需要的字段
$users = User::select('id', 'name', 'email')->get();

// 避免选择大字段
$users = User::select('id', 'name')->get();
// 不选择 content、description 等大字段
```

### 查询缓存

使用查询缓存提高性能。

**示例**：

```php
<?php
declare(strict_types=1);

// 缓存查询结果
$users = Cache::remember('users', 3600, function () {
    return User::all();
});

// 或者使用 remember 方法
$users = User::remember(3600)->get();
```

---

## 完整示例

### 用户管理示例

```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = ['name', 'email', 'password'];
    
    protected $hidden = ['password'];
    
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
    
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}

// 使用示例
// 创建用户
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password')
]);

// 查询用户
$user = User::find(1);
$user = User::where('email', 'john@example.com')->first();
$users = User::where('status', 'active')->get();

// 更新用户
$user->update(['name' => 'Jane Doe']);

// 删除用户
$user->delete();

// 关系查询
$posts = $user->posts;
$profile = $user->profile;

// 预加载
$users = User::with('posts', 'profile')->get();
```

---

## 使用场景

### Laravel 应用

Eloquent 是 Laravel 框架的 ORM，适合 Laravel 应用。

### 快速开发

Eloquent 提供简洁的 API，适合快速开发。

### 关系处理

Eloquent 简化表关系的处理，适合需要处理复杂关系的应用。

---

## 注意事项

### 模型约定

遵循 Eloquent 的约定，减少配置。

### 查询性能

注意查询性能，避免 N+1 问题。

### N+1 问题

使用预加载避免 N+1 问题。

### 批量操作

使用批量操作提高性能。

---

## 常见问题

### 如何定义 Eloquent 模型？

继承 `Model` 类，遵循约定。

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = ['name', 'email'];
}
```

### 如何进行查询？

使用查询构建器方法。

```php
<?php
declare(strict_types=1);

$users = User::where('status', 'active')->get();
```

### 如何处理 N+1 问题？

使用 `with` 方法预加载关联数据。

```php
<?php
declare(strict_types=1);

$users = User::with('posts')->get();
```

### Eloquent 的性能优化？

优化方法：

- 使用预加载
- 选择需要的字段
- 使用查询缓存
- 使用批量操作

---

## 最佳实践

### 遵循模型约定

遵循 Eloquent 的约定，减少配置。

### 使用查询优化

使用预加载、选择字段等方法优化查询。

### 处理 N+1 问题

使用 `with` 方法预加载关联数据。

### 使用批量操作

使用批量操作提高性能。

---

## 练习任务

1. **模型定义**
   - 创建 Eloquent 模型
   - 定义模型属性
   - 测试模型功能
   - 验证模型约定

2. **CRUD 操作**
   - 实现创建操作
   - 实现查询操作
   - 实现更新操作
   - 实现删除操作

3. **查询构建**
   - 实现条件查询
   - 实现排序和分页
   - 实现聚合查询
   - 测试查询性能

4. **性能优化**
   - 识别 N+1 问题
   - 使用预加载优化
   - 测试性能差异
   - 编写优化报告

5. **综合应用**
   - 创建一个完整的应用
   - 使用 Eloquent 实现功能
   - 优化查询性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.4.1 ORM 基础](section-01-orm-basics.md)
- [6.4.3 数据迁移](section-03-migrations.md)
- [6.4.4 关系映射](section-04-relationships.md)

# 6.4.2 Eloquent 模型与 CRUD 操作

## 概述

在 Eloquent ORM 中，**模型 (Model)** 是一个核心概念。它是一个直接与数据库表交互的 PHP 类，封装了表的业务逻辑、关系和数据操作。每一个模型类的实例，都对应着表中的一行数据。

本节将详细介绍如何定义一个 Eloquent 模型，并利用它来执行优雅、直观的 CRUD (Create, Read, Update, Delete) 操作。

## 定义第一个模型

创建模型非常简单，只需创建一个继承自 `Illuminate\Database\Eloquent\Model` 的类即可。

**核心原则：约定优于配置 (Convention over Configuration)**
Eloquent 会基于**命名约定**来自动推断表名、主键等信息，从而极大地简化了配置。
-   **类名**：使用单数形式，首字母大写（如 `User`）。
-   **表名**：Eloquent 会自动推断为类名的**复数、蛇形命名**形式（如 `User` -> `users`）。
-   **主键**：Eloquent 假定每张表都有一个名为 `id` 的自增主键。
-   **时间戳**：Eloquent 假定表中有 `created_at` 和 `updated_at` 两个 `TIMESTAMP` 类型的字段，并在创建和更新时自动维护它们。

**示例：创建一个 User 模型**
在你的项目中创建一个目录，例如 `app/Models/`，然后在其中创建 `User.php` 文件。

`app/Models/User.php`
```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    // 模型代码将写在这里
}
```
**注意**：为了让 Composer 能够找到这个类，你需要在 `composer.json` 中配置 PSR-4 自动加载：
```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    }
},
```
配置后，记得运行 `composer dump-autoload`。

### 覆盖默认约定

如果你的表结构不符合 Eloquent 的约定，可以轻松地在模型中通过属性来覆盖它们：

```php
class User extends Model
{
    /**
     * @var string 指定模型关联的表名
     */
    protected $table = 'my_users';

    /**
     * @var string 指定表的主键
     */
    protected $primaryKey = 'user_id';

    /**
     * @var bool 指定主键是否为自增
     */
    public $incrementing = false;

    /**
     * @var string 指定主键的类型
     */
    protected $keyType = 'string';

    /**
     * @var bool 指定模型是否自动维护时间戳
     */
    public $timestamps = false;
}
```

### 批量赋值 (Mass Assignment)

为了方便，我们有时会希望通过一个数组来一次性创建或更新模型。但这是一个安全风险，因为恶意用户可能会提交额外的字段来修改他们不应访问的数据（例如，`is_admin` 字段）。

为了防止这种情况，Eloquent 提供了**批量赋值保护**。你必须在模型中明确指定哪些字段是“可以”被批量赋值的。

-   `$fillable` 属性：一个“白名单”，数组中列出的字段**允许**被批量赋值。
-   `$guarded` 属性：一个“黑名单”，数组中列出的字段**禁止**被批量赋值。`$guarded = ['*']` 会禁止所有字段。

**最佳实践**：使用 `$fillable`，因为它更明确、更安全。

```php
class User extends Model
{
    /**
     * @var array 允许被批量赋值的属性
     */
    protected $fillable = [
        'username',
        'email',
        'password_hash', // 在实际项目中，密码应在赋值前单独处理
    ];
}
```

## CRUD 操作

在你的业务逻辑代码中，确保先引入 Eloquent 启动文件和需要使用的模型类。
```php
require_once __DIR__ . '/bootstrap.php';
use App\Models\User;
```

### 1. 创建 (Create)

**方法一：实例化并保存**
```php
$user = new User();
$user->username = 'eloquent_user';
$user->email = 'eloquent@example.com';
$user->password_hash = password_hash('secret', PASSWORD_DEFAULT);
$user->save(); // save() 会执行 INSERT 操作

echo "新用户已创建，ID: " . $user->id;
```

**方法二：使用 `create()` 静态方法（批量赋值）**
这种方法更简洁，但要求 `fillable` 属性已被正确设置。
```php
$newUser = User::create([
    'username' => 'another_user',
    'email' => 'another@example.com',
    'password_hash' => password_hash('secret123', PASSWORD_DEFAULT),
]);

echo "新用户已创建，ID: " . $newUser->id;
```

### 2. 读取 (Read)

**获取所有记录**:
```php
$users = User::all(); // 返回一个 Eloquent 集合对象

foreach ($users as $user) {
    echo $user->username . "\n";
}
```
**警告**：`all()` 会一次性加载所有记录，对于大表，请谨慎使用。

**通过主键查找**:
```php
// 查找 id=1 的用户
$user = User::find(1);
if ($user) {
    echo "找到用户: " . $user->username;
}

// 查找，如果不存在则抛出 ModelNotFoundException 异常
$user = User::findOrFail(1);
```

**使用查询构造器 (Query Builder)**
Eloquent 最强大的功能之一是它的查询构造器。你可以像调用普通方法一样，链式地构建复杂的查询。

```php
// 获取所有活跃用户，并按用户名排序
$activeUsers = User::where('status', 'active')
                   ->orderBy('username', 'desc')
                   ->get(); // get() 执行查询并返回集合

// 获取第一个匹配的用户
$firstAdmin = User::where('is_admin', true)->first();

// 查找并返回特定列
$emails = User::where('status', 'active')->pluck('email');
```

### 3. 更新 (Update)

**方法一：先查找，再保存**
这是最常用的方式，因为它会触发模型事件（我们将在后续章节学习）。
```php
$user = User::find(1);
$user->email = 'admin.new@example.com';
$user->save(); // save() 会自动执行 UPDATE 操作
```

**方法二：批量更新**
对符合条件的记录进行批量更新。这会直接在数据库层面执行，**不会**触发模型事件。
```php
$affectedRows = User::where('status', 'inactive')->update(['status' => 'archived']);
echo "更新了 {" . $affectedRows . "} 个用户。";
```

### 4. 删除 (Delete)

**方法一：通过模型实例删除**
```php
$user = User::find(10);
if ($user) {
    $user->delete();
}
```

**方法二：通过主键删除**
```php
// 删除一个或多个用户
User::destroy(10);
User::destroy([11, 12, 13]);
```

**方法三：批量删除**
```php
$deletedRows = User::where('last_login_at', '<', '2020-01-01')->delete();
```

### 软删除 (Soft Deletes)

有时我们不希望真正地从数据库中删除一条记录，而是想“标记”它为已删除。Eloquent 提供了优雅的软删除功能。

1.  **修改数据表**：在表中增加一个 `deleted_at` 字段，类型为 `TIMESTAMP`，允许为 `NULL`。
2.  **在模型中启用**：在模型类中 `use Illuminate\Database\Eloquent\SoftDeletes;` 这个 Trait。

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes; // 启用软删除
    
    // ... 其他模型代码
}
```
现在，当你调用 `$user->delete()` 时，Eloquent 不会执行 `DELETE` 语句，而是会自动设置 `deleted_at` 字段为当前时间。同时，所有正常的查询（如 `all()`, `find()`, `where()`）都会**自动**在查询条件中加入 `WHERE deleted_at IS NULL`，从而将软删除的记录排除在外。

---

## 练习任务

1.  **创建 Product 模型**：
    为你之前创建的 `products` 表（包含 `id`, `name`, `price`, `stock`, `version`）创建一个 `Product` 模型。确保正确配置了 `$fillable` 属性。

2.  **实现 CRUD 逻辑**：
    编写一个 `product_manager.php` 脚本，在其中：
    -   使用 `Product::create()` 创建一个新产品。
    -   使用 `Product::find()` 查找该产品并打印其名称。
    -   修改该产品的价格，并使用 `save()` 方法更新。
    -   最后，使用 `destroy()` 方法将其删除。

3.  **查询练习**：
    使用 Eloquent 查询构造器，编写代码来完成以下查询：
    -   查找所有价格高于 100 且库存小于 10 的商品。
    -   查找所有商品中价格最高的前 5 件商品。
    -   统计共有多少种不同的商品。

```
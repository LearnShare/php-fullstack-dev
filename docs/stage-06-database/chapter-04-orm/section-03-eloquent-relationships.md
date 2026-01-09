# 6.4.3 Eloquent 模型关系 (Eloquent Model Relationships)

## 概述

ORM 最强大的功能之一，就是能够以面向对象的方式来定义和操作数据表之间的关系。通过在模型中定义关系，我们可以轻松地获取关联数据，而无需手动编写 `JOIN` 查询。例如，获取一个用户的所有文章，可以简单地通过 `$user->posts` 来实现。

Eloquent 支持所有主要的关系类型，本节将重点介绍三种最核心的关系：一对一、一对多和多对多。此外，我们还将探讨一个使用 ORM 时至关重要的性能优化主题——**预加载 (Eager Loading)**。

## 定义关系

在 Eloquent 中，关系是通过在模型类中定义一个**公共方法**来实现的。该方法的名称通常是关联模型的名称（单数或复数形式）。在这个方法内部，你会调用 Eloquent 提供的关系函数（如 `hasOne`, `hasMany`, `belongsTo` 等）并返回其结果。

返回的关系对象本身也是一个**查询构造器**，这意味着你可以在其上继续链式调用 `where`, `orderBy` 等方法。

---

## 一对一 (One-to-One)

**场景**：一个 `User` (用户) 只有一个 `UserProfile` (用户资料)。

**数据表结构**：
-   `users`: `id`, `username`, ...
-   `user_profiles`: `id`, `user_id`, `full_name`, `bio`, ... (`user_id` 是外键)

### 定义关系

在 `User` 模型中，我们定义一个 `profile()` 方法来获取其关联的资料：
`app/Models/User.php`
```php
public function profile()
{
    // 一个 User 拥有一个 UserProfile
    // Eloquent 会自动假定外键是 'user_id' (基于 User 类名)
    return $this->hasOne(UserProfile::class); 
}
```

在 `UserProfile` 模型中，我们定义反向关系：
`app/Models/UserProfile.php`
```php
public function user()
{
    // 一个 UserProfile 归属于一个 User
    return $this->belongsTo(User::class);
}
```

### 使用关系

```php
// 获取用户的资料
$user = User::find(1);
$profile = $user->profile; // 动态属性，首次访问时会执行查询
echo $profile->full_name;

// 获取资料所属的用户
$profile = UserProfile::find(1);
$user = $profile->user;
echo $user->username;

// 创建关联记录
$user = User::find(1);
$user->profile()->create([
    'full_name' => 'John Doe',
    'bio' => 'A passionate PHP developer.'
]);
```

---

## 一对多 (One-to-Many)

**场景**：一个 `User` (用户) 可以发表多篇 `Post` (文章)。

**数据表结构**：
-   `users`: `id`, `username`, ...
-   `posts`: `id`, `user_id`, `title`, ... (`user_id` 是外键)

### 定义关系

在 `User` 模型中：
`app/Models/User.php`
```php
public function posts()
{
    // 一个 User 拥有多篇 Post
    return $this->hasMany(Post::class);
}
```

在 `Post` 模型中定义反向关系：
`app/Models/Post.php`
```php
public function user()
{
    // 一篇 Post 归属于一个 User
    return $this->belongsTo(User::class);
}
```

### 使用关系

```php
// 获取一个用户的所有文章
$user = User::find(1);
$posts = $user->posts; // 返回一个 Eloquent 集合对象

foreach ($posts as $post) {
    echo $post->title . "\n";
}

// 在关系上进行二次查询
$publishedPosts = $user->posts()->where('is_published', true)->get();

// 创建一篇新文章
$user = User::find(1);
$newPost = $user->posts()->create([
    'title' => 'My First Eloquent Post',
    'content' => '...'
]);
```

---

## 多对多 (Many-to-Many)

**场景**：一篇 `Post` (文章)可以有多个 `Tag` (标签)，一个 `Tag` 也可以被用于多篇文章。

**数据表结构**：
-   `posts`: `id`, `title`, ...
-   `tags`: `id`, `name`, ...
-   `post_tag` (连接表): `post_id`, `tag_id`

### 定义关系

Eloquent 会自动推断连接表的名称是两个模型名按字母顺序排列的单数形式（`post_tag`）。

在 `Post` 模型中：
`app/Models/Post.php`
```php
public function tags()
{
    return $this->belongsToMany(Tag::class);
}
```

在 `Tag` 模型中：
`app/Models/Tag.php`
```php
public function posts()
{
    return $this->belongsToMany(Post::class);
}
```
### 使用关系

多对多关系的操作略有不同，主要涉及“附加”和“分离”关系。

```php
$post = Post::find(1);
$tag = Tag::find(5);

// 为文章附加一个标签
$post->tags()->attach($tag->id);

// 附加多个标签
$post->tags()->attach([2, 3, 4]);

// 从文章中移除一个标签
$post->tags()->detach($tag->id);

// 同步关系：让文章的标签只包含给定的 ID 列表
// Eloquent 会自动添加不存在的，移除多余的
$post->tags()->sync([1, 5, 8]);

// 获取文章的所有标签
$tags = $post->tags;
foreach ($tags as $tag) {
    echo $tag->name;
}
```

### 操作连接表中的额外字段

如果你的连接表（如 `post_tag`）还包含其他字段（例如 `created_at`），可以在定义关系时使用 `withTimestamps()` 或 `withPivot()` 来声明。

```php
// 在 Post 模型中
public function tags()
{
    return $this->belongsToMany(Tag::class)->withTimestamps();
}

// 使用时
foreach ($post->tags as $tag) {
    // 访问连接表中的 created_at 字段
    echo $tag->pivot->created_at;
}
```

---

## 预加载：解决 N+1 查询问题

这是使用 ORM 时**必须掌握**的性能优化技巧。

**问题（N+1 查询）**：
假设我们要显示 100 篇文章的列表，以及每篇文章的作者名。

**错误的代码**：
```php
$posts = Post::all(); // 执行了 1 次查询

foreach ($posts as $post) {
    // 在循环中，每次访问 $post->user 都会触发一次新的查询来获取作者信息
    echo $post->title . ' by ' . $post->user->username . "\n"; // 执行了 100 次查询
}
```
总计执行了 `1 + 100 = 101` 次查询，这就是 **N+1 问题**。当 N 很大时，这会给数据库带来巨大压力。

**解决方案：预加载 (Eager Loading)**
使用 `with()` 方法，我们可以告诉 Eloquent 在查询文章的同时，一次性地把所有相关的作者信息也提前加载出来。

**正确的代码**：
```php
// 使用 with() 进行预加载
$posts = Post::with('user')->get(); // 只执行了 2 次查询！

foreach ($posts as $post) {
    // 这里访问 $post->user 不会再触发新的查询，因为数据已经提前加载好了
    echo $post->title . ' by ' . $post->user->username . "\n";
}
```
Eloquent 是如何做到的？
1.  `SELECT * FROM posts`
2.  `SELECT * FROM users WHERE id IN (1, 2, 5, ...)` (这里的 ID 列表是第一步查出的所有 post 的 `user_id`)

**预加载多个和嵌套的关系**：
```php
// 同时预加载作者和标签
$posts = Post::with('user', 'tags')->get();

// 预加载嵌套关系：作者及其个人资料
$posts = Post::with('user.profile')->get();
```

**经验法则**：当你需要在一个循环中访问某个模型的关联模型时，几乎总是应该使用 `with()` 进行预加载。

---

## 练习任务

1.  **定义模型与关系**：
    为我们之前设计的 `authors` (作者) 和 `posts` (文章) 表创建 `Author` 和 `Post` 两个模型。并在模型中正确地定义它们之间的一对多关系。

2.  **实现文章和标签**：
    创建 `Tag` 模型，并实现 `Post` 和 `Tag` 之间的多对多关系。编写一个脚本，为一篇文章添加几个标签，然后再查询这篇文章及其所有标签并打印出来。

3.  **修复 N+1 问题**：
    假设你有一个 `categories` 表和 `products` 表（一对多关系）。编写一段代码，先获取所有分类，然后遍历每个分类，打印出该分类下所有商品的名称。
    -   首先，用“错误”的方式（会导致 N+1 问题）来实现它。
    -   然后，使用 `with()` 预加载来重构你的代码，解决 N+1 问题。

```
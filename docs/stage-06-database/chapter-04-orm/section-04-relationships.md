# 6.4.4 关系映射

## 概述

关系映射是 ORM 的核心功能之一，用于处理数据库表之间的关系。Eloquent 提供了简洁的 API 来定义和查询表之间的关系，包括一对一、一对多、多对多等关系类型。

本节详细介绍 Eloquent 的关系映射，包括关系类型（一对一、一对多、多对多、多态关系）、关系定义、关系查询、预加载优化等，帮助零基础学员掌握关系映射的使用方法。

**主要内容**：
- 关系映射的概念和类型
- 一对一关系（hasOne、belongsTo）
- 一对多关系（hasMany、belongsTo）
- 多对多关系（belongsToMany）
- 多态关系（morphTo、morphMany）
- 关系查询和预加载
- 完整示例和最佳实践

---

## 特性

- **关系定义**：简洁的关系定义方法
- **自动处理**：自动处理外键和关联表
- **便捷查询**：通过对象属性访问关联数据
- **预加载优化**：支持预加载，避免 N+1 问题
- **关系约束**：支持关系约束和条件查询

---

## 关系映射概念

### 什么是关系映射

关系映射是将数据库表之间的关系映射为对象之间的关系，使开发者可以使用面向对象的方式处理表关系。

**关系类型**：

- **一对一（1:1）**：一个模型对应另一个模型的一个实例
- **一对多（1:N）**：一个模型对应另一个模型的多个实例
- **多对多（M:N）**：一个模型的多个实例对应另一个模型的多个实例

### 关系映射的优势

关系映射的优势：

- **代码简洁**：使用对象属性访问关联数据
- **自动处理**：自动处理外键和关联表
- **类型安全**：提供类型检查和自动转换
- **查询优化**：支持预加载，优化查询性能

---

## 一对一关系

### hasOne 关系

`hasOne` 关系表示一个模型拥有另一个模型的一个实例。

**语法**：`hasOne(RelatedModel::class, $foreignKey, $localKey)`

**示例**：

```php
<?php
declare(strict_types=1);

// 用户表（users）
// id | name | email
// 1  | John | john@example.com

// 用户资料表（profiles）
// id | user_id | bio | avatar
// 1  | 1       | ... | ...

class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
        // 默认外键：user_id
        // 默认本地键：id
    }
}

// 使用
$user = User::find(1);
$profile = $user->profile;  // 自动查询关联的 Profile
```

### belongsTo 关系

`belongsTo` 关系表示一个模型属于另一个模型。

**语法**：`belongsTo(RelatedModel::class, $foreignKey, $ownerKey)`

**示例**：

```php
<?php
declare(strict_types=1);

class Profile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
        // 默认外键：user_id
        // 默认所有者键：id
    }
}

// 使用
$profile = Profile::find(1);
$user = $profile->user;  // 自动查询关联的 User
```

### 一对一关系示例

**完整示例**：

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

// 使用
$user = User::with('profile')->find(1);
echo $user->profile->bio;

$profile = Profile::with('user')->find(1);
echo $profile->user->name;
```

---

## 一对多关系

### hasMany 关系

`hasMany` 关系表示一个模型拥有另一个模型的多个实例。

**语法**：`hasMany(RelatedModel::class, $foreignKey, $localKey)`

**示例**：

```php
<?php
declare(strict_types=1);

// 用户表（users）
// id | name | email
// 1  | John | john@example.com

// 文章表（posts）
// id | user_id | title | content
// 1  | 1       | ...   | ...
// 2  | 1       | ...   | ...

class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class);
        // 默认外键：user_id
        // 默认本地键：id
    }
}

// 使用
$user = User::find(1);
$posts = $user->posts;  // 自动查询关联的所有 Post
```

### belongsTo 关系（一对多）

在一对多关系中，多的一方使用 `belongsTo`。

**示例**：

```php
<?php
declare(strict_types=1);

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
        // 默认外键：user_id
        // 默认所有者键：id
    }
}

// 使用
$post = Post::find(1);
$user = $post->user;  // 自动查询关联的 User
```

### 一对多关系示例

**完整示例**：

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

// 使用
$user = User::with('posts')->find(1);
foreach ($user->posts as $post) {
    echo $post->title;
}

$post = Post::with('user')->find(1);
echo $post->user->name;
```

---

## 多对多关系

### belongsToMany 关系

`belongsToMany` 关系表示多对多关系，需要通过中间表关联。

**语法**：`belongsToMany(RelatedModel::class, $table, $foreignPivotKey, $relatedPivotKey, $parentKey, $relatedKey)`

**示例**：

```php
<?php
declare(strict_types=1);

// 文章表（posts）
// id | title | content
// 1  | ...   | ...

// 标签表（tags）
// id | name
// 1  | PHP
// 2  | MySQL

// 中间表（post_tag）
// post_id | tag_id
// 1       | 1
// 1       | 2

class Post extends Model
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
        // 默认中间表：post_tag（按字母顺序）
        // 默认外键：post_id, tag_id
    }
}

class Tag extends Model
{
    public function posts()
    {
        return $this->belongsToMany(Post::class);
    }
}

// 使用
$post = Post::find(1);
$tags = $post->tags;  // 自动查询关联的所有 Tag

$tag = Tag::find(1);
$posts = $tag->posts;  // 自动查询关联的所有 Post
```

### 自定义中间表

可以自定义中间表名称和字段。

**示例**：

```php
<?php
declare(strict_types=1);

class Post extends Model
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class, 'article_tags', 'article_id', 'tag_id');
        // 中间表：article_tags
        // 外键：article_id, tag_id
    }
}
```

### 中间表数据

可以访问和操作中间表的数据。

**示例**：

```php
<?php
declare(strict_types=1);

class Post extends Model
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class)
            ->withPivot('created_at')  // 包含中间表字段
            ->withTimestamps();  // 包含时间戳
    }
}

// 使用
$post = Post::find(1);
foreach ($post->tags as $tag) {
    echo $tag->pivot->created_at;  // 访问中间表数据
}

// 附加关系
$post->tags()->attach(1, ['created_at' => now()]);
$post->tags()->detach(1);
$post->tags()->sync([1, 2, 3]);
```

---

## 多态关系

### morphTo 关系

`morphTo` 关系表示多态关系，一个模型可以属于多个不同类型的模型。

**语法**：`morphTo($name = null, $type = null, $id = null, $ownerKey = null)`

**示例**：

```php
<?php
declare(strict_types=1);

// 评论表（comments）
// id | commentable_type | commentable_id | content
// 1  | App\Models\Post   | 1              | ...
// 2  | App\Models\Video  | 1              | ...

class Comment extends Model
{
    public function commentable()
    {
        return $this->morphTo();
        // 默认类型字段：commentable_type
        // 默认 ID 字段：commentable_id
    }
}

// 使用
$comment = Comment::find(1);
$commentable = $comment->commentable;  // 可能是 Post 或 Video
```

### morphMany 关系

`morphMany` 关系表示一个模型拥有多个多态关联。

**语法**：`morphMany(RelatedModel::class, $name, $type = null, $id = null, $localKey = null)`

**示例**：

```php
<?php
declare(strict_types=1);

class Post extends Model
{
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

class Video extends Model
{
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

// 使用
$post = Post::find(1);
$comments = $post->comments;

$video = Video::find(1);
$comments = $video->comments;
```

---

## 关系查询

### 关系查询方法

使用关系方法进行查询。

**示例**：

```php
<?php
declare(strict_types=1);

// 查询关联数据
$user = User::find(1);
$posts = $user->posts()->where('status', 'published')->get();

// 条件查询
$user = User::find(1);
$activePosts = $user->posts()->where('status', 'active')->get();

// 计数
$user = User::find(1);
$postCount = $user->posts()->count();

// 存在性检查
$user = User::find(1);
$hasPosts = $user->posts()->exists();
```

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

// 嵌套预加载
$users = User::with('posts.tags')->get();
```

### 延迟加载（Lazy Loading）

关联数据在访问时自动加载。

**示例**：

```php
<?php
declare(strict_types=1);

$user = User::find(1);
// 此时还没有加载 posts

$posts = $user->posts;  // 访问时自动加载
// 执行查询：SELECT * FROM posts WHERE user_id = 1
```

---

## 关系约束

### 条件约束

在关系定义中添加条件约束。

**示例**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class)->where('status', 'published');
    }
    
    public function activePosts()
    {
        return $this->hasMany(Post::class)->where('status', 'active');
    }
}
```

### 排序约束

在关系定义中添加排序。

**示例**：

```php
<?php
declare(strict_types=1);

class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class)->orderBy('created_at', 'desc');
    }
}
```

---

## 完整示例

### 博客系统关系示例

```php
<?php
declare(strict_types=1);

// 用户模型
class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
    
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
    
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}

// 文章模型
class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
    
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
    
    public function category()
    {
        return $this->belongsTo(Category::class);
    }
}

// 评论模型
class Comment extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    public function commentable()
    {
        return $this->morphTo();
    }
}

// 标签模型
class Tag extends Model
{
    public function posts()
    {
        return $this->belongsToMany(Post::class);
    }
}

// 使用示例
// 查询用户及其文章
$user = User::with('posts')->find(1);
foreach ($user->posts as $post) {
    echo $post->title;
}

// 查询文章及其标签
$post = Post::with('tags')->find(1);
foreach ($post->tags as $tag) {
    echo $tag->name;
}

// 查询文章及其评论
$post = Post::with('comments.user')->find(1);
foreach ($post->comments as $comment) {
    echo $comment->user->name . ': ' . $comment->content;
}
```

---

## 使用场景

### 关系处理

使用关系映射处理表之间的关系。

### 数据查询

通过关系查询关联数据。

### 数据操作

通过关系操作关联数据。

---

## 注意事项

### N+1 问题

使用预加载避免 N+1 问题。

### 关系定义

正确定义关系，确保外键正确。

### 性能优化

使用预加载和条件查询优化性能。

---

## 常见问题

### 如何定义关系？

使用关系方法定义关系。

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
```

### 如何查询关联数据？

使用对象属性或关系方法查询。

```php
<?php
declare(strict_types=1);

$user = User::find(1);
$posts = $user->posts;  // 延迟加载
$posts = $user->posts()->get();  // 关系查询
```

### 如何处理 N+1 问题？

使用 `with` 方法预加载关联数据。

```php
<?php
declare(strict_types=1);

$users = User::with('posts')->get();
```

---

## 最佳实践

### 正确定义关系

确保关系定义正确，外键匹配。

### 使用预加载

使用预加载避免 N+1 问题。

### 优化查询

使用条件查询和排序优化性能。

---

## 练习任务

1. **关系定义**
   - 定义一对一关系
   - 定义一对多关系
   - 定义多对多关系
   - 测试关系功能

2. **关系查询**
   - 实现关系查询
   - 使用预加载优化
   - 测试查询性能
   - 验证数据正确性

3. **多态关系**
   - 实现多态关系
   - 测试多态查询
   - 验证关系功能
   - 优化查询性能

4. **性能优化**
   - 识别 N+1 问题
   - 使用预加载优化
   - 测试性能差异
   - 编写优化报告

5. **综合应用**
   - 创建一个完整的应用
   - 实现所有关系类型
   - 优化查询性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.4.1 ORM 基础](section-01-orm-basics.md)
- [6.4.2 Eloquent 使用](section-02-eloquent.md)
- [6.4.3 数据迁移](section-03-migrations.md)
- [6.5 Redis 缓存策略与性能优化](../chapter-05-redis/readme.md)

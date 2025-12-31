# 6.4.1 ORM 基础

## 概述

ORM（Object-Relational Mapping，对象关系映射）是一种编程技术，用于在面向对象编程语言和关系型数据库之间建立映射关系。ORM 将数据库表映射为类，将表中的行映射为对象，将表中的列映射为对象的属性，从而可以使用面向对象的方式操作数据库。

本节详细介绍 ORM 的概念、工作原理、优势与劣势、主要设计模式（Active Record、Data Mapper）、主流 PHP ORM 框架对比等，帮助零基础学员理解 ORM 的价值和使用场景。

**主要内容**：
- ORM 的概念和工作原理
- ORM 的优势与劣势
- ORM 设计模式（Active Record、Data Mapper）
- 主流 PHP ORM 框架对比
- ORM 的适用场景
- 完整示例和最佳实践

---

## 特性

- **面向对象**：使用面向对象的方式操作数据库
- **代码简化**：减少 SQL 代码，提高开发效率
- **数据库抽象**：隐藏数据库细节，提高可移植性
- **关系处理**：简化表关系的处理
- **类型安全**：提供类型检查和自动转换

---

## ORM 概念

### 什么是 ORM

ORM（Object-Relational Mapping，对象关系映射）是一种编程技术，用于在面向对象编程语言和关系型数据库之间建立映射关系。

**ORM 的核心思想**：

- **表映射为类**：数据库表对应 PHP 类
- **行映射为对象**：表中的行对应类的实例
- **列映射为属性**：表中的列对应对象的属性
- **关系映射**：表之间的关系对应对象之间的关系

**示例**：

```php
<?php
declare(strict_types=1);

// 数据库表：users
// id | name  | email
// 1  | John  | john@example.com
// 2  | Jane  | jane@example.com

// ORM 映射
class User
{
    public int $id;
    public string $name;
    public string $email;
}

// 使用 ORM
$user = User::find(1);
// 等价于：SELECT * FROM users WHERE id = 1
// 返回 User 对象，$user->name = 'John'
```

### ORM 的作用

ORM 的主要作用包括：

1. **简化数据库操作**：使用面向对象的方式操作数据库，无需编写 SQL
2. **提高开发效率**：减少重复代码，加快开发速度
3. **数据库抽象**：隐藏数据库细节，提高代码可移植性
4. **关系处理**：简化表关系的处理，如一对一、一对多、多对多

**示例**：

```php
<?php
declare(strict_types=1);

// 不使用 ORM（PDO）
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);
$data = $stmt->fetch(PDO::FETCH_ASSOC);
$user = new User();
$user->id = $data['id'];
$user->name = $data['name'];
$user->email = $data['email'];

// 使用 ORM
$user = User::find(1);
// 更简洁、更直观
```

### ORM 的工作原理

ORM 的工作原理：

1. **元数据映射**：定义表与类的映射关系
2. **SQL 生成**：将对象操作转换为 SQL 语句
3. **结果映射**：将查询结果映射为对象
4. **关系处理**：处理表之间的关系

**工作流程**：

```
对象操作 → ORM → SQL 语句 → 数据库 → 结果集 → ORM → 对象
```

**示例**：

```php
<?php
declare(strict_types=1);

// 1. 对象操作
$user = new User();
$user->name = 'John';
$user->email = 'john@example.com';
$user->save();

// 2. ORM 转换为 SQL
// INSERT INTO users (name, email) VALUES ('John', 'john@example.com')

// 3. 执行 SQL

// 4. 结果映射为对象
// $user->id = 1（自动填充自增 ID）
```

---

## ORM 的优势

### 开发效率

ORM 可以显著提高开发效率：

- **减少代码量**：无需编写大量 SQL 代码
- **快速开发**：快速实现 CRUD 操作
- **代码复用**：模型可以复用

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 ORM：简洁
$user = User::create([
    'name' => 'John',
    'email' => 'john@example.com'
]);

// 不使用 ORM：繁琐
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute([
    'name' => 'John',
    'email' => 'john@example.com'
]);
$id = $pdo->lastInsertId();
```

### 代码可读性

ORM 代码更易读、易理解：

- **面向对象**：使用面向对象的方式，更符合编程习惯
- **语义清晰**：代码语义更清晰，易于理解
- **维护方便**：代码结构更清晰，易于维护

**示例**：

```php
<?php
declare(strict_types=1);

// ORM：语义清晰
$user = User::where('email', 'john@example.com')->first();
$user->name = 'Jane';
$user->save();

// 原生 SQL：需要理解 SQL 语法
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => 'john@example.com']);
$data = $stmt->fetch();
$stmt = $pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
$stmt->execute(['name' => 'Jane', 'id' => $data['id']]);
```

### 数据库抽象

ORM 提供数据库抽象层：

- **数据库无关**：代码可以在不同数据库间移植
- **隐藏细节**：隐藏数据库特定的语法和特性
- **统一接口**：提供统一的 API 操作不同数据库

**示例**：

```php
<?php
declare(strict_types=1);

// ORM：数据库无关
$user = User::find(1);
// 可以在 MySQL、PostgreSQL、SQLite 等数据库上运行

// 原生 SQL：数据库相关
// MySQL
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
// PostgreSQL
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = $1');
```

### 关系处理

ORM 简化表关系的处理：

- **自动处理**：自动处理外键关系
- **便捷访问**：通过对象属性访问关联数据
- **延迟加载**：支持延迟加载，提高性能

**示例**：

```php
<?php
declare(strict_types=1);

// ORM：关系处理简单
$user = User::find(1);
$posts = $user->posts;  // 自动加载关联的文章

// 原生 SQL：需要手动 JOIN
$stmt = $pdo->prepare('
    SELECT u.*, p.* 
    FROM users u 
    LEFT JOIN posts p ON u.id = p.user_id 
    WHERE u.id = :id
');
$stmt->execute(['id' => 1]);
// 需要手动处理结果集
```

---

## ORM 的劣势

### 性能开销

ORM 可能带来性能开销：

- **查询生成**：需要将对象操作转换为 SQL，有额外开销
- **结果映射**：需要将结果集映射为对象，有额外开销
- **N+1 问题**：可能产生 N+1 查询问题

**示例**：

```php
<?php
declare(strict_types=1);

// ORM：可能产生 N+1 问题
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count();  // 每个用户都执行一次查询
}
// 执行了 1 + N 次查询（N 为用户数量）

// 优化：使用预加载
$users = User::with('posts')->get();
foreach ($users as $user) {
    echo $user->posts->count();  // 只执行 2 次查询
}
```

### 学习曲线

ORM 需要学习成本：

- **框架学习**：需要学习 ORM 框架的 API
- **概念理解**：需要理解 ORM 的概念和工作原理
- **最佳实践**：需要掌握 ORM 的最佳实践

### 灵活性限制

ORM 可能限制灵活性：

- **复杂查询**：复杂查询可能难以用 ORM 表达
- **性能优化**：某些性能优化可能需要绕过 ORM
- **数据库特性**：可能无法使用数据库特定特性

**示例**：

```php
<?php
declare(strict_types=1);

// 复杂查询：ORM 可能难以表达
// 需要使用原生 SQL
$users = DB::select('
    SELECT u.*, 
           COUNT(p.id) as post_count,
           SUM(p.views) as total_views
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    GROUP BY u.id
    HAVING post_count > 10
    ORDER BY total_views DESC
');
```

### 复杂查询

某些复杂查询可能难以用 ORM 表达：

- **子查询**：复杂的子查询
- **窗口函数**：窗口函数
- **存储过程**：存储过程调用

**解决方案**：

- 使用原生 SQL
- 使用查询构建器
- 结合 ORM 和原生 SQL

---

## ORM 设计模式

### Active Record 模式

Active Record 模式将数据访问逻辑封装在模型类中，模型类既表示数据，又包含操作数据的方法。

**特点**：

- **模型即数据**：模型类既表示数据，又包含操作方法
- **简单直观**：使用简单，代码直观
- **耦合度高**：模型与数据库耦合度高

**示例**：

```php
<?php
declare(strict_types=1);

// Eloquent（Active Record 模式）
class User extends Model
{
    protected $table = 'users';
}

// 使用
$user = User::find(1);  // 静态方法
$user->name = 'John';
$user->save();  // 实例方法
```

**代表框架**：

- **Eloquent**（Laravel）
- **Yii ActiveRecord**

### Data Mapper 模式

Data Mapper 模式将数据访问逻辑与模型类分离，模型类只表示数据，数据访问由 Mapper 类处理。

**特点**：

- **分离关注点**：模型与数据访问逻辑分离
- **解耦**：模型与数据库解耦
- **复杂**：实现相对复杂

**示例**：

```php
<?php
declare(strict_types=1);

// Doctrine（Data Mapper 模式）
class User
{
    private int $id;
    private string $name;
    private string $email;
    
    // 只有数据，没有操作方法
}

// 使用 EntityManager 操作
$user = $entityManager->find(User::class, 1);
$user->setName('John');
$entityManager->persist($user);
$entityManager->flush();
```

**代表框架**：

- **Doctrine**（Symfony）
- **Propel**

### 模式对比

| 特性 | Active Record | Data Mapper |
|:-----|:-------------|:------------|
| 复杂度 | 简单 | 复杂 |
| 耦合度 | 高 | 低 |
| 灵活性 | 较低 | 较高 |
| 学习曲线 | 平缓 | 陡峭 |
| 适用场景 | 快速开发 | 复杂应用 |

---

## ORM 框架对比

### Eloquent（Laravel）

Eloquent 是 Laravel 框架的 ORM，采用 Active Record 模式。

**特点**：

- **简单易用**：API 简洁，易于使用
- **功能丰富**：提供丰富的功能
- **文档完善**：文档详细，社区活跃
- **Laravel 集成**：与 Laravel 深度集成

**示例**：

```php
<?php
declare(strict_types=1);

// Eloquent
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'users';
    
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}

// 使用
$user = User::find(1);
$posts = $user->posts;
```

**适用场景**：

- Laravel 项目
- 快速开发
- 中小型应用

### Doctrine（Symfony）

Doctrine 是 Symfony 框架的 ORM，采用 Data Mapper 模式。

**特点**：

- **功能强大**：功能非常强大
- **灵活性高**：灵活性高，可定制性强
- **性能优化**：提供多种性能优化选项
- **企业级**：适合企业级应用

**示例**：

```php
<?php
declare(strict_types=1);

// Doctrine
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ORM\Table(name: 'users')]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;
    
    #[ORM\Column(type: 'string')]
    private string $name;
    
    #[ORM\OneToMany(targetEntity: Post::class, mappedBy: 'user')]
    private Collection $posts;
}

// 使用
$user = $entityManager->find(User::class, 1);
$posts = $user->getPosts();
```

**适用场景**：

- Symfony 项目
- 大型应用
- 企业级应用

### Propel

Propel 是一个独立的 ORM 框架，支持 Active Record 和 Data Mapper 两种模式。

**特点**：

- **独立框架**：不依赖特定框架
- **双模式**：支持两种模式
- **代码生成**：通过代码生成提高性能

**适用场景**：

- 独立项目
- 需要灵活性的项目

### 框架选择

选择 ORM 框架的考虑因素：

1. **项目框架**：如果使用 Laravel，选择 Eloquent；如果使用 Symfony，选择 Doctrine
2. **项目规模**：小型项目选择 Active Record，大型项目选择 Data Mapper
3. **团队经验**：选择团队熟悉的框架
4. **性能要求**：根据性能要求选择

---

## 完整示例

### 使用 Eloquent 的完整示例

```php
<?php
declare(strict_types=1);

// 模型定义
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
    
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}

class Post extends Model
{
    protected $table = 'posts';
    
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
}

// 使用示例
// 创建
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password')
]);

// 查询
$user = User::find(1);
$user = User::where('email', 'john@example.com')->first();
$users = User::where('age', '>', 18)->get();

// 更新
$user->name = 'Jane Doe';
$user->save();

// 删除
$user->delete();

// 关系查询
$posts = $user->posts;
$profile = $user->profile;

// 预加载
$users = User::with('posts', 'profile')->get();
```

---

## 使用场景

### 快速开发

ORM 适合快速开发场景：

- **原型开发**：快速构建原型
- **中小型应用**：中小型应用的开发
- **标准 CRUD**：标准的增删改查操作

### 关系处理

ORM 适合需要处理复杂关系的场景：

- **多表关联**：需要处理多个表的关联
- **关系查询**：需要频繁查询关联数据
- **关系维护**：需要维护表之间的关系

### 代码维护

ORM 适合需要长期维护的项目：

- **代码可读性**：代码更易读、易维护
- **团队协作**：团队成员更容易理解代码
- **重构方便**：重构更方便

---

## 注意事项

### 性能考虑

使用 ORM 时需要注意性能：

- **N+1 问题**：避免 N+1 查询问题
- **预加载**：使用预加载减少查询次数
- **查询优化**：优化查询，避免不必要的查询
- **原生 SQL**：复杂查询使用原生 SQL

### 复杂查询

对于复杂查询，可能需要使用原生 SQL：

- **子查询**：复杂的子查询
- **窗口函数**：窗口函数
- **存储过程**：存储过程调用

### 学习成本

ORM 需要学习成本：

- **框架学习**：需要学习 ORM 框架
- **最佳实践**：需要掌握最佳实践
- **调试技巧**：需要掌握调试技巧

### 框架选择

选择合适的 ORM 框架：

- **项目框架**：根据项目框架选择
- **项目规模**：根据项目规模选择
- **团队经验**：根据团队经验选择

---

## 常见问题

### 什么是 ORM？

ORM（Object-Relational Mapping）是对象关系映射，用于在面向对象编程语言和关系型数据库之间建立映射关系。

### ORM 的优势和劣势？

**优势**：

- 提高开发效率
- 代码可读性好
- 数据库抽象
- 关系处理简单

**劣势**：

- 性能开销
- 学习曲线
- 灵活性限制
- 复杂查询困难

### 如何选择 ORM 框架？

选择原则：

1. 根据项目框架选择
2. 根据项目规模选择
3. 根据团队经验选择
4. 根据性能要求选择

### ORM 的性能影响？

ORM 的性能影响：

- **查询生成**：有额外开销
- **结果映射**：有额外开销
- **N+1 问题**：可能产生性能问题

**优化方法**：

- 使用预加载
- 优化查询
- 使用原生 SQL
- 使用查询缓存

---

## 最佳实践

### 理解 ORM 的适用场景

- **适合**：快速开发、关系处理、标准 CRUD
- **不适合**：复杂查询、高性能要求、数据库特定特性

### 选择合适的 ORM 框架

- Laravel 项目：Eloquent
- Symfony 项目：Doctrine
- 独立项目：根据需求选择

### 注意性能优化

- 避免 N+1 问题
- 使用预加载
- 优化查询
- 使用原生 SQL

### 结合原生 SQL 使用

- 复杂查询使用原生 SQL
- 性能关键路径使用原生 SQL
- ORM 和原生 SQL 结合使用

---

## 练习任务

1. **ORM 基础**
   - 创建一个简单的 ORM 模型
   - 实现基本的 CRUD 操作
   - 测试 ORM 功能
   - 对比 ORM 和原生 SQL

2. **关系处理**
   - 定义模型关系
   - 实现关系查询
   - 测试关系功能
   - 优化关系查询

3. **性能优化**
   - 识别 N+1 问题
   - 使用预加载优化
   - 测试性能差异
   - 编写优化报告

4. **框架对比**
   - 对比 Eloquent 和 Doctrine
   - 测试不同框架的性能
   - 分析适用场景
   - 编写对比报告

5. **综合应用**
   - 创建一个完整的应用
   - 使用 ORM 实现功能
   - 优化性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.4.2 Eloquent 使用](section-02-eloquent.md)
- [6.4.3 数据迁移](section-03-migrations.md)
- [6.4.4 关系映射](section-04-relationships.md)
- [6.2 PDO 入门与高安全模式](../chapter-02-pdo/readme.md)

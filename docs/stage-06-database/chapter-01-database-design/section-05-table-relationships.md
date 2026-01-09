# 6.1.5 表关系设计 (Table Relationship Design)

## 概述

在关系型数据库中，数据被分散在不同的表中，以实现规范化并减少冗余。**关系 (Relationship)** 描述了这些表之间是如何相互关联的。正是这些预先定义好的关系，使得我们能够通过连接（JOIN）操作，将来自多个表的数据组合成有意义的信息。可以说，关系是“关系型数据库”名称的由来和其强大功能的核心。

建立表关系主要依赖于**主键 (Primary Key)** 和**外键 (Foreign Key)**。
-   **主-外键约束**是实现关系和保证**参照完整性**的主要机制。
-   外键在一个表中的值，对应着另一个表中的主键值。

表之间主要存在三种类型的关系：一对一 (One-to-One)、一对多 (One-to-Many) 和多对多 (Many-to-Many)。正确地识别和设计这些关系是数据库设计的关键步骤。

---

## 一对一关系 (One-to-One)

### 1. 定义

一对一关系指的是，表 A 中的一条记录最多只能与表 B 中的一条记录相关联，反之亦然。这是一种相对特殊但非常有用的关系。

### 2. 实现方式

主要有两种实现方式：

1.  **共享主键**：让两个表使用相同的主键值。表 B 的主键既是主键，也是指向表 A 的外键。
2.  **唯一外键**：在表 B 中创建一个外键字段指向表 A 的主键，并对这个外键字段添加**唯一约束 (Unique Constraint)**。

### 3. 应用场景

-   **分割大表（优化性能）**：当一个表包含非常多的列，其中一些列（如个人简介、大段文本）不常被查询时，可以将其拆分到另一个表中。这样，主表的“宽度”变小，查询常用信息时性能更高。
-   **隔离敏感数据（增强安全性）**：将普通数据和敏感数据（如财务信息、密码凭证）分表存储，并为敏感数据表设置更严格的访问权限。
-   **扩展第三方表**：当你使用一个无法修改的第三方或核心表时，可以通过一对一关系创建一个附加信息表来“扩展”它。

### 4. 示例

假设我们有一个 `users` 表，为了优化和安全，我们想将用户的详细个人资料 `profiles` 分离出去。

`users` 表 (主表):
```sql
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash CHAR(60) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

`user_profiles` 表 (从表，使用共享主键):
```sql
CREATE TABLE user_profiles (
    user_id INT UNSIGNED PRIMARY KEY,
    full_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(255),
    -- 将 user_id 同时作为主键和外键
    CONSTRAINT fk_user_profiles_users FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE -- 如果用户被删除，其个人资料也应一并删除
) ENGINE=InnoDB;
```
在这个例子中，`user_profiles.user_id` 必须对应一个 `users.id`，并且由于它也是主键，所以必须是唯一的。这就强制实现了一对一的关系。

---

## 一对多关系 (One-to-Many)

### 1. 定义

一对多关系是数据库设计中**最常见**的关系类型。它指的是，表 A 中的一条记录可以与表 B 中的多条记录相关联，但表 B 中的一条记录只能与表 A 中的一条记录相关联。

### 2. 实现方式

在“多”的那一方的表中，创建一个外键字段，指向“一”的那一方的表的主键。

### 3. 示例

一个典型的例子是“作者”与“文章”的关系：一个作者可以写多篇文章，但一篇文章只能有一个作者。

`authors` 表 ("一"方):
```sql
CREATE TABLE authors (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    bio TEXT
) ENGINE=InnoDB;
```

`posts` 表 ("多"方):
```sql
CREATE TABLE posts (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    author_id INT UNSIGNED NOT NULL, -- 这是外键
    title VARCHAR(255) NOT NULL,
    content MEDIUMTEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_posts_authors FOREIGN KEY (author_id)
        REFERENCES authors(id)
        ON DELETE CASCADE -- 如果作者被删除，其所有文章也一并删除
) ENGINE=InnoDB;
```
`posts` 表中的 `author_id` 字段作为外键，指向了 `authors` 表的 `id`。任何一篇文章都必须归属于一个已存在的作者。

---

## 多对多关系 (Many-to-Many)

### 1. 定义

多对多关系指的是，表 A 中的一条记录可以与表 B 中的多条记录相关联，反之，表 B 中的一条记录也可以与表 A 中的多条记录相关联。

### 2. 实现方式

多对多关系不能通过两个表直接实现，必须引入第三个表，这个表被称为**连接表 (Junction Table)**，也叫作桥接表 (Bridge Table) 或关联表 (Associative Table)。

连接表的结构通常包含：
1.  一个自己的主键（可选但推荐）。
2.  两个外键，分别指向另外两个表的主键。
3.  这两个外键的组合通常需要设置一个**唯一约束**，以防止重复的关联关系。
4.  连接表本身也可以包含额外的属性，例如关系创建的时间戳。

### 3. 示例

一个经典的例子是“文章”与“标签”的关系：一篇文章可以有多个标签，一个标签也可以被用于多篇文章。

`posts` 表:
```sql
CREATE TABLE posts (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content MEDIUMTEXT
) ENGINE=InnoDB;
```

`tags` 表:
```sql
CREATE TABLE tags (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
) ENGINE=InnoDB;
```

`post_tag` 表 (连接表):
```sql
CREATE TABLE post_tag (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    post_id INT UNSIGNED NOT NULL,
    tag_id INT UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- 创建外键约束
    CONSTRAINT fk_post_tag_posts FOREIGN KEY (post_id)
        REFERENCES posts(id) ON DELETE CASCADE,
    CONSTRAINT fk_post_tag_tags FOREIGN KEY (tag_id)
        REFERENCES tags(id) ON DELETE CASCADE,

    -- 创建唯一约束，确保同一篇文章不会被重复打上同一个标签
    UNIQUE KEY uk_post_tag (post_id, tag_id)
) ENGINE=InnoDB;
```
通过 `post_tag` 这个连接表，我们成功地建立了 `posts` 和 `tags` 之间的多对多关系。要查询某篇文章的所有标签，或者查询某个标签下的所有文章，都需要连接这三个表。

---

## 总结

| 关系类型     | 定义                               | 实现方式                                                     | 示例             |
|:-------------|:-----------------------------------|:-------------------------------------------------------------|:-----------------|
| **一对一**   | 一条记录对一条记录                 | 共享主键 或 唯一外键                                         | 用户 - 用户资料  |
| **一对多**   | 一条记录对多条记录                 | 在“多”方创建外键，指向“一”方的主键                           | 作者 - 文章      |
| **多对多**   | 多条记录对多条记录                 | 使用第三个“连接表”，包含两个分别指向双方主键的外键           | 文章 - 标签      |

---

## 练习任务

1.  **识别关系类型**：
    请判断以下实体之间的关系类型（一对一、一对多、多对多）：
    -   国家 (Country) 与 城市 (City)
    -   学生 (Student) 与 课程 (Course)
    -   用户 (User) 与 收货地址 (ShippingAddress)
    -   电影 (Movie) 与 演员 (Actor)
    -   公司 (Company) 与 CEO

2.  **设计学校数据库**：
    为一个学校数据库设计表结构，至少包含 `students` (学生), `courses` (课程), 和 `enrollments` (选课记录) 三个表。请思考它们之间的关系，并写出 `CREATE TABLE` 语句，确保外键和约束都已正确设置。

3.  **电商平台关系设计**：
    在一个电商平台中，存在“订单 (Orders)”和“商品 (Products)”两个实体。一个订单里可以包含多种商品，而同一种商品也会出现在不同的订单里。此外，每个订单详情（如购买数量、单价）可能不同。
    请问“订单”和“商品”之间是什么关系？你需要设计几个表来实现这个业务逻辑？请简要描述每个表的结构和作用。

# 9.1.4 数据库设计

## 一、数据库设计原则

### 1.1 设计原则

1. **规范化设计**：遵循数据库范式，减少数据冗余
2. **性能优化**：合理设计索引，优化查询性能
3. **可扩展性**：支持未来扩展和分表分库
4. **数据完整性**：使用外键约束和检查约束
5. **安全性**：敏感数据加密存储

### 1.2 命名规范

- **表名**：复数形式，小写，下划线分隔（`users`、`articles`）
- **字段名**：小写，下划线分隔（`user_id`、`created_at`）
- **索引名**：`idx_表名_字段名`（`idx_articles_author_id`）
- **外键名**：`fk_表名_字段名`（`fk_articles_author_id`）

## 二、核心表设计

### 2.1 用户表（users）

**表名**：`users`

**用途**：存储用户基本信息

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 用户 ID（主键） | PRIMARY |
| `username` | VARCHAR | 50 | NO | - | 用户名（唯一） | UNIQUE |
| `email` | VARCHAR | 255 | NO | - | 邮箱（唯一） | UNIQUE |
| `email_verified_at` | TIMESTAMP | - | YES | NULL | 邮箱验证时间 | - |
| `password` | VARCHAR | 255 | NO | - | 密码哈希 | - |
| `nickname` | VARCHAR | 50 | YES | NULL | 昵称 | - |
| `avatar` | VARCHAR | 500 | YES | NULL | 头像 URL | - |
| `bio` | TEXT | - | YES | NULL | 个人简介 | - |
| `website` | VARCHAR | 255 | YES | NULL | 个人网站 | - |
| `status` | TINYINT | - | NO | 1 | 状态（1=正常，2=禁用，3=删除） | INDEX |
| `role` | VARCHAR | 20 | NO | 'user' | 角色（user/vip/admin） | INDEX |
| `last_login_at` | TIMESTAMP | - | YES | NULL | 最后登录时间 | - |
| `last_login_ip` | VARCHAR | 45 | YES | NULL | 最后登录 IP | - |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | INDEX |
| `updated_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 更新时间 | - |
| `deleted_at` | TIMESTAMP | - | YES | NULL | 删除时间（软删除） | INDEX |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_users_username` (`username`)
- UNIQUE KEY `uk_users_email` (`email`)
- INDEX `idx_users_status` (`status`)
- INDEX `idx_users_role` (`role`)
- INDEX `idx_users_created_at` (`created_at`)
- INDEX `idx_users_deleted_at` (`deleted_at`)

**建表 SQL**：
```sql
CREATE TABLE `users` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(50) NOT NULL COMMENT '用户名',
    `email` VARCHAR(255) NOT NULL COMMENT '邮箱',
    `email_verified_at` TIMESTAMP NULL DEFAULT NULL COMMENT '邮箱验证时间',
    `password` VARCHAR(255) NOT NULL COMMENT '密码哈希',
    `nickname` VARCHAR(50) NULL DEFAULT NULL COMMENT '昵称',
    `avatar` VARCHAR(500) NULL DEFAULT NULL COMMENT '头像URL',
    `bio` TEXT NULL DEFAULT NULL COMMENT '个人简介',
    `website` VARCHAR(255) NULL DEFAULT NULL COMMENT '个人网站',
    `status` TINYINT NOT NULL DEFAULT 1 COMMENT '状态：1=正常，2=禁用，3=删除',
    `role` VARCHAR(20) NOT NULL DEFAULT 'user' COMMENT '角色：user/vip/admin',
    `last_login_at` TIMESTAMP NULL DEFAULT NULL COMMENT '最后登录时间',
    `last_login_ip` VARCHAR(45) NULL DEFAULT NULL COMMENT '最后登录IP',
    `created_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted_at` TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_users_username` (`username`),
    UNIQUE KEY `uk_users_email` (`email`),
    INDEX `idx_users_status` (`status`),
    INDEX `idx_users_role` (`role`),
    INDEX `idx_users_created_at` (`created_at`),
    INDEX `idx_users_deleted_at` (`deleted_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';
```

### 2.2 文章表（articles）

**表名**：`articles`

**用途**：存储文章信息

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 文章 ID（主键） | PRIMARY |
| `author_id` | BIGINT UNSIGNED | - | NO | - | 作者 ID（外键） | INDEX |
| `title` | VARCHAR | 200 | NO | - | 文章标题 | FULLTEXT |
| `slug` | VARCHAR | 255 | NO | - | URL 友好标识（唯一） | UNIQUE |
| `summary` | VARCHAR | 500 | YES | NULL | 文章摘要 | - |
| `content` | LONGTEXT | - | NO | - | 文章正文（Markdown） | FULLTEXT |
| `cover_image` | VARCHAR | 500 | YES | NULL | 封面图片 URL | - |
| `category_id` | BIGINT UNSIGNED | - | YES | NULL | 分类 ID（外键） | INDEX |
| `status` | VARCHAR | 20 | NO | 'draft' | 状态（draft/published/deleted） | INDEX |
| `visibility` | VARCHAR | 20 | NO | 'public' | 可见性（public/private/followers） | INDEX |
| `view_count` | INT UNSIGNED | - | NO | 0 | 浏览次数 | - |
| `like_count` | INT UNSIGNED | - | NO | 0 | 点赞数 | INDEX |
| `comment_count` | INT UNSIGNED | - | NO | 0 | 评论数 | - |
| `published_at` | TIMESTAMP | - | YES | NULL | 发布时间 | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | INDEX |
| `updated_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 更新时间 | - |
| `deleted_at` | TIMESTAMP | - | YES | NULL | 删除时间（软删除） | INDEX |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_articles_slug` (`slug`)
- INDEX `idx_articles_author_id` (`author_id`)
- INDEX `idx_articles_category_id` (`category_id`)
- INDEX `idx_articles_status` (`status`)
- INDEX `idx_articles_visibility` (`visibility`)
- INDEX `idx_articles_published_at` (`published_at`)
- INDEX `idx_articles_created_at` (`created_at`)
- INDEX `idx_articles_like_count` (`like_count`)
- FULLTEXT INDEX `ft_articles_title_content` (`title`, `content`)
- FOREIGN KEY `fk_articles_author_id` (`author_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_articles_category_id` (`category_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL

**建表 SQL**：
```sql
CREATE TABLE `articles` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `author_id` BIGINT UNSIGNED NOT NULL COMMENT '作者ID',
    `title` VARCHAR(200) NOT NULL COMMENT '文章标题',
    `slug` VARCHAR(255) NOT NULL COMMENT 'URL友好标识',
    `summary` VARCHAR(500) NULL DEFAULT NULL COMMENT '文章摘要',
    `content` LONGTEXT NOT NULL COMMENT '文章正文',
    `cover_image` VARCHAR(500) NULL DEFAULT NULL COMMENT '封面图片URL',
    `category_id` BIGINT UNSIGNED NULL DEFAULT NULL COMMENT '分类ID',
    `status` VARCHAR(20) NOT NULL DEFAULT 'draft' COMMENT '状态：draft/published/deleted',
    `visibility` VARCHAR(20) NOT NULL DEFAULT 'public' COMMENT '可见性：public/private/followers',
    `view_count` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT '浏览次数',
    `like_count` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT '点赞数',
    `comment_count` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT '评论数',
    `published_at` TIMESTAMP NULL DEFAULT NULL COMMENT '发布时间',
    `created_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted_at` TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_articles_slug` (`slug`),
    INDEX `idx_articles_author_id` (`author_id`),
    INDEX `idx_articles_category_id` (`category_id`),
    INDEX `idx_articles_status` (`status`),
    INDEX `idx_articles_visibility` (`visibility`),
    INDEX `idx_articles_published_at` (`published_at`),
    INDEX `idx_articles_created_at` (`created_at`),
    INDEX `idx_articles_like_count` (`like_count`),
    FULLTEXT INDEX `ft_articles_title_content` (`title`, `content`),
    CONSTRAINT `fk_articles_author_id` FOREIGN KEY (`author_id`) REFERENCES `users` (`id`) ON DELETE CASCADE,
    CONSTRAINT `fk_articles_category_id` FOREIGN KEY (`category_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='文章表';
```

### 2.3 评论表（comments）

**表名**：`comments`

**用途**：存储评论信息

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 评论 ID（主键） | PRIMARY |
| `article_id` | BIGINT UNSIGNED | - | NO | - | 文章 ID（外键） | INDEX |
| `author_id` | BIGINT UNSIGNED | - | NO | - | 评论作者 ID（外键） | INDEX |
| `parent_id` | BIGINT UNSIGNED | - | YES | NULL | 父评论 ID（支持嵌套） | INDEX |
| `content` | TEXT | - | NO | - | 评论内容 | - |
| `status` | VARCHAR | 20 | NO | 'pending' | 状态（pending/approved/deleted） | INDEX |
| `like_count` | INT UNSIGNED | - | NO | 0 | 点赞数 | - |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | INDEX |
| `updated_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 更新时间 | - |
| `deleted_at` | TIMESTAMP | - | YES | NULL | 删除时间（软删除） | INDEX |

**索引设计**：
- PRIMARY KEY (`id`)
- INDEX `idx_comments_article_id` (`article_id`)
- INDEX `idx_comments_author_id` (`author_id`)
- INDEX `idx_comments_parent_id` (`parent_id`)
- INDEX `idx_comments_status` (`status`)
- INDEX `idx_comments_created_at` (`created_at`)
- FOREIGN KEY `fk_comments_article_id` (`article_id`) REFERENCES `articles` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_comments_author_id` (`author_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_comments_parent_id` (`parent_id`) REFERENCES `comments` (`id`) ON DELETE CASCADE

**建表 SQL**：
```sql
CREATE TABLE `comments` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `article_id` BIGINT UNSIGNED NOT NULL COMMENT '文章ID',
    `author_id` BIGINT UNSIGNED NOT NULL COMMENT '评论作者ID',
    `parent_id` BIGINT UNSIGNED NULL DEFAULT NULL COMMENT '父评论ID',
    `content` TEXT NOT NULL COMMENT '评论内容',
    `status` VARCHAR(20) NOT NULL DEFAULT 'pending' COMMENT '状态：pending/approved/deleted',
    `like_count` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT '点赞数',
    `created_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted_at` TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (`id`),
    INDEX `idx_comments_article_id` (`article_id`),
    INDEX `idx_comments_author_id` (`author_id`),
    INDEX `idx_comments_parent_id` (`parent_id`),
    INDEX `idx_comments_status` (`status`),
    INDEX `idx_comments_created_at` (`created_at`),
    CONSTRAINT `fk_comments_article_id` FOREIGN KEY (`article_id`) REFERENCES `articles` (`id`) ON DELETE CASCADE,
    CONSTRAINT `fk_comments_author_id` FOREIGN KEY (`author_id`) REFERENCES `users` (`id`) ON DELETE CASCADE,
    CONSTRAINT `fk_comments_parent_id` FOREIGN KEY (`parent_id`) REFERENCES `comments` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='评论表';
```

### 2.4 分类表（categories）

**表名**：`categories`

**用途**：存储文章分类信息

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 分类 ID（主键） | PRIMARY |
| `name` | VARCHAR | 50 | NO | - | 分类名称（唯一） | UNIQUE |
| `slug` | VARCHAR | 100 | NO | - | URL 友好标识（唯一） | UNIQUE |
| `description` | VARCHAR | 500 | YES | NULL | 分类描述 | - |
| `icon` | VARCHAR | 255 | YES | NULL | 分类图标 URL | - |
| `parent_id` | BIGINT UNSIGNED | - | YES | NULL | 父分类 ID（支持二级分类） | INDEX |
| `sort_order` | INT | - | NO | 0 | 排序顺序 | INDEX |
| `article_count` | INT UNSIGNED | - | NO | 0 | 文章数量 | - |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | - |
| `updated_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 更新时间 | - |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_categories_name` (`name`)
- UNIQUE KEY `uk_categories_slug` (`slug`)
- INDEX `idx_categories_parent_id` (`parent_id`)
- INDEX `idx_categories_sort_order` (`sort_order`)
- FOREIGN KEY `fk_categories_parent_id` (`parent_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL

### 2.5 标签表（tags）

**表名**：`tags`

**用途**：存储标签信息

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 标签 ID（主键） | PRIMARY |
| `name` | VARCHAR | 50 | NO | - | 标签名称（唯一） | UNIQUE |
| `slug` | VARCHAR | 100 | NO | - | URL 友好标识（唯一） | UNIQUE |
| `description` | VARCHAR | 200 | YES | NULL | 标签描述 | - |
| `color` | VARCHAR | 20 | YES | NULL | 标签颜色 | - |
| `usage_count` | INT UNSIGNED | - | NO | 0 | 使用次数 | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | - |
| `updated_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 更新时间 | - |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_tags_name` (`name`)
- UNIQUE KEY `uk_tags_slug` (`slug`)
- INDEX `idx_tags_usage_count` (`usage_count`)

## 三、关联表设计

### 3.1 文章标签关联表（article_tag）

**表名**：`article_tag`

**用途**：文章和标签的多对多关系

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 主键 | PRIMARY |
| `article_id` | BIGINT UNSIGNED | - | NO | - | 文章 ID | INDEX |
| `tag_id` | BIGINT UNSIGNED | - | NO | - | 标签 ID | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | - |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_article_tag` (`article_id`, `tag_id`)
- INDEX `idx_article_tag_article_id` (`article_id`)
- INDEX `idx_article_tag_tag_id` (`tag_id`)
- FOREIGN KEY `fk_article_tag_article_id` (`article_id`) REFERENCES `articles` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_article_tag_tag_id` (`tag_id`) REFERENCES `tags` (`id`) ON DELETE CASCADE

### 3.2 用户关注表（user_follows）

**表名**：`user_follows`

**用途**：用户关注关系

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 主键 | PRIMARY |
| `follower_id` | BIGINT UNSIGNED | - | NO | - | 关注者 ID | INDEX |
| `following_id` | BIGINT UNSIGNED | - | NO | - | 被关注者 ID | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | - |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_user_follows` (`follower_id`, `following_id`)
- INDEX `idx_user_follows_follower_id` (`follower_id`)
- INDEX `idx_user_follows_following_id` (`following_id`)
- FOREIGN KEY `fk_user_follows_follower_id` (`follower_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_user_follows_following_id` (`following_id`) REFERENCES `users` (`id`) ON DELETE CASCADE

### 3.3 文章点赞表（article_likes）

**表名**：`article_likes`

**用途**：文章点赞记录

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 主键 | PRIMARY |
| `article_id` | BIGINT UNSIGNED | - | NO | - | 文章 ID | INDEX |
| `user_id` | BIGINT UNSIGNED | - | NO | - | 用户 ID | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | - |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_article_likes` (`article_id`, `user_id`)
- INDEX `idx_article_likes_article_id` (`article_id`)
- INDEX `idx_article_likes_user_id` (`user_id`)
- FOREIGN KEY `fk_article_likes_article_id` (`article_id`) REFERENCES `articles` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_article_likes_user_id` (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE

### 3.4 文章收藏表（article_favorites）

**表名**：`article_favorites`

**用途**：文章收藏记录

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 主键 | PRIMARY |
| `article_id` | BIGINT UNSIGNED | - | NO | - | 文章 ID | INDEX |
| `user_id` | BIGINT UNSIGNED | - | NO | - | 用户 ID | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | - |

**索引设计**：
- PRIMARY KEY (`id`)
- UNIQUE KEY `uk_article_favorites` (`article_id`, `user_id`)
- INDEX `idx_article_favorites_article_id` (`article_id`)
- INDEX `idx_article_favorites_user_id` (`user_id`)
- FOREIGN KEY `fk_article_favorites_article_id` (`article_id`) REFERENCES `articles` (`id`) ON DELETE CASCADE
- FOREIGN KEY `fk_article_favorites_user_id` (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE

### 3.5 通知表（notifications）

**表名**：`notifications`

**用途**：站内通知

**字段设计**：

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 | 索引 |
|--------|------|------|--------|--------|------|------|
| `id` | BIGINT UNSIGNED | - | NO | AUTO_INCREMENT | 通知 ID（主键） | PRIMARY |
| `user_id` | BIGINT UNSIGNED | - | NO | - | 接收用户 ID | INDEX |
| `type` | VARCHAR | 50 | NO | - | 通知类型 | INDEX |
| `title` | VARCHAR | 200 | NO | - | 通知标题 | - |
| `content` | TEXT | - | YES | NULL | 通知内容 | - |
| `data` | JSON | - | YES | NULL | 附加数据 | - |
| `read_at` | TIMESTAMP | - | YES | NULL | 阅读时间 | INDEX |
| `created_at` | TIMESTAMP | - | YES | CURRENT_TIMESTAMP | 创建时间 | INDEX |

**索引设计**：
- PRIMARY KEY (`id`)
- INDEX `idx_notifications_user_id` (`user_id`)
- INDEX `idx_notifications_type` (`type`)
- INDEX `idx_notifications_read_at` (`read_at`)
- INDEX `idx_notifications_created_at` (`created_at`)
- FOREIGN KEY `fk_notifications_user_id` (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE

## 四、数据库优化策略

### 4.1 索引优化

1. **主键索引**：所有表都有自增主键
2. **唯一索引**：用户名、邮箱、slug 等唯一字段
3. **普通索引**：外键字段、查询字段、排序字段
4. **全文索引**：文章标题和内容（支持全文搜索）
5. **复合索引**：多字段查询（如 `article_id` + `user_id`）

### 4.2 查询优化

1. **避免 N+1 查询**：使用 Eager Loading
2. **分页查询**：使用 LIMIT 和 OFFSET
3. **统计查询**：使用计数器字段（`like_count`、`comment_count`）
4. **软删除**：使用 `deleted_at` 字段，保留数据

### 4.3 数据分区（未来）

**分区策略**：
- 按时间分区（articles 表按 `created_at` 分区）
- 按用户分区（user_follows 表按 `follower_id` 分区）

### 4.4 读写分离（未来）

**主从复制**：
- 主库：写操作
- 从库：读操作（查询、统计）

## 五、数据迁移

### 5.1 Laravel Migration

**创建迁移文件**：
```bash
php artisan make:migration create_users_table
php artisan make:migration create_articles_table
php artisan make:migration create_comments_table
# ... 其他表
```

**迁移文件示例**（users 表）：
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('username', 50)->unique();
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->string('nickname', 50)->nullable();
            $table->string('avatar', 500)->nullable();
            $table->text('bio')->nullable();
            $table->string('website')->nullable();
            $table->tinyInteger('status')->default(1)->index();
            $table->string('role', 20)->default('user')->index();
            $table->timestamp('last_login_at')->nullable();
            $table->string('last_login_ip', 45)->nullable();
            $table->timestamps();
            $table->softDeletes();
            
            $table->index('created_at');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

## 六、数据库关系图

```
users (1) ────< (N) articles
users (1) ────< (N) comments
users (1) ────< (N) user_follows (N) ────> (1) users
articles (1) ────< (N) comments
articles (N) ────> (N) tags (article_tag)
articles (1) ────< (N) article_likes
articles (1) ────< (N) article_favorites
categories (1) ────< (N) articles
categories (1) ────< (N) categories (parent_id)
users (1) ────< (N) notifications
```

## 七、下一步

完成数据库设计后，继续学习：
- [API 接口设计](section-05-api-design.md)：设计 RESTful API 接口规范

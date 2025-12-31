# 6.4.3 数据迁移

## 概述

数据迁移（Migration）是数据库版本控制的重要工具，用于管理数据库结构的变更。通过迁移文件，可以记录数据库结构的每次变更，实现数据库结构的版本控制、团队协作和部署自动化。

本节详细介绍数据迁移的概念、使用方法、迁移文件的创建和执行、回滚操作、迁移最佳实践等，帮助零基础学员掌握数据迁移的使用方法。

**主要内容**：
- 数据迁移的概念和重要性
- 迁移文件的创建和结构
- 迁移的执行和回滚
- 表结构操作（创建、修改、删除）
- 数据迁移操作
- 完整示例和最佳实践

---

## 特性

- **版本控制**：数据库结构的版本控制
- **团队协作**：团队成员共享数据库结构变更
- **部署自动化**：自动化数据库结构部署
- **可回滚**：支持回滚操作，恢复数据库结构
- **环境一致性**：保证不同环境数据库结构一致

---

## 数据迁移概念

### 什么是数据迁移

数据迁移是数据库结构变更的版本控制系统，通过迁移文件记录每次数据库结构的变更。

**迁移的核心思想**：

- **版本控制**：每个迁移文件代表一个版本
- **可追溯**：可以追溯每次数据库结构变更
- **可回滚**：可以回滚到之前的版本
- **团队协作**：团队成员共享数据库结构变更

### 为什么需要数据迁移

**没有迁移的问题**：

- 数据库结构变更难以管理
- 团队成员数据库结构不一致
- 部署时数据库结构变更困难
- 无法回滚数据库结构变更

**迁移的优势**：

- **版本控制**：数据库结构的版本控制
- **团队协作**：团队成员共享数据库结构变更
- **部署自动化**：自动化数据库结构部署
- **可回滚**：支持回滚操作

### 迁移的工作原理

迁移的工作原理：

1. **创建迁移文件**：定义数据库结构变更
2. **执行迁移**：执行迁移文件，应用变更
3. **记录状态**：记录迁移执行状态
4. **回滚迁移**：回滚迁移，恢复之前的结构

---

## 迁移文件创建

### 创建迁移文件

使用 Artisan 命令创建迁移文件。

**语法**：`php artisan make:migration create_table_name_table`

**示例**：

```bash
# 创建表
php artisan make:migration create_users_table

# 修改表
php artisan make:migration add_email_to_users_table --table=users

# 删除表
php artisan make:migration drop_users_table
```

### 迁移文件结构

迁移文件包含 `up()` 和 `down()` 方法。

**示例**：

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        // 执行迁移时的操作
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamps();
        });
    }
    
    public function down(): void
    {
        // 回滚迁移时的操作
        Schema::dropIfExists('users');
    }
};
```

### 表结构操作

使用 Schema 构建器操作表结构。

**创建表**：

```php
<?php
declare(strict_types=1);

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamp('email_verified_at')->nullable();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});
```

**修改表**：

```php
<?php
declare(strict_types=1);

Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable()->after('email');
    $table->index('email');
});
```

**删除表**：

```php
<?php
declare(strict_types=1);

Schema::dropIfExists('users');
```

---

## 字段类型

### 常用字段类型

Schema 构建器提供多种字段类型。

**示例**：

```php
<?php
declare(strict_types=1);

Schema::create('users', function (Blueprint $table) {
    // 整数类型
    $table->id();  // BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
    $table->integer('age');  // INT
    $table->bigInteger('views');  // BIGINT
    $table->tinyInteger('status');  // TINYINT
    $table->unsignedInteger('count');  // UNSIGNED INT
    
    // 字符串类型
    $table->string('name', 100);  // VARCHAR(100)
    $table->text('bio');  // TEXT
    $table->longText('content');  // LONGTEXT
    $table->char('code', 10);  // CHAR(10)
    
    // 日期时间类型
    $table->date('birthday');  // DATE
    $table->dateTime('published_at');  // DATETIME
    $table->timestamp('created_at');  // TIMESTAMP
    $table->timestamps();  // created_at, updated_at
    $table->softDeletes();  // deleted_at
    
    // 其他类型
    $table->boolean('is_active');  // BOOLEAN
    $table->decimal('price', 10, 2);  // DECIMAL(10,2)
    $table->float('rating');  // FLOAT
    $table->json('metadata');  // JSON
    $table->uuid('uuid');  // UUID
});
```

### 字段修饰符

字段可以添加修饰符。

**示例**：

```php
<?php
declare(strict_types=1);

Schema::create('users', function (Blueprint $table) {
    // 可空
    $table->string('phone')->nullable();
    
    // 默认值
    $table->string('status')->default('active');
    
    // 唯一
    $table->string('email')->unique();
    
    // 索引
    $table->string('name')->index();
    
    // 注释
    $table->string('name')->comment('用户姓名');
    
    // 位置
    $table->string('phone')->after('email');
    $table->string('phone')->first();
});
```

---

## 索引和约束

### 索引操作

使用索引方法创建索引。

**示例**：

```php
<?php
declare(strict_types=1);

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('email');
    $table->string('name');
    
    // 单列索引
    $table->index('email');
    
    // 唯一索引
    $table->unique('email');
    
    // 复合索引
    $table->index(['name', 'email']);
    
    // 命名索引
    $table->index('email', 'idx_email');
});
```

### 外键约束

使用外键方法创建外键约束。

**示例**：

```php
<?php
declare(strict_types=1);

Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->string('title');
    $table->text('content');
    $table->timestamps();
});

// 或者手动指定
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')
        ->references('id')
        ->on('users')
        ->onDelete('cascade');
    $table->string('title');
    $table->timestamps();
});
```

---

## 迁移执行

### 执行迁移

使用 Artisan 命令执行迁移。

**语法**：`php artisan migrate`

**示例**：

```bash
# 执行所有未执行的迁移
php artisan migrate

# 执行迁移（强制，生产环境）
php artisan migrate --force

# 执行迁移（显示 SQL）
php artisan migrate --pretend
```

### 回滚迁移

使用 Artisan 命令回滚迁移。

**语法**：`php artisan migrate:rollback`

**示例**：

```bash
# 回滚最后一次迁移
php artisan migrate:rollback

# 回滚指定次数
php artisan migrate:rollback --step=3

# 回滚所有迁移
php artisan migrate:reset

# 回滚并重新执行
php artisan migrate:refresh

# 回滚所有并重新执行
php artisan migrate:fresh
```

### 迁移状态

查看迁移执行状态。

**语法**：`php artisan migrate:status`

**示例**：

```bash
# 查看迁移状态
php artisan migrate:status
```

---

## 数据迁移

### 填充数据

在迁移中填充初始数据。

**示例**：

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\DB;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('roles', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
        
        // 填充初始数据
        DB::table('roles')->insert([
            ['name' => 'admin', 'created_at' => now(), 'updated_at' => now()],
            ['name' => 'user', 'created_at' => now(), 'updated_at' => now()],
        ]);
    }
    
    public function down(): void
    {
        Schema::dropIfExists('roles');
    }
};
```

### 数据迁移

迁移现有数据。

**示例**：

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\DB;

return new class extends Migration
{
    public function up(): void
    {
        // 添加新字段
        Schema::table('users', function (Blueprint $table) {
            $table->string('full_name')->nullable()->after('name');
        });
        
        // 迁移数据
        DB::table('users')->chunkById(100, function ($users) {
            foreach ($users as $user) {
                DB::table('users')
                    ->where('id', $user->id)
                    ->update(['full_name' => $user->name]);
            }
        });
    }
    
    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('full_name');
        });
    }
};
```

---

## 完整示例

### 创建用户表迁移

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->string('phone')->nullable();
            $table->tinyInteger('status')->default(1);
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
            
            $table->index('email');
            $table->index('status');
        });
    }
    
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

### 修改用户表迁移

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('avatar')->nullable()->after('email');
            $table->date('birthday')->nullable()->after('phone');
            $table->index('birthday');
        });
    }
    
    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropIndex(['birthday']);
            $table->dropColumn(['avatar', 'birthday']);
        });
    }
};
```

---

## 使用场景

### 版本控制

使用迁移管理数据库结构版本。

### 团队协作

团队成员共享数据库结构变更。

### 部署自动化

自动化数据库结构部署。

---

## 注意事项

### 迁移顺序

迁移按文件名顺序执行，注意命名规范。

### 回滚操作

确保 `down()` 方法正确实现，支持回滚。

### 数据迁移

数据迁移要谨慎，确保数据安全。

### 生产环境

生产环境执行迁移要谨慎，建议先备份。

---

## 常见问题

### 如何创建迁移文件？

使用 Artisan 命令创建迁移文件。

```bash
php artisan make:migration create_users_table
```

### 如何执行迁移？

使用 Artisan 命令执行迁移。

```bash
php artisan migrate
```

### 如何回滚迁移？

使用 Artisan 命令回滚迁移。

```bash
php artisan migrate:rollback
```

### 迁移文件命名规范？

迁移文件命名规范：

- 使用描述性名称
- 使用下划线分隔
- 包含操作类型（create、add、modify、drop）

---

## 最佳实践

### 遵循命名规范

遵循迁移文件命名规范。

### 实现回滚操作

确保 `down()` 方法正确实现。

### 测试迁移

在开发环境测试迁移，确保正确。

### 备份数据

生产环境执行迁移前先备份数据。

---

## 练习任务

1. **创建迁移**
   - 创建用户表迁移
   - 创建文章表迁移
   - 创建评论表迁移
   - 测试迁移执行

2. **修改表结构**
   - 添加字段迁移
   - 修改字段迁移
   - 删除字段迁移
   - 测试回滚操作

3. **索引和约束**
   - 创建索引迁移
   - 创建外键迁移
   - 测试索引和约束
   - 优化表结构

4. **数据迁移**
   - 填充初始数据
   - 迁移现有数据
   - 测试数据迁移
   - 验证数据完整性

5. **综合应用**
   - 创建一个完整的数据库结构
   - 使用迁移管理结构变更
   - 测试迁移和回滚
   - 编写迁移文档

---

**相关章节**：

- [6.4.1 ORM 基础](section-01-orm-basics.md)
- [6.4.2 Eloquent 使用](section-02-eloquent.md)
- [6.4.4 关系映射](section-04-relationships.md)

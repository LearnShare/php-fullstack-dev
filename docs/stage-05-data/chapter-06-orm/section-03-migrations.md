# 5.6.3 数据迁移

## 概述

数据库迁移（Migration）是版本化数据库结构变更的机制。本节详细介绍迁移基础、创建迁移、运行迁移、回滚迁移，以及迁移最佳实践。

## 什么是迁移

- **迁移**：版本化数据库结构变更
- 记录数据库的每次变更
- 支持回滚和版本管理

## Eloquent 迁移

### 创建迁移

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUsersTable extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('username')->unique();
            $table->string('email')->unique();
            $table->string('password_hash');
            $table->timestamps();
            $table->index('email');
        });
    }
    
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
}
```

### 修改表结构

```php
<?php
declare(strict_types=1);

class AddPhoneToUsersTable extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('phone')->nullable()->after('email');
        });
    }
    
    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('phone');
        });
    }
}
```

## 迁移命令

```bash
# 运行迁移
php artisan migrate

# 回滚迁移
php artisan migrate:rollback

# 回滚所有迁移
php artisan migrate:reset

# 查看迁移状态
php artisan migrate:status

# 刷新数据库（回滚后重新运行）
php artisan migrate:refresh
```

## Doctrine 迁移

```php
<?php
declare(strict_types=1);

namespace DoctrineMigrations;

use Doctrine\DBAL\Schema\Schema;
use Doctrine\Migrations\AbstractMigration;

final class Version20250101000000 extends AbstractMigration
{
    public function up(Schema $schema): void
    {
        $this->addSql('
            CREATE TABLE users (
                id INT AUTO_INCREMENT PRIMARY KEY,
                username VARCHAR(255) NOT NULL,
                email VARCHAR(255) NOT NULL,
                password_hash VARCHAR(255) NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                UNIQUE KEY uk_username (username),
                UNIQUE KEY uk_email (email),
                INDEX idx_email (email)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
        ');
    }
    
    public function down(Schema $schema): void
    {
        $this->addSql('DROP TABLE users');
    }
}
```

## 迁移最佳实践

### 1. 原子性迁移

```php
<?php
declare(strict_types=1);

public function up(): void
{
    DB::transaction(function () {
        Schema::create('users', function (Blueprint $table) {
            // ...
        });
        
        // 数据迁移
        // ...
    });
}
```

### 2. 可回滚迁移

```php
<?php
declare(strict_types=1);

public function down(): void
{
    // 确保可以完全回滚
    Schema::dropIfExists('users');
}
```

### 3. 数据迁移

```php
<?php
declare(strict_types=1);

public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('full_name')->nullable();
    });
    
    // 数据迁移
    DB::table('users')->update([
        'full_name' => DB::raw("CONCAT(first_name, ' ', last_name)")
    ]);
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CreateOrdersTable extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->decimal('total', 10, 2);
            $table->enum('status', ['pending', 'paid', 'shipped', 'completed', 'cancelled'])->default('pending');
            $table->timestamps();
            
            $table->index('user_id');
            $table->index('status');
            $table->index('created_at');
        });
    }
    
    public function down(): void
    {
        Schema::dropIfExists('orders');
    }
}
```

## 注意事项

1. **原子性**：确保迁移是原子的
2. **可回滚**：每个迁移都要有对应的回滚方法
3. **数据迁移**：数据迁移要小心处理
4. **版本控制**：迁移文件要提交到版本控制

## 练习

1. 创建完整的数据库迁移，包含所有表结构。

2. 实现数据迁移，迁移现有数据到新结构。

3. 编写迁移回滚脚本，确保可以安全回滚。

4. 实现迁移测试，验证迁移的正确性。

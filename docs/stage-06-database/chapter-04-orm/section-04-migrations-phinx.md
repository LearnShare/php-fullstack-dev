# 6.4.4 数据迁移与 Phinx (Data Migrations & Phinx)

## 概述

随着项目的迭代，数据库的结构（Schema）几乎不可避免地会发生变化：需要添加新表、增加新字段、修改索引……我们如何在一个团队中，跨越多个环境（开发、测试、生产），来安全、可靠地管理这些变更呢？

这就是**数据迁移 (Database Migrations)** 发挥作用的地方。本节将介绍数据迁移的核心思想，并学习如何使用一个流行的独立迁移工具——**Phinx**。

## 为什么需要数据迁移？

想象一下没有迁移工具时的“旧世界”：
-   **手动变更**：开发者 A 在自己的本地数据库中用 `ALTER TABLE` 加了一个字段。他需要通过聊天工具告诉开发者 B、C、D 也执行同样的操作。
-   **共享 SQL 文件**：团队把所有的 `ALTER` 语句保存在一个 `.sql` 文件中。每次部署时，数据库管理员（DBA）或运维人员需要手动执行这个文件。

这种方式充满了问题：
-   **极易出错**：很容易忘记在某个环境上执行，或者执行了错误的顺序。
-   **无版本控制**：你无法知道数据库在任意时间点的确切结构，更无法轻松地回滚到上一个版本。
-   **协作困难**：新成员加入项目时，需要花费大量时间来搭建一个与团队其他人一致的数据库。
-   **部署噩梦**：部署新功能常常伴随着复杂的手动数据库变更，风险极高。

### 数据迁移的解决方案

数据迁移的核心思想是：**用代码来管理数据库结构**。

-   每一个数据库结构的变更，都对应一个**迁移文件**（通常是一个 PHP 类）。
-   这些迁移文件被纳入 **Git 等版本控制系统**中，和你的业务代码一起管理。
-   通过专门的命令行工具，可以自动地应用（`migrate`）或撤销（`rollback`）这些变更。

**带来的好处**：
-   **版本化**：数据库结构有了清晰的版本历史。
-   **自动化**：一个命令就能让任何环境的数据库更新到最新结构。
-   **协作性**：团队成员只需拉取最新代码，运行迁移命令，即可同步数据库。
-   **可靠性**：变更过程是可重复、可预测的，大大降低了部署风险。

## Phinx 简介

**Phinx** 是一个用 PHP 编写的、非常流行的数据库迁移管理工具。它不依赖于任何框架，可以轻松地集成到任何 PHP 项目中。

## 使用 Phinx

### 1. 安装

在你的项目根目录下，通过 Composer 安装 Phinx：
```bash
composer require robmorgan/phinx
```

### 2. 初始化

执行以下命令来初始化 Phinx，它会在你的项目根目录创建一个配置文件：
```bash
vendor/bin/phinx init
```
默认会创建 `phinx.php` 文件。你也可以指定格式，如 `phinx.yml`。

### 3. 配置

打开 `phinx.php`，这是你配置迁移路径和数据库连接的地方。

**`phinx.php` (示例)**
```php
<?php
return [
    'paths' => [
        // 迁移文件的存放目录
        'migrations' => '%%PHINX_CONFIG_DIR%%/db/migrations',
        'seeds' => '%%PHINX_CONFIG_DIR%%/db/seeds'
    ],
    'environments' => [
        'default_migration_table' => 'phinxlog',
        'default_environment' => 'development', // 默认使用的环境
        'production' => [
            'adapter' => 'mysql',
            'host' => 'prod_host',
            'name' => 'prod_db',
            'user' => 'prod_user',
            'pass' => getenv('DB_PASS'), // 推荐从环境变量获取密码
            'port' => 3306,
            'charset' => 'utf8mb4',
        ],
        'development' => [
            'adapter' => 'mysql',
            'host' => 'localhost',
            'name' => 'my_blog', // 你的本地数据库
            'user' => 'root',
            'pass' => 'your_password',
            'port' => 3306,
            'charset' => 'utf8mb4',
        ],
    ]
];
```
配置完成后，请确保 `db/migrations` 目录存在。

### 4. 创建第一个迁移

使用 `create` 命令来创建一个新的迁移文件。文件名使用驼峰式命名，Phinx 会自动生成一个带有时间戳前缀的 PHP 文件。

```bash
vendor/bin/phinx create CreateUsersTable
```
这会生成一个类似 `db/migrations/20260110123456_create_users_table.php` 的文件。

**迁移文件结构**：
每个迁移类都包含一个 `change()` 方法，或者 `up()` 和 `down()` 两个方法。`change()` 更方便，Phinx 通常能自动推断出如何回滚。`up()`/`down()` 则更明确。

```php
<?php
use Phinx\Migration\AbstractMigration;

class CreateUsersTable extends AbstractMigration
{
    /**
     * `up()` 方法在执行 `migrate` 时运行。
     */
    public function up(): void
    {
        // 创建 users 表
        $table = $this->table('users'); // Phinx 会自动添加 'id' 作为自增主键
        $table->addColumn('username', 'string', ['limit' => 50])
              ->addColumn('email', 'string', ['limit' => 100])
              ->addColumn('password_hash', 'string', ['limit' => 255])
              ->addColumn('created_at', 'timestamp', ['default' => 'CURRENT_TIMESTAMP'])
              ->addColumn('updated_at', 'timestamp', ['default' => 'CURRENT_TIMESTAMP', 'update' => 'CURRENT_TIMESTAMP'])
              ->addIndex(['username'], ['unique' => true]) // 添加唯一索引
              ->addIndex(['email'], ['unique' => true])
              ->create();
    }

    /**
     * `down()` 方法在执行 `rollback` 时运行。
     */
    public function down(): void
    {
        $this->table('users')->drop()->save();
    }
}
```

### 5. 运行迁移

执行以下命令来应用所有尚未执行的迁移：
```bash
vendor/bin/phinx migrate
```
Phinx 会连接到数据库，检查 `phinxlog` 表（如果不存在会自动创建），然后按顺序执行所有新的迁移文件中的 `up()` 方法，并记录下已执行的版本。

### 6. 回滚迁移

**回滚上一次的迁移**:
```bash
vendor/bin/phinx rollback
```
这会找到最近一次执行的迁移，并运行其 `down()` 方法。

**回滚到指定版本**:
```bash
# -t 表示 target
vendor/bin/phinx rollback -t 20260110123456
```

**回滚所有迁移**:
```bash
vendor/bin/phinx rollback -t 0
```

### 7. 查看状态

`status` 命令可以让你清晰地看到所有迁移文件的状态（已执行/未执行）。

```bash
vendor/bin/phinx status
```

## 最佳实践

1.  **保持原子性**：一个迁移文件应该只做一件相关的、逻辑上原子性的事（例如，创建一个表、给一个表加一个字段、修改一个字段）。
2.  **确保可回滚**：始终为你写的 `up()` 方法提供一个对应的、逻辑相反的 `down()` 方法。
3.  **不要修改已合并的迁移**：一旦一个迁移文件被合并到主分支或在生产环境上运行过，**绝对不要**再回去修改它。如果你需要变更，请创建一个**新的**迁移文件。
4.  **与团队同步**：在开始开发新功能前，先拉取最新代码并运行 `phinx migrate`，以确保你的本地数据库结构与团队保持一致。

---

## 练习任务

1.  **初始化 Phinx**：
    在你之前的项目中，安装并初始化 Phinx，然后正确地配置 `phinx.php` 文件，使其连接到你的本地数据库。

2.  **创建 Posts 表**：
    使用 `phinx create CreatePostsTable` 命令创建一个迁移文件。在其中编写 `up()` 和 `down()` 方法来创建和删除 `posts` 表。`posts` 表应至少包含 `id`, `user_id` (外键), `title`, `content`, `created_at`, `updated_at` 字段。记得为 `user_id` 添加外键约束。

3.  **执行与回滚**：
    -   运行 `phinx migrate` 来创建 `posts` 表，并使用 `phinx status` 检查状态。
    -   连接到你的数据库，确认 `posts` 表和 `phinxlog` 表都已成功创建。
    -   运行 `phinx rollback`，并确认 `posts` 表已被成功删除。

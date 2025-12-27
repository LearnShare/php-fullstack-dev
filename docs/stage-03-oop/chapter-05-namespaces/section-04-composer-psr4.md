# 3.5.4 Composer 自动加载与 PSR-4

## 概述

PSR-4 是 PHP 标准的自动加载规范，定义了命名空间与文件系统目录的映射关系。Composer 实现了 PSR-4 标准，是现代 PHP 项目的标准自动加载方式。理解 PSR-4 和 Composer 的自动加载机制对于组织和维护大型项目至关重要。

PSR-4 标准解决了之前自动加载规范（如 PSR-0）的问题，提供了更简洁和高效的映射规则。Composer 作为 PHP 的依赖管理工具，内置了对 PSR-4 的支持，只需在 `composer.json` 中配置，就可以自动生成高效的自动加载器。

掌握 Composer 自动加载和 PSR-4 标准是现代 PHP 开发的基础。几乎所有现代 PHP 框架和库都使用这种方式组织代码，理解它有助于理解框架的工作原理和最佳实践。

**主要内容**：
- PSR-4 自动加载规则和原理
- Composer 自动加载配置（composer.json）
- 目录结构与命名空间的对应关系
- 自动加载优化（classmap、files）
- 调试自动加载问题
- 生产环境优化
- 命名空间最佳实践

## 特性

- **标准规范**：遵循 PSR-4 标准，业界通用
- **自动生成**：Composer 自动生成高效的加载器
- **性能优化**：支持类映射（classmap）等多种优化方式
- **简单配置**：只需在 composer.json 中配置
- **灵活映射**：支持多个命名空间前缀映射

## 语法/定义

### PSR-4 规则

**基本规则**：
1. **命名空间前缀**：完全限定类名的顶级命名空间（如 `App\`）
2. **基础目录**：命名空间前缀对应的文件系统目录（如 `src/`）
3. **子命名空间**：对应子目录（如 `App\Models\` 对应 `src/Models/`）
4. **类名**：对应文件名（不含 `.php` 扩展名，如 `User` 对应 `User.php`）

**映射公式**：
- 完全限定类名：`NamespacePrefix\SubNamespace\ClassName`
- 文件路径：`BaseDirectory/SubNamespace/ClassName.php`

### composer.json 配置

**语法**：

```json
{
    "autoload": {
        "psr-4": {
            "NamespacePrefix\\": "BaseDirectory/"
        }
    }
}
```

**组成部分**：
- `"autoload"`：自动加载配置键
- `"psr-4"`：PSR-4 自动加载配置
- `"NamespacePrefix\\"`：命名空间前缀（必须以 `\` 结尾）
- `"BaseDirectory/"`：对应的基础目录

### 生成自动加载器

**命令**：`composer dump-autoload`

**参数**：
- `-o` 或 `--optimize`：生成优化的类映射
- `-a` 或 `--classmap-authoritative`：只使用类映射，不扫描文件系统

## 基本用法

### 示例 1：基础 PSR-4 配置

```json
{
    "name": "myapp/app",
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

**目录结构**：

```
project/
├── composer.json
├── src/
│   ├── Models/
│   │   └── User.php          (namespace App\Models;)
│   ├── Controllers/
│   │   └── UserController.php (namespace App\Controllers;)
│   └── Services/
│       └── UserService.php    (namespace App\Services;)
└── vendor/
    └── autoload.php
```

**类文件示例**：

```php
<?php
// src/Models/User.php
declare(strict_types=1);

namespace App\Models;

class User
{
    public function __construct(
        private int $id,
        private string $name
    ) {}
}
```

**使用**：

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use App\Models\User;

$user = new User(1, 'John');
```

**说明**：
- `App\` 命名空间前缀映射到 `src/` 目录
- `App\Models\User` 对应 `src/Models/User.php`
- Composer 自动生成加载逻辑

### 示例 2：多个命名空间前缀

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Tests\\": "tests/",
            "App\\Commands\\": "app/Commands/"
        }
    }
}
```

**目录结构**：

```
project/
├── src/
│   └── Models/User.php        (namespace App\Models;)
├── tests/
│   └── Models/UserTest.php    (namespace App\Tests\Models;)
└── app/
    └── Commands/
        └── CreateUser.php     (namespace App\Commands;)
```

**说明**：
- 可以配置多个命名空间前缀映射
- 每个前缀映射到不同的目录
- 更细粒度的映射优先级更高

### 示例 3：完整的项目配置

```json
{
    "name": "mycompany/myproject",
    "description": "My PHP Project",
    "type": "project",
    "require": {
        "php": "^8.2"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Tests\\": "tests/"
        },
        "classmap": [
            "src/Legacy/"
        ],
        "files": [
            "src/helpers.php"
        ]
    }
}
```

**说明**：
- 使用 PSR-4 自动加载主要代码
- 使用 `classmap` 处理不符合 PSR-4 的遗留代码
- 使用 `files` 自动加载辅助文件

### 示例 4：类映射（classmap）

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        },
        "classmap": [
            "src/Legacy/",
            "src/ThirdParty/"
        ]
    }
}
```

**说明**：
- `classmap` 会扫描指定目录，生成类名到文件路径的映射
- 适用于不符合 PSR-4 的代码
- 性能更好（不需要解析命名空间）

**目录结构**：

```
src/
├── Legacy/
│   ├── OldClass.php    (没有命名空间或不符合 PSR-4)
│   └── AnotherClass.php
└── ThirdParty/
    └── VendorClass.php
```

### 示例 5：文件自动加载（files）

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        },
        "files": [
            "src/helpers.php",
            "src/functions.php"
        ]
    }
}
```

**helper 文件**：

```php
<?php
// src/helpers.php
declare(strict_types=1);

if (!function_exists('helper_function')) {
    function helper_function(): void
    {
        echo "Helper function\n";
    }
}
```

**使用**：

```php
<?php
require __DIR__ . '/vendor/autoload.php';

// helper 文件中的函数自动可用
helper_function();
```

**说明**：
- `files` 会在自动加载时立即加载指定文件
- 适用于全局函数和常量的定义
- 谨慎使用，避免过度使用

### 示例 6：生成和优化自动加载器

```bash
# 生成自动加载器（开发环境）
composer dump-autoload

# 生成优化的自动加载器（生产环境）
composer dump-autoload -o
# 或
composer dump-autoload --optimize

# 生成权威类映射（只使用类映射，不扫描文件系统）
composer dump-autoload -a
# 或
composer dump-autoload --classmap-authoritative
```

**说明**：
- 开发环境：使用 `composer dump-autoload`
- 生产环境：使用 `composer dump-autoload -o` 或 `-a`
- `-o` 选项会生成类映射，提升性能
- `-a` 选项只使用类映射，不扫描文件系统，性能最佳

## 使用场景

### 场景 1：标准项目结构

遵循 PSR-4 标准的项目结构。

**示例**：见"示例 1：基础 PSR-4 配置"

### 场景 2：框架开发

框架使用 PSR-4 组织代码。

**示例**：

```json
{
    "autoload": {
        "psr-4": {
            "Framework\\": "framework/",
            "App\\": "app/"
        }
    }
}
```

### 场景 3：库开发

库使用 PSR-4 组织代码，便于其他项目使用。

**示例**：

```json
{
    "name": "vendor/package",
    "autoload": {
        "psr-4": {
            "Vendor\\Package\\": "src/"
        }
    }
}
```

## 注意事项

### 命名空间前缀必须以 `\` 结尾

**正确**：`"App\\": "src/"`

**错误**：`"App": "src/"`（缺少 `\`）

### 目录结构与命名空间必须对应

- 命名空间结构必须与目录结构一致
- 类名必须与文件名一致（大小写敏感）
- 一个文件一个类

**示例**：

```
命名空间：App\Models\User
文件路径：src/Models/User.php  ✅ 正确

命名空间：App\Models\User
文件路径：src/models/user.php  ❌ 错误（大小写不一致）

命名空间：App\Models\User
文件路径：src/Models/UserModel.php  ❌ 错误（文件名不匹配）
```

### 修改配置后需要重新生成

修改 `composer.json` 中的自动加载配置后，需要运行 `composer dump-autoload` 重新生成自动加载器。

### 生产环境优化

在生产环境使用优化选项：
- `composer dump-autoload -o`：生成类映射
- `composer dump-autoload -a`：只使用类映射（最优化）

## 常见问题

### 问题 1：类找不到错误

**错误信息**：`Class 'App\Models\User' not found`

**原因**：
- 命名空间与目录结构不匹配
- 文件路径不正确
- 未运行 `composer dump-autoload`

**解决方案**：
1. 检查命名空间是否正确
2. 检查文件路径是否与命名空间对应
3. 运行 `composer dump-autoload`
4. 检查 `composer.json` 配置是否正确

**调试方法**：

```php
<?php
require __DIR__ . '/vendor/autoload.php';

// 检查类是否可以加载
if (class_exists('App\Models\User')) {
    echo "类存在\n";
} else {
    echo "类不存在\n";
}

// 查看自动加载器映射
$loader = require __DIR__ . '/vendor/autoload.php';
$classMap = $loader->getClassMap();
print_r($classMap);
```

### 问题 2：自动加载器未更新

**症状**：修改了代码或配置，但类仍然找不到

**原因**：未运行 `composer dump-autoload`

**解决方案**：运行 `composer dump-autoload` 重新生成自动加载器

### 问题 3：性能问题

**症状**：应用启动较慢

**原因**：未使用优化选项

**解决方案**：
- 使用 `composer dump-autoload -o` 生成类映射
- 生产环境使用 `composer dump-autoload -a`

### 问题 4：命名空间前缀冲突

**症状**：类加载到错误的文件

**原因**：命名空间前缀配置有重叠

**解决方案**：确保更具体的前缀在前面

**示例**：

```json
{
    "autoload": {
        "psr-4": {
            "App\\Controllers\\": "src/Controllers/",  // 更具体的前缀在前
            "App\\": "src/"                           // 更通用的前缀在后
        }
    }
}
```

## 最佳实践

### 1. 遵循 PSR-4 标准

所有新代码遵循 PSR-4 标准，命名空间与目录结构对应。

**示例**：

```
命名空间：App\Models\User
目录：src/Models/
文件：User.php
```

### 2. 使用有意义的前缀

使用有意义的命名空间前缀，通常与项目或公司名称相关。

**示例**：

```json
{
    "autoload": {
        "psr-4": {
            "MyCompany\\MyApp\\": "src/"
        }
    }
}
```

### 3. 生产环境使用优化

在生产环境使用优化选项提升性能。

**示例**：

```bash
# 生产环境部署脚本
composer install --no-dev --optimize-autoloader
# 或
composer dump-autoload -a
```

### 4. 合理使用 classmap 和 files

- **PSR-4**：用于符合标准的新代码
- **classmap**：用于不符合标准的遗留代码
- **files**：谨慎使用，只在必要时使用

### 5. 保持目录结构清晰

保持清晰的目录结构，便于理解和维护。

**示例**：

```
src/
├── Controllers/    # 控制器
├── Models/         # 模型
├── Services/       # 服务
├── Repositories/   # 仓库
└── Utils/          # 工具类
```

## 对比分析

### PSR-4 vs PSR-0

| 特性         | PSR-4                           | PSR-0                            |
|:-------------|:--------------------------------|:--------------------------------|
| **映射规则**  | 更简洁，前缀直接映射到目录        | 更复杂，命名空间全路径映射         |
| **性能**      | ✅ 更好                          | ⚠️ 较差                           |
| **灵活性**    | ✅ 更灵活                        | ⚠️ 不够灵活                       |
| **状态**      | ✅ 当前标准                      | ❌ 已废弃                         |
| **推荐**      | ✅ 推荐使用                      | ❌ 不推荐使用                     |

### Composer 自动加载 vs 手动自动加载

| 特性         | Composer 自动加载                | 手动自动加载                      |
|:-------------|:--------------------------------|:--------------------------------|
| **配置**      | ✅ 简单（JSON 配置）              | ⚠️ 需要编写代码                   |
| **标准化**    | ✅ 遵循 PSR-4 标准               | ⚠️ 自定义实现                     |
| **性能**      | ✅ 经过优化                      | ⚠️ 取决于实现                     |
| **维护**      | ✅ 自动维护                      | ⚠️ 需要手动维护                   |
| **推荐**      | ✅ 推荐使用                      | ❌ 特殊情况才使用                 |

## 练习任务

1. **配置 PSR-4 自动加载**：创建一个项目，配置 PSR-4 自动加载，创建多个类并测试自动加载。

2. **多个命名空间前缀**：配置多个命名空间前缀映射到不同的目录，创建类并测试。

3. **使用 classmap**：创建一个包含遗留代码的项目，使用 classmap 自动加载这些代码。

4. **优化自动加载器**：比较优化前后的性能差异，理解优化选项的作用。

5. **完整项目结构**：创建一个完整的项目结构，使用 PSR-4、classmap 和 files 组合配置自动加载。

## 相关章节

- **[3.5.1 命名空间基础](section-01-basics.md)**：了解命名空间的基础知识
- **[3.5.2 use 语句](section-02-use-statements.md)**：了解 use 语句的使用
- **[3.5.3 自动加载基础](section-03-autoloading-basics.md)**：了解自动加载的基础知识
- **阶段一：Composer 使用**：了解 Composer 的基础知识

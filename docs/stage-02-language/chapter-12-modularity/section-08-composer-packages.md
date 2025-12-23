# 2.12.8 编写和发布 Composer 包

## 概述

编写和发布自己的 Composer 包是 PHP 开发中的重要技能。本节详细介绍如何创建、测试、版本控制和发布 Composer 包到 Packagist。

## 创建包的基本结构

### 目录结构

**推荐的包结构**：

```
my-package/
├── composer.json
├── README.md
├── LICENSE
├── .gitignore
├── src/
│   └── MyClass.php
├── tests/
│   └── MyClassTest.php
└── .github/
    └── workflows/
        └── ci.yml
```

### 最小结构

**最简单的包结构**：

```
my-package/
├── composer.json
├── README.md
└── src/
    └── MyClass.php
```

## 编写 composer.json

### 基本配置

```json
{
    "name": "your-vendor/package-name",
    "description": "包的描述",
    "type": "library",
    "keywords": ["keyword1", "keyword2"],
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your.email@example.com"
        }
    ],
    "require": {
        "php": "^8.2"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0"
    },
    "autoload": {
        "psr-4": {
            "YourVendor\\PackageName\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "YourVendor\\PackageName\\Tests\\": "tests/"
        }
    }
}
```

### 字段详解

**必需字段**：

- `name`：包名，格式为 `vendor/package`，必须唯一（在 Packagist 上）
- `description`：包的简短描述
- `type`：包类型（`library`、`project`、`metapackage` 等）

**推荐字段**：

- `keywords`：关键词数组，便于搜索
- `license`：许可证（MIT、Apache-2.0、GPL-3.0 等）
- `authors`：作者信息数组
- `homepage`：项目主页 URL
- `support`：支持信息（issues、source、docs 等）

**完整示例**：

```json
{
    "name": "myvendor/validator",
    "description": "A simple validation library for PHP",
    "type": "library",
    "keywords": ["validation", "validator", "php"],
    "license": "MIT",
    "authors": [
        {
            "name": "John Doe",
            "email": "john@example.com",
            "homepage": "https://johndoe.com"
        }
    ],
    "homepage": "https://github.com/myvendor/validator",
    "support": {
        "issues": "https://github.com/myvendor/validator/issues",
        "source": "https://github.com/myvendor/validator"
    },
    "require": {
        "php": "^8.2"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0",
        "phpstan/phpstan": "^1.0"
    },
    "autoload": {
        "psr-4": {
            "MyVendor\\Validator\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "MyVendor\\Validator\\Tests\\": "tests/"
        }
    },
    "scripts": {
        "test": "phpunit",
        "analyse": "phpstan analyse"
    }
}
```

## 编写包代码

### 基本类结构

```php
<?php
// src/Validator.php
declare(strict_types=1);

namespace MyVendor\Validator;

/**
 * 验证器类
 * 
 * 提供常用的验证方法
 */
class Validator
{
    /**
     * 验证邮箱地址
     * 
     * @param string $email 邮箱地址
     * @return bool 验证结果
     */
    public static function email(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
    
    /**
     * 验证 URL
     * 
     * @param string $url URL 地址
     * @return bool 验证结果
     */
    public static function url(string $url): bool
    {
        return filter_var($url, FILTER_VALIDATE_URL) !== false;
    }
    
    /**
     * 验证最小长度
     * 
     * @param string $value 要验证的值
     * @param int $min 最小长度
     * @return bool 验证结果
     */
    public static function minLength(string $value, int $min): bool
    {
        return strlen($value) >= $min;
    }
}
```

### 代码规范

- 遵循 PSR-1 和 PSR-12 编码规范
- 使用类型声明
- 添加 PHPDoc 注释
- 编写单元测试

## 编写 README.md

### 基本结构

```markdown
# Package Name

包的描述和使用说明。

## 安装

```bash
composer require your-vendor/package-name
```

## 使用

```php
use YourVendor\PackageName\MyClass;

$obj = new MyClass();
echo $obj->doSomething();
```

## 文档

更多文档请参考 [文档链接]

## 许可证

MIT License
```

### 完整 README 示例

```markdown
# Validator

一个简单易用的 PHP 验证库。

## 特性

- 邮箱验证
- URL 验证
- 长度验证
- 类型安全

## 安装

```bash
composer require myvendor/validator
```

## 快速开始

```php
use MyVendor\Validator\Validator;

// 验证邮箱
if (Validator::email('test@example.com')) {
    echo "有效的邮箱地址";
}

// 验证 URL
if (Validator::url('https://example.com')) {
    echo "有效的 URL";
}

// 验证长度
if (Validator::minLength('hello', 3)) {
    echo "长度符合要求";
}
```

## API 文档

### Validator::email()

验证邮箱地址。

**参数**：
- `string $email`：要验证的邮箱地址

**返回值**：
- `bool`：验证结果

### Validator::url()

验证 URL 地址。

**参数**：
- `string $url`：要验证的 URL

**返回值**：
- `bool`：验证结果

## 测试

```bash
composer test
```

## 贡献

欢迎提交 Issue 和 Pull Request。

## 许可证

MIT License
```

## 版本控制

### 初始化 Git 仓库

```bash
# 初始化 Git 仓库
git init

# 创建 .gitignore
cat > .gitignore << EOF
vendor/
composer.lock
.idea/
.vscode/
*.log
EOF

# 提交代码
git add .
git commit -m "Initial commit"
```

### 创建版本标签

```bash
# 创建版本标签
git tag -a v1.0.0 -m "Version 1.0.0"

# 推送标签
git push origin v1.0.0

# 推送所有标签
git push --tags
```

### 版本号规范

**遵循语义化版本（Semantic Versioning）**：

- **主版本号（Major）**：不兼容的 API 修改（如 `1.0.0` → `2.0.0`）
- **次版本号（Minor）**：向后兼容的功能性新增（如 `1.0.0` → `1.1.0`）
- **修订号（Patch）**：向后兼容的问题修正（如 `1.0.0` → `1.0.1`）

**版本标签格式**：

```bash
# 主版本
v1.0.0
v2.0.0

# 次版本
v1.1.0
v1.2.0

# 修订版本
v1.0.1
v1.0.2

# 预发布版本
v1.0.0-alpha
v1.0.0-beta
v1.0.0-rc1
```

## 发布到 Packagist

### 步骤 1：在 GitHub/GitLab 创建仓库

1. 在 GitHub 创建新仓库
2. 推送代码到仓库

```bash
# 添加远程仓库
git remote add origin https://github.com/your-username/package-name.git

# 推送代码
git push -u origin main

# 推送标签
git push --tags
```

### 步骤 2：在 Packagist 提交包

1. 访问 [packagist.org](https://packagist.org)
2. 注册/登录账号
3. 点击 "Submit"
4. 输入仓库 URL（如 `https://github.com/your-username/package-name`）
5. 点击 "Check" 验证
6. 点击 "Submit" 提交

### 步骤 3：配置自动更新（推荐）

1. 在 Packagist 包页面点击 "Settings"
2. 复制 Webhook URL
3. 在 GitHub 仓库设置中添加 Webhook：
   - URL：Packagist 提供的 Webhook URL
   - Content type：`application/json`
   - Events：选择 "Just the push event"

**配置后，每次推送代码或标签，Packagist 会自动更新包。**

## 更新包

### 发布新版本

```bash
# 1. 更新代码
# ... 修改代码 ...

# 2. 更新版本号（可选，Composer 会从 Git 标签检测）
# 编辑 composer.json，更新 version 字段（可选）

# 3. 提交更改
git add .
git commit -m "Add new feature"

# 4. 创建新版本标签
git tag -a v1.1.0 -m "Version 1.1.0"

# 5. 推送代码和标签
git push origin main
git push origin v1.1.0

# Packagist 会自动检测新版本（如果配置了 Webhook）
```

### 手动更新 Packagist

如果未配置 Webhook，可以手动更新：

1. 访问 Packagist 包页面
2. 点击 "Update" 按钮
3. Packagist 会检查仓库并更新包信息

## 完整示例：创建一个验证器包

### 1. 创建项目结构

```bash
mkdir my-validator
cd my-validator
mkdir src tests
```

### 2. 编写 composer.json

```json
{
    "name": "myvendor/validator",
    "description": "A simple validation library",
    "type": "library",
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your.email@example.com"
        }
    ],
    "require": {
        "php": "^8.2"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0"
    },
    "autoload": {
        "psr-4": {
            "MyVendor\\Validator\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "MyVendor\\Validator\\Tests\\": "tests/"
        }
    }
}
```

### 3. 编写验证器类

```php
<?php
// src/Validator.php
declare(strict_types=1);

namespace MyVendor\Validator;

class Validator
{
    public static function email(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
    
    public static function url(string $url): bool
    {
        return filter_var($url, FILTER_VALIDATE_URL) !== false;
    }
    
    public static function minLength(string $value, int $min): bool
    {
        return strlen($value) >= $min;
    }
}
```

### 4. 编写测试

```php
<?php
// tests/ValidatorTest.php
declare(strict_types=1);

namespace MyVendor\Validator\Tests;

use PHPUnit\Framework\TestCase;
use MyVendor\Validator\Validator;

class ValidatorTest extends TestCase
{
    public function testEmail(): void
    {
        $this->assertTrue(Validator::email('test@example.com'));
        $this->assertFalse(Validator::email('invalid-email'));
    }
    
    public function testUrl(): void
    {
        $this->assertTrue(Validator::url('https://example.com'));
        $this->assertFalse(Validator::url('not-a-url'));
    }
    
    public function testMinLength(): void
    {
        $this->assertTrue(Validator::minLength('hello', 3));
        $this->assertFalse(Validator::minLength('hi', 3));
    }
}
```

### 5. 初始化 Composer

```bash
composer install
```

### 6. 运行测试

```bash
composer test
# 或
vendor/bin/phpunit
```

### 7. 创建 Git 仓库

```bash
git init
git add .
git commit -m "Initial commit"
git tag -a v1.0.0 -m "Version 1.0.0"
```

### 8. 推送到 GitHub

```bash
git remote add origin https://github.com/your-username/my-validator.git
git push -u origin main
git push --tags
```

### 9. 提交到 Packagist

1. 访问 packagist.org
2. 提交仓库 URL
3. 配置 Webhook（可选）

## 使用私有仓库

### 配置私有仓库

**在 composer.json 中配置**：

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/your-username/private-package.git"
        }
    ],
    "require": {
        "your-username/private-package": "dev-main"
    }
}
```

### 使用认证

**对于需要认证的私有仓库**：

```bash
# 配置 GitHub Token
composer config github-oauth.github.com your-token

# 配置 GitLab Token
composer config gitlab-token.gitlab.com your-token
```

### 使用 Satis（私有 Packagist）

**Satis** 是 Packagist 的简化版本，可以搭建私有包仓库：

```bash
# 安装 Satis
composer create-project composer/satis --stability=dev

# 配置 satis.json
# 运行 Satis
php bin/satis build satis.json public/
```

## 最佳实践

### 1. 代码质量

- 遵循 PSR 编码规范
- 编写单元测试
- 使用静态分析工具（PHPStan、Psalm）
- 代码覆盖率 > 80%

### 2. 文档

- 编写清晰的 README
- 添加代码注释
- 提供使用示例
- 维护变更日志（CHANGELOG.md）

### 3. 版本管理

- 遵循语义化版本
- 使用 Git 标签管理版本
- 维护变更日志

### 4. 持续集成

**GitHub Actions 示例**：

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - run: composer install
      - run: composer test
```

### 5. 安全性

- 定期更新依赖
- 检查安全漏洞
- 使用依赖扫描工具

## 注意事项

1. **包名唯一性**：包名在 Packagist 上必须唯一
2. **版本标签**：使用 Git 标签管理版本
3. **自动更新**：配置 Webhook 实现自动更新
4. **文档完整**：提供清晰的使用文档
5. **测试覆盖**：编写充分的单元测试

## 练习

1. 创建一个简单的 Composer 包，包含验证器类，并配置自动加载。

2. 编写包的 README 文档，包含安装、使用和 API 说明。

3. 为包编写单元测试，使用 PHPUnit。

4. 创建 Git 仓库，管理版本标签，并推送到 GitHub。

5. 将包提交到 Packagist，并配置自动更新。

6. 创建一个完整的包，包含多个类、测试、文档和 CI 配置。

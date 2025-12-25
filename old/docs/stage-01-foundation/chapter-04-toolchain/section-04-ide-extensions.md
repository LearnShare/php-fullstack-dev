# 1.3.4 IDE 与扩展配置

## 概述

选择合适的 IDE 和配置扩展可以显著提高开发效率。本章介绍主流 IDE 的选择和配置方法。

## IDE 选择

### PhpStorm

**特点**：
- 功能强大，专为 PHP 设计
- 内置调试、重构、代码分析
- 支持 Laravel、Symfony 等框架

**适用场景**：
- 专业 PHP 开发
- 大型项目
- 需要强大的调试功能

### VS Code

**特点**：
- 轻量、跨平台
- 丰富的扩展生态
- 免费开源

**适用场景**：
- 中小型项目
- 多语言开发
- 预算有限

### 对比

| 特性 | PhpStorm | VS Code |
| :--- | :------- | :------ |
| 价格 | 付费 | 免费 |
| 性能 | 高 | 中 |
| 调试 | 内置 | 需扩展 |
| 重构 | 强大 | 基础 |
| 扩展 | 内置 | 丰富 |

## PhpStorm 配置

### 基本设置

1. **PHP 解释器**
   - File → Settings → PHP
   - 选择 PHP 解释器路径
   - 设置 PHP 版本

2. **代码样式**
   - File → Settings → Editor → Code Style → PHP
   - 选择 PSR-12 标准
   - 配置缩进、换行等

3. **代码检查**
   - File → Settings → Editor → Inspections
   - 启用 PHP 代码检查
   - 配置检查级别

### 推荐扩展

- **Laravel Idea**：Laravel 框架支持
- **PHP Annotations**：注解支持
- **PHPUnit**：测试支持

### 调试配置

1. **配置服务器**
   - File → Settings → PHP → Servers
   - 添加服务器配置
   - 设置路径映射

2. **启动调试**
   - 点击工具栏的 "Start Listening" 按钮
   - 设置断点
   - 访问页面触发调试

## VS Code 配置

### 推荐扩展

#### PHP Intelephense

**功能**：PHP 语言支持、代码补全、类型检查

**安装**：
```bash
# 在 VS Code 扩展市场搜索 "PHP Intelephense"
# 或使用命令行
code --install-extension bmewburn.vscode-intelephense-client
```

**配置**：
```json
{
    "intelephense.files.maxSize": 5000000,
    "intelephense.completion.insertUseDeclaration": true,
    "intelephense.completion.fullyQualifyGlobalConstantsAndFunctions": false
}
```

#### PHP CS Fixer

**功能**：代码格式化，遵循 PSR 标准

**安装**：
```bash
code --install-extension junstyle.php-cs-fixer
```

**配置**：
```json
{
    "php-cs-fixer.executablePath": "${workspaceFolder}/vendor/bin/php-cs-fixer",
    "php-cs-fixer.rules": "@PSR12",
    "editor.formatOnSave": true
}
```

#### PHP Debug

**功能**：Xdebug 调试支持

**安装**：
```bash
code --install-extension xdebug.php-debug
```

**配置**（`.vscode/launch.json`）：
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/var/www/html": "${workspaceFolder}"
            }
        }
    ]
}
```

#### Pest

**功能**：Pest 测试框架支持

**安装**：
```bash
code --install-extension pestphp.pest
```

#### Blade Syntax

**功能**：Laravel Blade 模板语法高亮

**安装**：
```bash
code --install-extension onecentlin.laravel-blade
```

### VS Code 设置

**推荐配置**（`.vscode/settings.json`）：

```json
{
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll": true
    },
    "php.validate.executablePath": "/usr/local/bin/php",
    "php.suggest.basic": false,
    "[php]": {
        "editor.defaultFormatter": "junstyle.php-cs-fixer",
        "editor.tabSize": 4
    }
}
```

## EditorConfig

### 作用

统一不同编辑器的代码风格配置。

### 配置文件

创建 `.editorconfig` 文件：

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.php]
indent_style = space
indent_size = 4

[*.{js,json,yml,yaml}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### 安装扩展

- **VS Code**：EditorConfig for VS Code
- **PhpStorm**：内置支持

## 代码格式化

### PHP CS Fixer

**安装**：

```bash
composer require --dev friendsofphp/php-cs-fixer
```

**配置**（`.php-cs-fixer.dist.php`）：

```php
<?php

$finder = PhpCsFixer\Finder::create()
    ->in(__DIR__ . '/src')
    ->in(__DIR__ . '/tests');

return (new PhpCsFixer\Config())
    ->setRules([
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
    ])
    ->setFinder($finder);
```

**使用**：

```bash
# 检查代码
vendor/bin/php-cs-fixer fix --dry-run --diff

# 修复代码
vendor/bin/php-cs-fixer fix
```

### PHP_CodeSniffer

**安装**：

```bash
composer require --dev squizlabs/php_codesniffer
```

**使用**：

```bash
# 检查代码
vendor/bin/phpcs --standard=PSR12 src/

# 自动修复
vendor/bin/phpcbf --standard=PSR12 src/
```

## 静态分析

### PHPStan

**安装**：

```bash
composer require --dev phpstan/phpstan
```

**配置**（`phpstan.neon`）：

```yaml
parameters:
    level: 5
    paths:
        - src
```

**使用**：

```bash
vendor/bin/phpstan analyse
```

### Psalm

**安装**：

```bash
composer require --dev vimeo/psalm
```

**初始化**：

```bash
vendor/bin/psalm --init
```

**使用**：

```bash
vendor/bin/psalm
```

## 环境变量管理

### .env.example

创建 `.env.example` 文件，列出所有必需的环境变量：

```env
APP_ENV=local
APP_DEBUG=true
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=
```

### 使用 phpdotenv

**安装**：

```bash
composer require vlucas/phpdotenv
```

**使用**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Dotenv\Dotenv;

$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();

// 获取环境变量
$dbHost = $_ENV['DB_HOST'] ?? 'localhost';
```

## 完整示例

### 项目结构

```
my-project/
├── .editorconfig
├── .env.example
├── .php-cs-fixer.dist.php
├── composer.json
├── src/
│   └── App.php
└── .vscode/
    ├── settings.json
    └── launch.json
```

### 配置流程

1. **创建项目**
   ```bash
   mkdir my-project
   cd my-project
   ```

2. **初始化 Composer**
   ```bash
   composer init
   ```

3. **安装开发工具**
   ```bash
   composer require --dev friendsofphp/php-cs-fixer phpstan/phpstan
   ```

4. **创建配置文件**
   - `.editorconfig`
   - `.php-cs-fixer.dist.php`
   - `phpstan.neon`

5. **配置 IDE**
   - VS Code：安装扩展，配置 `settings.json`
   - PhpStorm：配置代码样式和检查

## 注意事项

1. **团队协作**：统一 IDE 配置，使用 EditorConfig 确保代码风格一致。

2. **版本控制**：将 `.editorconfig`、`.php-cs-fixer.dist.php` 等配置文件提交到版本控制。

3. **CI/CD**：在 CI 流程中集成代码检查和格式化。

4. **性能**：大型项目考虑使用 PHPStan Level 5+，逐步提升。

5. **扩展选择**：根据项目需求选择合适的扩展，避免安装过多扩展影响性能。

## 练习

1. 配置 VS Code 或 PhpStorm，启用代码格式化和静态分析。

2. 创建 `.editorconfig` 文件，统一代码风格。

3. 配置 PHP CS Fixer，设置 PSR-12 标准。

4. 使用 PHPStan 分析项目代码，修复发现的问题。

5. 配置 Xdebug 调试，设置断点并调试代码。

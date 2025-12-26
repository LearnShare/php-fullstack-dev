# 1.4.2 IDE 与扩展配置

## 概述

选择合适的 IDE 和配置必要的扩展可以显著提高开发效率。本节介绍主流的 PHP IDE（PhpStorm、VS Code）的选择、配置和推荐扩展。

## IDE 选择

### PhpStorm

**特点**：
- 功能强大的商业 IDE
- 智能代码补全
- 强大的调试功能
- 集成版本控制

**下载**：https://www.jetbrains.com/phpstorm/

### VS Code

**特点**：
- 免费开源的轻量级编辑器
- 丰富的扩展生态
- 跨平台支持
- 轻量快速

**下载**：https://code.visualstudio.com/

## VS Code 推荐扩展

### PHP Intelephense

**功能**：
- 智能代码补全
- 代码导航
- 类型检查

**安装**：

```bash
code --install-extension bmewburn.vscode-intelephense-client
```

### PHP Debug

**功能**：
- Xdebug 调试支持
- 断点调试
- 变量查看

**安装**：

```bash
code --install-extension xdebug.php-debug
```

### PHP DocBlocker

**功能**：
- 自动生成 PHPDoc 注释
- 代码文档生成

**安装**：

```bash
code --install-extension neilbrayfield.php-docblocker
```

## EditorConfig 配置

创建 `.editorconfig` 文件：

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4

[*.{yml,yaml}]
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

## 代码格式化

### PHP CS Fixer

**安装**：

```bash
composer require --dev friendsofphp/php-cs-fixer
```

**配置** `.php-cs-fixer.php`：

```php
<?php

$config = new PhpCsFixer\Config();
return $config
    ->setRules([
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
    ])
    ->setFinder(
        PhpCsFixer\Finder::create()
            ->in(__DIR__)
            ->exclude('vendor')
    );
```

**使用**：

```bash
vendor/bin/php-cs-fixer fix
```

## 静态分析工具

### PHPStan

**安装**：

```bash
composer require --dev phpstan/phpstan
```

**使用**：

```bash
vendor/bin/phpstan analyse src
```

### Psalm

**安装**：

```bash
composer require --dev vimeo/psalm
```

**使用**：

```bash
vendor/bin/psalm
```

## 完整示例

### 示例 1：VS Code 配置

创建 `.vscode/settings.json`：

```json
{
    "php.validate.executablePath": "/usr/local/bin/php",
    "php.suggest.basic": false,
    "intelephense.files.maxSize": 5000000,
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll": true
    }
}
```

### 示例 2：调试配置

创建 `.vscode/launch.json`：

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

## 注意事项

- **扩展选择**：根据项目需求选择合适的扩展
- **性能考虑**：避免安装过多扩展影响性能
- **配置同步**：使用 VS Code 设置同步功能
- **团队协作**：统一 IDE 配置，使用 EditorConfig

## 最佳实践

- 使用 EditorConfig 统一代码风格
- 配置代码格式化工具，自动格式化代码
- 使用静态分析工具检查代码质量
- 配置调试工具，提高调试效率
- 定期更新扩展和工具

## 相关章节

- **1.5 PHP 配置与扩展**：了解 PHP 配置和扩展
- **2.13 代码规范**：了解代码规范详细要求

## 练习任务

1. 安装 VS Code 或 PhpStorm，并配置 PHP 开发环境。
2. 安装推荐的 PHP 扩展（PHP Intelephense、PHP Debug）。
3. 创建 `.editorconfig` 文件，配置代码风格。
4. 安装并配置 PHP CS Fixer，设置自动格式化。
5. 配置 Xdebug 调试环境，尝试断点调试。

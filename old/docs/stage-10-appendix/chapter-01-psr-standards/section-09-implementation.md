# 10.1.9 PSR 标准实施

## 工具链

### PHP CS Fixer

**安装**：

```bash
composer require --dev friendsofphp/php-cs-fixer
```

**配置 `.php-cs-fixer.php`**：

```php
<?php

$config = new PhpCsFixer\Config();

return $config
    ->setRules([
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'no_unused_imports' => true,
    ])
    ->setFinder(
        PhpCsFixer\Finder::create()
            ->in(__DIR__ . '/src')
            ->in(__DIR__ . '/tests')
            ->name('*.php')
    );
```

**使用**：

```bash
# 检查
vendor/bin/php-cs-fixer fix --dry-run --diff

# 修复
vendor/bin/php-cs-fixer fix
```

### PHP CodeSniffer

**安装**：

```bash
composer require --dev squizlabs/php_codesniffer
```

**配置 `phpcs.xml`**：

```xml
<?xml version="1.0"?>
<ruleset name="PSR12">
    <description>PSR-12 Coding Standard</description>
    
    <file>./src</file>
    <file>./tests</file>
    
    <rule ref="PSR12"/>
    
    <exclude-pattern>*/vendor/*</exclude-pattern>
</ruleset>
```

**使用**：

```bash
# 检查
vendor/bin/phpcs

# 修复（部分）
vendor/bin/phpcbf
```

## CI/CD 集成

### GitHub Actions

```yaml
name: Code Style Check

on: [push, pull_request]

jobs:
  php-cs-fixer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install dependencies
        run: composer install
      - name: Run PHP CS Fixer
        run: vendor/bin/php-cs-fixer fix --dry-run --diff
```

### GitLab CI

```yaml
php-cs-fixer:
  image: php:8.2
  script:
    - composer install
    - vendor/bin/php-cs-fixer fix --dry-run --diff
```

## 代码审查清单

- [ ] 代码符合 PSR-1 基础编码标准
- [ ] 代码符合 PSR-12 编码风格指南
- [ ] 使用 PSR-4 自动加载
- [ ] 使用 PSR-3 日志接口（如需要）
- [ ] 使用 PSR-7 HTTP 消息接口（如需要）
- [ ] 使用 PSR-11 容器接口（如需要）
- [ ] 通过代码风格检查工具
- [ ] 通过静态分析工具

## 团队协作

### 1. 统一工具配置

在项目中包含工具配置文件，确保所有团队成员使用相同的配置。

### 2. 预提交钩子

**`.git/hooks/pre-commit`**：

```bash
#!/bin/bash
vendor/bin/php-cs-fixer fix --dry-run --diff
if [ $? -ne 0 ]; then
    echo "Code style check failed. Run: vendor/bin/php-cs-fixer fix"
    exit 1
fi
```

### 3. 代码审查流程

1. 提交前运行代码风格检查
2. 代码审查时检查 PSR 标准遵循情况
3. CI/CD 中自动检查
4. 不符合标准的代码不允许合并

## 总结

PSR 标准的实施需要：

- 选择合适的工具
- 配置 CI/CD 流程
- 建立代码审查机制
- 团队统一标准
- 持续改进

通过系统化的实施，可以确保代码质量和团队协作效率。

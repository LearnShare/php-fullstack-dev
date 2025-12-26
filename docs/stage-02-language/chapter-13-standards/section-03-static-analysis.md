# 2.13.3 静态代码分析

## 概述

静态代码分析是提高代码质量的重要工具，可以在不运行代码的情况下发现潜在错误、类型问题、代码质量问题等。本节简要介绍 PHPStan 和 Psalm 的基本使用。

静态代码分析帮助开发者在代码提交前及早发现问题，提高代码质量和可维护性。

## 特性

- **早期发现问题**：在代码运行前发现潜在错误
- **类型安全**：检查类型使用是否正确
- **代码质量**：发现代码质量问题和不规范用法
- **团队协作**：统一代码标准，提高团队代码质量

## 基本用法

### PHPStan

#### 安装

```bash
composer require --dev phpstan/phpstan
```

#### 初始化配置

```bash
vendor/bin/phpstan init
```

初始化后会生成 `phpstan.neon` 配置文件。

#### 运行分析

```bash
# 使用配置文件中的级别
vendor/bin/phpstan analyse

# 指定级别（0-9，级别越高检查越严格）
vendor/bin/phpstan analyse --level=5
```

#### 配置文件示例

```neon
# phpstan.neon
parameters:
    level: 5
    paths:
        - src
    excludePaths:
        - src/vendor
```

### Psalm

#### 安装

```bash
composer require --dev vimeo/psalm
```

#### 初始化配置

```bash
vendor/bin/psalm --init
```

初始化后会生成 `psalm.xml` 配置文件。

#### 运行分析

```bash
# 使用配置文件
vendor/bin/psalm

# 指定级别（1-8，级别越高检查越严格）
vendor/bin/psalm --level=3
```

#### 配置文件示例

```xml
<?xml version="1.0"?>
<psalm>
    <projectFiles>
        <directory name="src" />
    </projectFiles>
</psalm>
```

## 基本示例

### 示例 1：PHPStan 分析结果

```php
<?php
declare(strict_types=1);

class User
{
    public function find($id)  // PHPStan: 缺少类型声明
    {
        // ...
    }
}
```

**分析结果**：
```
Line   src/User.php
25     Parameter #1 $id of method User::find() expects int, string given.
```

### 示例 2：修复问题

```php
<?php
declare(strict_types=1);

// 修复后：添加类型声明
class User
{
    public function find(int $id): ?self
    {
        // ...
    }
}
```

## 分析级别

### PHPStan 级别（0-9）

- **级别 0-3**：基础检查，类型检查
- **级别 4-6**：更严格的类型检查
- **级别 7-9**：最严格的检查，包括 null 安全、泛型等

**建议**：从低级别开始，逐步提高级别。

### Psalm 级别（1-8）

- **级别 1-3**：基础检查，类型检查
- **级别 4-5**：更严格的检查
- **级别 6-8**：最严格的检查

**建议**：根据项目情况选择合适的级别。

## CI/CD 集成

### GitHub Actions 示例

```yaml
name: Static Analysis

on: [push, pull_request]

jobs:
  phpstan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install dependencies
        run: composer install --no-progress --prefer-dist
      - name: Run PHPStan
        run: vendor/bin/phpstan analyse
```

## 注意事项

### 级别选择

- **从低级别开始**：不要一开始就使用最高级别
- **逐步提高**：每次提高一个级别，修复问题后再提高
- **项目适配**：根据项目情况选择合适的级别

### 性能优化

- **使用缓存**：PHPStan 和 Psalm 都支持缓存，提高分析速度
- **排除路径**：排除不需要分析的路径（如 vendor）
- **增量分析**：只分析修改的文件

### 误报处理

- **理解规则**：理解分析工具的规则，区分误报和真实问题
- **添加注释**：对于确认的误报，使用注释忽略（`@phpstan-ignore-next-line` 或 `@psalm-suppress`）

## 对比分析

### PHPStan vs Psalm

| 特性 | PHPStan | Psalm |
|:-----|:--------|:------|
| 分析级别 | 0-9 | 1-8 |
| 配置格式 | NEON | XML |
| 性能 | 优秀 | 优秀 |
| IDE 支持 | 支持 | 支持 |
| 推荐度 | 推荐 | 推荐 |

**选择建议**：
- **PHPStan**：配置简单，适合大多数项目
- **Psalm**：类型检查更严格，适合大型项目
- **两者结合**：可以同时使用两个工具

## 相关章节

- **2.13.1 PSR 标准**：了解编码标准
- **2.13.2 PHPDoc**：了解文档注释规范（PHPDoc 注释有助于静态分析）
- **2.10 函数与作用域**：了解函数类型声明

## 练习任务

1. **工具安装练习**：
   - 安装 PHPStan 或 Psalm
   - 初始化配置
   - 运行分析，查看分析结果

2. **配置练习**：
   - 调整分析级别
   - 配置排除路径
   - 找到适合项目的配置

3. **问题修复练习**：
   - 运行分析工具
   - 修复发现的问题
   - 添加类型声明和 PHPDoc 注释

4. **CI/CD 集成练习**：
   - 配置 GitHub Actions 或 GitLab CI
   - 集成静态分析
   - 测试 CI/CD 流程

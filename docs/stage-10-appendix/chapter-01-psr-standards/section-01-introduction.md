# 10.1.1 PSR 标准概述

## 什么是 PSR

PSR（PHP Standards Recommendations）是 PHP-FIG（PHP Framework Interop Group）制定的 PHP 编码规范和接口标准。这些标准旨在提高 PHP 代码的互操作性、可读性和可维护性。

## PHP-FIG 组织

### 组织简介

PHP-FIG（PHP Framework Interop Group）成立于 2009 年，是一个由 PHP 框架和库的开发者组成的组织。其目标是：

- 促进 PHP 框架和库之间的互操作性
- 制定和推广 PHP 编码标准
- 提供可重用的接口和组件
- 推动 PHP 生态系统的发展

### 成员框架

PHP-FIG 的成员包括：

- Laravel
- Symfony
- Zend Framework / Laminas
- CakePHP
- Drupal
- Joomla
- 以及其他众多 PHP 框架和库

### 标准制定流程

1. **提案（Proposal）**：成员提出新的标准提案
2. **讨论（Discussion）**：在邮件列表和 GitHub 上讨论
3. **投票（Voting）**：成员投票决定是否接受
4. **发布（Publication）**：正式发布为 PSR 标准

## PSR 标准分类

### 编码规范类

| 标准 | 名称 | 状态 | 说明 |
| :--- | :--- | :--- | :--- |
| PSR-1 | Basic Coding Standard | 已接受 | 基础编码标准 |
| PSR-2 | Coding Style Guide | 已废弃 | 编码风格指南（已被 PSR-12 替代） |
| PSR-12 | Extended Coding Style Guide | 已接受 | 扩展编码风格指南 |

### 自动加载类

| 标准 | 名称 | 状态 | 说明 |
| :--- | :--- | :--- | :--- |
| PSR-0 | Autoloading Standard | 已废弃 | 自动加载标准（已被 PSR-4 替代） |
| PSR-4 | Autoloader | 已接受 | 自动加载标准 |

### 接口类

| 标准 | 名称 | 状态 | 说明 |
| :--- | :--- | :--- | :--- |
| PSR-3 | Logger Interface | 已接受 | 日志接口 |
| PSR-6 | Cache | 已接受 | 缓存接口 |
| PSR-7 | HTTP Message Interfaces | 已接受 | HTTP 消息接口 |
| PSR-11 | Container Interface | 已接受 | 容器接口 |
| PSR-13 | Hypermedia Links | 已接受 | 超媒体链接接口 |
| PSR-14 | Event Dispatcher | 已接受 | 事件分发器接口 |
| PSR-15 | HTTP Handlers | 已接受 | HTTP 处理器接口 |
| PSR-16 | Simple Cache | 已接受 | 简单缓存接口 |
| PSR-17 | HTTP Factories | 已接受 | HTTP 工厂接口 |
| PSR-18 | HTTP Client | 已接受 | HTTP 客户端接口 |

### 文档类

| 标准 | 名称 | 状态 | 说明 |
| :--- | :--- | :--- | :--- |
| PSR-5 | PHPDoc | 草案 | PHPDoc 标准（未正式发布） |
| PSR-19 | PHPDoc tags | 已接受 | PHPDoc 标签标准 |

### 其他类

| 标准 | 名称 | 状态 | 说明 |
| :--- | :--- | :--- | :--- |
| PSR-8 | Huggable Interface | 已废弃 | 拥抱接口（愚人节玩笑） |
| PSR-9 | Security Advisories | 已接受 | 安全公告 |
| PSR-10 | Security Reporting | 已接受 | 安全报告 |
| PSR-20 | Clock | 已接受 | 时钟接口 |
| PSR-21 | Strategy Pattern | 已接受 | 策略模式接口 |

## 核心 PSR 标准详解

### PSR-1：基础编码标准

**重要性**：⭐⭐⭐⭐⭐

PSR-1 定义了 PHP 代码的基础规范，是所有其他标准的基础。

**主要内容**：
- 文件必须只使用 `<?php` 和 `<?=` 标签
- 文件必须使用 UTF-8 编码（无 BOM）
- 文件应该只声明符号（类、函数、常量等）或产生副作用（输出、修改 ini 设置等），不应同时做两件事
- 命名空间和类必须遵循自动加载标准（PSR-4）
- 类名必须使用 `StudlyCaps`（大驼峰）命名
- 类常量必须使用全大写字母，单词间用下划线分隔
- 方法名必须使用 `camelCase`（小驼峰）命名

### PSR-12：扩展编码风格指南

**重要性**：⭐⭐⭐⭐⭐

PSR-12 扩展了 PSR-1，提供了详细的代码风格规范。

**主要内容**：
- 代码必须使用 4 个空格缩进，不能使用 Tab
- 每行代码长度不应超过 120 个字符
- 文件必须以空行结束
- 关键字必须小写
- `true`、`false`、`null` 必须小写
- 花括号必须独占一行
- 控制结构关键字后必须有一个空格
- 闭包声明时，`function` 关键字后以及 `use` 关键字前后必须有一个空格

### PSR-4：自动加载标准

**重要性**：⭐⭐⭐⭐⭐

PSR-4 定义了自动加载的标准，是现代 PHP 项目的基础。

**主要内容**：
- 完全限定的类名格式：`\<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>`
- 完全限定的类名必须有一个顶级命名空间（"Vendor namespace"）
- 完全限定的类名可以有多个子命名空间
- 命名空间前缀与基础目录一一对应
- 类名中的下划线没有特殊含义
- 字母大小写在自动加载时是大小写敏感的

### PSR-3：日志接口

**重要性**：⭐⭐⭐⭐

PSR-3 定义了日志接口标准，实现了框架和库之间的日志互操作。

**主要内容**：
- 定义了 `Psr\Log\LoggerInterface` 接口
- 定义了 8 个日志级别：`emergency`、`alert`、`critical`、`error`、`warning`、`notice`、`info`、`debug`
- 支持上下文信息（context）
- 支持占位符插值

### PSR-7：HTTP 消息接口

**重要性**：⭐⭐⭐⭐

PSR-7 定义了 HTTP 消息的标准接口，实现了不同框架之间的 HTTP 消息互操作。

**主要内容**：
- `MessageInterface`：基础消息接口
- `RequestInterface`：请求接口
- `ResponseInterface`：响应接口
- `ServerRequestInterface`：服务器请求接口
- `StreamInterface`：流接口
- `UriInterface`：URI 接口
- `UploadedFileInterface`：上传文件接口

### PSR-11：容器接口

**重要性**：⭐⭐⭐⭐

PSR-11 定义了依赖注入容器的标准接口。

**主要内容**：
- `ContainerInterface`：容器接口
- `ContainerExceptionInterface`：容器异常接口
- `NotFoundExceptionInterface`：未找到异常接口

## PSR 标准的意义

### 1. 提高代码互操作性

遵循 PSR 标准的代码可以在不同的框架和库之间无缝使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 PSR-3 日志接口，可以在任何框架中使用
use Psr\Log\LoggerInterface;

class UserService
{
    public function __construct(
        private LoggerInterface $logger
    ) {}
    
    public function createUser(string $name): void
    {
        $this->logger->info('Creating user', ['name' => $name]);
        // ...
    }
}

// 在 Laravel 中使用
$service = new UserService(app(LoggerInterface::class));

// 在 Symfony 中使用
$service = new UserService($container->get(LoggerInterface::class));
```

### 2. 提高代码可读性

统一的编码风格使代码更易读、易维护。

**示例对比**：

**不符合 PSR-12**：

```php
<?php
class UserService{
    public function getUser($id){
        if($id>0){
            return $this->repository->find($id);
        }
        return null;
    }
}
```

**符合 PSR-12**：

```php
<?php
declare(strict_types=1);

class UserService
{
    public function getUser(int $id): ?User
    {
        if ($id > 0) {
            return $this->repository->find($id);
        }
        
        return null;
    }
}
```

### 3. 提高团队协作效率

统一的编码标准减少了代码审查时的争议，提高了团队协作效率。

### 4. 推动生态系统发展

PSR 标准推动了 PHP 生态系统的发展，使不同框架和库能够更好地协作。

## 如何应用 PSR 标准

### 1. 使用代码检查工具

**PHP CS Fixer**：

```bash
# 安装
composer require --dev friendsofphp/php-cs-fixer

# 配置 .php-cs-fixer.php
<?php
$config = new PhpCsFixer\Config();
return $config
    ->setRules([
        '@PSR12' => true,
    ])
    ->setFinder(
        PhpCsFixer\Finder::create()
            ->in(__DIR__ . '/src')
    );

# 检查代码
vendor/bin/php-cs-fixer fix --dry-run

# 自动修复
vendor/bin/php-cs-fixer fix
```

**PHP CodeSniffer**：

```bash
# 安装
composer require --dev squizlabs/php_codesniffer

# 检查代码
vendor/bin/phpcs --standard=PSR12 src/

# 自动修复（部分）
vendor/bin/phpcbf --standard=PSR12 src/
```

### 2. 在 IDE 中配置

**PhpStorm**：
1. Settings → Editor → Code Style → PHP
2. 选择 "PSR-12" 预设
3. 启用 "Reformat code" 快捷键

**VS Code**：
1. 安装 "PHP CS Fixer" 扩展
2. 配置 `.vscode/settings.json`：

```json
{
    "php-cs-fixer.executablePath": "vendor/bin/php-cs-fixer",
    "php-cs-fixer.rules": "@PSR12"
}
```

### 3. 在 CI/CD 中集成

**GitHub Actions 示例**：

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

## 标准版本与状态

### 已接受的标准

- PSR-1、PSR-3、PSR-4、PSR-6、PSR-7、PSR-11、PSR-12、PSR-13、PSR-14、PSR-15、PSR-16、PSR-17、PSR-18、PSR-19、PSR-20、PSR-21

### 已废弃的标准

- PSR-0（被 PSR-4 替代）
- PSR-2（被 PSR-12 替代）
- PSR-8（愚人节玩笑）

### 草案标准

- PSR-5（PHPDoc 标准，未正式发布）

## 学习路径建议

1. **基础阶段**：重点学习 PSR-1 和 PSR-12，掌握代码风格规范
2. **进阶阶段**：学习 PSR-4 自动加载，理解现代 PHP 项目结构
3. **应用阶段**：学习 PSR-3、PSR-7、PSR-11 等接口标准，实现框架互操作
4. **实践阶段**：在实际项目中应用 PSR 标准，使用工具自动检查和修复

## 参考资料

- [PHP-FIG 官方网站](https://www.php-fig.org/)
- [PSR 标准列表](https://www.php-fig.org/psr/)
- [PHP CS Fixer 文档](https://cs.symfony.com/)
- [PHP CodeSniffer 文档](https://github.com/squizlabs/PHP_CodeSniffer)

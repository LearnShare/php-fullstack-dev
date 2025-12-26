# 2.11.2 include 与 require

## 概述

`include` 和 `require` 是导入其他文件的基本方法。本节详细介绍 `include`、`require`、`include_once`、`require_once` 的区别、工作原理、导入位置的影响、执行时机、作用域、错误处理和性能优化。

理解这些语句的区别和使用场景对于正确导入和使用模块非常重要。掌握错误处理和性能优化可以帮助编写更可靠的代码。

## 特性

- **include**：失败时发出警告，脚本继续执行
- **require**：失败时抛出致命错误，脚本终止
- **include_once/require_once**：避免重复导入
- **返回值**：可以返回值和检查返回值

## 语法/定义

### include

**语法**：`include 'file.php';` 或 `include('file.php');`

**行为**：
- 失败时发出 `E_WARNING` 警告
- 脚本继续执行
- 可以多次导入同一文件

**返回值**：
- 成功返回 `1`
- 失败返回 `false`

### require

**语法**：`require 'file.php';` 或 `require('file.php');`

**行为**：
- 失败时抛出 `E_COMPILE_ERROR` 致命错误
- 脚本终止执行
- 可以多次导入同一文件

**返回值**：
- 成功返回 `1`
- 失败不返回（脚本终止）

### include_once

**语法**：`include_once 'file.php';` 或 `include_once('file.php');`

**行为**：
- 仅在第一次导入时执行
- 避免重复定义错误
- 失败时发出警告

**返回值**：
- 成功返回 `true`
- 已导入返回 `true`
- 失败返回 `false`

### require_once

**语法**：`require_once 'file.php';` 或 `require_once('file.php');`

**行为**：
- 仅在第一次导入时执行
- 避免重复定义错误
- 失败时抛出致命错误

**返回值**：
- 成功返回 `true`
- 已导入返回 `true`
- 失败不返回（脚本终止）

## 基本用法

### 示例 1：include 基本使用

```php
<?php
declare(strict_types=1);

// 导入配置文件
include 'config.php';

// 导入函数文件
include 'functions.php';

// 使用导入的内容
echo DB_HOST . "\n";
$result = add(3, 4);
```

### 示例 2：require 基本使用

```php
<?php
declare(strict_types=1);

// 导入必需的类文件
require 'User.php';
require 'Database.php';

// 使用导入的类
$user = new User();
$db = new Database();
```

### 示例 3：include_once/require_once

```php
<?php
declare(strict_types=1);

// 避免重复导入
require_once 'config.php';
require_once 'config.php';  // 不会重复执行

// 在函数中使用
function loadConfig(): void
{
    require_once 'config.php';
}

loadConfig();
loadConfig();  // 不会重复执行
```

### 示例 4：检查返回值

```php
<?php
declare(strict_types=1);

// 检查 include 返回值
if (include 'optional.php') {
    echo "File loaded successfully\n";
} else {
    echo "File not found\n";
}

// require 失败会终止脚本，通常不需要检查
require 'required.php';  // 如果失败，脚本终止
```

## 完整代码示例

### 示例 1：条件导入

```php
<?php
declare(strict_types=1);

// 根据环境导入不同配置
if (getenv('APP_ENV') === 'production') {
    require 'config.prod.php';
} else {
    require 'config.dev.php';
}

// 根据功能导入模块
if ($featureEnabled) {
    include 'feature.php';
}
```

### 示例 2：循环导入处理

```php
<?php
declare(strict_types=1);

// 使用 require_once 避免重复导入
function loadDependencies(): void
{
    require_once __DIR__ . '/config.php';
    require_once __DIR__ . '/functions.php';
    require_once __DIR__ . '/classes/User.php';
}

loadDependencies();
loadDependencies();  // 安全，不会重复导入
```

### 示例 3：错误处理

```php
<?php
declare(strict_types=1);

// 处理 include 错误
$loaded = @include 'optional.php';
if (!$loaded) {
    error_log("Failed to load optional.php");
    // 使用默认配置
    $config = ['default' => 'value'];
}

// require 错误处理（使用 try-catch 在 PHP 7.0+）
try {
    require 'required.php';
} catch (Throwable $e) {
    error_log("Fatal error: " . $e->getMessage());
    exit(1);
}
```

### 示例 4：动态导入

```php
<?php
declare(strict_types=1);

// 动态导入模块
function loadModule(string $moduleName): bool
{
    $file = __DIR__ . "/modules/{$moduleName}.php";
    if (file_exists($file)) {
        require_once $file;
        return true;
    }
    return false;
}

// 使用
if (loadModule('user')) {
    echo "User module loaded\n";
}
```

## 使用场景

### include

- **可选模块**：可选的功能模块
- **模板文件**：HTML 模板文件
- **条件加载**：根据条件加载的模块

### require

- **必需依赖**：必需的类文件、配置文件
- **核心功能**：核心功能模块
- **关键文件**：关键的系统文件

### include_once/require_once

- **避免重复**：避免重复定义错误
- **配置文件**：配置文件通常使用 `require_once`
- **类文件**：类文件通常使用 `require_once`

## 注意事项

### 路径处理

- **相对路径**：注意相对路径的基准
- **绝对路径**：使用 `__DIR__` 或绝对路径
- **跨平台**：注意不同操作系统的路径分隔符

### 错误处理

- **include 错误**：检查返回值
- **require 错误**：使用 try-catch（PHP 7.0+）
- **文件存在**：使用 `file_exists()` 检查

### 性能考虑

- **once 版本**：`once` 版本有性能开销
- **缓存**：PHP 会缓存已导入的文件
- **避免过度**：避免过度使用 `include_once`

## 常见问题

### 问题 1：路径错误

**症状**：找不到文件错误

**原因**：文件路径不正确

**错误示例**：

```php
<?php
declare(strict_types=1);

// 相对路径可能不正确
include 'config.php';  // 可能找不到文件
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 使用 __DIR__ 处理路径
require __DIR__ . '/config.php';

// 或使用绝对路径
require '/path/to/config.php';

// 或使用相对路径（明确基准）
require dirname(__FILE__) . '/config.php';
```

### 问题 2：重复导入

**症状**：重复定义错误

**原因**：多次导入同一文件

**错误示例**：

```php
<?php
declare(strict_types=1);

require 'config.php';
require 'config.php';  // 可能重复定义常量或函数
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 使用 require_once
require_once 'config.php';
require_once 'config.php';  // 安全，不会重复执行

// 或检查是否已导入
if (!defined('CONFIG_LOADED')) {
    require 'config.php';
    define('CONFIG_LOADED', true);
}
```

### 问题 3：作用域问题

**症状**：导入的内容不可用

**原因**：不理解导入的作用域

**解决方法**：

```php
<?php
declare(strict_types=1);

// 导入的文件在全局作用域执行
require 'functions.php';

// 函数在全局作用域可用
$result = add(3, 4);

// 在函数内部也可以使用
function test(): void
{
    $result = add(3, 4);  // 可以使用导入的函数
}
```

## 最佳实践

### 导入选择

- **关键依赖**：使用 `require_once`
- **可选模块**：使用 `include`
- **配置文件**：使用 `require_once`
- **模板文件**：使用 `include`

### 路径处理

- **使用 __DIR__**：使用 `__DIR__` 处理相对路径
- **绝对路径**：关键文件使用绝对路径
- **路径常量**：定义路径常量

### 错误处理

- **检查返回值**：检查 `include` 的返回值
- **文件存在检查**：使用 `file_exists()` 检查
- **错误日志**：记录导入错误

## 对比分析

### include vs require

| 特性 | include | require |
|:-----|:---------|:--------|
| 失败行为 | 警告，继续执行 | 致命错误，终止执行 |
| 适用场景 | 可选模块 | 必需依赖 |
| 返回值检查 | 需要 | 不需要 |
| 推荐度 | 推荐（可选） | 推荐（必需） |

**选择建议**：
- **必需文件**：使用 `require`
- **可选文件**：使用 `include`

### 普通版本 vs once 版本

| 特性 | 普通版本 | once 版本 |
|:-----|:---------|:----------|
| 重复导入 | 允许 | 不允许 |
| 性能 | 稍好 | 稍差 |
| 安全性 | 低 | 高 |
| 推荐度 | 按需使用 | 推荐（避免重复） |

**选择建议**：
- **避免重复**：使用 `once` 版本
- **性能敏感**：按需使用普通版本

## 相关章节

- **2.11.1 模块基础**：了解模块的概念
- **2.11.3 使用导入的内容**：了解如何使用导入的内容
- **2.11.5 路径处理和返回值**：了解路径处理

## 练习任务

1. **导入语句练习**：
   - 练习使用 `include`、`require`、`include_once`、`require_once`
   - 理解不同语句的区别
   - 测试各种导入场景
   - 观察错误处理行为

2. **路径处理练习**：
   - 练习使用 `__DIR__` 处理路径
   - 测试相对路径和绝对路径
   - 处理跨平台路径问题
   - 测试各种路径场景

3. **错误处理练习**：
   - 练习处理导入错误
   - 检查返回值
   - 使用 try-catch 处理错误
   - 测试各种错误场景

4. **实际应用练习**：
   - 实现条件导入功能
   - 实现动态导入功能
   - 实现模块加载系统
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用导入语句的项目
   - 实现各种导入功能
   - 处理路径和错误
   - 进行代码审查，确保正确性

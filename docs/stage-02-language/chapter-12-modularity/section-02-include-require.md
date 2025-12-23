# 2.12.2 include 与 require

## 概述

`include` 和 `require` 是 PHP 中用于导入其他文件/模块的关键字。理解它们的工作原理、区别和使用场景对于编写模块化的 PHP 代码至关重要。

## include / require 对比

### 基本区别

| 关键字      | 失败时行为         | 返回值 |
| :---------- | :----------------- | :----- |
| `include`   | 发出警告 (`E_WARNING`)，脚本继续执行 | 成功返回 1，失败返回 `false` |
| `require`   | 抛出致命错误 (`E_ERROR`)，脚本终止 | 成功返回 1，失败不返回（脚本终止） |
| `include_once` | 仅在第一次包含文件时执行，避免重复定义 | 同 `include` |
| `require_once` | 同上，但致命错误行为不变 | 同 `require` |

### 选择建议

- **关键依赖**：使用 `require` 或 `require_once`
- **可选文件**：使用 `include` 或 `include_once`
- **可能被多次包含**：使用 `_once` 版本

## 工作原理：文件内容如何合并

### 核心概念

**`include` 和 `require` 会将指定文件的内容直接插入到当前文件的对应位置**，就像把被包含文件的代码复制粘贴到当前位置一样。

### 文件内容的合并方式

**示例 1：基本合并**

```php
<?php
// utils/math.php
function add(int $a, int $b): int
{
    return $a + $b;
}
```

```php
<?php
// main.php
echo "开始\n";

require __DIR__ . '/utils/math.php';  // 这里插入 math.php 的内容

echo "结束\n";
```

**实际执行效果**（PHP 内部处理方式）：

```php
<?php
// main.php（执行时的等效代码）
echo "开始\n";

// math.php 的内容被插入到这里
function add(int $a, int $b): int
{
    return $a + $b;
}

echo "结束\n";
```

### 导入位置的影响

**导入位置非常重要**，它决定了：

1. **执行顺序**：导入的文件会立即执行
2. **变量可用性**：导入位置之前的变量可以在导入文件中使用
3. **函数/类可用性**：导入位置之后才能使用导入文件中定义的函数和类

**示例 2：导入位置的影响**

```php
<?php
// config.php
$appName = 'My App';
```

```php
<?php
// functions.php
function getAppName(): string
{
    global $appName;  // 使用全局变量
    return $appName;
}
```

```php
<?php
// main.php
declare(strict_types=1);

// 情况1：先导入 config，再导入 functions（正确顺序）
require __DIR__ . '/config.php';      // $appName 被定义
require __DIR__ . '/functions.php';   // functions.php 可以使用 $appName

echo getAppName() . "\n";  // 输出：My App
```

```php
<?php
// main.php（错误示例）
declare(strict_types=1);

// 情况2：先导入 functions，再导入 config（错误顺序）
require __DIR__ . '/functions.php';   // functions.php 中 $appName 还未定义
require __DIR__ . '/config.php';      // $appName 在这里才定义

echo getAppName() . "\n";  // 可能输出空值或警告
```

**示例 3：函数必须在定义后才能使用**

```php
<?php
// utils.php
function helper(): void
{
    echo "Helper function\n";
}
```

```php
<?php
// main.php
declare(strict_types=1);

// 错误：在导入之前使用函数
helper();  // 错误：函数未定义

require __DIR__ . '/utils.php';
```

```php
<?php
// main.php（正确）
declare(strict_types=1);

// 正确：先导入，再使用
require __DIR__ . '/utils.php';
helper();  // 正确：函数已定义
```

### 导入内容的执行时机

**重要**：导入的文件会**立即执行**，包括：

- 函数和类的定义（立即生效）
- 文件顶部的代码（立即执行）
- 变量赋值（立即执行）

**示例 4：导入文件的立即执行**

```php
<?php
// init.php
echo "init.php 开始执行\n";

$counter = 0;

function increment(): void
{
    global $counter;
    $counter++;
    echo "Counter: {$counter}\n";
}

echo "init.php 执行完成\n";
```

```php
<?php
// main.php
declare(strict_types=1);

echo "main.php 开始\n";

require __DIR__ . '/init.php';
// 输出：
// main.php 开始
// init.php 开始执行
// init.php 执行完成

echo "main.php 继续\n";
// 输出：main.php 继续

increment();  // 输出：Counter: 1
increment();  // 输出：Counter: 2
```

### 作用域的影响

**导入文件中的代码与当前文件共享作用域**：

1. **全局作用域**：导入文件中的函数、类、常量在全局作用域可用
2. **变量作用域**：导入文件中的变量在当前作用域可用（取决于导入位置）

**示例 5：作用域共享**

```php
<?php
// module.php
$moduleVar = 'Module Variable';

function moduleFunction(): void
{
    global $moduleVar;
    echo $moduleVar . "\n";
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/module.php';

// 可以直接访问导入文件中的变量（如果在全局作用域）
echo $moduleVar . "\n";  // 输出：Module Variable

// 可以直接调用导入文件中的函数
moduleFunction();  // 输出：Module Variable
```

**示例 6：函数内部的作用域**

```php
<?php
// utils.php
$globalVar = 'Global';

function test(): void
{
    global $globalVar;
    echo $globalVar . "\n";
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/utils.php';

function myFunction(): void
{
    // 在函数内部，需要 global 关键字访问全局变量
    global $globalVar;
    echo $globalVar . "\n";  // 输出：Global
    
    // 可以直接调用导入的函数（函数在全局作用域）
    test();  // 输出：Global
}

myFunction();
```

### 执行流程总结

**导入文件的执行流程**：

1. **遇到 `include`/`require` 语句**
2. **立即读取并解析目标文件**
3. **将文件内容插入到当前位置**
4. **立即执行插入的代码**
   - 函数/类定义立即生效
   - 变量赋值立即执行
   - 其他代码立即执行
5. **继续执行后续代码**

**示例 7：完整的执行流程**

```php
<?php
// step1.php
echo "步骤 1\n";
$step1 = 'Step 1 Complete';
```

```php
<?php
// step2.php
echo "步骤 2\n";
require __DIR__ . '/step1.php';
echo "步骤 2 继续，step1 变量值：{$step1}\n";
```

```php
<?php
// main.php
declare(strict_types=1);

echo "主程序开始\n";

require __DIR__ . '/step2.php';

echo "主程序结束\n";
```

**执行输出**：
```
主程序开始
步骤 2
步骤 1
步骤 2 继续，step1 变量值：Step 1 Complete
主程序结束
```

## include 语句

### 基本语法

```php
include 'filename.php';
include $filename;
include(__DIR__ . '/file.php');
```

### 特点

- 文件不存在时发出警告 (`E_WARNING`)，脚本继续执行
- 成功返回 1，失败返回 `false`
- 可以用于可选文件

### 使用场景

- 模板文件（可选）
- 可选的辅助文件
- 非关键的配置文件

### 示例

```php
<?php
declare(strict_types=1);

// 包含函数模块
include __DIR__ . '/utils/math.php';

// 使用导入的函数
if (function_exists('add')) {
    echo add(5, 3) . "\n";  // 8
}

// 包含可选模板
if (file_exists(__DIR__ . '/templates/header.php')) {
    include __DIR__ . '/templates/header.php';
}
```

### 错误处理

```php
<?php
declare(strict_types=1);

$result = include __DIR__ . '/optional.php';
if ($result === false) {
    echo "文件加载失败，但脚本继续执行\n";
    // 使用默认值或备用方案
}
```

## require 语句

### 基本语法

```php
require 'filename.php';
require $filename;
require(__DIR__ . '/file.php');
```

### 特点

- 文件不存在时抛出致命错误 (`E_ERROR`)，脚本终止
- 成功返回 1，失败不返回（脚本终止）
- 用于必需文件

### 使用场景

- 关键依赖文件
- 类定义文件
- 必须存在的配置文件

### 示例

```php
<?php
declare(strict_types=1);

// 包含必需的类文件
require __DIR__ . '/src/User.php';

// 使用导入的类
$user = new User('Alice', 25);
echo $user->getName() . "\n";  // Alice
```

### 错误处理

```php
<?php
declare(strict_types=1);

// require 失败会直接终止脚本，无法捕获
// 因此应该确保文件存在
if (!file_exists(__DIR__ . '/required.php')) {
    die("必需文件不存在\n");
}

require __DIR__ . '/required.php';
```

## include_once / require_once

### 基本语法

```php
include_once 'filename.php';
require_once 'filename.php';
```

### 特点

- 确保文件只被包含一次
- 避免重复定义错误
- 避免副作用代码重复执行

### 为什么需要 `_once`

**问题场景**：

```php
<?php
// functions.php
function helper(): void
{
    echo "Helper\n";
}
```

```php
<?php
// main.php
require __DIR__ . '/functions.php';
require __DIR__ . '/functions.php';  // 错误：函数 helper 已定义
```

**解决方案**：

```php
<?php
// main.php
require_once __DIR__ . '/functions.php';
require_once __DIR__ . '/functions.php';  // 安全，不会重复执行
```

### 使用场景

- 可能被多个文件包含的模块
- 包含初始化代码的文件
- 定义函数、类、常量的文件

### 性能考虑

`_once` 版本需要检查文件是否已包含，性能略低于普通版本：

```php
<?php
// 性能对比
// require_once：需要检查已包含文件列表
// require：直接包含，更快

// 如果确定文件只会被包含一次，使用 require
// 如果不确定，使用 require_once
```

## 错误处理和调试

### 检查文件是否存在

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/config.php';
if (!file_exists($file)) {
    throw new RuntimeException("配置文件不存在: {$file}");
}
require $file;
```

### 使用 try-catch（仅对 include 有效）

```php
<?php
declare(strict_types=1);

try {
    $result = include __DIR__ . '/optional.php';
    if ($result === false) {
        throw new RuntimeException('文件加载失败');
    }
} catch (Throwable $e) {
    echo "错误: " . $e->getMessage() . "\n";
    // 使用默认值
}
```

**注意**：`require` 失败会抛出致命错误，无法用 try-catch 捕获。

### 调试技巧

```php
<?php
declare(strict_types=1);

// 1. 检查文件路径
$file = __DIR__ . '/config.php';
echo "尝试加载: {$file}\n";
echo "文件存在: " . (file_exists($file) ? '是' : '否') . "\n";

// 2. 检查文件权限
if (file_exists($file)) {
    echo "可读: " . (is_readable($file) ? '是' : '否') . "\n";
}

// 3. 使用 include 测试（不会终止脚本）
$result = include $file;
if ($result === false) {
    echo "加载失败\n";
}
```

## 性能优化

### 避免重复包含

```php
<?php
// ❌ 不好的做法：可能重复包含
function loadConfig(): array
{
    require __DIR__ . '/config.php';
    return $config;
}

// ✅ 好的做法：使用 _once
function loadConfig(): array
{
    require_once __DIR__ . '/config.php';
    return $config;
}
```

### 使用 OPcache

在生产环境中，启用 OPcache 可以缓存已编译的 PHP 文件，显著提升 `include`/`require` 的性能。OPcache 会缓存操作码，避免每次请求都重新编译文件。

> **详细说明**：关于 OPcache 的工作原理、配置优化、缓存管理和性能监控，请参考 [6.5.2 OPcache 配置](../../../stage-06-security/chapter-05-performance/section-02-opcache.md)。

### 减少文件包含次数

```php
<?php
// ❌ 不好的做法：在循环中重复包含
foreach ($files as $file) {
    require __DIR__ . "/modules/{$file}.php";
}

// ✅ 好的做法：先包含所有文件
$modules = ['module1', 'module2', 'module3'];
foreach ($modules as $module) {
    require_once __DIR__ . "/modules/{$module}.php";
}
```

## 安全注意事项

### 路径遍历攻击

```php
<?php
// ❌ 危险：用户输入直接用于文件路径
$file = $_GET['file'];
require $file;  // 可能包含任意文件

// ✅ 安全：验证和限制文件路径
$allowed = ['config', 'utils'];
$file = $_GET['file'] ?? 'config';
if (!in_array($file, $allowed)) {
    throw new InvalidArgumentException('不允许的文件');
}
require __DIR__ . "/{$file}.php";
```

### 文件权限

确保包含的文件有适当的权限：

```bash
# 配置文件应该只读
chmod 644 config.php

# 包含敏感信息的文件应该限制访问
chmod 600 sensitive.php
```

## 完整示例

```php
<?php
// bootstrap.php
declare(strict_types=1);

// 1. 加载配置（必需）
require_once __DIR__ . '/config/app.php';

// 2. 加载工具函数（必需）
require_once __DIR__ . '/utils/functions.php';

// 3. 加载类文件（必需）
require_once __DIR__ . '/src/User.php';
require_once __DIR__ . '/src/UserRepository.php';

// 4. 加载可选模块
if (file_exists(__DIR__ . '/modules/cache.php')) {
    include __DIR__ . '/modules/cache.php';
}

// 5. 初始化应用
$config = require __DIR__ . '/config/app.php';
echo "应用: {$config['app_name']}\n";
```

## 注意事项

1. **导入位置很重要**：必须在需要使用之前导入
2. **执行顺序**：导入文件会立即执行，注意副作用
3. **作用域共享**：导入文件与当前文件共享作用域
4. **重复导入**：使用 `_once` 版本避免重复定义错误
5. **错误处理**：`require` 失败会终止脚本，`include` 失败可以处理
6. **性能考虑**：`_once` 版本性能略低，但更安全
7. **安全性**：验证文件路径，防止路径遍历攻击

## 练习

1. 创建一个配置文件，使用 `require` 加载并验证配置内容。

2. 编写一个函数，安全地包含模板文件，处理文件不存在的情况。

3. 实现一个模块加载器，使用 `require_once` 避免重复加载。

4. 创建一个包含多个步骤的初始化脚本，演示导入文件的执行顺序。

5. 编写代码，演示 `include` 和 `require` 在错误处理上的区别。

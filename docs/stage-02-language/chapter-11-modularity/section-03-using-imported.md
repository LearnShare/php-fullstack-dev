# 2.11.3 使用导入的内容

## 概述

导入文件后，需要使用导入的内容。本节详细介绍如何使用导入的函数、类、配置、常量和变量，检查内容是否存在，处理名称冲突。

理解如何使用导入的内容对于正确使用模块非常重要。掌握检查方法和冲突处理可以帮助编写更可靠的代码。

## 特性

- **直接使用**：导入后可以直接使用函数、类、常量
- **存在检查**：可以检查内容是否存在
- **冲突处理**：可以处理名称冲突
- **作用域**：导入的内容在全局作用域可用

## 语法/定义

### 使用导入的函数

**直接调用**：导入后直接调用函数

**检查存在**：使用 `function_exists(string $function_name): bool` 检查函数是否存在

### 使用导入的类

**实例化**：使用 `new` 关键字实例化类

**检查存在**：使用 `class_exists(string $class_name, bool $autoload = true): bool` 检查类是否存在

### 使用导入的常量

**直接使用**：导入后直接使用常量

**检查存在**：使用 `defined(string $constant_name): bool` 检查常量是否已定义

### 使用导入的变量

**直接使用**：导入后直接使用变量（不推荐）

**注意**：变量在导入时执行，可能不在预期作用域

## 基本用法

### 示例 1：使用导入的函数

```php
<?php
declare(strict_types=1);

// 导入函数文件
require 'functions/math.php';

// 直接使用导入的函数
$result = add(3, 4);
echo $result . "\n";  // 7

$product = multiply(3, 4);
echo $product . "\n";  // 12
```

### 示例 2：使用导入的类

```php
<?php
declare(strict_types=1);

// 导入类文件
require 'classes/User.php';

// 实例化导入的类
$user = new User('John', 25);
echo $user->getName() . "\n";  // John
```

### 示例 3：使用导入的常量

```php
<?php
declare(strict_types=1);

// 导入配置文件
require 'config/database.php';

// 直接使用导入的常量
echo DB_HOST . "\n";  // localhost
echo DB_NAME . "\n";  // mydb
```

### 示例 4：检查内容是否存在

```php
<?php
declare(strict_types=1);

// 检查函数是否存在
if (function_exists('add')) {
    $result = add(3, 4);
} else {
    echo "Function 'add' not found\n";
}

// 检查类是否存在
if (class_exists('User')) {
    $user = new User('John', 25);
} else {
    echo "Class 'User' not found\n";
}

// 检查常量是否已定义
if (defined('DB_HOST')) {
    echo DB_HOST . "\n";
} else {
    echo "Constant 'DB_HOST' not defined\n";
}
```

## 完整代码示例

### 示例 1：安全使用导入的内容

```php
<?php
declare(strict_types=1);

class ModuleLoader
{
    public static function loadFunction(string $file, string $function): bool
    {
        if (!file_exists($file)) {
            return false;
        }
        
        require_once $file;
        
        if (!function_exists($function)) {
            return false;
        }
        
        return true;
    }
    
    public static function loadClass(string $file, string $class): bool
    {
        if (!file_exists($file)) {
            return false;
        }
        
        require_once $file;
        
        if (!class_exists($class)) {
            return false;
        }
        
        return true;
    }
}

// 使用
if (ModuleLoader::loadFunction('functions/math.php', 'add')) {
    $result = add(3, 4);
    echo $result . "\n";
}
```

### 示例 2：动态使用导入的内容

```php
<?php
declare(strict_types=1);

// 动态调用函数
function callFunction(string $functionName, ...$args): mixed
{
    if (!function_exists($functionName)) {
        throw new RuntimeException("Function '{$functionName}' not found");
    }
    
    return call_user_func($functionName, ...$args);
}

// 动态实例化类
function createInstance(string $className, ...$args): object
{
    if (!class_exists($className)) {
        throw new RuntimeException("Class '{$className}' not found");
    }
    
    return new $className(...$args);
}

// 使用
$result = callFunction('add', 3, 4);
$user = createInstance('User', 'John', 25);
```

### 示例 3：处理名称冲突

```php
<?php
declare(strict_types=1);

// 方法1：使用别名（需要命名空间支持，阶段三学习）
// namespace App;
// use Math\add as mathAdd;

// 方法2：使用前缀（当前阶段）
require 'functions/math.php';
require 'functions/string.php';

// 如果函数名冲突，使用不同的函数名
function math_add(int $a, int $b): int
{
    return $a + $b;
}

function string_add(string $a, string $b): string
{
    return $a . $b;
}

// 使用
$numResult = math_add(3, 4);
$strResult = string_add('Hello', 'World');
```

### 示例 4：配置管理

```php
<?php
declare(strict_types=1);

// config.php
return [
    'database' => [
        'host' => 'localhost',
        'name' => 'mydb'
    ],
    'app' => [
        'name' => 'My App',
        'version' => '1.0.0'
    ]
];

// 使用配置
$config = require 'config.php';

// 检查配置项是否存在
if (isset($config['database']['host'])) {
    $host = $config['database']['host'];
    echo "Database host: {$host}\n";
}

// 使用空合并运算符提供默认值
$port = $config['database']['port'] ?? 3306;
echo "Database port: {$port}\n";
```

## 使用场景

### 代码复用

- **函数复用**：使用导入的函数
- **类复用**：使用导入的类
- **配置复用**：使用导入的配置

### 动态加载

- **条件加载**：根据条件加载和使用模块
- **插件系统**：动态加载插件
- **功能开关**：根据功能开关加载模块

### 配置管理

- **集中配置**：集中管理配置
- **环境配置**：根据环境加载不同配置
- **配置验证**：验证配置的完整性

## 注意事项

### 检查存在

- **函数检查**：使用 `function_exists()` 检查函数
- **类检查**：使用 `class_exists()` 检查类
- **常量检查**：使用 `defined()` 检查常量

### 名称冲突

- **避免冲突**：使用有意义的名称避免冲突
- **前缀使用**：使用前缀区分不同模块的函数
- **命名空间**：使用命名空间避免冲突（阶段三学习）

### 作用域

- **全局作用域**：导入的内容在全局作用域可用
- **函数内部**：在函数内部也可以使用导入的内容
- **变量作用域**：注意变量的作用域

## 常见问题

### 问题 1：函数未定义错误

**症状**：调用函数时提示未定义

**原因**：函数未导入或不存在

**错误示例**：

```php
<?php
declare(strict_types=1);

// 忘记导入函数文件
$result = add(3, 4);  // Fatal error: Call to undefined function add()
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：导入函数文件
require 'functions/math.php';
$result = add(3, 4);

// 方法2：检查函数是否存在
if (function_exists('add')) {
    $result = add(3, 4);
} else {
    echo "Function 'add' not found\n";
}
```

### 问题 2：名称冲突

**症状**：函数或类名冲突

**原因**：多个模块定义相同名称

**错误示例**：

```php
<?php
declare(strict_types=1);

require 'functions/math.php';  // 定义了 add()
require 'functions/string.php';  // 也定义了 add()

$result = add(3, 4);  // 冲突，不知道调用哪个
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：使用不同的函数名
// math.php
function math_add(int $a, int $b): int { }

// string.php
function string_add(string $a, string $b): string { }

// 方法2：使用命名空间（阶段三学习）
// namespace Math;
// function add(int $a, int $b): int { }

// namespace String;
// function add(string $a, string $b): string { }
```

### 问题 3：常量未定义

**症状**：使用常量时提示未定义

**原因**：常量未导入或未定义

**解决方法**：

```php
<?php
declare(strict_types=1);

// 检查常量是否已定义
if (defined('DB_HOST')) {
    echo DB_HOST . "\n";
} else {
    echo "Constant 'DB_HOST' not defined\n";
    // 使用默认值
    $host = 'localhost';
}
```

## 最佳实践

### 使用前检查

- **检查存在**：使用前检查内容是否存在
- **错误处理**：处理内容不存在的情况
- **默认值**：提供默认值

### 避免冲突

- **有意义的名称**：使用有意义的名称
- **前缀使用**：使用前缀区分不同模块
- **命名空间**：使用命名空间（阶段三学习）

### 代码组织

- **明确依赖**：明确声明模块的依赖
- **文档说明**：为模块添加文档说明
- **测试验证**：测试导入和使用

## 对比分析

### 直接使用 vs 检查后使用

| 特性 | 直接使用 | 检查后使用 |
|:-----|:---------|:-----------|
| 简洁性 | 高 | 中 |
| 安全性 | 低 | 高 |
| 推荐度 | 推荐（确定存在） | 推荐（不确定存在） |

**选择建议**：
- **确定存在**：直接使用
- **不确定存在**：检查后使用

## 相关章节

- **2.11.2 include 与 require**：了解如何导入文件
- **2.11.4 避免冲突和循环导入**：了解如何避免冲突
- **2.10.5 可调用类型与内置函数**：了解动态调用

## 练习任务

1. **使用导入内容练习**：
   - 练习使用导入的函数、类、常量
   - 理解作用域的概念
   - 测试各种使用场景
   - 观察使用行为

2. **存在检查练习**：
   - 练习使用 `function_exists()`、`class_exists()`、`defined()`
   - 理解检查的作用
   - 测试各种检查场景
   - 处理不存在的情况

3. **冲突处理练习**：
   - 练习处理名称冲突
   - 使用前缀避免冲突
   - 测试各种冲突场景
   - 实现冲突处理方案

4. **实际应用练习**：
   - 实现模块加载系统
   - 实现动态调用功能
   - 实现配置管理系统
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用导入内容的项目
   - 实现各种使用功能
   - 处理冲突和错误
   - 进行代码审查，确保正确性

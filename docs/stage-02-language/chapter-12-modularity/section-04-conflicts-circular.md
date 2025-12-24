# 2.12.4 避免冲突和循环导入

## 概述

在模块化开发中，经常会遇到名称冲突和循环导入的问题。本节详细介绍这些问题的原因、影响以及解决方案。

## 如何避免导入内容的冲突

### 问题：函数名冲突

**场景**：两个模块定义了同名的函数

```php
<?php
// module1.php
function format(string $text): string
{
    return strtoupper($text);
}
```

```php
<?php
// module2.php
function format(string $text): string
{
    return strtolower($text);
}
```

```php
<?php
// main.php
require 'module1.php';
require 'module2.php';  // 错误：函数 format 已定义
```

**解决方案 1：使用命名空间**

```php
<?php
// module1.php
namespace Module1;

function format(string $text): string
{
    return strtoupper($text);
}
```

```php
<?php
// module2.php
namespace Module2;

function format(string $text): string
{
    return strtolower($text);
}
```

```php
<?php
// main.php
require 'module1.php';
require 'module2.php';

// 使用完全限定名
echo \Module1\format('hello') . "\n";  // HELLO
echo \Module2\format('HELLO') . "\n";  // hello

// 或使用 use 导入
use function Module1\format as formatUpper;
use function Module2\format as formatLower;

echo formatUpper('hello') . "\n";  // HELLO
echo formatLower('HELLO') . "\n";  // hello
```

**解决方案 2：重命名函数**

```php
<?php
// module1.php
function formatToUpper(string $text): string
{
    return strtoupper($text);
}
```

```php
<?php
// module2.php
function formatToLower(string $text): string
{
    return strtolower($text);
}
```

**解决方案 3：使用函数前缀**

```php
<?php
// module1.php
function module1_format(string $text): string
{
    return strtoupper($text);
}
```

```php
<?php
// module2.php
function module2_format(string $text): string
{
    return strtolower($text);
}
```

### 问题：类名冲突

**场景**：两个模块定义了同名的类

```php
<?php
// database/Connection.php
class Connection
{
    public function connect(): void
    {
        echo "数据库连接\n";
    }
}
```

```php
<?php
// cache/Connection.php
class Connection  // 错误：类 Connection 已定义
{
    public function connect(): void
    {
        echo "缓存连接\n";
    }
}
```

**解决方案：使用命名空间**

```php
<?php
// database/Connection.php
namespace Database;

class Connection
{
    public function connect(): void
    {
        echo "数据库连接\n";
    }
}
```

```php
<?php
// cache/Connection.php
namespace Cache;

class Connection
{
    public function connect(): void
    {
        echo "缓存连接\n";
    }
}
```

```php
<?php
// main.php
require 'database/Connection.php';
require 'cache/Connection.php';

// 使用别名避免冲突
use Database\Connection as DatabaseConnection;
use Cache\Connection as CacheConnection;

$db = new DatabaseConnection();
$db->connect();  // 数据库连接

$cache = new CacheConnection();
$cache->connect();  // 缓存连接
```

### 问题：常量名冲突

**场景**：两个模块定义了同名的常量

```php
<?php
// config1.php
const MAX_SIZE = 100;
```

```php
<?php
// config2.php
const MAX_SIZE = 200;  // 错误：常量 MAX_SIZE 已定义
```

**解决方案 1：使用命名空间**

```php
<?php
// config1.php
namespace Config1;

const MAX_SIZE = 100;
```

```php
<?php
// config2.php
namespace Config2;

const MAX_SIZE = 200;
```

```php
<?php
// main.php
require 'config1.php';
require 'config2.php';

// 使用完全限定名
echo \Config1\MAX_SIZE . "\n";  // 100
echo \Config2\MAX_SIZE . "\n";  // 200

// 或使用 use const
use const Config1\MAX_SIZE as CONFIG1_MAX_SIZE;
use const Config2\MAX_SIZE as CONFIG2_MAX_SIZE;

echo CONFIG1_MAX_SIZE . "\n";  // 100
echo CONFIG2_MAX_SIZE . "\n";  // 200
```

**解决方案 2：使用不同的常量名**

```php
<?php
// config1.php
const CONFIG1_MAX_SIZE = 100;
```

```php
<?php
// config2.php
const CONFIG2_MAX_SIZE = 200;
```

### 冲突检测和预防

**检测函数是否已定义**：

```php
<?php
if (function_exists('format')) {
    echo "函数 format 已存在\n";
    // 使用命名空间或重命名
}
```

**检测类是否已定义**：

```php
<?php
if (class_exists('Connection')) {
    echo "类 Connection 已存在\n";
    // 使用命名空间或重命名
}
```

**检测常量是否已定义**：

```php
<?php
if (defined('MAX_SIZE')) {
    echo "常量 MAX_SIZE 已存在\n";
    // 使用命名空间或重命名
}
```

## 如何避免循环导入

### 什么是循环导入

**循环导入**（Circular Dependency）是指两个或多个模块相互依赖，形成循环引用。

**示例**：

```php
<?php
// module_a.php
require 'module_b.php';

function functionA(): void
{
    functionB();  // 使用 module_b 的函数
}
```

```php
<?php
// module_b.php
require 'module_a.php';

function functionB(): void
{
    functionA();  // 使用 module_a 的函数
}
```

**问题**：
- 可能导致无限循环
- 可能导致函数/类在使用前未定义
- 难以调试和维护
- 可能导致内存溢出

### 检测循环导入

**使用调试工具**：

```php
<?php
// 跟踪已加载的文件
$loadedFiles = [];

function safeRequire(string $file): void
{
    global $loadedFiles;
    
    if (in_array($file, $loadedFiles)) {
        throw new RuntimeException("检测到循环导入: {$file}");
    }
    
    $loadedFiles[] = $file;
    require $file;
}
```

### 解决方案 1：重构代码结构

**将共同依赖提取到第三个模块**：

```php
<?php
// common.php
function commonFunction(): void
{
    echo "Common function\n";
}
```

```php
<?php
// module_a.php
require 'common.php';

function functionA(): void
{
    commonFunction();
    echo "Function A\n";
}
```

```php
<?php
// module_b.php
require 'common.php';

function functionB(): void
{
    commonFunction();
    echo "Function B\n";
}
```

**优势**：
- 消除循环依赖
- 代码更清晰
- 易于维护

### 解决方案 2：延迟加载

**使用函数包装，延迟加载依赖**：

```php
<?php
// module_a.php
function functionA(): void
{
    // 延迟加载 module_b
    if (!function_exists('functionB')) {
        require __DIR__ . '/module_b.php';
    }
    functionB();
    echo "Function A\n";
}
```

```php
<?php
// module_b.php
function functionB(): void
{
    // 延迟加载 module_a
    if (!function_exists('functionA')) {
        require __DIR__ . '/module_a.php';
    }
    functionA();
    echo "Function B\n";
}
```

**注意**：这种方法可能导致无限递归，需要谨慎使用。

**改进的延迟加载**：

```php
<?php
// module_a.php
$moduleALoaded = true;

function functionA(): void
{
    // 只在需要时加载 module_b
    static $moduleBLoaded = false;
    if (!$moduleBLoaded && !function_exists('functionB')) {
        require __DIR__ . '/module_b.php';
        $moduleBLoaded = true;
    }
    functionB();
    echo "Function A\n";
}
```

### 解决方案 3：使用接口和依赖注入

**定义接口，通过参数传递依赖**：

```php
<?php
// interfaces.php
interface ServiceInterface
{
    public function doSomething(): void;
}
```

```php
<?php
// module_a.php
require 'interfaces.php';

class ServiceA implements ServiceInterface
{
    public function __construct(
        private ?ServiceInterface $dependency = null
    ) {
    }
    
    public function doSomething(): void
    {
        if ($this->dependency) {
            $this->dependency->doSomething();
        }
        echo "Service A\n";
    }
}
```

```php
<?php
// module_b.php
require 'interfaces.php';

class ServiceB implements ServiceInterface
{
    public function __construct(
        private ?ServiceInterface $dependency = null
    ) {
    }
    
    public function doSomething(): void
    {
        if ($this->dependency) {
            $this->dependency->doSomething();
        }
        echo "Service B\n";
    }
}
```

```php
<?php
// main.php
require 'module_a.php';
require 'module_b.php';

// 通过依赖注入避免循环
$serviceA = new ServiceA();
$serviceB = new ServiceB($serviceA);

// 如果需要双向依赖，可以在创建后设置
// $serviceA->setDependency($serviceB);
```

### 解决方案 4：使用命名空间和自动加载

**使用 Composer autoload 和命名空间，避免手动 require**：

```php
<?php
// src/ModuleA/ServiceA.php
namespace ModuleA;

use ModuleB\ServiceBInterface;

class ServiceA
{
    public function __construct(
        private ServiceBInterface $serviceB
    ) {
    }
    
    public function doSomething(): void
    {
        $this->serviceB->doSomething();
        echo "Service A\n";
    }
}
```

```php
<?php
// src/ModuleB/ServiceB.php
namespace ModuleB;

use ModuleA\ServiceAInterface;

class ServiceB
{
    public function __construct(
        private ServiceAInterface $serviceA
    ) {
    }
    
    public function doSomething(): void
    {
        $this->serviceA->doSomething();
        echo "Service B\n";
    }
}
```

Composer autoload 会在需要时自动加载类，避免循环导入问题。

> **详细说明**：关于 Composer 自动加载的详细配置和使用，请参考 [1.3.2 Composer 自动加载](../../stage-01-foundation/chapter-04-toolchain/section-02-composer-autoload.md)。

### 解决方案 5：事件驱动架构

**使用事件解耦模块**：

```php
<?php
// EventDispatcher.php
class EventDispatcher
{
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }
    
    public function emit(string $event, mixed $data = null): void
    {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                $listener($data);
            }
        }
    }
}
```

```php
<?php
// module_a.php
require 'EventDispatcher.php';

class ModuleA
{
    public function __construct(
        private EventDispatcher $dispatcher
    ) {
        $this->dispatcher->on('module_b.ready', function() {
            echo "Module B is ready\n";
        });
    }
    
    public function init(): void
    {
        $this->dispatcher->emit('module_a.ready');
    }
}
```

```php
<?php
// module_b.php
require 'EventDispatcher.php';

class ModuleB
{
    public function __construct(
        private EventDispatcher $dispatcher
    ) {
        $this->dispatcher->on('module_a.ready', function() {
            echo "Module A is ready\n";
        });
    }
    
    public function init(): void
    {
        $this->dispatcher->emit('module_b.ready');
    }
}
```

## 最佳实践

### 1. 设计清晰的依赖关系

- 避免双向依赖
- 使用依赖注入
- 提取共同依赖

### 2. 使用命名空间

- 避免名称冲突
- 组织代码结构
- 提高可维护性

### 3. 使用自动加载

- 避免手动 require
- 按需加载
- 减少循环依赖风险

### 4. 模块化设计

- 单一职责
- 高内聚低耦合
- 接口隔离

### 5. 文档化依赖

```php
<?php
/**
 * ModuleA
 * 
 * 依赖：
 * - ModuleB (通过接口)
 * - Common (工具函数)
 * 
 * 被依赖：
 * - ModuleC
 */
class ModuleA
{
    // ...
}
```

## 调试技巧

### 跟踪文件加载

```php
<?php
$loadedFiles = [];

function trackRequire(string $file): void
{
    global $loadedFiles;
    
    echo "加载文件: {$file}\n";
    $loadedFiles[] = $file;
    
    if (count($loadedFiles) > 50) {
        throw new RuntimeException("可能陷入循环导入");
    }
    
    require $file;
}
```

### 可视化依赖关系

```php
<?php
// 生成依赖图
$dependencies = [
    'module_a.php' => ['module_b.php'],
    'module_b.php' => ['module_a.php'],  // 循环依赖
];

// 检测循环
function hasCycle(array $dependencies): bool
{
    // 使用深度优先搜索检测循环
    // ...
}
```

## 完整示例

### 重构前（有循环依赖）

```php
<?php
// UserService.php
require 'UserRepository.php';

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {
    }
}
```

```php
<?php
// UserRepository.php
require 'UserService.php';  // 循环依赖

class UserRepository
{
    public function __construct(
        private UserService $service
    ) {
    }
}
```

### 重构后（消除循环）

```php
<?php
// User.php
class User
{
    // 用户实体
}
```

```php
<?php
// UserRepository.php
require 'User.php';

class UserRepository
{
    public function find(int $id): ?User
    {
        // 查找用户
    }
}
```

```php
<?php
// UserService.php
require 'UserRepository.php';

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {
    }
    
    public function getUser(int $id): ?User
    {
        return $this->repository->find($id);
    }
}
```

## 注意事项

1. **设计阶段**：在设计时就避免循环依赖
2. **代码审查**：定期检查依赖关系
3. **使用工具**：使用静态分析工具检测循环依赖
4. **文档化**：记录模块之间的依赖关系
5. **重构**：及时重构消除循环依赖

## 练习

1. 创建两个模块，它们都定义了 `format()` 函数，使用命名空间避免冲突。

2. 创建一个循环依赖的场景，然后重构代码消除循环依赖。

3. 实现一个依赖注入容器，管理模块之间的依赖关系。

4. 编写代码，检测和报告循环导入问题。

5. 设计一个模块系统，使用事件驱动架构避免循环依赖。

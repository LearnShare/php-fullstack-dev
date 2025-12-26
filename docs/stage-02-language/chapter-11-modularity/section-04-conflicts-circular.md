# 2.11.4 避免冲突和循环导入

## 概述

模块化开发中，需要避免名称冲突和循环导入问题。本节详细介绍如何避免函数名、类名、常量名冲突，循环导入的概念和问题，多种解决方案（重构、延迟加载、依赖注入、自动加载、事件驱动）。

理解如何避免冲突和循环导入对于编写可维护的模块化代码非常重要。掌握各种解决方案可以帮助解决复杂的依赖问题。

## 特性

- **名称冲突**：多个模块定义相同名称
- **循环导入**：模块间相互依赖
- **解决方案**：多种解决方案可供选择
- **最佳实践**：遵循最佳实践避免问题

## 语法/定义

### 名称冲突

**类型**：
- 函数名冲突
- 类名冲突
- 常量名冲突

**原因**：多个模块定义相同名称

**影响**：导致重复定义错误或行为不确定

### 循环导入

**概念**：模块 A 导入模块 B，模块 B 导入模块 A

**问题**：
- 重复定义错误
- 无限循环风险
- 代码难以维护

**示例**：
```
A.php -> require B.php
B.php -> require A.php
```

## 基本用法

### 示例 1：避免名称冲突 - 使用前缀

```php
<?php
declare(strict_types=1);

// math.php
function math_add(int $a, int $b): int
{
    return $a + $b;
}

function math_multiply(int $a, int $b): int
{
    return $a * $b;
}

// string.php
function string_add(string $a, string $b): string
{
    return $a . $b;
}

function string_length(string $str): int
{
    return strlen($str);
}
```

### 示例 2：避免循环导入 - 延迟加载

```php
<?php
declare(strict_types=1);

// Database.php
class Database
{
    private static ?Database $instance = null;
    
    public static function getInstance(): Database
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    public function query(string $sql): array
    {
        // 延迟加载 Logger
        if (!class_exists('Logger')) {
            require_once __DIR__ . '/Logger.php';
        }
        
        $logger = new Logger();
        $logger->log("Executing query: {$sql}");
        
        // 执行查询
        return [];
    }
}

// Logger.php
class Logger
{
    public function log(string $message): void
    {
        // 延迟加载 Database（如果需要）
        if (!class_exists('Database')) {
            require_once __DIR__ . '/Database.php';
        }
        
        // 记录日志
        echo "[LOG] {$message}\n";
    }
}
```

### 示例 3：避免循环导入 - 依赖注入

```php
<?php
declare(strict_types=1);

// Database.php
class Database
{
    public function __construct(private ?Logger $logger = null) {}
    
    public function query(string $sql): array
    {
        if ($this->logger !== null) {
            $this->logger->log("Executing query: {$sql}");
        }
        
        // 执行查询
        return [];
    }
}

// Logger.php
class Logger
{
    public function log(string $message): void
    {
        echo "[LOG] {$message}\n";
    }
}

// 使用依赖注入
require_once __DIR__ . '/Logger.php';
require_once __DIR__ . '/Database.php';

$logger = new Logger();
$db = new Database($logger);
```

## 完整代码示例

### 示例 1：模块注册系统

```php
<?php
declare(strict_types=1);

class ModuleRegistry
{
    private static array $modules = [];
    private static array $loaded = [];
    
    public static function register(string $name, string $file): void
    {
        self::$modules[$name] = $file;
    }
    
    public static function load(string $name): bool
    {
        // 检查是否已加载
        if (isset(self::$loaded[$name])) {
            return true;
        }
        
        // 检查是否在加载中（防止循环）
        if (isset(self::$loading[$name])) {
            throw new RuntimeException("Circular dependency detected: {$name}");
        }
        
        if (!isset(self::$modules[$name])) {
            return false;
        }
        
        // 标记为加载中
        self::$loading[$name] = true;
        
        // 加载模块
        require_once self::$modules[$name];
        
        // 标记为已加载
        self::$loaded[$name] = true;
        unset(self::$loading[$name]);
        
        return true;
    }
    
    private static array $loading = [];
}

// 注册模块
ModuleRegistry::register('math', __DIR__ . '/math.php');
ModuleRegistry::register('string', __DIR__ . '/string.php');

// 加载模块
ModuleRegistry::load('math');
```

### 示例 2：依赖解析系统

```php
<?php
declare(strict_types=1);

class DependencyResolver
{
    private array $dependencies = [];
    private array $loaded = [];
    
    public function addDependency(string $module, array $deps): void
    {
        $this->dependencies[$module] = $deps;
    }
    
    public function load(string $module): void
    {
        if (isset($this->loaded[$module])) {
            return;  // 已加载
        }
        
        // 加载依赖
        if (isset($this->dependencies[$module])) {
            foreach ($this->dependencies[$module] as $dep) {
                $this->load($dep);
            }
        }
        
        // 加载模块
        $file = __DIR__ . "/modules/{$module}.php";
        if (file_exists($file)) {
            require_once $file;
            $this->loaded[$module] = true;
        }
    }
}

// 使用
$resolver = new DependencyResolver();
$resolver->addDependency('user', ['database', 'logger']);
$resolver->addDependency('database', []);
$resolver->addDependency('logger', []);

$resolver->load('user');  // 自动加载依赖
```

### 示例 3：事件驱动解耦

```php
<?php
declare(strict_types=1);

// EventEmitter.php
class EventEmitter
{
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }
    
    public function emit(string $event, ...$args): void
    {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                $listener(...$args);
            }
        }
    }
}

// Database.php - 不直接依赖 Logger
class Database
{
    public function __construct(private EventEmitter $emitter) {}
    
    public function query(string $sql): array
    {
        $this->emitter->emit('database.query', $sql);
        return [];
    }
}

// Logger.php - 监听事件，不直接依赖 Database
class Logger
{
    public function __construct(EventEmitter $emitter)
    {
        $emitter->on('database.query', function($sql) {
            $this->log("Query: {$sql}");
        });
    }
    
    public function log(string $message): void
    {
        echo "[LOG] {$message}\n";
    }
}

// 使用 - 通过事件解耦
$emitter = new EventEmitter();
$logger = new Logger($emitter);
$db = new Database($emitter);
```

## 使用场景

### 大型项目

- **避免冲突**：使用前缀或命名空间避免名称冲突
- **依赖管理**：管理模块间的依赖关系
- **代码组织**：组织清晰的代码结构

### 模块化开发

- **避免循环**：避免循环导入问题
- **延迟加载**：按需加载模块
- **依赖注入**：使用依赖注入解耦

### 代码质量

- **可维护性**：提高代码可维护性
- **可测试性**：提高代码可测试性
- **可扩展性**：提高代码可扩展性

## 注意事项

### 名称冲突

- **使用前缀**：使用前缀区分不同模块
- **命名空间**：使用命名空间避免冲突（阶段三学习）
- **命名规范**：遵循命名规范

### 循环导入

- **依赖分析**：分析模块间的依赖关系
- **重构代码**：必要时重构代码消除循环
- **延迟加载**：使用延迟加载避免循环

### 依赖管理

- **明确依赖**：明确声明模块的依赖
- **最小依赖**：最小化模块的依赖
- **依赖注入**：使用依赖注入解耦

## 常见问题

### 问题 1：名称冲突

**症状**：重复定义错误

**原因**：多个模块定义相同名称

**错误示例**：

```php
<?php
declare(strict_types=1);

// math.php
function add(int $a, int $b): int { }

// string.php
function add(string $a, string $b): string { }

// 导入两个文件
require 'math.php';
require 'string.php';  // Fatal error: Cannot redeclare add()
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：使用前缀
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

### 问题 2：循环导入

**症状**：重复定义错误或无限循环

**原因**：模块间相互依赖

**错误示例**：

```php
<?php
declare(strict_types=1);

// A.php
require 'B.php';
class A { }

// B.php
require 'A.php';
class B { }
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：延迟加载
// A.php
class A
{
    public function useB(): void
    {
        if (!class_exists('B')) {
            require_once __DIR__ . '/B.php';
        }
        $b = new B();
    }
}

// 方法2：依赖注入
// A.php
class A
{
    public function __construct(private ?B $b = null) {}
}

// 方法3：重构代码，消除循环依赖
// 提取公共部分到独立模块
```

## 最佳实践

### 避免冲突

- **使用前缀**：使用前缀区分不同模块
- **命名空间**：使用命名空间（阶段三学习）
- **命名规范**：遵循命名规范

### 避免循环

- **依赖分析**：分析模块间的依赖关系
- **重构代码**：必要时重构代码
- **延迟加载**：使用延迟加载
- **依赖注入**：使用依赖注入

### 代码组织

- **清晰结构**：组织清晰的代码结构
- **明确依赖**：明确声明依赖
- **文档说明**：为模块添加文档说明

## 对比分析

### 延迟加载 vs 立即加载

| 特性 | 延迟加载 | 立即加载 |
|:-----|:---------|:---------|
| 性能 | 按需加载 | 一次性加载 |
| 循环风险 | 低 | 高 |
| 复杂度 | 中 | 低 |
| 推荐度 | 推荐（避免循环） | 推荐（简单场景） |

**选择建议**：
- **避免循环**：使用延迟加载
- **简单场景**：使用立即加载

### 依赖注入 vs 直接依赖

| 特性 | 依赖注入 | 直接依赖 |
|:-----|:---------|:----------|
| 耦合度 | 低 | 高 |
| 可测试性 | 高 | 低 |
| 复杂度 | 中 | 低 |
| 推荐度 | 推荐（解耦） | 推荐（简单场景） |

**选择建议**：
- **解耦需求**：使用依赖注入
- **简单场景**：使用直接依赖

## 相关章节

- **2.11.3 使用导入的内容**：了解如何使用导入的内容
- **2.11.5 路径处理和返回值**：了解路径处理
- **阶段三：面向对象编程基础**：深入学习命名空间和依赖注入

## 练习任务

1. **冲突避免练习**：
   - 练习使用前缀避免名称冲突
   - 理解冲突的原因和影响
   - 测试各种冲突场景
   - 实现冲突处理方案

2. **循环导入避免练习**：
   - 练习使用延迟加载避免循环导入
   - 练习使用依赖注入解耦
   - 测试各种循环导入场景
   - 实现循环导入检测

3. **依赖管理练习**：
   - 实现模块注册系统
   - 实现依赖解析系统
   - 实现事件驱动解耦
   - 测试各种依赖管理场景

4. **实际应用练习**：
   - 创建一个模块化项目
   - 实现各种避免冲突和循环的方案
   - 测试模块间的依赖关系
   - 进行代码审查，确保正确性

5. **综合练习**：
   - 创建一个复杂的模块化项目
   - 实现完整的依赖管理系统
   - 处理各种冲突和循环问题
   - 进行代码审查，确保代码质量

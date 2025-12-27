# 4.5.3 垃圾回收机制

## 概述

垃圾回收（Garbage Collection, GC）是 PHP 自动管理内存的机制，用于自动释放不再使用的内存。理解垃圾回收的工作原理、引用计数机制、循环引用问题，以及如何手动触发垃圾回收，对于优化内存使用和解决内存泄漏问题至关重要。

PHP 使用引用计数和循环引用检测两种机制来管理内存。虽然垃圾回收是自动进行的，但在某些情况下，了解其工作原理可以帮助我们编写更高效的程序。

**主要内容**：
- 垃圾回收概述（什么是垃圾回收、为什么需要）
- 引用计数机制（PHP 内存管理的基础）
- 循环引用问题（为什么引用计数无法处理循环引用）
- 循环引用检测（垃圾回收器如何处理循环引用）
- `gc_collect_cycles()` 函数（手动触发垃圾回收）
- `gc_enable()` 和 `gc_disable()` 函数（启用/禁用垃圾回收）
- 实际应用场景和注意事项

## 特性

- **自动管理**：自动释放不再使用的内存
- **引用计数**：基于引用计数的基础机制
- **循环检测**：自动检测和清理循环引用
- **手动控制**：支持手动触发垃圾回收
- **性能优化**：可以禁用垃圾回收以提高性能

## 语法/定义

### gc_collect_cycles() 函数

**语法**：`gc_collect_cycles(): int`

**参数**：无参数

**返回值**：返回收集的循环引用数量。

### gc_enable() 函数

**语法**：`gc_enable(): void`

**参数**：无参数

**返回值**：无返回值。

### gc_disable() 函数

**语法**：`gc_disable(): void`

**参数**：无参数

**返回值**：无返回值。

### gc_status() 函数

**语法**：`gc_status(): array`

**参数**：无参数

**返回值**：返回垃圾回收器状态的数组。

## 基本用法

### 示例 1：理解引用计数

```php
<?php
declare(strict_types=1);

// 创建变量，引用计数 = 1
$a = new stdClass();
$refCount = 1;

// 赋值给另一个变量，引用计数 = 2
$b = $a;
$refCount = 2;

// 取消引用，引用计数 = 1
unset($b);
$refCount = 1;

// 取消最后一个引用，引用计数 = 0，对象被销毁
unset($a);
// 此时对象可以被垃圾回收
```

**说明**：
- PHP 使用引用计数管理内存
- 当引用计数为 0 时，对象可以被回收
- 这是 PHP 内存管理的基础机制

### 示例 2：循环引用问题

```php
<?php
declare(strict_types=1);

class Node
{
    public ?Node $next = null;
    
    public function __construct(public string $value)
    {
    }
}

// 创建循环引用
$node1 = new Node('A');
$node2 = new Node('B');

$node1->next = $node2;
$node2->next = $node1;

// 即使取消变量引用，对象仍然无法被回收（引用计数 > 0）
unset($node1);
unset($node2);

// 垃圾回收器会检测并清理这种循环引用
```

**说明**：
- 循环引用导致引用计数无法归零
- 垃圾回收器会检测并清理循环引用
- 这是引用计数机制的局限性

### 示例 3：手动触发垃圾回收

```php
<?php
declare(strict_types=1);

// 创建循环引用
class ParentClass
{
    public ?ChildClass $child = null;
}

class ChildClass
{
    public ?ParentClass $parent = null;
}

$parent = new ParentClass();
$child = new ChildClass();

$parent->child = $child;
$child->parent = $parent;

// 取消变量引用
unset($parent);
unset($child);

// 手动触发垃圾回收
$collected = gc_collect_cycles();
echo "收集了 {$collected} 个循环引用\n";
```

**说明**：
- `gc_collect_cycles()` 手动触发垃圾回收
- 返回收集的循环引用数量
- 通常不需要手动触发，垃圾回收器会自动工作

### 示例 4：检查垃圾回收器状态

```php
<?php
declare(strict_types=1);

// 获取垃圾回收器状态
$status = gc_status();

echo "垃圾回收器状态:\n";
echo "运行: " . ($status['runs'] ? '是' : '否') . "\n";
echo "收集的对象数: {$status['collected']}\n";
echo "阈值: {$status['threshold']}\n";
echo "根数: {$status['roots']}\n";

// 输出示例：
// 垃圾回收器状态:
// 运行: 是
// 收集的对象数: 0
// 阈值: 10001
// 根数: 0
```

**说明**：
- `gc_status()` 返回垃圾回收器的状态信息
- 可以用于监控垃圾回收器的运行情况

### 示例 5：启用/禁用垃圾回收

```php
<?php
declare(strict_types=1);

// 禁用垃圾回收（可以提高性能，但不推荐）
gc_disable();
echo "垃圾回收已禁用\n";

// 执行一些操作
// ...

// 手动触发垃圾回收
gc_collect_cycles();

// 重新启用垃圾回收
gc_enable();
echo "垃圾回收已启用\n";
```

**说明**：
- `gc_disable()` 禁用垃圾回收
- `gc_enable()` 启用垃圾回收
- 禁用垃圾回收可以提高性能，但可能导致内存泄漏

### 示例 6：垃圾回收性能测试

```php
<?php
declare(strict_types=1);

// 测试垃圾回收的性能影响
function testWithGC(bool $enableGC): float
{
    if ($enableGC) {
        gc_enable();
    } else {
        gc_disable();
    }
    
    $start = microtime(true);
    
    // 创建大量循环引用
    for ($i = 0; $i < 10000; $i++) {
        $parent = new stdClass();
        $child = new stdClass();
        $parent->child = $child;
        $child->parent = $parent;
        unset($parent, $child);
    }
    
    if ($enableGC) {
        gc_collect_cycles();
    }
    
    $end = microtime(true);
    return $end - $start;
}

// 测试
$timeWithGC = testWithGC(true);
$timeWithoutGC = testWithGC(false);

echo "启用垃圾回收: {$timeWithGC} 秒\n";
echo "禁用垃圾回收: {$timeWithoutGC} 秒\n";
echo "性能差异: " . (($timeWithGC - $timeWithoutGC) / $timeWithoutGC * 100) . "%\n";
```

**说明**：
- 测试垃圾回收的性能影响
- 垃圾回收有性能开销，但通常可以忽略

## 使用场景

### 场景 1：处理大量循环引用

在处理大量循环引用时，可能需要手动触发垃圾回收。

**示例**：

```php
<?php
declare(strict_types=1);

// 处理大量数据，创建循环引用
for ($i = 0; $i < 100000; $i++) {
    $obj1 = new stdClass();
    $obj2 = new stdClass();
    $obj1->ref = $obj2;
    $obj2->ref = $obj1;
    unset($obj1, $obj2);
    
    // 定期手动触发垃圾回收
    if ($i % 10000 === 0) {
        gc_collect_cycles();
    }
}
```

### 场景 2：内存敏感的操作

在内存敏感的操作中，手动触发垃圾回收释放内存。

**示例**：

```php
<?php
declare(strict_types=1);

// 在处理大量数据前，清理内存
gc_collect_cycles();

// 处理大量数据
// ...

// 处理完成后，再次清理
gc_collect_cycles();
```

## 注意事项

### 通常不需要手动触发

垃圾回收器会自动工作，通常不需要手动触发。

**示例**：

```php
<?php
declare(strict_types=1);

// 通常不需要手动触发
// gc_collect_cycles();  // 通常不需要

// 垃圾回收器会在适当时机自动运行
```

### 性能影响

垃圾回收有性能开销，但通常可以忽略。只有在特殊情况下才需要禁用。

**示例**：

```php
<?php
declare(strict_types=1);

// 不推荐禁用垃圾回收（可能导致内存泄漏）
// gc_disable();  // 不推荐

// 只有在性能非常关键且确保没有循环引用时才考虑禁用
```

### 循环引用的影响

循环引用会阻止对象被及时回收，应该尽量避免创建不必要的循环引用。

**示例**：

```php
<?php
declare(strict_types=1);

// 如果可能，避免创建循环引用
class ParentClass
{
    public ?ChildClass $child = null;
}

class ChildClass
{
    // 如果不需要反向引用，就不要创建
    // public ?ParentClass $parent = null;  // 避免不必要的循环引用
}
```

## 常见问题

### 问题 1：什么是垃圾回收？

**回答**：垃圾回收是自动释放不再使用的内存的机制。PHP 使用引用计数和循环引用检测两种机制。

### 问题 2：什么时候需要手动触发垃圾回收？

**回答**：通常不需要手动触发。只有在处理大量循环引用或内存敏感的操作时，才可能需要手动触发。

**示例**：

```php
<?php
declare(strict_types=1);

// 处理大量循环引用时
for ($i = 0; $i < 100000; $i++) {
    // 创建循环引用
    // ...
    
    if ($i % 10000 === 0) {
        gc_collect_cycles();  // 定期手动触发
    }
}
```

### 问题 3：如何检测内存泄漏？

**回答**：通过监控内存使用趋势，如果在不应该增长的地方内存持续增长，可能存在内存泄漏。

**示例**：

```php
<?php
declare(strict_types=1);

// 监控内存使用
$before = memory_get_usage();

// 执行操作
// ...

$after = memory_get_usage();
$increase = $after - $before;

// 如果内存持续增长，可能存在泄漏
if ($increase > 10 * 1024 * 1024) {  // 超过 10MB
    echo "警告: 可能存在内存泄漏\n";
}
```

### 问题 4：垃圾回收的性能影响？

**回答**：垃圾回收有性能开销，但通常可以忽略。只有在特殊情况下才需要考虑禁用。

## 最佳实践

### 1. 依赖自动垃圾回收

通常依赖 PHP 的自动垃圾回收机制，不需要手动触发。

**示例**：

```php
<?php
declare(strict_types=1);

// 通常不需要手动触发
// 垃圾回收器会自动工作
```

### 2. 避免不必要的循环引用

如果可能，避免创建不必要的循环引用。

**示例**：

```php
<?php
declare(strict_types=1);

// 如果不需要反向引用，就不要创建
class ParentClass
{
    public ?ChildClass $child = null;
}

class ChildClass
{
    // 避免不必要的循环引用
    // public ?ParentClass $parent = null;
}
```

### 3. 在特殊情况下手动触发

在处理大量循环引用或内存敏感的操作时，可以手动触发垃圾回收。

**示例**：

```php
<?php
declare(strict_types=1);

// 处理大量循环引用时
for ($i = 0; $i < 100000; $i++) {
    // 创建循环引用
    // ...
    
    if ($i % 10000 === 0) {
        gc_collect_cycles();
    }
}
```

### 4. 监控垃圾回收状态

可以监控垃圾回收器的状态，了解其运行情况。

**示例**：

```php
<?php
declare(strict_types=1);

$status = gc_status();
echo "垃圾回收运行次数: {$status['runs']}\n";
echo "收集的对象数: {$status['collected']}\n";
```

### 5. 不要禁用垃圾回收

除非特殊情况，不要禁用垃圾回收，否则可能导致内存泄漏。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：禁用垃圾回收
// gc_disable();

// ✅ 推荐：依赖自动垃圾回收
// gc_enable();  // 默认启用
```

## 对比分析

### 引用计数 vs 循环引用检测

| 特性         | 引用计数                       | 循环引用检测                   |
|:-------------|:-------------------------------|:-------------------------------|
| **机制**     | 跟踪每个对象的引用数量         | 检测循环引用并清理             |
| **速度**     | ✅ 快速（实时回收）            | ⚠️ 较慢（定期检测）            |
| **适用场景** | 大多数情况（非循环引用）       | 循环引用情况                   |
| **局限性**   | ⚠️ 无法处理循环引用            | ✅ 可以处理循环引用            |

### 自动垃圾回收 vs 手动触发

| 特性         | 自动垃圾回收                   | 手动触发                       |
|:-------------|:-------------------------------|:-------------------------------|
| **便利性**   | ✅ 自动进行，无需干预          | ⚠️ 需要手动调用                |
| **控制力**   | ⚠️ 无法精确控制时机            | ✅ 可以精确控制时机            |
| **使用场景** | 大多数情况（推荐）             | 特殊需求（大量循环引用等）     |

## 练习任务

1. **垃圾回收监控工具**：创建一个工具，监控垃圾回收器的运行情况。

2. **循环引用检测工具**：实现一个工具，检测代码中可能存在的循环引用。

3. **内存泄漏检测工具**：创建一个工具，检测可能的内存泄漏。

4. **垃圾回收性能测试工具**：编写一个工具，测试垃圾回收的性能影响。

5. **内存优化工具**：实现一个工具，分析内存使用并提供优化建议。

## 相关章节

- **[4.5.1 内存使用监控](section-01-memory-monitoring.md)**：了解内存监控的方法
- **[4.5.4 内存优化实践](section-04-memory-optimization.md)**：学习内存优化方法
- **[3.1.4 对象克隆与引用](../stage-03-oop/chapter-01-classes/section-04-clone-references.md)**：了解对象引用的相关内容

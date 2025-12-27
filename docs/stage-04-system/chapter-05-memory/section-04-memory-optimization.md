# 4.5.4 内存优化实践

## 概述

内存优化是提高程序性能和稳定性的重要手段。在实际应用中，我们需要优化内存使用，减少内存占用，避免内存溢出等问题。理解内存优化的策略、常见的内存占用场景，以及如何优化数组、对象等数据结构的内存使用，对于开发高效的 PHP 应用至关重要。

本节介绍内存优化的实践方法，包括减少内存占用、优化大数组处理、对象优化等策略，帮助零基础学员掌握内存优化技巧。

**主要内容**：
- 内存优化策略概述
- 减少内存占用的方法
- 大数组处理优化
- 对象优化策略
- 流式处理
- 内存优化工具
- 实际应用场景和最佳实践

## 特性

- **策略多样**：提供多种内存优化策略
- **实用性强**：提供具体的优化方法和示例
- **性能提升**：优化后可以显著减少内存占用
- **通用适用**：适用于各种场景

## 语法/定义

### 内存优化要点

内存优化没有单一的语法定义，而是综合性的优化策略和实践方法。

## 基本用法

### 示例 1：使用生成器减少内存占用

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：创建大数组（占用大量内存）
function getLargeArray(): array
{
    $result = [];
    for ($i = 0; $i < 1000000; $i++) {
        $result[] = $i;
    }
    return $result;
}

// ✅ 推荐：使用生成器（内存友好）
function getLargeGenerator(): Generator
{
    for ($i = 0; $i < 1000000; $i++) {
        yield $i;
    }
}

// 使用生成器
foreach (getLargeGenerator() as $value) {
    // 处理每个值，不会一次性加载到内存
    echo $value . "\n";
}
```

**说明**：
- 生成器逐个生成值，不会一次性加载到内存
- 适合处理大量数据

### 示例 2：及时释放大变量

```php
<?php
declare(strict_types=1);

// 处理大文件
$largeData = file_get_contents(__DIR__ . '/large-file.txt');

// 处理数据
$processed = processData($largeData);

// 及时释放不再需要的大变量
unset($largeData);

// 继续使用处理后的数据
echo $processed;
```

**说明**：
- 使用 `unset()` 及时释放不再需要的大变量
- 可以立即释放内存

### 示例 3：流式处理大文件

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：一次性读取整个文件
// $content = file_get_contents(__DIR__ . '/large-file.txt');

// ✅ 推荐：流式读取（内存友好）
function processLargeFile(string $filename): void
{
    $handle = fopen($filename, 'r');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        while (($line = fgets($handle)) !== false) {
            // 处理每一行，不会一次性加载整个文件
            processLine($line);
        }
    } finally {
        fclose($handle);
    }
}

processLargeFile(__DIR__ . '/large-file.txt');
```

**说明**：
- 流式处理适合大文件
- 每次只处理一部分数据，不占用太多内存

### 示例 4：优化数组内存使用

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：存储大量重复的字符串
$array = [];
for ($i = 0; $i < 10000; $i++) {
    $array[] = 'very long string that is repeated many times';
}

// ✅ 推荐：使用引用或常量
$commonString = 'very long string that is repeated many times';
$array = [];
for ($i = 0; $i < 10000; $i++) {
    $array[] = &$commonString;  // 使用引用
}
// 或者
const COMMON_STRING = 'very long string that is repeated many times';
$array = array_fill(0, 10000, COMMON_STRING);
```

**说明**：
- 使用引用或常量避免重复存储相同的字符串
- 可以显著减少内存占用

### 示例 5：优化对象内存使用

```php
<?php
declare(strict_types=1);

// 优化前：对象占用较多内存
class LargeObject
{
    public string $data1;
    public string $data2;
    public string $data3;
    // ... 很多属性
}

// 优化后：使用数组存储（在某些情况下内存占用更小）
class OptimizedObject
{
    private array $data = [];
    
    public function setData(string $key, string $value): void
    {
        $this->data[$key] = $value;
    }
    
    public function getData(string $key): ?string
    {
        return $this->data[$key] ?? null;
    }
}
```

**说明**：
- 在某些情况下，使用数组存储数据比使用多个属性更节省内存
- 需要权衡可读性和内存使用

### 示例 6：使用 SplFixedArray 优化固定大小数组

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：普通数组（对于固定大小，内存占用较大）
$array = [];
for ($i = 0; $i < 10000; $i++) {
    $array[] = $i;
}

// ✅ 推荐：SplFixedArray（固定大小数组，内存占用更小）
$fixedArray = new SplFixedArray(10000);
for ($i = 0; $i < 10000; $i++) {
    $fixedArray[$i] = $i;
}

// 使用完后释放
unset($fixedArray);
```

**说明**：
- `SplFixedArray` 适合固定大小的数组
- 内存占用比普通数组小

### 示例 7：分批处理大数据

```php
<?php
declare(strict_types=1);

function processBatch(array $data, int $batchSize = 1000): void
{
    $batches = array_chunk($data, $batchSize);
    
    foreach ($batches as $batch) {
        // 处理每批数据
        processBatchData($batch);
        
        // 清理批数据
        unset($batch);
        
        // 可选：手动触发垃圾回收
        if (count($batches) > 10) {
            gc_collect_cycles();
        }
    }
}

// 使用
$largeData = range(1, 100000);
processBatch($largeData, 1000);
```

**说明**：
- 分批处理大数据，避免一次性处理导致内存溢出
- 每批处理完后释放内存

### 示例 8：内存优化工具类

```php
<?php
declare(strict_types=1);

class MemoryOptimizer
{
    public static function optimizeArray(array &$array): void
    {
        // 压缩数组，释放未使用的空间
        $array = array_values($array);
    }
    
    public static function clearLargeVariables(array $variables): void
    {
        foreach ($variables as $var) {
            unset($var);
        }
    }
    
    public static function getMemoryUsage(): array
    {
        return [
            'current' => memory_get_usage(),
            'peak' => memory_get_peak_usage(),
            'limit' => ini_get('memory_limit'),
        ];
    }
    
    public static function optimizeMemory(): void
    {
        // 触发垃圾回收
        gc_collect_cycles();
    }
}

// 使用
$data = range(1, 100000);
MemoryOptimizer::optimizeMemory();

$usage = MemoryOptimizer::getMemoryUsage();
print_r($usage);
```

**说明**：
- 封装内存优化方法
- 提供便捷的内存优化工具

## 使用场景

### 场景 1：处理大量数据

处理大量数据时，使用生成器、流式处理等方法优化内存使用。

**示例**：见"示例 1：使用生成器减少内存占用"和"示例 3：流式处理大文件"

### 场景 2：内存受限环境

在内存受限的环境中，需要优化内存使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 在内存受限环境中，使用流式处理
ini_set('memory_limit', '64M');

// 使用生成器处理大数据
function processData(): Generator
{
    for ($i = 0; $i < 1000000; $i++) {
        yield processItem($i);
    }
}

foreach (processData() as $item) {
    // 处理每个项目
}
```

## 注意事项

### 权衡性能和内存

内存优化可能会影响性能，需要权衡。

**示例**：

```php
<?php
declare(strict_types=1);

// 流式处理节省内存，但可能比一次性读取稍慢
// 需要根据实际情况选择
```

### 及时释放大变量

处理完大变量后，及时释放内存。

**示例**：见"示例 2：及时释放大变量"

### 避免过早优化

不要过早优化，先确保代码正确，再考虑优化。

**示例**：

```php
<?php
declare(strict_types=1);

// 先编写正确的代码
// 如果遇到内存问题，再考虑优化
```

## 常见问题

### 问题 1：如何减少内存占用？

**回答**：使用生成器、流式处理、及时释放变量、优化数据结构等方法。

**示例**：见"示例 1：使用生成器减少内存占用"

### 问题 2：如何处理大数组？

**回答**：使用生成器、分批处理、SplFixedArray（固定大小）等方法。

**示例**：见"示例 7：分批处理大数据"和"示例 6：使用 SplFixedArray 优化固定大小数组"

### 问题 3：如何优化对象内存使用？

**回答**：减少对象属性、使用数组存储、及时释放对象引用等。

**示例**：见"示例 5：优化对象内存使用"

### 问题 4：内存优化会影响性能吗？

**回答**：可能会，需要权衡。例如流式处理可能比一次性读取稍慢，但可以节省内存。

## 最佳实践

### 1. 使用生成器处理大量数据

对于大量数据，使用生成器而不是数组。

**示例**：见"示例 1：使用生成器减少内存占用"

### 2. 流式处理大文件

对于大文件，使用流式处理而不是一次性读取。

**示例**：见"示例 3：流式处理大文件"

### 3. 及时释放大变量

处理完大变量后，使用 `unset()` 及时释放。

**示例**：见"示例 2：及时释放大变量"

### 4. 分批处理大数据

将大数据分成小批处理，避免一次性处理。

**示例**：见"示例 7：分批处理大数据"

### 5. 监控内存使用

在优化过程中，监控内存使用情况，验证优化效果。

**示例**：

```php
<?php
declare(strict_types=1);

$before = memory_get_usage();
// 执行操作
$after = memory_get_usage();
$used = $after - $before;

echo "内存使用: " . formatBytes($used) . "\n";
```

## 对比分析

### 数组 vs 生成器

| 特性         | 数组                         | 生成器（Generator）          |
|:-------------|:-----------------------------|:-----------------------------|
| **内存占用** | ⚠️ 占用大量内存（所有数据）  | ✅ 内存友好（逐个生成）      |
| **性能**     | ✅ 随机访问快                | ⚠️ 只能顺序访问              |
| **适用场景** | 数据量较小                   | 大量数据                     |
| **灵活性**   | ✅ 支持各种操作              | ⚠️ 功能有限                  |

### 一次性读取 vs 流式处理

| 特性         | 一次性读取                   | 流式处理                     |
|:-------------|:-----------------------------|:-----------------------------|
| **内存占用** | ⚠️ 占用大量内存（整个文件）  | ✅ 内存友好（部分数据）      |
| **速度**     | ✅ 可能更快                  | ⚠️ 可能稍慢                  |
| **适用场景** | 小文件                       | 大文件                       |
| **稳定性**   | ⚠️ 大文件可能溢出            | ✅ 稳定                      |

## 练习任务

1. **内存优化工具类**：创建一个工具类，封装常用的内存优化方法。

2. **内存使用分析工具**：实现一个工具，分析代码的内存使用并提供优化建议。

3. **大数组优化工具**：创建一个工具，优化大数组的内存使用。

4. **流式处理工具**：编写一个工具，将一次性处理转换为流式处理。

5. **内存优化最佳实践指南**：创建一个指南，总结内存优化的最佳实践。

## 相关章节

- **[4.5.1 内存使用监控](section-01-memory-monitoring.md)**：了解内存监控的方法
- **[4.5.2 内存限制设置](section-02-memory-limits.md)**：了解内存限制的设置
- **[4.2.10 大文件处理](../chapter-02-filesystem/section-10-large-files.md)**：了解大文件处理中的内存优化
- **[4.3.2 HTTP 客户端编程](../chapter-03-networking/section-02-http-client.md)**：了解网络编程中的内存优化

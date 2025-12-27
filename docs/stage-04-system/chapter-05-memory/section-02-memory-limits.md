# 4.5.2 内存限制设置

## 概述

内存限制是 PHP 脚本执行的重要配置，用于防止脚本消耗过多内存导致系统问题。在实际应用中，我们需要根据脚本的实际需求设置合适的内存限制，既不能过小导致脚本无法正常运行，也不能过大导致资源浪费或安全问题。

理解内存限制的设置方法、配置优先级、限制值的格式，以及如何在运行时调整限制，对于开发健壮的 PHP 应用至关重要。

**主要内容**：
- 内存限制概述（什么是内存限制、为什么需要限制）
- `php.ini` 配置（`memory_limit` 配置项）
- `ini_set()` 函数（运行时设置内存限制）
- 限制值格式（字节数、单位表示、无限制）
- 配置优先级（php.ini vs ini_set vs .htaccess）
- 实际应用场景和最佳实践

## 特性

- **配置灵活**：支持在 php.ini 和运行时设置
- **单位支持**：支持字节、KB、MB、GB 等单位
- **无限制选项**：支持设置无限制（-1）
- **优先级控制**：不同配置方式的优先级不同

## 语法/定义

### ini_set() 函数

**语法**：`ini_set(string $option, string|int|float|bool|null $value): string|false`

**参数**：
- `$option`：配置项名称（如 `'memory_limit'`）
- `$value`：配置值（如 `'256M'`）

**返回值**：成功返回旧值，失败返回 `false`。

### ini_get() 函数

**语法**：`ini_get(string $option): string|false`

**参数**：
- `$option`：配置项名称（如 `'memory_limit'`）

**返回值**：成功返回配置值，失败返回 `false`。

## 基本用法

### 示例 1：获取当前内存限制

```php
<?php
declare(strict_types=1);

// 获取当前内存限制
$limit = ini_get('memory_limit');
echo "当前内存限制: {$limit}\n";

// 解析内存限制值
function parseMemoryLimit(string $limit): int
{
    $limit = trim($limit);
    if ($limit === '-1') {
        return -1;  // 无限制
    }
    
    $last = strtolower($limit[strlen($limit) - 1]);
    $value = (int)$limit;
    
    return match ($last) {
        'g' => $value * 1024 * 1024 * 1024,
        'm' => $value * 1024 * 1024,
        'k' => $value * 1024,
        default => $value,
    };
}

$limitBytes = parseMemoryLimit($limit);
if ($limitBytes === -1) {
    echo "内存限制: 无限制\n";
} else {
    echo "内存限制: " . formatBytes($limitBytes) . "\n";
}

function formatBytes(int $bytes): string
{
    $units = ['B', 'KB', 'MB', 'GB'];
    $bytes = max($bytes, 0);
    $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
    $pow = min($pow, count($units) - 1);
    $bytes /= (1 << (10 * $pow));
    
    return round($bytes, 2) . ' ' . $units[$pow];
}
```

**说明**：
- `ini_get('memory_limit')` 获取当前内存限制
- 返回值可能是字符串格式（如 `'256M'`），需要解析
- `-1` 表示无限制

### 示例 2：设置内存限制

```php
<?php
declare(strict_types=1);

// 获取原始限制
$originalLimit = ini_get('memory_limit');
echo "原始内存限制: {$originalLimit}\n";

// 设置新的内存限制
$newLimit = '512M';
$result = ini_set('memory_limit', $newLimit);
if ($result !== false) {
    echo "内存限制已设置为: {$newLimit}\n";
    echo "原有限制: {$result}\n";
} else {
    echo "无法设置内存限制\n";
}

// 验证设置
$currentLimit = ini_get('memory_limit');
echo "当前内存限制: {$currentLimit}\n";
```

**说明**：
- `ini_set()` 在运行时设置内存限制
- 返回之前的限制值
- 设置失败返回 `false`

### 示例 3：不同单位的内存限制

```php
<?php
declare(strict_types=1);

// 使用不同单位设置内存限制
$limits = [
    '128M',   // 128 MB
    '256M',   // 256 MB
    '512M',   // 512 MB
    '1G',     // 1 GB
    '2048K',  // 2048 KB
    '-1',     // 无限制
];

foreach ($limits as $limit) {
    if (ini_set('memory_limit', $limit) !== false) {
        echo "设置成功: {$limit}\n";
        echo "当前限制: " . ini_get('memory_limit') . "\n";
    } else {
        echo "设置失败: {$limit}\n";
    }
    echo "\n";
}
```

**说明**：
- 支持 K（KB）、M（MB）、G（GB）等单位
- `-1` 表示无限制
- 也可以使用纯数字（字节）

### 示例 4：临时提高内存限制

```php
<?php
declare(strict_types=1);

function withIncreasedMemoryLimit(string $newLimit, callable $callback): mixed
{
    // 保存原始限制
    $originalLimit = ini_get('memory_limit');
    
    // 设置新限制
    ini_set('memory_limit', $newLimit);
    
    try {
        // 执行回调
        return $callback();
    } finally {
        // 恢复原始限制
        ini_set('memory_limit', $originalLimit);
    }
}

// 使用：临时提高内存限制执行操作
$result = withIncreasedMemoryLimit('512M', function () {
    // 需要更多内存的操作
    $largeArray = range(1, 1000000);
    return count($largeArray);
});

echo "结果: {$result}\n";
echo "当前限制: " . ini_get('memory_limit') . "\n";
```

**说明**：
- 临时提高内存限制执行操作
- 使用 try-finally 确保恢复原始限制

### 示例 5：检查内存限制是否足够

```php
<?php
declare(strict_types=1);

function checkMemoryLimit(int $requiredMB): bool
{
    $limitStr = ini_get('memory_limit');
    if ($limitStr === '-1') {
        return true;  // 无限制，总是足够
    }
    
    $limitBytes = parseMemoryLimit($limitStr);
    $requiredBytes = $requiredMB * 1024 * 1024;
    
    return $limitBytes >= $requiredBytes;
}

function parseMemoryLimit(string $limit): int
{
    $limit = trim($limit);
    if ($limit === '-1') {
        return -1;
    }
    
    $last = strtolower($limit[strlen($limit) - 1]);
    $value = (int)$limit;
    
    return match ($last) {
        'g' => $value * 1024 * 1024 * 1024,
        'm' => $value * 1024 * 1024,
        'k' => $value * 1024,
        default => $value,
    };
}

// 使用
if (!checkMemoryLimit(256)) {
    echo "警告: 内存限制可能不足 256MB\n";
    // 尝试提高限制
    ini_set('memory_limit', '512M');
}
```

**说明**：
- 检查内存限制是否满足需求
- 根据需求动态调整限制

## 使用场景

### 场景 1：处理大数据

处理大文件或大数据集时，需要提高内存限制。

**示例**：

```php
<?php
declare(strict_types=1);

// 处理大文件前提高内存限制
ini_set('memory_limit', '512M');

// 处理大文件
$data = file_get_contents(__DIR__ . '/large-file.txt');
// 处理数据...
```

### 场景 2：内存密集型操作

执行内存密集型操作时，临时提高内存限制。

**示例**：见"示例 4：临时提高内存限制"

## 注意事项

### 限制设置的权限

某些配置项可能被 `php.ini` 限制，无法通过 `ini_set()` 修改。

**示例**：

```php
<?php
declare(strict_types=1);

$result = ini_set('memory_limit', '1024M');
if ($result === false) {
    echo "无法设置内存限制（可能被 php.ini 限制）\n";
}
```

### 系统内存限制

PHP 的内存限制不能超过系统的可用内存。

**示例**：

```php
<?php
declare(strict_types=1);

// 即使设置了很大的限制，也受系统内存限制
ini_set('memory_limit', '10G');  // 如果系统内存不足，仍然可能失败
```

### 安全考虑

设置过大的内存限制可能导致安全问题（内存耗尽攻击）。

**示例**：

```php
<?php
declare(strict_types=1);

// 在生产环境，应该设置合理的限制
// 不推荐设置为 -1（无限制）
// ini_set('memory_limit', '-1');  // 不推荐
```

### 配置优先级

不同配置方式的优先级不同：`.htaccess` > `ini_set()` > `php.ini`。

**示例**：

```php
<?php
declare(strict_types=1);

// 优先级顺序（从高到低）：
// 1. .htaccess（Web 环境）
// 2. ini_set()（运行时设置）
// 3. php.ini（全局配置）
```

## 常见问题

### 问题 1：如何设置内存限制？

**回答**：可以在 `php.ini` 中设置 `memory_limit`，或使用 `ini_set()` 在运行时设置。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法1：php.ini
// memory_limit = 256M

// 方法2：运行时设置
ini_set('memory_limit', '256M');
```

### 问题 2：内存限制的单位是什么？

**回答**：支持字节（纯数字）、K（KB）、M（MB）、G（GB）等单位。

**示例**：

```php
<?php
declare(strict_types=1);

ini_set('memory_limit', '256M');  // 256 MB
ini_set('memory_limit', '1G');    // 1 GB
ini_set('memory_limit', '2048K'); // 2048 KB
ini_set('memory_limit', '268435456'); // 字节
```

### 问题 3：如何设置无限制？

**回答**：设置值为 `-1` 表示无限制（不推荐用于生产环境）。

**示例**：

```php
<?php
declare(strict_types=1);

ini_set('memory_limit', '-1');  // 无限制（不推荐）
```

### 问题 4：限制设置的优先级？

**回答**：`.htaccess`（Web 环境）> `ini_set()`（运行时）> `php.ini`（全局配置）。

## 最佳实践

### 1. 根据实际需求设置限制

根据脚本的实际内存需求设置合适的限制，不要设置过大。

**示例**：

```php
<?php
declare(strict_types=1);

// 根据操作类型设置限制
if ($operation === 'large_file_processing') {
    ini_set('memory_limit', '512M');
} else {
    ini_set('memory_limit', '128M');
}
```

### 2. 避免设置过大的限制

设置过大的限制可能导致资源浪费和安全问题。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：根据实际需求设置
ini_set('memory_limit', '256M');

// ❌ 不推荐：设置过大
// ini_set('memory_limit', '10G');

// ❌ 不推荐：生产环境无限制
// ini_set('memory_limit', '-1');
```

### 3. 使用 ini_set() 临时调整

对于特定操作，使用 `ini_set()` 临时提高限制，操作后恢复。

**示例**：见"示例 4：临时提高内存限制"

### 4. 监控内存使用情况

设置内存限制后，监控实际的内存使用情况。

**示例**：

```php
<?php
declare(strict_types=1);

ini_set('memory_limit', '256M');

// 监控内存使用
$usage = memory_get_usage();
$limit = parseMemoryLimit(ini_get('memory_limit'));
$percent = ($usage / $limit) * 100;

if ($percent > 80) {
    echo "警告: 内存使用超过 80%\n";
}
```

## 对比分析

### php.ini vs ini_set()

| 特性         | php.ini                     | ini_set()                   |
|:-------------|:----------------------------|:----------------------------|
| **作用范围** | 全局配置                    | 当前脚本                    |
| **持久性**   | ✅ 永久生效                 | ⚠️ 脚本结束后失效           |
| **优先级**   | ⚠️ 较低                     | ✅ 较高                     |
| **适用场景** | 全局默认配置                | 脚本特定需求                |

### 不同配置方式的优先级

| 配置方式     | 优先级 | 说明                        |
|:-------------|:-------|:----------------------------|
| **.htaccess**| 最高   | Web 环境，仅 Apache         |
| **ini_set()**| 中等   | 运行时设置，当前脚本        |
| **php.ini**  | 最低   | 全局配置，所有脚本          |

## 练习任务

1. **内存限制管理工具**：创建一个工具类，封装内存限制的获取和设置功能。

2. **内存限制检查工具**：实现一个工具，检查内存限制是否满足需求。

3. **动态内存限制调整工具**：创建一个工具，根据操作需求动态调整内存限制。

4. **内存限制配置分析工具**：编写一个工具，分析不同配置方式的内存限制设置。

5. **内存限制最佳实践工具**：实现一个工具，根据脚本特征推荐合适的内存限制值。

## 相关章节

- **[4.5.1 内存使用监控](section-01-memory-monitoring.md)**：了解内存监控的方法
- **[4.5.4 内存优化实践](section-04-memory-optimization.md)**：学习内存优化方法
- **[1.5.1 php.ini 配置](../../stage-01-foundation/chapter-05-config/section-01-php-ini.md)**：了解 php.ini 配置的相关内容

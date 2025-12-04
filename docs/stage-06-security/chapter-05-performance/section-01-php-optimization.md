# 6.5.1 PHP 性能优化

## 概述

PHP 性能优化是提升应用响应速度和吞吐量的关键。本节介绍性能优化的基本原则、代码优化技巧、内存优化方法，以及性能测试工具。

## 性能优化原则

### 1. 测量优先

在优化之前，先测量性能瓶颈：

```php
<?php
declare(strict_types=1);

// 使用 microtime 测量执行时间
$start = microtime(true);

// 执行代码
for ($i = 0; $i < 1000000; $i++) {
    // ...
}

$end = microtime(true);
$executionTime = $end - $start;
echo "Execution time: {$executionTime} seconds\n";
```

### 2. 优化热点代码

使用 Xdebug 或 Xhprof 找出性能瓶颈：

```php
<?php
declare(strict_types=1);

// 使用 Xhprof
xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);

// 执行代码
// ...

$data = xhprof_disable();
include_once '/path/to/xhprof_lib/utils/xhprof_lib.php';
include_once '/path/to/xhprof_lib/utils/xhprof_runs.php';

$xhprof_runs = new XHProfRuns_Default();
$run_id = $xhprof_runs->save_run($data, "xhprof_test");
```

### 3. 避免过早优化

只优化真正影响性能的代码。

## 代码优化

### 1. 使用合适的函数

```php
<?php
declare(strict_types=1);

// 不推荐：使用 count() 在循环中
for ($i = 0; $i < count($array); $i++) {
    // ...
}

// 推荐：预先计算
$count = count($array);
for ($i = 0; $i < $count; $i++) {
    // ...
}

// 更推荐：使用 foreach
foreach ($array as $item) {
    // ...
}
```

### 2. 避免不必要的函数调用

```php
<?php
declare(strict_types=1);

// 不推荐：重复调用函数
if (strlen($string) > 10 && strlen($string) < 100) {
    // ...
}

// 推荐：缓存结果
$length = strlen($string);
if ($length > 10 && $length < 100) {
    // ...
}
```

### 3. 使用类型提示

```php
<?php
declare(strict_types=1);

// 类型提示帮助 PHP 优化
function processUser(User $user): void
{
    // PHP 不需要类型检查
}
```

### 4. 避免使用魔术方法

```php
<?php
declare(strict_types=1);

// 不推荐：使用 __get
class User
{
    private array $data = [];
    
    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }
}

// 推荐：直接访问属性
class User
{
    public function __construct(
        public string $name,
        public string $email,
    ) {}
}
```

### 5. 使用生成器处理大数组

```php
<?php
declare(strict_types=1);

// 不推荐：加载所有数据到内存
function getAllUsers(): array
{
    $users = [];
    $stmt = $pdo->query("SELECT * FROM users");
    while ($row = $stmt->fetch()) {
        $users[] = $row;
    }
    return $users;
}

// 推荐：使用生成器
function getAllUsers(): Generator
{
    $stmt = $pdo->query("SELECT * FROM users");
    while ($row = $stmt->fetch()) {
        yield $row;
    }
}

// 使用
foreach (getAllUsers() as $user) {
    // 处理单个用户，内存占用小
}
```

## 内存优化

### 1. 及时释放大变量

```php
<?php
declare(strict_types=1);

// 处理大文件
$largeData = file_get_contents('large_file.txt');
processData($largeData);
unset($largeData); // 及时释放内存
```

### 2. 使用流处理大文件

```php
<?php
declare(strict_types=1);

// 不推荐：一次性读取
$content = file_get_contents('large_file.txt');

// 推荐：流式处理
$handle = fopen('large_file.txt', 'r');
while (($line = fgets($handle)) !== false) {
    processLine($line);
}
fclose($handle);
```

### 3. 避免循环引用

```php
<?php
declare(strict_types=1);

// 避免循环引用
class Node
{
    public ?Node $parent = null;
    public array $children = [];
    
    public function addChild(Node $child): void
    {
        $this->children[] = $child;
        $child->parent = $this;
    }
    
    public function __destruct()
    {
        // 清理引用
        $this->parent = null;
        $this->children = [];
    }
}
```

## PHP-FPM 优化

### 进程池配置

```ini
; php-fpm.conf
[www]
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 1000
```

### 慢日志配置

```ini
; php-fpm.conf
slowlog = /var/log/php-fpm/slow.log
request_slowlog_timeout = 5s
```

## 性能测试

### 基准测试类

```php
<?php
declare(strict_types=1);

class Benchmark
{
    private array $results = [];
    
    public function measure(string $name, callable $callback): float
    {
        $start = microtime(true);
        $startMemory = memory_get_usage();
        
        $callback();
        
        $end = microtime(true);
        $endMemory = memory_get_usage();
        
        $time = $end - $start;
        $memory = $endMemory - $startMemory;
        
        $this->results[$name] = [
            'time' => $time,
            'memory' => $memory,
        ];
        
        return $time;
    }
    
    public function compare(array $tests): void
    {
        foreach ($tests as $name => $callback) {
            $this->measure($name, $callback);
        }
        
        $this->report();
    }
    
    public function report(): void
    {
        echo "Benchmark Results:\n";
        echo str_repeat("-", 60) . "\n";
        
        uasort($this->results, fn($a, $b) => $a['time'] <=> $b['time']);
        
        foreach ($this->results as $name => $result) {
            printf(
                "%-30s Time: %8.4fs  Memory: %10s\n",
                $name,
                $result['time'],
                $this->formatBytes($result['memory'])
            );
        }
    }
    
    private function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB'];
        $i = 0;
        while ($bytes >= 1024 && $i < count($units) - 1) {
            $bytes /= 1024;
            $i++;
        }
        return round($bytes, 2) . ' ' . $units[$i];
    }
}

// 使用示例
$benchmark = new Benchmark();

$benchmark->compare([
    'Array iteration' => function() {
        $array = range(1, 100000);
        foreach ($array as $value) {
            // ...
        }
    },
    'Generator iteration' => function() {
        $generator = function() {
            for ($i = 1; $i <= 100000; $i++) {
                yield $i;
            }
        };
        foreach ($generator() as $value) {
            // ...
        }
    },
]);
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 性能优化示例：用户列表处理
class UserService
{
    private PDO $db;
    
    public function __construct(PDO $db)
    {
        $this->db = $db;
    }
    
    // 优化前：加载所有用户到内存
    public function getAllUsersOld(): array
    {
        $stmt = $this->db->query("SELECT * FROM users");
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    // 优化后：使用生成器
    public function getAllUsers(): Generator
    {
        $stmt = $this->db->query("SELECT * FROM users");
        while ($user = $stmt->fetch(PDO::FETCH_ASSOC)) {
            yield $user;
        }
    }
    
    // 优化：批量处理
    public function processUsersBatch(int $batchSize = 1000): void
    {
        $offset = 0;
        
        while (true) {
            $stmt = $this->db->prepare("SELECT * FROM users LIMIT ? OFFSET ?");
            $stmt->execute([$batchSize, $offset]);
            $users = $stmt->fetchAll(PDO::FETCH_ASSOC);
            
            if (empty($users)) {
                break;
            }
            
            foreach ($users as $user) {
                $this->processUser($user);
            }
            
            $offset += $batchSize;
            
            // 释放内存
            unset($users);
        }
    }
    
    private function processUser(array $user): void
    {
        // 处理用户
    }
}
```

## 最佳实践

1. **测量优先**：使用工具找出性能瓶颈
2. **优化热点**：只优化真正影响性能的代码
3. **使用生成器**：处理大数据集时使用生成器
4. **及时释放**：及时释放大变量
5. **配置优化**：优化 PHP-FPM 配置

## 注意事项

1. 避免过早优化
2. 保持代码可读性
3. 定期进行性能测试
4. 监控生产环境性能

## 练习

1. 创建一个性能测试工具，比较不同实现方式的性能。

2. 优化一个处理大文件的函数，使用流式处理减少内存占用。

3. 使用生成器重构一个返回大量数据的函数。

4. 配置 PHP-FPM，优化进程池设置。

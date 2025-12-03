# 1.7.4 PHP 8 Fibers

## 概述

Fibers 是 PHP 8.1+ 引入的轻量级协程实现。它提供了原生的协程支持，无需扩展即可使用。

## 什么是 Fibers

### 基本概念

- **Fibers**：PHP 8.1+ 引入的轻量级协程实现
- **特点**：原生支持，无需扩展
- **类比**：类似 JavaScript 的 Generator，但更强大

### 与 Node.js Promise/async-await 对比

如果你熟悉 Node.js 的 Promise 和 async-await，理解 PHP 异步处理的关键差异：

**Node.js 异步模型**：
```javascript
// Promise
fetch('https://api.example.com/users/1')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));

// async/await
async function fetchUser(id) {
    try {
        const response = await fetch(`https://api.example.com/users/${id}`);
        const data = await response.json();
        return data;
    } catch (error) {
        console.error(error);
    }
}
```

**PHP 异步模型**：
```php
<?php
// 传统同步方式（阻塞）
$response = file_get_contents('https://api.example.com/users/1');
$data = json_decode($response, true);

// 使用协程（非阻塞，类似 async/await）
use OpenSwoole\Coroutine;

Coroutine::create(function () {
    $response = Coroutine\Http\get('https://api.example.com/users/1');
    $data = json_decode($response->getBody(), true);
    return $data;
});
```

**关键差异**：

| 特性 | Node.js | PHP (传统) | PHP (协程) |
| :--- | :------ | :--------- | :---------- |
| 异步模型 | Promise/async-await | 同步阻塞 | 协程（OpenSwoole/Fibers） |
| 非阻塞 I/O | 原生支持 | 不支持 | 需要协程运行时 |
| 并发处理 | 事件循环 | 多进程 | 协程池 |
| 语法 | `async/await` | 同步语法 | 协程语法 |

## Fibers 基础

### 基本语法

```php
<?php
declare(strict_types=1);

$fiber = new Fiber(function (): void {
    echo "Fiber started\n";
    
    // 暂停执行
    Fiber::suspend('First suspend');
    
    echo "Fiber resumed\n";
    
    // 再次暂停
    Fiber::suspend('Second suspend');
    
    echo "Fiber finished\n";
});

// 启动 Fiber
$value = $fiber->start();
echo "Suspended with: {$value}\n";

// 恢复执行
$value = $fiber->resume();
echo "Resumed with: {$value}\n";

// 再次恢复
$fiber->resume();
```

### Fiber 方法

| 方法 | 说明 | 示例 |
| :--- | :--- | :--- |
| `start()` | 启动 Fiber | `$fiber->start()` |
| `resume(mixed $value)` | 恢复执行 | `$fiber->resume($value)` |
| `suspend(mixed $value)` | 暂停执行 | `Fiber::suspend($value)` |
| `isStarted()` | 是否已启动 | `$fiber->isStarted()` |
| `isSuspended()` | 是否已暂停 | `$fiber->isSuspended()` |
| `isRunning()` | 是否运行中 | `$fiber->isRunning()` |
| `isTerminated()` | 是否已终止 | `$fiber->isTerminated()` |

## Fibers 实际应用

### 异步 HTTP 客户端

```php
<?php
declare(strict_types=1);

class AsyncHttpClient
{
    private array $fibers = [];
    
    public function get(string $url): string
    {
        $fiber = new Fiber(function () use ($url): string {
            // 模拟异步 HTTP 请求
            $ch = curl_init($url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            
            // 暂停，等待响应
            Fiber::suspend();
            
            $response = curl_exec($ch);
            curl_close($ch);
            
            return $response;
        });
        
        $this->fibers[] = $fiber;
        $fiber->start();
        
        // 返回一个占位符，实际数据稍后获取
        return "pending";
    }
    
    public function waitAll(): array
    {
        $results = [];
        
        foreach ($this->fibers as $fiber) {
            if ($fiber->isSuspended()) {
                // 模拟响应到达，恢复 Fiber
                $results[] = $fiber->resume();
            }
        }
        
        return $results;
    }
}
```

### 任务调度器

```php
<?php
declare(strict_types=1);

class TaskScheduler
{
    private array $tasks = [];
    
    public function addTask(callable $task): void
    {
        $fiber = new Fiber(function () use ($task) {
            return $task();
        });
        
        $this->tasks[] = $fiber;
    }
    
    public function run(): array
    {
        $results = [];
        
        // 启动所有任务
        foreach ($this->tasks as $fiber) {
            $fiber->start();
        }
        
        // 等待所有任务完成
        while (!empty($this->tasks)) {
            foreach ($this->tasks as $key => $fiber) {
                if ($fiber->isSuspended()) {
                    // 恢复执行
                    $fiber->resume();
                }
                
                if ($fiber->isTerminated()) {
                    $results[] = $fiber->getReturn();
                    unset($this->tasks[$key]);
                }
            }
        }
        
        return $results;
    }
}
```

## Fibers 与生成器

### 使用生成器实现类似效果

```php
<?php
declare(strict_types=1);

// 使用生成器实现类似效果
function asyncTask(): Generator
{
    yield 'Task 1';
    yield 'Task 2';
    yield 'Task 3';
}

$gen = asyncTask();
foreach ($gen as $value) {
    echo $value . "\n";
}
```

### 使用 Fibers 实现

```php
<?php
declare(strict_types=1);

// 使用 Fibers 实现
$fiber = new Fiber(function (): void {
    echo "Task 1\n";
    Fiber::suspend();
    echo "Task 2\n";
    Fiber::suspend();
    echo "Task 3\n";
});

$fiber->start();
$fiber->resume();
$fiber->resume();
```

### 对比

| 特性 | Generator | Fiber |
| :--- | :--------- | :---- |
| 返回值 | 只能 yield 值 | 可以返回值 |
| 双向通信 | 只能 yield 值 | 可以传递值 |
| 异常处理 | 基础 | 完整 |
| 嵌套支持 | 有限 | 完整 |

## 完整示例

```php
<?php
declare(strict_types=1);

class FibersExample
{
    public static function demonstrate(): void
    {
        echo "=== Basic Fiber ===\n";
        $fiber = new Fiber(function (): void {
            echo "Fiber started\n";
            Fiber::suspend('First');
            echo "Fiber resumed\n";
            Fiber::suspend('Second');
            echo "Fiber finished\n";
        });
        
        echo "Start: " . $fiber->start() . "\n";
        echo "Resume 1: " . $fiber->resume() . "\n";
        $fiber->resume();
        
        echo "\n=== Task Scheduler ===\n";
        $scheduler = new TaskScheduler();
        $scheduler->addTask(fn() => 'Task 1');
        $scheduler->addTask(fn() => 'Task 2');
        $scheduler->addTask(fn() => 'Task 3');
        
        $results = $scheduler->run();
        print_r($results);
    }
}
```

## 注意事项

1. **运行时支持**：Fibers 需要 PHP 8.1+，但实际异步 I/O 仍需要 OpenSwoole 等扩展。

2. **调度器**：Fibers 本身不提供调度器，需要自己实现或使用框架。

3. **性能考虑**：Fibers 是轻量级的，但大量 Fibers 仍会影响性能。

4. **错误处理**：妥善处理 Fiber 中的异常。

5. **实际应用**：Fibers 主要用于框架和库的实现，而不是直接用于业务代码。

## 练习

1. 创建一个基于 Fibers 的异步任务系统。

2. 实现一个 Fiber 调度器，管理多个 Fiber 的执行。

3. 使用 Fibers 实现简单的协程功能。

4. 对比 Fibers 和 Generator 的差异。

5. 实现一个基于 Fibers 的 HTTP 客户端。

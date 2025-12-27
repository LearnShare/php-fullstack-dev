# 4.4.4 进程管理与守护进程

## 概述

进程管理是系统级编程的重要部分。在实际应用中，我们经常需要创建子进程、实现守护进程、处理信号等。PHP 通过 `pcntl` 扩展提供了进程管理功能，包括进程创建、信号处理等。理解进程管理的基本概念和方法，对于开发后台服务、任务队列等应用至关重要。

注意：`pcntl` 扩展只在 Unix/Linux/macOS 系统上可用，Windows 不支持。在生产环境中，通常使用 supervisor、systemd 等工具来管理进程。

**主要内容**：
- 进程概念和生命周期
- `pcntl_fork()` 函数（创建子进程）
- `pcntl_signal()` 函数（信号处理）
- `pcntl_waitpid()` 函数（等待子进程）
- 守护进程实现
- 僵尸进程处理
- 信号处理
- 实际应用场景和注意事项

## 特性

- **进程创建**：支持创建子进程
- **信号处理**：支持处理系统信号
- **进程控制**：支持等待子进程、获取进程状态等
- **守护进程**：支持实现守护进程
- **Unix 专用**：只在 Unix/Linux/macOS 上可用

## 语法/定义

### pcntl_fork() 函数

**语法**：`pcntl_fork(): int`

**参数**：无参数

**返回值**：
- 成功时，在父进程中返回子进程 ID，在子进程中返回 0
- 失败时返回 -1

**注意**：需要启用 `pcntl` 扩展。

### pcntl_signal() 函数

**语法**：`pcntl_signal(int $signal, callable|int $handler, bool $restart_syscalls = true): bool`

**参数**：
- `$signal`：信号编号（如 `SIGTERM`、`SIGINT`）
- `$handler`：信号处理函数或 `SIG_IGN`（忽略）、`SIG_DFL`（默认）
- `$restart_syscalls`：可选，是否重启系统调用

**返回值**：成功返回 `true`，失败返回 `false`。

### pcntl_waitpid() 函数

**语法**：`pcntl_waitpid(int $process_id, int &$status, int $flags = 0, array $resource_usage = []): int`

**参数**：
- `$process_id`：进程 ID（-1 表示等待任意子进程）
- `&$status`：用于接收进程状态
- `$flags`：可选，等待选项
- `$resource_usage`：可选，资源使用信息

**返回值**：成功返回进程 ID，失败返回 -1。

## 基本用法

### 示例 1：检查 pcntl 扩展

```php
<?php
declare(strict_types=1);

// 检查 pcntl 扩展是否可用
if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available. This feature only works on Unix/Linux/macOS.\n");
}

echo "pcntl extension is loaded\n";
```

**说明**：
- `pcntl` 扩展只在 Unix/Linux/macOS 上可用
- 在使用前应该检查扩展是否可用

### 示例 2：创建子进程

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available\n");
}

$pid = pcntl_fork();

if ($pid == -1) {
    // 创建失败
    die('进程创建失败\n');
} elseif ($pid) {
    // 父进程
    echo "父进程: 子进程 PID = {$pid}\n";
    echo "父进程: 当前 PID = " . getmypid() . "\n";
    
    // 等待子进程结束
    pcntl_waitpid($pid, $status);
    echo "子进程已结束\n";
} else {
    // 子进程
    echo "子进程: 当前 PID = " . getmypid() . "\n";
    echo "子进程: 父进程 PID = " . getmypid() . "\n";
    sleep(2);
    echo "子进程结束\n";
    exit(0);
}
```

**说明**：
- `pcntl_fork()` 创建子进程
- 父进程中返回子进程 ID（> 0）
- 子进程中返回 0
- 失败返回 -1
- 父进程应该等待子进程结束，避免僵尸进程

### 示例 3：处理信号

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available\n");
}

$running = true;

// 信号处理函数
function signalHandler(int $signo): void
{
    global $running;
    
    match ($signo) {
        SIGTERM, SIGINT => {
            echo "\n收到终止信号，正在关闭...\n";
            $running = false;
        },
        SIGHUP => {
            echo "\n收到挂起信号，重新加载配置...\n";
            // 重新加载配置
        },
        default => echo "收到未知信号: {$signo}\n",
    };
}

// 注册信号处理器
pcntl_signal(SIGTERM, 'signalHandler');
pcntl_signal(SIGINT, 'signalHandler');
pcntl_signal(SIGHUP, 'signalHandler');

echo "进程 PID: " . getmypid() . "\n";
echo "按 Ctrl+C 或发送 SIGTERM 信号终止进程\n";

// 主循环
while ($running) {
    // 处理待处理的信号
    pcntl_signal_dispatch();
    
    // 执行任务
    echo "运行中...\n";
    sleep(1);
}

echo "进程已关闭\n";
```

**说明**：
- `pcntl_signal()` 注册信号处理器
- 需要定期调用 `pcntl_signal_dispatch()` 处理信号
- 常见信号：`SIGTERM`（终止）、`SIGINT`（中断，Ctrl+C）、`SIGHUP`（挂起）

### 示例 4：等待子进程（避免僵尸进程）

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available\n");
}

// 创建多个子进程
for ($i = 0; $i < 3; $i++) {
    $pid = pcntl_fork();
    
    if ($pid == -1) {
        die("进程创建失败\n");
    } elseif ($pid) {
        // 父进程
        echo "创建子进程: {$pid}\n";
    } else {
        // 子进程
        $childPid = getmypid();
        echo "子进程 {$childPid} 开始执行\n";
        sleep(2);
        echo "子进程 {$childPid} 执行完成\n";
        exit(0);
    }
}

// 父进程等待所有子进程
while (($pid = pcntl_waitpid(-1, $status)) > 0) {
    $exitCode = pcntl_wexitstatus($status);
    echo "子进程 {$pid} 已结束，退出码: {$exitCode}\n";
}

echo "所有子进程已完成\n";
```

**说明**：
- `pcntl_waitpid(-1, $status)` 等待任意子进程结束
- 返回已结束的子进程 ID
- 使用 `pcntl_wexitstatus()` 获取退出码
- 父进程必须等待子进程，避免僵尸进程

### 示例 5：简单的守护进程实现

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available\n");
}

function daemonize(): void
{
    // 1. 创建子进程
    $pid = pcntl_fork();
    if ($pid < 0) {
        die("第一次 fork 失败\n");
    } elseif ($pid > 0) {
        // 父进程退出
        exit(0);
    }
    
    // 2. 成为会话领导者
    if (posix_setsid() < 0) {
        die("setsid 失败\n");
    }
    
    // 3. 第二次 fork（防止获取控制终端）
    $pid = pcntl_fork();
    if ($pid < 0) {
        die("第二次 fork 失败\n");
    } elseif ($pid > 0) {
        // 父进程退出
        exit(0);
    }
    
    // 4. 改变工作目录
    chdir('/');
    
    // 5. 关闭文件描述符
    fclose(STDIN);
    fclose(STDOUT);
    fclose(STDERR);
    
    // 重新打开标准文件描述符到 /dev/null
    $stdin = fopen('/dev/null', 'r');
    $stdout = fopen('/dev/null', 'w');
    $stderr = fopen('/dev/null', 'w');
}

// 转换为守护进程
daemonize();

// 写入 PID 文件
$pidFile = '/tmp/daemon.pid';
file_put_contents($pidFile, getmypid());

// 主循环
$running = true;

pcntl_signal(SIGTERM, function () use (&$running) {
    global $running;
    $running = false;
});

while ($running) {
    pcntl_signal_dispatch();
    
    // 执行任务
    // ...
    
    sleep(1);
}

// 清理
unlink($pidFile);
```

**说明**：
- 守护进程实现步骤：fork、setsid、再次 fork、改变工作目录、关闭文件描述符
- 守护进程在后台运行，不受终端影响

## 使用场景

### 场景 1：后台任务处理

创建后台进程处理任务。

**示例**：

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available\n");
}

$pid = pcntl_fork();

if ($pid == -1) {
    die("进程创建失败\n");
} elseif ($pid) {
    // 父进程
    echo "任务已在后台运行，PID: {$pid}\n";
    exit(0);
} else {
    // 子进程（后台任务）
    // 执行长时间运行的任务
    while (true) {
        // 处理任务
        sleep(1);
    }
}
```

### 场景 2：多进程任务处理

使用多个进程并行处理任务。

**示例**：

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available\n");
}

$tasks = ['task1', 'task2', 'task3', 'task4', 'task5'];
$workers = 3;  // 工作进程数

for ($i = 0; $i < $workers; $i++) {
    $pid = pcntl_fork();
    
    if ($pid == -1) {
        die("进程创建失败\n");
    } elseif ($pid == 0) {
        // 子进程（工作进程）
        $workerId = $i;
        echo "工作进程 {$workerId} 启动\n";
        
        // 处理分配给该工作进程的任务
        foreach ($tasks as $index => $task) {
            if ($index % $workers == $workerId) {
                echo "工作进程 {$workerId} 处理任务: {$task}\n";
                sleep(1);  // 模拟处理
            }
        }
        
        exit(0);
    }
}

// 父进程等待所有子进程
while (pcntl_waitpid(-1, $status) > 0) {
    // 等待子进程
}

echo "所有任务已完成\n";
```

## 注意事项

### pcntl 扩展的可用性

`pcntl` 扩展只在 Unix/Linux/macOS 上可用，Windows 不支持。

**示例**：

```php
<?php
declare(strict_types=1);

if (!extension_loaded('pcntl')) {
    die("pcntl extension is not available on this system\n");
}
```

### 僵尸进程处理

父进程必须等待子进程结束，避免僵尸进程。

**示例**：见"示例 4：等待子进程（避免僵尸进程）"

### 信号处理的安全性

信号处理函数应该尽量简单，避免在信号处理函数中执行复杂操作。

**示例**：

```php
<?php
declare(strict_types=1);

$shutdown = false;

pcntl_signal(SIGTERM, function () use (&$shutdown) {
    // 只设置标志，不在信号处理函数中执行复杂操作
    $shutdown = true;
});

while (!$shutdown) {
    pcntl_signal_dispatch();
    // 在主循环中检查标志并执行关闭操作
    // ...
}
```

## 常见问题

### 问题 1：如何创建子进程？

**回答**：使用 `pcntl_fork()` 函数，根据返回值判断是父进程还是子进程。

**示例**：见"示例 2：创建子进程"

### 问题 2：如何实现守护进程？

**回答**：按照标准步骤：fork、setsid、再次 fork、改变工作目录、关闭文件描述符。

**示例**：见"示例 5：简单的守护进程实现"

### 问题 3：如何处理进程信号？

**回答**：使用 `pcntl_signal()` 注册信号处理器，定期调用 `pcntl_signal_dispatch()` 处理信号。

**示例**：见"示例 3：处理信号"

### 问题 4：如何避免僵尸进程？

**回答**：父进程使用 `pcntl_waitpid()` 等待子进程结束。

**示例**：见"示例 4：等待子进程（避免僵尸进程）"

## 最佳实践

### 1. 检查 pcntl 扩展可用性

在使用 pcntl 功能前，检查扩展是否可用。

**示例**：见"示例 1：检查 pcntl 扩展"

### 2. 正确处理父子进程

根据 `pcntl_fork()` 的返回值正确区分父进程和子进程。

**示例**：见"示例 2：创建子进程"

### 3. 实现优雅关闭机制

使用信号处理实现优雅关闭。

**示例**：见"示例 3：处理信号"

### 4. 等待子进程结束

父进程应该等待子进程结束，避免僵尸进程。

**示例**：见"示例 4：等待子进程（避免僵尸进程）"

### 5. 使用进程管理工具

在生产环境，考虑使用 supervisor、systemd 等工具管理进程，而不是直接使用 pcntl 实现守护进程。

**示例**：

```bash
# 使用 supervisor 管理进程
# supervisor 配置文件示例
[program:myapp]
command=php /path/to/script.php
autostart=true
autorestart=true
```

## 对比分析

### pcntl vs 进程管理工具

| 特性         | pcntl                       | supervisor/systemd          |
|:-------------|:----------------------------|:----------------------------|
| **控制力**   | ✅ 完全控制                  | ⚠️ 受限于工具功能            |
| **复杂性**   | ⚠️ 需要手动实现              | ✅ 配置简单                  |
| **可靠性**   | ⚠️ 需要处理各种边界情况      | ✅ 经过验证，更可靠          |
| **适用场景** | 需要精细控制的场景          | 生产环境推荐                 |

## 练习任务

1. **进程管理工具类**：创建一个工具类，封装进程创建和管理功能。

2. **守护进程实现**：实现一个完整的守护进程，包括信号处理、PID 文件管理等。

3. **多进程任务处理工具**：创建一个工具，使用多个进程并行处理任务。

4. **信号处理框架**：实现一个信号处理框架，方便注册和处理各种信号。

5. **进程监控工具**：创建一个工具，监控进程状态并在进程异常时重启。

## 相关章节

- **[4.4.2 执行外部命令](section-02-exec-commands.md)**：了解命令执行的相关内容
- **[4.4.7 CLI 最佳实践](section-07-best-practices.md)**：了解 CLI 最佳实践
- **[4.9.1 日志体系设计](../chapter-09-logging/section-01-logging-system.md)**：了解日志系统的相关内容

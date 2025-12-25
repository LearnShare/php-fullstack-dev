# 1.7.1 Worker 模式

## 概述

Worker 模式是常驻运行时的基础。理解 Worker 模式的工作原理对于使用现代 PHP 运行时至关重要。

## 什么是 Worker

### 基本概念

- **Worker**：工作进程，重复处理请求的进程
- **特点**：进程常驻内存，可以处理多个请求
- **优势**：避免每次请求都创建新进程的开销

### 传统模式 vs Worker 模式

```
传统模式（PHP-FPM）：
请求 1 → 创建进程 → 处理 → 销毁进程
请求 2 → 创建进程 → 处理 → 销毁进程
请求 3 → 创建进程 → 处理 → 销毁进程

Worker 模式：
启动 → 创建 Worker 进程池
请求 1 → Worker 1 处理
请求 2 → Worker 2 处理
请求 3 → Worker 1 处理（复用）
```

## Worker 实现示例

### 简单 Worker 实现

```php
<?php
declare(strict_types=1);

class SimpleWorker
{
    private $socket;
    private bool $running = true;
    
    public function __construct(string $address, int $port)
    {
        $this->socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
        socket_bind($this->socket, $address, $port);
        socket_listen($this->socket);
    }
    
    public function start(): void
    {
        echo "Worker started on 0.0.0.0:8080\n";
        
        while ($this->running) {
            $client = socket_accept($this->socket);
            
            if ($client !== false) {
                $this->handleRequest($client);
            }
        }
    }
    
    private function handleRequest($client): void
    {
        $request = socket_read($client, 1024);
        
        // 处理请求
        $response = "HTTP/1.1 200 OK\r\n";
        $response .= "Content-Type: text/plain\r\n";
        $response .= "\r\n";
        $response .= "Hello from Worker";
        
        socket_write($client, $response);
        socket_close($client);
    }
    
    public function stop(): void
    {
        $this->running = false;
        socket_close($this->socket);
    }
}

// 使用
$worker = new SimpleWorker('0.0.0.0', 8080);
$worker->start();
```

## Worker 进程池

### 进程池概念

```
Master 进程
├── Worker 1 (处理请求)
├── Worker 2 (处理请求)
├── Worker 3 (处理请求)
└── Worker 4 (处理请求)
```

### 进程池实现

```php
<?php
declare(strict_types=1);

class WorkerPool
{
    private array $workers = [];
    private int $poolSize;
    private bool $running = true;
    
    public function __construct(int $poolSize = 4)
    {
        $this->poolSize = $poolSize;
    }
    
    public function start(): void
    {
        // 创建 Worker 进程池
        for ($i = 0; $i < $this->poolSize; $i++) {
            $pid = pcntl_fork();
            
            if ($pid === 0) {
                // 子进程（Worker）
                $this->workerLoop();
                exit(0);
            } else {
                // 父进程（Master）
                $this->workers[] = $pid;
            }
        }
        
        // Master 进程等待
        while ($this->running) {
            pcntl_waitpid(-1, $status, WNOHANG);
            usleep(100000); // 100ms
        }
    }
    
    private function workerLoop(): void
    {
        echo "Worker " . getmypid() . " started\n";
        
        while ($this->running) {
            // 处理任务
            $task = $this->getTask();
            if ($task !== null) {
                $this->processTask($task);
            }
            
            usleep(10000); // 10ms
        }
    }
    
    private function getTask()
    {
        // 从队列获取任务
        // 实际实现需要使用消息队列或共享内存
        return null;
    }
    
    private function processTask($task): void
    {
        // 处理任务
        echo "Worker " . getmypid() . " processing task\n";
    }
    
    public function stop(): void
    {
        $this->running = false;
        foreach ($this->workers as $pid) {
            posix_kill($pid, SIGTERM);
        }
    }
}
```

## 连接复用

### 数据库连接复用

```php
<?php
declare(strict_types=1);

class WorkerWithConnection
{
    private ?PDO $db = null;
    
    public function start(): void
    {
        // 在 Worker 启动时创建连接
        $this->db = new PDO(
            'mysql:host=localhost;dbname=test',
            'user',
            'password'
        );
        
        while (true) {
            $request = $this->getRequest();
            if ($request !== null) {
                $this->handleRequest($request);
            }
        }
    }
    
    private function handleRequest($request): void
    {
        // 复用数据库连接
        $stmt = $this->db->query('SELECT * FROM users');
        $users = $stmt->fetchAll(PDO::FETCH_ASSOC);
        
        // 处理请求
        $this->sendResponse($users);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class WorkerExample
{
    public static function demonstrate(): void
    {
        echo "=== Simple Worker ===\n";
        $worker = new SimpleWorker('0.0.0.0', 8080);
        // $worker->start(); // 实际运行时会阻塞
        
        echo "\n=== Worker Pool ===\n";
        $pool = new WorkerPool(4);
        // $pool->start(); // 实际运行时会阻塞
    }
}
```

## 注意事项

1. **状态管理**：Worker 进程常驻内存，避免在 Worker 中存储请求间共享的状态。

2. **内存泄漏**：定期清理不需要的数据，防止内存泄漏。

3. **错误处理**：妥善处理异常，避免 Worker 崩溃。

4. **进程管理**：正确管理 Worker 进程的生命周期。

5. **资源清理**：在 Worker 退出时清理资源。

## 练习

1. 实现一个简单的 Worker 服务器，处理 HTTP 请求。

2. 创建一个 Worker 进程池，实现任务分发。

3. 实现连接复用，在 Worker 中复用数据库连接。

4. 实现 Worker 的健康检查和自动重启机制。

5. 对比 Worker 模式和传统模式的性能差异。

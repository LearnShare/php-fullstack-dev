# 3.3.2 接口（Interface）

## 概述

接口（Interface）定义了一组方法签名，作为类必须实现的契约。接口实现了多态和代码解耦。

## 基础语法

### 接口定义

- **语法**：`interface InterfaceName { ... }`
- 接口定义方法签名，不包含实现。
- 类使用 `implements` 关键字实现接口。

```php
interface Logger
{
    public function log(string $message, string $level = 'info'): void;
}

class FileLogger implements Logger
{
    public function log(string $message, string $level = 'info'): void
    {
        $line = sprintf("[%s] %s: %s\n", date('Y-m-d H:i:s'), strtoupper($level), $message);
        file_put_contents('app.log', $line, FILE_APPEND);
    }
}

class DatabaseLogger implements Logger
{
    public function log(string $message, string $level = 'info'): void
    {
        // 写入数据库的逻辑
        echo "Database log: [{$level}] {$message}\n";
    }
}
```

## 接口方法可见性

### 默认可见性

- 接口方法默认为 `public`，不能使用其他可见性修饰符。
- PHP 8.0+ 支持在接口中定义常量。

```php
interface PaymentGateway
{
    public const STATUS_PENDING = 'pending';
    public const STATUS_COMPLETED = 'completed';
    public const STATUS_FAILED = 'failed';

    public function processPayment(float $amount): bool;
    public function refund(string $transactionId): bool;
}
```

## 实现多个接口

### 多接口实现

- 一个类可以实现多个接口。

```php
interface Serializable
{
    public function serialize(): string;
}

interface Deserializable
{
    public static function deserialize(string $data): self;
}

class User implements Serializable, Deserializable
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function serialize(): string
    {
        return json_encode(['id' => $this->id, 'name' => $this->name]);
    }

    public static function deserialize(string $data): self
    {
        $decoded = json_decode($data, true);
        return new self($decoded['id'], $decoded['name']);
    }
}
```

## 接口继承

### 接口扩展

- 接口可以继承其他接口。

```php
interface Readable
{
    public function read(): string;
}

interface Writable
{
    public function write(string $data): void;
}

interface ReadWritable extends Readable, Writable
{
    // 继承了两个接口的所有方法
}

class FileHandler implements ReadWritable
{
    public function read(): string
    {
        return 'File content';
    }

    public function write(string $data): void
    {
        echo "Writing: {$data}\n";
    }
}
```

## 接口作为类型提示

### 多态实现

- 接口可以用作类型提示，实现多态。

```php
interface Notifier
{
    public function send(string $message): void;
}

class EmailNotifier implements Notifier
{
    public function send(string $message): void
    {
        echo "Sending email: {$message}\n";
    }
}

class SmsNotifier implements Notifier
{
    public function send(string $message): void
    {
        echo "Sending SMS: {$message}\n";
    }
}

class NotificationService
{
    public function __construct(
        private Notifier $notifier
    ) {
    }

    public function notify(string $message): void
    {
        $this->notifier->send($message);
    }
}

$emailService = new NotificationService(new EmailNotifier());
$emailService->notify('Hello via email');

$smsService = new NotificationService(new SmsNotifier());
$smsService->notify('Hello via SMS');
```

## 完整示例

```php
<?php
declare(strict_types=1);

interface Cache
{
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
    public function delete(string $key): void;
    public function clear(): void;
}

class FileCache implements Cache
{
    private string $directory;

    public function __construct(string $directory = '/tmp/cache')
    {
        $this->directory = $directory;
        if (!is_dir($directory)) {
            mkdir($directory, 0755, true);
        }
    }

    public function get(string $key): mixed
    {
        $file = $this->getFilePath($key);
        if (!file_exists($file)) {
            return null;
        }
        $data = unserialize(file_get_contents($file));
        if ($data['expires'] < time()) {
            $this->delete($key);
            return null;
        }
        return $data['value'];
    }

    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $file = $this->getFilePath($key);
        $data = [
            'value' => $value,
            'expires' => time() + $ttl
        ];
        file_put_contents($file, serialize($data));
    }

    public function delete(string $key): void
    {
        $file = $this->getFilePath($key);
        if (file_exists($file)) {
            unlink($file);
        }
    }

    public function clear(): void
    {
        $files = glob($this->directory . '/*');
        foreach ($files as $file) {
            if (is_file($file)) {
                unlink($file);
            }
        }
    }

    private function getFilePath(string $key): string
    {
        return $this->directory . '/' . md5($key) . '.cache';
    }
}

class MemoryCache implements Cache
{
    private array $cache = [];

    public function get(string $key): mixed
    {
        return $this->cache[$key] ?? null;
    }

    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $this->cache[$key] = $value;
    }

    public function delete(string $key): void
    {
        unset($this->cache[$key]);
    }

    public function clear(): void
    {
        $this->cache = [];
    }
}

// 使用接口实现多态
class CacheService
{
    public function __construct(
        private Cache $cache
    ) {
    }

    public function get(string $key): mixed
    {
        return $this->cache->get($key);
    }
}

$fileCache = new CacheService(new FileCache());
$memoryCache = new CacheService(new MemoryCache());
```

## 注意事项

1. **方法实现**：实现接口的类必须实现接口中定义的所有方法。

2. **方法签名**：实现的方法签名必须与接口定义完全匹配。

3. **多接口实现**：一个类可以实现多个接口，用逗号分隔。

4. **接口继承**：接口可以继承其他接口，形成接口层次结构。

5. **类型提示**：接口可以用作类型提示，实现依赖注入和多态。

## 练习

1. 定义一个 `Cache` 接口，包含 `get()`、`set()`、`delete()` 方法，然后实现 `FileCache` 和 `MemoryCache` 类。

2. 创建一个 `Repository` 接口，定义 `find()`、`save()`、`delete()` 方法，实现 `UserRepository` 和 `ProductRepository`。

3. 定义一个 `Logger` 接口，实现 `FileLogger`、`DatabaseLogger`、`ConsoleLogger`。

4. 创建一个 `PaymentMethod` 接口，实现 `CreditCardPayment` 和 `PayPalPayment`。

5. 实现一个 `Serializable` 接口，包含 `serialize()` 和 `deserialize()` 方法，让多个类实现该接口。

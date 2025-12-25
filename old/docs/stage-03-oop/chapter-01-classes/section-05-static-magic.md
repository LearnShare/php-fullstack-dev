# 3.1.5 静态属性与方法、魔术方法

## 概述

静态成员属于类而非对象实例，可以在不创建对象的情况下使用。魔术方法提供了对象行为的特殊钩子，允许自定义对象的行为。

## 静态属性与方法

### 静态属性

- **语法**：`public static $property;`
- 属于类而非对象，所有实例共享同一静态属性。
- 通过 `ClassName::$property` 访问。

```php
class Counter
{
    public static int $count = 0;

    public function __construct()
    {
        self::$count++;
    }

    public static function getCount(): int
    {
        return self::$count;
    }
}

new Counter();
new Counter();
echo Counter::$count; // 2
echo Counter::getCount(); // 2
```

### 静态方法

- **语法**：`public static function methodName(): 返回类型 { ... }`
- 通过 `ClassName::methodName()` 调用，无需实例化。
- 静态方法内部不能使用 `$this`，只能访问静态属性或调用静态方法。

```php
class Math
{
    public static function add(float $a, float $b): float
    {
        return $a + $b;
    }

    public static function multiply(float $a, float $b): float
    {
        return $a * $b;
    }
}

echo Math::add(5, 3); // 8
echo Math::multiply(4, 7); // 28
```

### 静态工厂方法

```php
class User
{
    private function __construct(
        public int $id,
        public string $name
    ) {
    }

    public static function create(int $id, string $name): self
    {
        return new self($id, $name);
    }

    public static function fromArray(array $data): self
    {
        return new self($data['id'], $data['name']);
    }
}

$user1 = User::create(1, 'Alice');
$user2 = User::fromArray(['id' => 2, 'name' => 'Bob']);
```

### 静态属性注意事项

```php
// 不推荐：使用静态属性存储全局状态
class Config
{
    public static string $databaseHost = 'localhost';
    public static string $databaseName = 'app';
}

// 推荐：使用依赖注入
class DatabaseConnection
{
    public function __construct(
        private string $host,
        private string $database
    ) {
    }
}
```

## 魔术方法

### 常用魔术方法

| 方法名          | 触发时机                     | 用途示例                           |
| :-------------- | :--------------------------- | :--------------------------------- |
| `__construct`   | 对象创建时                   | 初始化属性                         |
| `__destruct`    | 对象销毁时                   | 释放资源                           |
| `__clone`       | `clone` 操作时                | 自定义克隆逻辑                     |
| `__toString`    | 对象被转换为字符串时         | 返回对象的字符串表示               |
| `__get`         | 访问不存在的属性时           | 动态属性访问                       |
| `__set`         | 设置不存在的属性时           | 动态属性设置                       |
| `__call`        | 调用不存在的方法时           | 动态方法调用                       |
| `__invoke`      | 对象被当作函数调用时         | 使对象可调用                       |

### `__toString` 方法

```php
class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function __toString(): string
    {
        return "User #{$this->id}: {$this->name}";
    }
}

$user = new User(1, 'Alice');
echo $user; // User #1: Alice
```

### `__get` 和 `__set` 方法

```php
class Config
{
    private array $data = [];

    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void
    {
        $this->data[$name] = $value;
    }
}

$config = new Config();
$config->database = 'myapp';
$config->host = 'localhost';
echo $config->database; // myapp
```

### `__call` 方法

```php
class DynamicAPI
{
    private array $endpoints = [];

    public function __call(string $name, array $arguments): mixed
    {
        if (isset($this->endpoints[$name])) {
            return call_user_func_array($this->endpoints[$name], $arguments);
        }
        throw new BadMethodCallException("Method {$name} does not exist");
    }

    public function register(string $name, callable $handler): void
    {
        $this->endpoints[$name] = $handler;
    }
}

$api = new DynamicAPI();
$api->register('getUser', fn($id) => "User {$id}");
echo $api->getUser(1); // User 1
```

### `__invoke` 方法

```php
class CallableClass
{
    public function __invoke(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$callable = new CallableClass();
echo $callable('Alice'); // Hello, Alice!

// 可以作为回调使用
$callback = new CallableClass();
array_map($callback, ['Alice', 'Bob']);
```

### `__isset` 和 `__unset` 方法

```php
class DynamicProperties
{
    private array $data = [];

    public function __isset(string $name): bool
    {
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void
    {
        unset($this->data[$name]);
    }

    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void
    {
        $this->data[$name] = $value;
    }
}

$obj = new DynamicProperties();
$obj->name = 'Alice';
var_dump(isset($obj->name)); // true
unset($obj->name);
var_dump(isset($obj->name)); // false
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Logger
{
    private static int $logCount = 0;
    private array $logs = [];

    public function log(string $message): void
    {
        $this->logs[] = $message;
        self::$logCount++;
    }

    public static function getLogCount(): int
    {
        return self::$logCount;
    }

    public function __toString(): string
    {
        return sprintf(
            "Logger: %d logs, %d total logs",
            count($this->logs),
            self::$logCount
        );
    }

    public function __invoke(string $message): void
    {
        $this->log($message);
    }
}

$logger = new Logger();
$logger->log('Message 1');
$logger->log('Message 2');

echo $logger; // Logger: 2 logs, 2 total logs
echo Logger::getLogCount(); // 2

// 作为可调用对象
$logger('Message 3');
```

## 注意事项

1. **静态属性共享**：所有实例共享静态属性，可能导致意外的状态共享。

2. **静态方法限制**：静态方法不能访问非静态成员，不能使用 `$this`。

3. **魔术方法性能**：魔术方法会增加方法调用的开销，只在必要时使用。

4. **`__toString` 要求**：`__toString()` 必须返回字符串，不能抛出异常。

5. **动态属性**：使用 `__get` 和 `__set` 可以实现动态属性，但要注意类型安全。

## 练习

1. 创建一个 `Logger` 类，使用静态属性记录日志数量，提供静态方法 `getLogCount()`，并实现 `__toString()` 方法。

2. 实现一个 `Config` 类，使用 `__get()` 和 `__set()` 魔术方法实现动态配置访问。

3. 创建一个 `Calculator` 类，使用 `__call()` 方法实现动态方法调用（如 `add()`、`subtract()`、`multiply()`）。

4. 实现一个 `CallableWrapper` 类，使用 `__invoke()` 方法包装函数调用。

5. 创建一个 `ArrayAccess` 类，使用 `__get()`、`__set()`、`__isset()`、`__unset()` 实现数组式访问。

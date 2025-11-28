# 3.4 代码模块化与元编程

## 目标

- 理解 Traits 的概念与使用场景，掌握如何通过 Traits 实现代码复用。
- 熟悉 Traits 的冲突解决机制，了解优先级与方法别名。
- 掌握 PHP 8.0+ Attributes（注解）系统，能够创建和使用自定义属性。
- 理解元编程的概念，能够在实际项目中应用 Traits 和 Attributes。

## Traits 基础

### 什么是 Traits

- Traits 是 PHP 5.4+ 引入的代码复用机制。
- 可以理解为"可复用的方法集合"，解决单继承限制。
- Traits 不能实例化，只能被类使用。

```php
trait Loggable
{
    public function log(string $message): void
    {
        $timestamp = date('Y-m-d H:i:s');
        echo "[{$timestamp}] {$message}\n";
    }
}

class User
{
    use Loggable;

    public function __construct(
        public string $name
    ) {
    }

    public function register(): void
    {
        $this->log("User {$this->name} registered");
    }
}

$user = new User('Alice');
$user->register(); // [2025-11-28 14:30:00] User Alice registered
```

### Traits 语法

- **定义**：`trait TraitName { ... }`
- **使用**：`use TraitName;`（在类内部）

```php
trait Timestampable
{
    public function getCreatedAt(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }

    public function getUpdatedAt(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }
}

class Post
{
    use Timestampable;

    public function __construct(
        public string $title,
        public string $content
    ) {
    }
}

$post = new Post('Hello', 'World');
echo $post->getCreatedAt()->format('Y-m-d H:i:s');
```

### 多个 Traits

- 一个类可以使用多个 Traits。

```php
trait Cacheable
{
    public function cache(string $key, mixed $value): void
    {
        echo "Caching {$key}\n";
    }
}

trait Validatable
{
    public function validate(): bool
    {
        echo "Validating...\n";
        return true;
    }
}

class Product
{
    use Cacheable, Validatable;

    public function __construct(
        public string $name,
        public float $price
    ) {
    }

    public function save(): void
    {
        if ($this->validate()) {
            $this->cache("product_{$this->name}", $this);
            echo "Product saved\n";
        }
    }
}

$product = new Product('Laptop', 999.99);
$product->save();
```

## Traits 冲突解决

### 方法冲突

- 当多个 Traits 包含同名方法时，会产生冲突。
- 需要使用 `insteadof` 选择使用哪个方法，或使用 `as` 创建别名。

```php
trait A
{
    public function method(): string
    {
        return 'A';
    }
}

trait B
{
    public function method(): string
    {
        return 'B';
    }
}

class MyClass
{
    use A, B {
        B::method insteadof A; // 使用 B 的 method，忽略 A 的
        A::method as methodA;  // 将 A 的 method 重命名为 methodA
    }
}

$obj = new MyClass();
echo $obj->method();  // B
echo $obj->methodA(); // A
```

### 可见性修改

- 可以使用 `as` 修改 Trait 方法的可见性。

```php
trait InternalHelper
{
    public function helperMethod(): string
    {
        return 'Helper';
    }
}

class MyClass
{
    use InternalHelper {
        helperMethod as private privateHelper;
    }

    public function publicMethod(): string
    {
        return $this->privateHelper();
    }
}

$obj = new MyClass();
// $obj->helperMethod(); // 错误：方法不存在（已被重命名）
// $obj->privateHelper(); // 错误：私有方法
echo $obj->publicMethod(); // Helper
```

### 完整冲突解决示例

```php
trait FileOperations
{
    public function save(): void
    {
        echo "Saving to file...\n";
    }
}

trait DatabaseOperations
{
    public function save(): void
    {
        echo "Saving to database...\n";
    }
}

class Document
{
    use FileOperations, DatabaseOperations {
        DatabaseOperations::save insteadof FileOperations;
        FileOperations::save as saveToFile;
    }

    public function saveToBoth(): void
    {
        $this->save();        // 调用 DatabaseOperations::save
        $this->saveToFile();  // 调用 FileOperations::save
    }
}

$doc = new Document();
$doc->saveToBoth();
```

## Traits 与抽象方法

- Traits 可以定义抽象方法，使用该 Trait 的类必须实现这些方法。

```php
trait Serializable
{
    abstract public function toArray(): array;

    public function serialize(): string
    {
        return json_encode($this->toArray());
    }

    public static function deserialize(string $data): static
    {
        $array = json_decode($data, true);
        return static::fromArray($array);
    }

    abstract public static function fromArray(array $data): static;
}

class User
{
    use Serializable;

    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function toArray(): array
    {
        return ['id' => $this->id, 'name' => $this->name];
    }

    public static function fromArray(array $data): static
    {
        return new self($data['id'], $data['name']);
    }
}

$user = new User(1, 'Alice');
$serialized = $user->serialize();
$deserialized = User::deserialize($serialized);
```

## Traits 中的属性

- Traits 可以定义属性，使用该 Trait 的类会继承这些属性。

```php
trait Timestamps
{
    public DateTimeImmutable $createdAt;
    public DateTimeImmutable $updatedAt;

    public function touch(): void
    {
        $this->updatedAt = new DateTimeImmutable();
    }

    public function initializeTimestamps(): void
    {
        $this->createdAt = new DateTimeImmutable();
        $this->updatedAt = new DateTimeImmutable();
    }
}

class Post
{
    use Timestamps;

    public function __construct(
        public string $title
    ) {
        $this->initializeTimestamps();
    }
}

$post = new Post('Hello');
$post->touch();
```

## Attributes（注解）（PHP 8.0+）

### 基础语法

- **语法**：`#[AttributeName]` 或 `#[AttributeName(arguments)]`
- Attributes 是元数据，用于为类、方法、属性等添加信息。
- 可以替代传统的 PHPDoc 注释，提供类型安全的元数据。

```php
#[Attribute]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {
    }
}

class UserController
{
    #[Route('/users', 'GET')]
    public function index(): array
    {
        return [];
    }

    #[Route('/users/{id}', 'GET')]
    public function show(int $id): array
    {
        return ['id' => $id];
    }
}
```

### 读取 Attributes

- 使用反射 API 读取 Attributes。

```php
#[Attribute]
class Deprecated
{
    public function __construct(
        public string $reason,
        public ?string $replacement = null
    ) {
    }
}

class OldClass
{
    #[Deprecated('Use NewClass instead', 'NewClass')]
    public function oldMethod(): void
    {
    }
}

$reflection = new ReflectionClass(OldClass::class);
$method = $reflection->getMethod('oldMethod');
$attributes = $method->getAttributes(Deprecated::class);

if (!empty($attributes)) {
    $deprecated = $attributes[0]->newInstance();
    echo "Deprecated: {$deprecated->reason}\n";
    if ($deprecated->replacement) {
        echo "Use instead: {$deprecated->replacement}\n";
    }
}
```

### Attribute 目标

- 可以指定 Attribute 的使用目标。

```php
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class Cache
{
    public function __construct(
        public int $ttl = 3600
    ) {
    }
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Column
{
    public function __construct(
        public string $name,
        public string $type
    ) {
    }
}

class User
{
    #[Column('user_id', 'INT')]
    public int $id;

    #[Column('user_name', 'VARCHAR(255)')]
    public string $name;
}
```

### 内置 Attributes

- PHP 8.0+ 提供了一些内置 Attributes。

```php
// 允许属性在序列化时包含 null 值
#[AllowDynamicProperties]
class DynamicClass
{
}

// 标记方法已弃用
class Api
{
    #[\ReturnTypeWillChange]
    public function oldMethod(): mixed
    {
        return null;
    }
}
```

### 自定义 Attribute 示例：验证器

```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class Validate
{
    public function __construct(
        public string $rule,
        public ?string $message = null
    ) {
    }
}

class User
{
    #[Validate('required', 'Name is required')]
    public string $name;

    #[Validate('email', 'Invalid email format')]
    public string $email;

    #[Validate('min:18', 'Age must be at least 18')]
    public int $age;
}

class Validator
{
    public static function validate(object $object): array
    {
        $errors = [];
        $reflection = new ReflectionClass($object);

        foreach ($reflection->getProperties() as $property) {
            $attributes = $property->getAttributes(Validate::class);
            if (empty($attributes)) {
                continue;
            }

            $validate = $attributes[0]->newInstance();
            $value = $property->getValue($object);

            if ($validate->rule === 'required' && empty($value)) {
                $errors[$property->getName()] = $validate->message ?? 'This field is required';
            } elseif ($validate->rule === 'email' && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
                $errors[$property->getName()] = $validate->message ?? 'Invalid email format';
            }
        }

        return $errors;
    }
}

$user = new User();
$user->name = '';
$user->email = 'invalid-email';
$user->age = 16;

$errors = Validator::validate($user);
print_r($errors);
```

### 路由系统示例

```php
#[Attribute(Attribute::TARGET_METHOD)]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET',
        public ?string $name = null
    ) {
    }
}

class UserController
{
    #[Route('/users', 'GET', 'users.index')]
    public function index(): array
    {
        return ['users' => []];
    }

    #[Route('/users/{id}', 'GET', 'users.show')]
    public function show(int $id): array
    {
        return ['user' => ['id' => $id]];
    }

    #[Route('/users', 'POST', 'users.store')]
    public function store(array $data): array
    {
        return ['user' => $data];
    }
}

class Router
{
    public static function registerRoutes(string $controllerClass): array
    {
        $routes = [];
        $reflection = new ReflectionClass($controllerClass);

        foreach ($reflection->getMethods() as $method) {
            $attributes = $method->getAttributes(Route::class);
            if (empty($attributes)) {
                continue;
            }

            $route = $attributes[0]->newInstance();
            $routes[] = [
                'path' => $route->path,
                'method' => $route->method,
                'name' => $route->name,
                'handler' => [$controllerClass, $method->getName()],
            ];
        }

        return $routes;
    }
}

$routes = Router::registerRoutes(UserController::class);
print_r($routes);
```

## Traits 与 Attributes 结合使用

```php
trait Timestampable
{
    #[Column('created_at', 'TIMESTAMP')]
    public DateTimeImmutable $createdAt;

    #[Column('updated_at', 'TIMESTAMP')]
    public DateTimeImmutable $updatedAt;

    public function initializeTimestamps(): void
    {
        $this->createdAt = new DateTimeImmutable();
        $this->updatedAt = new DateTimeImmutable();
    }
}

#[Table('users')]
class User
{
    use Timestampable;

    #[Column('id', 'INT'), PrimaryKey]
    public int $id;

    #[Column('name', 'VARCHAR(255)'), Validate('required')]
    public string $name;
}
```

## 最佳实践

### Traits 使用建议

1. **单一职责**：每个 Trait 应该只负责一个功能领域。
2. **避免过度使用**：不要为了复用而创建过多 Traits，保持代码可读性。
3. **文档说明**：为 Traits 添加清晰的文档，说明使用场景和依赖。

```php
/**
 * 提供日志功能的 Trait
 * 
 * 使用此 Trait 的类需要实现 getLogger() 方法
 */
trait Loggable
{
    abstract protected function getLogger(): Logger;

    protected function log(string $level, string $message, array $context = []): void
    {
        $this->getLogger()->log($level, $message, $context);
    }
}
```

### Attributes 使用建议

1. **类型安全**：使用强类型的 Attribute 类，避免字符串魔法值。
2. **目标限制**：明确指定 Attribute 的使用目标，防止误用。
3. **文档补充**：Attributes 不能完全替代文档，重要信息仍需注释说明。

```php
#[Attribute(Attribute::TARGET_CLASS)]
class Entity
{
    public function __construct(
        public string $table,
        public ?string $repository = null
    ) {
    }
}
```

## 练习

1. 创建一个 `Cacheable` Trait，提供 `cache()` 和 `getCached()` 方法，然后创建一个 `Product` 类使用该 Trait。

2. 定义两个 Traits：`Timestamps`（创建/更新时间）和 `SoftDeletes`（软删除），创建一个 `Post` 类同时使用这两个 Traits。

3. 创建一个 `Route` Attribute，用于标记控制器方法，然后编写一个路由注册器读取这些 Attributes 并注册路由。

4. 实现一个 `Validate` Attribute 系统，支持 `required`、`email`、`min`、`max` 等验证规则，创建一个验证器类读取 Attributes 并执行验证。

5. 创建一个 `Serializable` Trait，提供 `serialize()` 和 `deserialize()` 方法，要求使用该 Trait 的类实现 `toArray()` 和 `fromArray()` 方法。

6. 设计一个 `EventDispatcher` Trait，提供 `dispatch()` 方法，使用该 Trait 的类可以触发事件，然后创建一个 `Event` Attribute 用于标记事件处理方法。

# 3.4.1 Traits

## 概述

Traits 是 PHP 5.4+ 引入的代码复用机制，可以理解为"可复用的方法集合"，解决单继承限制。

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

## 多个 Traits

### 使用多个 Traits

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

## Traits 与抽象方法

### 抽象方法要求

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

### 属性定义

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

## 完整示例

```php
<?php
declare(strict_types=1);

trait SoftDeletes
{
    private ?DateTimeImmutable $deletedAt = null;

    public function delete(): void
    {
        $this->deletedAt = new DateTimeImmutable();
    }

    public function restore(): void
    {
        $this->deletedAt = null;
    }

    public function isDeleted(): bool
    {
        return $this->deletedAt !== null;
    }
}

trait Timestampable
{
    public DateTimeImmutable $createdAt;
    public DateTimeImmutable $updatedAt;

    public function initializeTimestamps(): void
    {
        $this->createdAt = new DateTimeImmutable();
        $this->updatedAt = new DateTimeImmutable();
    }

    public function touch(): void
    {
        $this->updatedAt = new DateTimeImmutable();
    }
}

class Post
{
    use SoftDeletes, Timestampable;

    public function __construct(
        public string $title,
        public string $content
    ) {
        $this->initializeTimestamps();
    }
}

$post = new Post('Hello', 'World');
$post->delete();
var_dump($post->isDeleted()); // true
```

## 注意事项

1. **单一职责**：每个 Trait 应该只负责一个功能领域。

2. **避免过度使用**：不要为了复用而创建过多 Traits，保持代码可读性。

3. **冲突解决**：当 Traits 有方法冲突时，使用 `insteadof` 和 `as` 解决。

4. **抽象方法**：Traits 可以定义抽象方法，使用该 Trait 的类必须实现。

5. **属性共享**：Traits 中的属性会被使用该 Trait 的类继承。

## 练习

1. 创建一个 `Cacheable` Trait，提供 `cache()` 和 `getCached()` 方法，然后创建一个 `Product` 类使用该 Trait。

2. 定义两个 Traits：`Timestamps`（创建/更新时间）和 `SoftDeletes`（软删除），创建一个 `Post` 类同时使用这两个 Traits。

3. 创建一个 `Serializable` Trait，提供 `serialize()` 和 `deserialize()` 方法，要求使用该 Trait 的类实现 `toArray()` 和 `fromArray()` 方法。

4. 实现一个 `EventDispatcher` Trait，提供 `dispatch()` 方法，使用该 Trait 的类可以触发事件。

5. 创建两个有方法冲突的 Traits，使用 `insteadof` 和 `as` 解决冲突。

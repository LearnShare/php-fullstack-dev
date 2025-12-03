# 3.4.2 Attributes（注解）

## 概述

Attributes（注解）是 PHP 8.0+ 引入的元数据系统，用于为类、方法、属性等添加信息。可以替代传统的 PHPDoc 注释，提供类型安全的元数据。

## 基础语法

### 定义 Attribute

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

## 读取 Attributes

### 反射 API

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

## Attribute 目标

### 指定使用目标

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

## 自定义 Attribute 示例

### 验证器 Attribute

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

## 完整示例

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_CLASS)]
class Entity
{
    public function __construct(
        public string $table
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

#[Entity('users')]
class User
{
    #[Column('id', 'INT')]
    public int $id;

    #[Column('name', 'VARCHAR(255)')]
    public string $name;

    #[Column('email', 'VARCHAR(255)')]
    public string $email;
}

class EntityMapper
{
    public static function getTableName(string $class): ?string
    {
        $reflection = new ReflectionClass($class);
        $attributes = $reflection->getAttributes(Entity::class);
        
        if (empty($attributes)) {
            return null;
        }
        
        $entity = $attributes[0]->newInstance();
        return $entity->table;
    }

    public static function getColumns(string $class): array
    {
        $columns = [];
        $reflection = new ReflectionClass($class);

        foreach ($reflection->getProperties() as $property) {
            $attributes = $property->getAttributes(Column::class);
            if (empty($attributes)) {
                continue;
            }

            $column = $attributes[0]->newInstance();
            $columns[$property->getName()] = [
                'name' => $column->name,
                'type' => $column->type
            ];
        }

        return $columns;
    }
}

$tableName = EntityMapper::getTableName(User::class);
$columns = EntityMapper::getColumns(User::class);
```

## 注意事项

1. **类型安全**：使用强类型的 Attribute 类，避免字符串魔法值。

2. **目标限制**：明确指定 Attribute 的使用目标，防止误用。

3. **文档补充**：Attributes 不能完全替代文档，重要信息仍需注释说明。

4. **性能考虑**：反射 API 有性能开销，避免在热路径中频繁使用。

5. **版本要求**：Attributes 需要 PHP 8.0+，确保运行环境支持。

## 练习

1. 创建一个 `Route` Attribute，用于标记控制器方法，然后编写一个路由注册器读取这些 Attributes 并注册路由。

2. 实现一个 `Validate` Attribute 系统，支持 `required`、`email`、`min`、`max` 等验证规则，创建一个验证器类读取 Attributes 并执行验证。

3. 创建一个 `Cache` Attribute，用于标记需要缓存的方法，实现一个缓存拦截器。

4. 设计一个 `Event` Attribute 用于标记事件处理方法，实现事件分发系统。

5. 实现一个 ORM 映射系统，使用 Attributes 定义实体类和列的映射关系。

# 3.4.2 Attributes（注解）

## 概述

Attributes（注解）是 PHP 8.0 引入的特性，用于为类、方法、属性、参数等代码元素添加元数据。Attributes 提供了一种类型安全的方式来添加元数据，可以替代传统的 PHPDoc 注释，提供更好的 IDE 支持和类型检查。

理解 Attributes 对于掌握现代 PHP 编程非常重要。Attributes 提供了一种声明式的编程方式，允许在代码中直接表达元数据，而无需使用字符串注释。这些元数据可以通过反射 API 读取，用于框架、库和工具的各种用途，如路由定义、数据验证、ORM 映射等。

Attributes 是元编程（Metaprogramming）的重要工具，允许编写操作代码的代码。通过 Attributes，可以在不修改原有代码逻辑的情况下，为代码添加额外的信息和行为。

**主要内容**：
- Attributes 的基础语法和使用
- 定义自定义 Attributes
- 读取 Attributes（反射 API）
- Attribute 目标（TARGET_CLASS、TARGET_METHOD 等）
- Attributes 的参数传递
- 内置 Attributes（如 `#[\Deprecated]`）
- Attributes 的应用场景（路由、验证、ORM 等）
- Attributes 的最佳实践

## 特性

- **元数据**：为代码添加元数据，提供额外信息
- **类型安全**：编译时类型检查，避免字符串错误
- **反射支持**：通过反射 API 读取 Attributes
- **目标限制**：可以限制 Attribute 的使用目标
- **参数支持**：可以传递参数给 Attributes

## 语法/定义

### Attributes 语法

**语法**：`#[AttributeName]` 或 `#[AttributeName(arguments)]`

**组成部分**：
- `#` 符号：表示 Attribute
- `[` 和 `]`：Attribute 的语法边界
- `AttributeName`：Attribute 类名
- `arguments`：可选的参数列表

**特点**：
- PHP 8.0+ 引入的特性
- 可以附加到类、方法、属性、参数、常量、函数等
- 支持位置参数和命名参数
- 支持多个 Attributes

### 定义自定义 Attribute

**语法**：`#[Attribute(flags)] class AttributeName { ... }`

**组成部分**：
- `#[Attribute]`：标记该类为 Attribute
- `flags`：可选的标志，指定使用目标（如 `Attribute::TARGET_CLASS`）
- `AttributeName`：Attribute 类名

**特点**：
- 使用 `#[Attribute]` 标记
- 可以指定使用目标
- 可以有构造函数接收参数

### 读取 Attributes

**方法**：使用反射 API

**相关类**：
- `ReflectionClass::getAttributes()`
- `ReflectionMethod::getAttributes()`
- `ReflectionProperty::getAttributes()`
- `ReflectionParameter::getAttributes()`

## 基本用法

### 示例 1：基础 Attributes 使用

```php
<?php
declare(strict_types=1);

#[Attribute]
class Route
{
    public function __construct(
        public string $path,
        public array $methods = ['GET']
    ) {}
}

#[Route('/users', methods: ['GET'])]
class UserController
{
    #[Route('/users/{id}', methods: ['GET'])]
    public function show(int $id): array
    {
        return ['id' => $id];
    }
}

// 读取 Attributes
$reflection = new ReflectionClass(UserController::class);
$classAttributes = $reflection->getAttributes(Route::class);

if (!empty($classAttributes)) {
    $route = $classAttributes[0]->newInstance();
    echo "Class route: {$route->path}, methods: " . implode(', ', $route->methods) . "\n";
}

$method = $reflection->getMethod('show');
$methodAttributes = $method->getAttributes(Route::class);

if (!empty($methodAttributes)) {
    $route = $methodAttributes[0]->newInstance();
    echo "Method route: {$route->path}, methods: " . implode(', ', $route->methods) . "\n";
}
```

**输出**：

```
Class route: /users, methods: GET
Method route: /users/{id}, methods: GET
```

**说明**：
- `Route` 是一个自定义 Attribute
- 可以附加到类和方法上
- 使用反射 API 读取 Attributes

### 示例 2：指定 Attribute 目标

```php
<?php
declare(strict_types=1);

// 只能用于类
#[Attribute(Attribute::TARGET_CLASS)]
class Entity
{
    public function __construct(
        public string $table
    ) {}
}

// 只能用于属性
#[Attribute(Attribute::TARGET_PROPERTY)]
class Column
{
    public function __construct(
        public string $name,
        public string $type
    ) {}
}

// 只能用于方法
#[Attribute(Attribute::TARGET_METHOD)]
class Authorize
{
    public function __construct(
        public array $roles = []
    ) {}
}

// 可以用于多个目标
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class Cache
{
    public function __construct(
        public int $ttl = 3600
    ) {}
}

#[Entity('users')]
class User
{
    #[Column('id', 'INT')]
    public int $id;
    
    #[Column('name', 'VARCHAR(255)')]
    public string $name;
    
    #[Authorize(['admin', 'user'])]
    public function update(): void {}
    
    #[Cache(ttl: 7200)]
    public function getData(): array { return []; }
}
```

**说明**：
- `Attribute::TARGET_CLASS`：只能用于类
- `Attribute::TARGET_PROPERTY`：只能用于属性
- `Attribute::TARGET_METHOD`：只能用于方法
- 可以使用 `|` 组合多个目标

### 示例 3：多个 Attributes

```php
<?php
declare(strict_types=1);

#[Attribute]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {}
}

#[Attribute]
class Authorize
{
    public function __construct(
        public array $roles = []
    ) {}
}

#[Attribute]
class Cache
{
    public function __construct(
        public int $ttl = 3600
    ) {}
}

class UserController
{
    #[Route('/users', 'GET')]
    #[Authorize(['admin'])]
    #[Cache(ttl: 600)]
    public function index(): array
    {
        return [];
    }
}

// 读取所有 Attributes
$reflection = new ReflectionClass(UserController::class);
$method = $reflection->getMethod('index');
$attributes = $method->getAttributes();

foreach ($attributes as $attribute) {
    $instance = $attribute->newInstance();
    echo "Attribute: " . $attribute->getName() . "\n";
}
```

**输出**：

```
Attribute: Route
Attribute: Authorize
Attribute: Cache
```

**说明**：
- 一个元素可以有多个 Attributes
- 每个 Attribute 独立定义和读取
- 可以组合使用多个 Attributes

### 示例 4：路由系统示例

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_METHOD)]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET',
        public ?string $name = null
    ) {}
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
foreach ($routes as $route) {
    echo "Route: {$route['method']} {$route['path']} -> {$route['name']}\n";
}
```

**输出**：

```
Route: GET /users -> users.index
Route: GET /users/{id} -> users.show
Route: POST /users -> users.store
```

**说明**：
- 使用 Attributes 定义路由
- 通过反射 API 读取路由信息
- 实现了声明式的路由定义

### 示例 5：数据验证示例

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PROPERTY)]
class Validate
{
    public function __construct(
        public string $rule,
        public ?string $message = null
    ) {}
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
            
            // 验证逻辑
            if ($validate->rule === 'required' && empty($value)) {
                $errors[$property->getName()] = $validate->message ?? 'This field is required';
            } elseif ($validate->rule === 'email' && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
                $errors[$property->getName()] = $validate->message ?? 'Invalid email format';
            } elseif (str_starts_with($validate->rule, 'min:')) {
                $min = (int) substr($validate->rule, 4);
                if ($value < $min) {
                    $errors[$property->getName()] = $validate->message ?? "Value must be at least {$min}";
                }
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
foreach ($errors as $field => $message) {
    echo "{$field}: {$message}\n";
}
```

**输出**：

```
name: Name is required
email: Invalid email format
age: Age must be at least 18
```

**说明**：
- 使用 Attributes 定义验证规则
- 通过反射 API 读取验证规则
- 实现了声明式的数据验证

### 示例 6：ORM 映射示例

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_CLASS)]
class Entity
{
    public function __construct(
        public string $table
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Column
{
    public function __construct(
        public string $name,
        public string $type
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class PrimaryKey {}

#[Entity('users')]
class User
{
    #[Column('id', 'INT')]
    #[PrimaryKey]
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
            $columnAttributes = $property->getAttributes(Column::class);
            if (empty($columnAttributes)) {
                continue;
            }
            
            $column = $columnAttributes[0]->newInstance();
            $isPrimaryKey = !empty($property->getAttributes(PrimaryKey::class));
            
            $columns[$property->getName()] = [
                'name' => $column->name,
                'type' => $column->type,
                'primary_key' => $isPrimaryKey,
            ];
        }
        
        return $columns;
    }
}

$tableName = EntityMapper::getTableName(User::class);
echo "Table: {$tableName}\n";

$columns = EntityMapper::getColumns(User::class);
foreach ($columns as $property => $column) {
    echo "Column {$property}: {$column['name']} ({$column['type']})" . ($column['primary_key'] ? ' [PK]' : '') . "\n";
}
```

**输出**：

```
Table: users
Column id: id (INT) [PK]
Column name: name (VARCHAR(255))
Column email: email (VARCHAR(255))
```

**说明**：
- 使用 Attributes 定义 ORM 映射
- 通过反射 API 读取映射信息
- 实现了声明式的数据库映射

### 示例 7：内置 Attributes（PHP 8.0+）

```php
<?php
declare(strict_types=1);

// 使用内置的 Deprecated Attribute
class OldClass
{
    #[\Deprecated('Use NewClass instead', replacement: 'NewClass')]
    public function oldMethod(): void
    {
    }
}

// 检查是否已弃用
$reflection = new ReflectionClass(OldClass::class);
$method = $reflection->getMethod('oldMethod');
$attributes = $method->getAttributes(\Deprecated::class);

if (!empty($attributes)) {
    $deprecated = $attributes[0]->newInstance();
    echo "Deprecated: " . $deprecated->reason . "\n";
    if ($deprecated->replacement !== null) {
        echo "Use instead: " . $deprecated->replacement . "\n";
    }
}
```

**输出**：

```
Deprecated: Use NewClass instead
Use instead: NewClass
```

**说明**：
- PHP 8.0+ 提供了一些内置 Attributes
- `\Deprecated` 用于标记已弃用的代码
- 可以通过反射 API 检查

### 示例 8：参数 Attributes

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PARAMETER)]
class SensitiveParameter {}

class UserService
{
    public function authenticate(
        string $username,
        #[\SensitiveParameter] string $password
    ): bool {
        // 认证逻辑
        return true;
    }
}

// 读取参数 Attributes
$reflection = new ReflectionClass(UserService::class);
$method = $reflection->getMethod('authenticate');
$parameters = $method->getParameters();

foreach ($parameters as $parameter) {
    $attributes = $parameter->getAttributes(\SensitiveParameter::class);
    if (!empty($attributes)) {
        echo "Parameter {$parameter->getName()} is sensitive\n";
    }
}
```

**输出**：

```
Parameter password is sensitive
```

**说明**：
- Attributes 可以附加到参数上
- 使用 `Attribute::TARGET_PARAMETER` 指定目标
- 可以通过反射 API 读取参数 Attributes

## 使用场景

### 场景 1：框架路由定义

Attributes 非常适合用于框架的路由定义。

**示例**：见"示例 4：路由系统示例"

### 场景 2：数据验证

Attributes 可以用于声明式的数据验证。

**示例**：见"示例 5：数据验证示例"

### 场景 3：ORM 映射

Attributes 可以用于 ORM 的数据库映射。

**示例**：见"示例 6：ORM 映射示例"

### 场景 4：依赖注入

Attributes 可以用于依赖注入容器的配置。

**示例**：依赖注入

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PROPERTY)]
class Inject
{
    public function __construct(
        public string $service
    ) {}
}

class UserService
{
    public function __construct(
        #[Inject('logger')] private $logger,
        #[Inject('cache')] private $cache
    ) {}
}
```

## 注意事项

### 版本要求

- **PHP 版本**：需要 PHP 8.0 或更高版本
- **兼容性**：旧版本 PHP 不支持此特性
- **检查方法**：使用 `phpversion()` 检查 PHP 版本

### 反射性能

- **性能开销**：反射 API 有性能开销
- **缓存建议**：考虑缓存反射结果
- **避免过度使用**：不要在热路径中频繁使用

### Attribute 目标限制

- **明确目标**：使用 `Attribute::TARGET_*` 标志限制使用目标
- **防止误用**：目标限制可以防止 Attribute 被误用到不合适的元素上
- **类型安全**：目标限制提供编译时检查

## 常见问题

### 问题 1：版本不兼容错误

**错误信息**：`Parse error: syntax error, unexpected '['`

**原因**：使用了低于 PHP 8.0 的版本

**解决方案**：
- 升级 PHP 版本到 8.0 或更高
- 检查 PHP 版本：`php -v`

### 问题 2：Attribute 目标错误

**错误信息**：`Attribute "X" cannot target property`

**原因**：Attribute 被用在了不允许的目标上

**解决方案**：
- 检查 Attribute 的目标限制
- 确保 Attribute 用在正确的目标上

**示例**：

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_METHOD)]  // 只能用于方法
class MethodOnly {}

// 错误：不能用于类
// #[MethodOnly]
class MyClass {}

// 正确：用于方法
class MyClass
{
    #[MethodOnly]
    public function method(): void {}
}
```

### 问题 3：反射性能问题

**症状**：代码运行较慢

**原因**：频繁使用反射 API

**解决方案**：
- 缓存反射结果
- 避免在热路径中使用反射
- 考虑使用静态分析工具

## 最佳实践

### 1. 明确指定 Attribute 目标

使用 `Attribute::TARGET_*` 标志明确指定使用目标。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的做法：明确指定目标
#[Attribute(Attribute::TARGET_PROPERTY)]
class Column {}

// 不好的做法：不指定目标
// #[Attribute]
// class Column {}
```

### 2. 使用类型安全的参数

Attribute 的参数应该使用类型声明，提供类型安全。

**示例**：

```php
<?php
declare(strict_types=1);

#[Attribute]
class Route
{
    public function __construct(
        public string $path,        // 类型声明
        public array $methods = ['GET']  // 类型声明和默认值
    ) {}
}
```

### 3. 合理命名 Attributes

Attribute 类名应该清晰描述其用途。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的命名
#[Attribute]
class Route {}
#[Attribute]
class Validate {}
#[Attribute]
class Authorize {}

// 不好的命名
// #[Attribute]
// class R {}  // 不清晰
// #[Attribute]
// class Attr {}  // 不清晰
```

### 4. 文档化 Attributes

为 Attribute 类添加文档注释，说明其用途和使用方法。

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * Route Attribute
 * 
 * 用于定义路由信息
 * 
 * @param string $path 路由路径
 * @param array $methods HTTP 方法列表
 */
#[Attribute]
class Route
{
    public function __construct(
        public string $path,
        public array $methods = ['GET']
    ) {}
}
```

### 5. 缓存反射结果

对于频繁使用的反射操作，考虑缓存结果。

**示例**：

```php
<?php
declare(strict_types=1);

class RouteCache
{
    private static array $cache = [];
    
    public static function getRoutes(string $controllerClass): array
    {
        if (isset(self::$cache[$controllerClass])) {
            return self::$cache[$controllerClass];
        }
        
        // 反射操作
        $routes = Router::registerRoutes($controllerClass);
        self::$cache[$controllerClass] = $routes;
        
        return $routes;
    }
}
```

## 对比分析

### Attributes vs PHPDoc 注释

| 特性         | Attributes                          | PHPDoc 注释                        |
|:-------------|:-----------------------------------|:-----------------------------------|
| **类型安全**  | ✅ 编译时检查                        | ❌ 运行时检查（字符串）              |
| **IDE 支持**  | ✅ 更好的支持                        | ⚠️ 有限的支持                       |
| **性能**      | 反射开销                           | 无开销                             |
| **版本要求**  | PHP 8.0+                           | 所有 PHP 版本                       |
| **可读性**    | ✅ 代码和元数据在一起                | ⚠️ 元数据在注释中                   |

### Attributes vs 配置文件

| 特性         | Attributes                          | 配置文件                           |
|:-------------|:-----------------------------------|:-----------------------------------|
| **位置**      | 代码中                              | 单独的配置文件                     |
| **类型安全**  | ✅ 编译时检查                        | ❌ 运行时检查                       |
| **维护**      | ✅ 代码和配置在一起                  | ⚠️ 需要单独维护                     |
| **灵活性**    | ⚠️ 需要重新编译                      | ✅ 可以动态修改                     |

## 练习任务

1. **定义路由 Attribute**：创建一个 `Route` Attribute，包含路径和方法参数，创建一个控制器类使用该 Attribute。

2. **验证 Attribute**：创建一个 `Validate` Attribute，包含验证规则和错误消息，创建一个类使用该 Attribute 进行验证。

3. **ORM Attribute**：创建 `Entity` 和 `Column` Attributes，创建一个实体类使用这些 Attributes，编写代码读取映射信息。

4. **多个 Attributes**：创建一个类，在同一方法上使用多个 Attributes（如 `Route`、`Authorize`、`Cache`），编写代码读取所有 Attributes。

5. **自定义 Attribute 目标**：创建多个不同目标的 Attributes，确保它们只能用在指定的目标上。

## 相关章节

- **[2.14.1 PHP 8.0 新特性](../../stage-02-language/chapter-14-php-versions/section-01-php80.md)**：了解 PHP 8.0 的其他新特性
- **[3.4.1 Traits](section-01-traits.md)**：了解 Traits 的使用
- **阶段七：高级架构与设计模式**：了解 Attributes 在框架中的应用

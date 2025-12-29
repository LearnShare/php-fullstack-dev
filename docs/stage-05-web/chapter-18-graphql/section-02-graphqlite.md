# 5.18.2 GraphQLite

## 概述

GraphQLite 是 PHP 的 GraphQL 库，通过注解定义 Schema。理解 GraphQLite 的使用方法、掌握 Schema 定义、实现查询处理，对于快速构建 GraphQL API 至关重要。本节详细介绍 GraphQLite 概述、安装配置、Schema 定义、查询处理等内容，帮助零基础学员掌握 GraphQLite 的使用。

GraphQLite 通过注解定义 GraphQL Schema，简化了 GraphQL API 的开发。掌握 GraphQLite 的使用可以快速构建 GraphQL API。

**主要内容**：
- GraphQLite 概述（GraphQLite 的功能、GraphQLite 的优势、使用场景）
- 安装配置（Composer 安装、基本配置、服务器创建）
- Schema 定义（类型定义、字段定义、查询定义、变更定义）
- 查询处理（查询解析、字段解析、参数处理、错误处理）
- 实际应用示例和最佳实践

## 特性

- **注解驱动**：通过注解定义 Schema
- **易于使用**：简洁的 API
- **类型安全**：类型安全
- **自动生成**：自动生成 Schema
- **文档完善**：文档完善

## GraphQLite 概述

### GraphQLite 的功能

1. **Schema 定义**：通过注解定义 Schema
2. **查询处理**：处理 GraphQL 查询
3. **类型系统**：强类型系统
4. **自动生成**：自动生成 Schema

### GraphQLite 的优势

1. **易于使用**：简洁的 API
2. **注解驱动**：通过注解定义
3. **类型安全**：类型安全
4. **自动生成**：自动生成 Schema

### 使用场景

1. **GraphQL API**：构建 GraphQL API
2. **快速开发**：快速开发 GraphQL API
3. **类型安全**：需要类型安全的 API

## 安装配置

### Composer 安装

**安装命令**：
```bash
composer require thecodingmachine/graphqlite
```

### 基本配置

**示例**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use TheCodingMachine\GraphQLite\Schema;
use TheCodingMachine\GraphQLite\Context;

$schema = new Schema();
$context = new Context();

// 注册类型
$schema->registerType(User::class);
$schema->registerType(Post::class);
```

### 服务器创建

**示例**：
```php
<?php
declare(strict_types=1);

use TheCodingMachine\GraphQLite\Schema;
use GraphQL\GraphQL;

$schema = $this->buildSchema();

// 处理 GraphQL 请求
$input = json_decode(file_get_contents('php://input'), true);
$query = $input['query'] ?? '';
$variables = $input['variables'] ?? [];

$result = GraphQL::executeQuery($schema, $query, null, null, $variables);
echo json_encode($result->toArray());
```

## Schema 定义

### 类型定义

**示例**：
```php
<?php
declare(strict_types=1);

use TheCodingMachine\GraphQLite\Annotations\Type;
use TheCodingMachine\GraphQLite\Annotations\Field;

#[Type]
class User
{
    public function __construct(
        private int $id,
        private string $name,
        private string $email
    ) {}

    #[Field]
    public function getId(): int
    {
        return $this->id;
    }

    #[Field]
    public function getName(): string
    {
        return $this->name;
    }

    #[Field]
    public function getEmail(): string
    {
        return $this->email;
    }

    #[Field]
    public function getPosts(): array
    {
        // 返回用户的文章列表
        return [];
    }
}
```

### 字段定义

**示例**：
```php
<?php
declare(strict_types=1);

use TheCodingMachine\GraphQLite\Annotations\Type;
use TheCodingMachine\GraphQLite\Annotations\Field;

#[Type]
class Post
{
    public function __construct(
        private int $id,
        private string $title,
        private string $content,
        private int $authorId
    ) {}

    #[Field]
    public function getId(): int
    {
        return $this->id;
    }

    #[Field]
    public function getTitle(): string
    {
        return $this->title;
    }

    #[Field]
    public function getContent(): string
    {
        return $this->content;
    }

    #[Field]
    public function getAuthor(): User
    {
        // 返回作者信息
        return new User($this->authorId, 'Author', 'author@example.com');
    }
}
```

### 查询定义

**示例**：
```php
<?php
declare(strict_types=1);

use TheCodingMachine\GraphQLite\Annotations\Query;

class UserQuery
{
    #[Query]
    public function user(int $id): ?User
    {
        // 根据 ID 查询用户
        return new User($id, 'John', 'john@example.com');
    }

    #[Query]
    public function users(int $limit = 10, int $offset = 0): array
    {
        // 查询用户列表
        return [
            new User(1, 'John', 'john@example.com'),
            new User(2, 'Jane', 'jane@example.com'),
        ];
    }
}
```

### 变更定义

**示例**：
```php
<?php
declare(strict_types=1);

use TheCodingMachine\GraphQLite\Annotations\Mutation;
use TheCodingMachine\GraphQLite\Annotations\Input;

#[Input]
class CreateUserInput
{
    public function __construct(
        public string $name,
        public string $email
    ) {}
}

class UserMutation
{
    #[Mutation]
    public function createUser(CreateUserInput $input): User
    {
        // 创建用户
        return new User(1, $input->name, $input->email);
    }

    #[Mutation]
    public function updateUser(int $id, CreateUserInput $input): User
    {
        // 更新用户
        return new User($id, $input->name, $input->email);
    }

    #[Mutation]
    public function deleteUser(int $id): bool
    {
        // 删除用户
        return true;
    }
}
```

## 查询处理

### 查询解析

**示例**：
```php
<?php
declare(strict_types=1);

use GraphQL\GraphQL;
use TheCodingMachine\GraphQLite\Schema;

$schema = $this->buildSchema();

$query = '
    query {
        user(id: 1) {
            id
            name
            email
            posts {
                id
                title
            }
        }
    }
';

$result = GraphQL::executeQuery($schema, $query);
echo json_encode($result->toArray());
```

### 字段解析

**示例**：
```php
<?php
declare(strict_types=1);

#[Type]
class User
{
    #[Field]
    public function getId(): int
    {
        return $this->id;
    }

    #[Field]
    public function getName(): string
    {
        return $this->name;
    }

    // GraphQLite 自动调用这些方法解析字段
}
```

### 参数处理

**示例**：
```php
<?php
declare(strict_types=1);

use TheCodingMachine\GraphQLite\Annotations\Query;

class UserQuery
{
    #[Query]
    public function user(
        int $id,
        ?string $name = null,
        ?string $email = null
    ): ?User {
        // 处理查询参数
        if ($name !== null) {
            // 按名称查询
        }
        
        if ($email !== null) {
            // 按邮箱查询
        }
        
        // 按 ID 查询
        return $this->findUserById($id);
    }
}
```

### 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

use GraphQL\GraphQL;
use GraphQL\Error\Error;

try {
    $result = GraphQL::executeQuery($schema, $query);
    
    if ($result->errors) {
        foreach ($result->errors as $error) {
            error_log($error->getMessage());
        }
    }
    
    echo json_encode($result->toArray());
} catch (\Exception $e) {
    http_response_code(500);
    echo json_encode([
        'errors' => [
            ['message' => $e->getMessage()]
        ]
    ]);
}
```

## 使用场景

### GraphQL API

- 构建 GraphQL API
- 快速开发
- 类型安全

### 快速开发

- 快速原型开发
- 简化 Schema 定义
- 自动生成 Schema

### 类型安全

- 类型安全的 API
- 强类型系统
- 编译时检查

## 注意事项

### Schema 定义

- **注解使用**：正确使用注解
- **类型定义**：定义清晰的类型
- **字段定义**：定义所有字段

### 查询处理

- **参数验证**：验证查询参数
- **错误处理**：处理查询错误
- **性能优化**：优化查询性能

### 类型安全

- **类型检查**：确保类型正确
- **类型转换**：正确处理类型转换
- **空值处理**：处理空值情况

### 性能考虑

- **查询优化**：优化查询性能
- **缓存使用**：使用缓存
- **批量查询**：实现批量查询

## 常见问题

### 如何安装 GraphQLite？

使用 Composer 安装：
```bash
composer require thecodingmachine/graphqlite
```

### 如何定义 Schema？

使用注解定义类型、字段、查询和变更。

### 如何处理查询？

GraphQLite 自动处理查询，调用相应的方法解析字段。

### 如何优化性能？

使用 DataLoader、缓存、批量查询等方法优化性能。

## 最佳实践

### 使用注解定义 Schema

- 使用注解定义类型
- 使用注解定义字段
- 保持注解清晰

### 实现类型安全

- 使用强类型
- 验证类型
- 处理类型转换

### 优化查询性能

- 使用 DataLoader
- 实现缓存
- 优化解析器

### 处理错误

- 捕获所有异常
- 返回清晰的错误信息
- 记录错误日志

## 相关章节

- **[5.18.1 GraphQL 概述](section-01-overview.md)**：了解 GraphQL 概述的详细内容
- **[5.18.3 Lighthouse](section-03-lighthouse.md)**：了解 Lighthouse 的详细内容
- **[5.18.4 GraphQL 最佳实践](section-04-best-practices.md)**：了解 GraphQL 最佳实践的详细内容

## 练习任务

1. **安装和配置 GraphQLite**
   - Composer 安装
   - 基本配置
   - Schema 创建

2. **实现基本 Schema**
   - 类型定义
   - 字段定义
   - 查询定义

3. **实现查询处理**
   - 查询解析
   - 字段解析
   - 参数处理

4. **实现变更处理**
   - Mutation 定义
   - 输入类型定义
   - 变更处理

5. **实现完整的 GraphQL API**
   - Schema 设计
   - 查询和变更
   - 错误处理和优化

# 5.18.3 Lighthouse

## 概述

Lighthouse 是 Laravel 的 GraphQL 库，提供了强大的 GraphQL 功能。理解 Lighthouse 的使用方法、掌握 Schema 定义、实现 Laravel 集成，对于在 Laravel 应用中构建 GraphQL API 至关重要。本节详细介绍 Lighthouse 概述、安装配置、Schema 定义、Laravel 集成等内容，帮助零基础学员掌握 Lighthouse 的使用。

Lighthouse 是 Laravel 生态中最流行的 GraphQL 库，提供了与 Laravel 深度集成的 GraphQL 功能。掌握 Lighthouse 的使用可以在 Laravel 应用中快速构建 GraphQL API。

**主要内容**：
- Lighthouse 概述（Lighthouse 的功能、Lighthouse 的优势、使用场景）
- 安装配置（Composer 安装、基本配置、路由配置）
- Schema 定义（类型定义、字段定义、查询定义、变更定义）
- Laravel 集成（模型集成、认证集成、授权集成、验证集成）
- 实际应用示例和最佳实践

## 特性

- **Laravel 集成**：深度集成 Laravel
- **功能完整**：功能完整且强大
- **易于使用**：易于使用和扩展
- **文档完善**：文档完善
- **社区活跃**：社区活跃

## Lighthouse 概述

### Lighthouse 的功能

1. **GraphQL 服务器**：实现 GraphQL 服务器
2. **Schema 定义**：定义 GraphQL Schema
3. **Laravel 集成**：与 Laravel 深度集成
4. **类型系统**：强类型系统

### Lighthouse 的优势

1. **Laravel 集成**：深度集成 Laravel
2. **易于使用**：简洁的 API
3. **功能完整**：功能完整
4. **文档完善**：文档完善

### 使用场景

1. **Laravel 应用**：Laravel 应用的 GraphQL API
2. **快速开发**：快速开发 GraphQL API
3. **Laravel 生态**：利用 Laravel 生态

## 安装配置

### Composer 安装

**安装命令**：
```bash
composer require nuwave/lighthouse
```

### 基本配置

**发布配置**：
```bash
php artisan vendor:publish --tag=lighthouse-config
```

**配置文件** (`config/lighthouse.php`)：
```php
<?php
return [
    'schema' => [
        'register' => base_path('graphql/schema.graphql'),
    ],
    'namespaces' => [
        'models' => 'App\\Models',
        'queries' => 'App\\GraphQL\\Queries',
        'mutations' => 'App\\GraphQL\\Mutations',
    ],
];
```

### 路由配置

**示例** (`routes/api.php`)：
```php
<?php
use Illuminate\Support\Facades\Route;

Route::post('/graphql', [\Nuwave\Lighthouse\Http\Controllers\GraphQLController::class, 'query']);
```

## Schema 定义

### 类型定义

**示例** (`graphql/schema.graphql`)：
```graphql
type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]! @hasMany
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User! @belongsTo
}
```

### 字段定义

**示例**：
```graphql
type User {
    id: ID!
    name: String!
    email: String!
    age: Int
    created_at: DateTime!
    updated_at: DateTime!
}
```

### 查询定义

**示例**：
```graphql
type Query {
    user(id: ID! @eq): User @find
    users: [User!]! @all
    posts: [Post!]! @all
}
```

### 变更定义

**示例**：
```graphql
type Mutation {
    createUser(input: CreateUserInput! @spread): User @create
    updateUser(id: ID!, input: UpdateUserInput! @spread): User @update
    deleteUser(id: ID!): User @delete
}

input CreateUserInput {
    name: String!
    email: String!
    age: Int
}

input UpdateUserInput {
    name: String
    email: String
    age: Int
}
```

## Laravel 集成

### 模型集成

**示例** (`app/Models/User.php`)：
```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = ['name', 'email', 'age'];

    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

**GraphQL Schema**：
```graphql
type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]! @hasMany
}
```

### 认证集成

**示例**：
```graphql
type Query {
    me: User @auth
    users: [User!]! @can(ability: "viewAny")
}

type Mutation {
    createUser(input: CreateUserInput!): User @create @can(ability: "create")
}
```

### 授权集成

**示例**：
```graphql
type User {
    id: ID!
    name: String!
    email: String! @can(ability: "viewEmail")
    posts: [Post!]! @hasMany @can(ability: "viewPosts")
}
```

### 验证集成

**示例**：
```graphql
input CreateUserInput {
    name: String! @rules(apply: ["required", "min:2", "max:50"])
    email: String! @rules(apply: ["required", "email"])
    age: Int @rules(apply: ["min:0", "max:120"])
}
```

## 使用场景

### Laravel 应用

- Laravel 应用的 GraphQL API
- 快速开发
- Laravel 生态利用

### 快速开发

- 快速原型开发
- 简化 Schema 定义
- 自动生成 Schema

### Laravel 生态

- 利用 Laravel 功能
- 认证和授权
- 验证和模型

## 注意事项

### Schema 定义

- **GraphQL 文件**：使用 .graphql 文件定义 Schema
- **指令使用**：正确使用 Lighthouse 指令
- **类型定义**：定义清晰的类型

### Laravel 集成

- **模型使用**：正确使用 Laravel 模型
- **认证集成**：集成 Laravel 认证
- **授权集成**：集成 Laravel 授权

### 性能考虑

- **查询优化**：优化查询性能
- **N+1 问题**：解决 N+1 问题
- **缓存使用**：使用缓存

### 错误处理

- **验证错误**：处理验证错误
- **授权错误**：处理授权错误
- **系统错误**：处理系统错误

## 常见问题

### 如何安装 Lighthouse？

使用 Composer 安装：
```bash
composer require nuwave/lighthouse
```

### 如何定义 Schema？

使用 .graphql 文件定义 Schema，使用 Lighthouse 指令。

### 如何集成 Laravel？

Lighthouse 自动集成 Laravel 模型、认证、授权等功能。

### 如何优化性能？

使用 DataLoader、缓存、查询优化等方法。

## 最佳实践

### 使用 GraphQL 文件定义 Schema

- 使用 .graphql 文件
- 使用 Lighthouse 指令
- 保持 Schema 清晰

### 利用 Laravel 功能

- 使用 Laravel 模型
- 集成认证和授权
- 使用验证规则

### 优化查询性能

- 使用 DataLoader
- 实现缓存
- 优化查询

### 处理错误

- 处理验证错误
- 处理授权错误
- 记录错误日志

## 相关章节

- **[5.18.1 GraphQL 概述](section-01-overview.md)**：了解 GraphQL 概述的详细内容
- **[5.18.2 GraphQLite](section-02-graphqlite.md)**：了解 GraphQLite 的详细内容
- **[5.18.4 GraphQL 最佳实践](section-04-best-practices.md)**：了解 GraphQL 最佳实践的详细内容

## 练习任务

1. **安装和配置 Lighthouse**
   - Composer 安装
   - 基本配置
   - 路由配置

2. **实现基本 Schema**
   - 类型定义
   - 字段定义
   - 查询定义

3. **实现 Laravel 集成**
   - 模型集成
   - 认证集成
   - 授权集成

4. **实现查询和变更**
   - Query 查询
   - Mutation 变更
   - 输入验证

5. **实现完整的 GraphQL API**
   - Schema 设计
   - Laravel 集成
   - 错误处理和优化

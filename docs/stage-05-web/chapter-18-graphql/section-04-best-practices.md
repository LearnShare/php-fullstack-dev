# 5.18.4 GraphQL 最佳实践

## 概述

GraphQL API 的设计需要遵循最佳实践以确保性能、安全性和可维护性。理解 GraphQL 设计原则、掌握查询优化、实现错误处理和安全考虑，对于构建高质量的 GraphQL API 至关重要。本节总结 GraphQL 设计的最佳实践，包括设计原则、查询优化、错误处理、安全考虑等内容，帮助零基础学员设计高质量的 GraphQL API。

遵循最佳实践可以确保 GraphQL API 的性能、安全性和可维护性。理解这些最佳实践对于设计和实现 GraphQL API 至关重要。

**主要内容**：
- GraphQL 设计原则（Schema 设计原则、查询设计原则、变更设计原则、类型设计原则）
- 查询优化（查询复杂度限制、DataLoader 使用、批量查询、缓存策略）
- 错误处理（错误分类、错误格式、错误处理、错误日志）
- 安全考虑（查询限制、权限控制、输入验证、速率限制）
- 实际应用示例和最佳实践

## 特性

- **高性能**：优化的查询性能
- **安全可靠**：安全的设计和实现
- **易于维护**：易于维护和扩展
- **标准化**：遵循标准和规范
- **可扩展**：支持水平扩展

## GraphQL 设计原则

### Schema 设计原则

1. **清晰命名**：使用清晰的类型和字段名
2. **类型安全**：使用强类型系统
3. **向后兼容**：保持向后兼容
4. **文档完善**：完善的 Schema 文档

### 查询设计原则

1. **查询深度限制**：限制查询深度
2. **查询复杂度限制**：限制查询复杂度
3. **字段选择**：允许字段选择
4. **查询优化**：优化查询性能

### 变更设计原则

1. **幂等性**：设计幂等的变更
2. **输入验证**：验证所有输入
3. **错误处理**：处理变更错误
4. **事务处理**：使用事务处理

### 类型设计原则

1. **类型清晰**：定义清晰的类型
2. **类型复用**：复用类型定义
3. **类型扩展**：支持类型扩展
4. **类型文档**：文档化类型

## 查询优化

### 查询复杂度限制

**示例**：
```php
<?php
declare(strict_types=1);

use GraphQL\GraphQL;
use GraphQL\Validator\Rules\QueryComplexity;

$rules = [
    new QueryComplexity(100),  // 最大复杂度 100
];

$result = GraphQL::executeQuery(
    $schema,
    $query,
    null,
    null,
    $variables,
    null,
    $rules
);
```

### DataLoader 使用

**示例**：
```php
<?php
declare(strict_types=1);

use GraphQL\Executor\Promise\Promise;
use GraphQL\Executor\Promise\PromiseAdapter;

class UserDataLoader
{
    private array $batch = [];
    private PromiseAdapter $promiseAdapter;

    public function load(int $id): Promise
    {
        $this->batch[] = $id;
        
        return $this->promiseAdapter->createFulfilled(
            $this->loadUser($id)
        );
    }

    public function loadMany(array $ids): Promise
    {
        // 批量加载用户
        $users = $this->loadUsersByIds($ids);
        
        return $this->promiseAdapter->createFulfilled($users);
    }
}
```

### 批量查询

**示例**：
```php
<?php
declare(strict_types=1);

class BatchLoader
{
    private array $pending = [];
    private \React\EventLoop\LoopInterface $loop;

    public function load(string $key, callable $loader): Promise
    {
        if (!isset($this->pending[$key])) {
            $this->pending[$key] = [];
        }
        
        $this->pending[$key][] = $loader;
        
        // 延迟执行批量加载
        $this->loop->addTimer(0.01, function () use ($key) {
            $this->executeBatch($key);
        });
    }

    private function executeBatch(string $key): void
    {
        $loaders = $this->pending[$key] ?? [];
        unset($this->pending[$key]);
        
        // 批量执行
        foreach ($loaders as $loader) {
            $loader();
        }
    }
}
```

### 缓存策略

**示例**：
```php
<?php
declare(strict_types=1);

class QueryCache
{
    private array $cache = [];
    private int $ttl = 3600;

    public function get(string $query, array $variables = []): ?array
    {
        $key = $this->generateKey($query, $variables);
        
        if (isset($this->cache[$key])) {
            $item = $this->cache[$key];
            if (time() - $item['time'] < $this->ttl) {
                return $item['data'];
            }
            unset($this->cache[$key]);
        }
        
        return null;
    }

    public function set(string $query, array $variables, array $data): void
    {
        $key = $this->generateKey($query, $variables);
        $this->cache[$key] = [
            'data' => $data,
            'time' => time(),
        ];
    }

    private function generateKey(string $query, array $variables): string
    {
        return md5($query . serialize($variables));
    }
}
```

## 错误处理

### 错误分类

**示例**：
```php
<?php
declare(strict_types=1);

enum GraphQLErrorType: string
{
    case VALIDATION = 'VALIDATION_ERROR';
    case AUTHENTICATION = 'AUTHENTICATION_ERROR';
    case AUTHORIZATION = 'AUTHORIZATION_ERROR';
    case NOT_FOUND = 'NOT_FOUND_ERROR';
    case INTERNAL = 'INTERNAL_ERROR';
}
```

### 错误格式

**示例**：
```php
<?php
declare(strict_types=1);

function formatGraphQLError(\Exception $e): array
{
    return [
        'message' => $e->getMessage(),
        'extensions' => [
            'code' => $this->getErrorCode($e),
            'category' => $this->getErrorCategory($e),
        ],
    ];
}
```

### 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

try {
    $result = GraphQL::executeQuery($schema, $query);
    
    if ($result->errors) {
        $formattedErrors = [];
        foreach ($result->errors as $error) {
            $formattedErrors[] = [
                'message' => $error->getMessage(),
                'extensions' => $error->getExtensions(),
            ];
        }
        
        return [
            'data' => $result->data,
            'errors' => $formattedErrors,
        ];
    }
    
    return ['data' => $result->data];
} catch (\Exception $e) {
    return [
        'errors' => [
            [
                'message' => 'Internal server error',
                'extensions' => [
                    'code' => 'INTERNAL_ERROR',
                ],
            ],
        ],
    ];
}
```

### 错误日志

**示例**：
```php
<?php
declare(strict_types=1);

function logGraphQLError(\Exception $e, string $query): void
{
    error_log(json_encode([
        'timestamp' => date('c'),
        'query' => $query,
        'error' => $e->getMessage(),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
        'trace' => $e->getTraceAsString(),
    ]));
}
```

## 安全考虑

### 查询限制

**示例**：
```php
<?php
declare(strict_types=1);

class QueryLimiter
{
    private int $maxDepth = 10;
    private int $maxComplexity = 1000;

    public function validate(string $query): void
    {
        $depth = $this->calculateDepth($query);
        if ($depth > $this->maxDepth) {
            throw new \Exception('Query depth exceeds maximum');
        }
        
        $complexity = $this->calculateComplexity($query);
        if ($complexity > $this->maxComplexity) {
            throw new \Exception('Query complexity exceeds maximum');
        }
    }
}
```

### 权限控制

**示例**：
```graphql
type User {
    id: ID!
    name: String!
    email: String! @can(ability: "viewEmail")
    posts: [Post!]! @can(ability: "viewPosts")
}

type Query {
    users: [User!]! @can(ability: "viewAny")
    user(id: ID!): User @can(ability: "view")
}
```

### 输入验证

**示例**：
```graphql
input CreateUserInput {
    name: String! @rules(apply: ["required", "min:2", "max:50"])
    email: String! @rules(apply: ["required", "email"])
    age: Int @rules(apply: ["min:0", "max:120"])
}
```

### 速率限制

**示例**：
```php
<?php
declare(strict_types=1);

class RateLimiter
{
    private int $maxQueries = 100;
    private int $window = 60;  // 60 秒

    public function check(string $clientId): bool
    {
        $key = "graphql:rate_limit:{$clientId}";
        $count = $this->getCount($key);
        
        if ($count >= $this->maxQueries) {
            return false;
        }
        
        $this->incrementCount($key);
        return true;
    }
}
```

## 使用场景

### 所有 GraphQL API

- Schema 设计
- 查询优化
- 错误处理
- 安全考虑

### 高性能 API

- 查询优化
- 缓存策略
- 批量查询

### 安全 API

- 权限控制
- 输入验证
- 速率限制

### 生产环境

- 错误处理
- 性能监控
- 安全防护

## 注意事项

### Schema 设计

- **清晰命名**：使用清晰的命名
- **类型安全**：使用强类型
- **向后兼容**：保持向后兼容

### 查询优化

- **复杂度限制**：限制查询复杂度
- **DataLoader**：使用 DataLoader
- **缓存策略**：实现缓存策略

### 错误处理

- **错误分类**：分类错误类型
- **错误格式**：统一错误格式
- **错误日志**：记录错误日志

### 安全考虑

- **查询限制**：限制查询深度和复杂度
- **权限控制**：实现权限控制
- **输入验证**：验证所有输入

## 常见问题

### 如何优化 GraphQL 查询？

使用 DataLoader、批量查询、缓存策略等方法优化查询。

### 如何处理 N+1 问题？

使用 DataLoader 批量加载数据，避免 N+1 问题。

### 如何实现权限控制？

使用字段级和查询级权限控制，集成认证和授权系统。

### 如何限制查询复杂度？

使用查询复杂度限制规则，限制查询深度和复杂度。

## 最佳实践

### 设计清晰的 Schema

- 使用清晰的命名
- 使用强类型系统
- 保持向后兼容

### 优化查询性能

- 使用 DataLoader
- 实现缓存策略
- 优化查询复杂度

### 实现权限控制

- 字段级权限
- 查询级权限
- 变更级权限

### 处理错误

- 分类错误类型
- 统一错误格式
- 记录错误日志

## 相关章节

- **[5.18.1 GraphQL 概述](section-01-overview.md)**：了解 GraphQL 概述的详细内容
- **[5.18.2 GraphQLite](section-02-graphqlite.md)**：了解 GraphQLite 的详细内容
- **[5.18.3 Lighthouse](section-03-lighthouse.md)**：了解 Lighthouse 的详细内容

## 练习任务

1. **设计 GraphQL Schema**
   - 类型定义
   - 字段定义
   - 查询和变更定义

2. **实现查询优化**
   - DataLoader 使用
   - 批量查询
   - 缓存策略

3. **实现错误处理**
   - 错误分类
   - 错误格式
   - 错误日志

4. **实现安全控制**
   - 查询限制
   - 权限控制
   - 输入验证

5. **实现完整的 GraphQL API**
   - Schema 设计
   - 查询优化
   - 错误处理和安全

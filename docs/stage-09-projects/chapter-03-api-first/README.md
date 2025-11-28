# 9.3 API-First 企业服务

## 目标

- 设计符合 OpenAPI 规范的 API。
- 自动生成 API 文档。
- 实现 SDK 自动生成。
- 建立完整的测试体系。

## 项目概述

本项目是一个 API-First 的企业级服务，采用 OpenAPI 规范定义 API，自动生成文档和 SDK，建立完整的测试体系。适合需要对外提供 API 服务或构建微服务架构的场景。

## OpenAPI 规范

### 完整的 API 定义

```yaml
openapi: 3.1.0
info:
  title: User Management API
  version: 1.0.0
  description: 用户管理 API，提供完整的用户 CRUD 操作
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

paths:
  /users:
    get:
      summary: 获取用户列表
      description: 分页获取用户列表，支持筛选和排序
      operationId: listUsers
      tags:
        - Users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
        - name: status
          in: query
          schema:
            type: string
            enum: [active, inactive]
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'
    post:
      summary: 创建用户
      operationId: createUser
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: 创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '422':
          $ref: '#/components/responses/ValidationError'

  /users/{id}:
    get:
      summary: 获取用户详情
      operationId: getUser
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        status:
          type: string
          enum: [active, inactive]
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time
      required:
        - id
        - email
        - name

    CreateUserRequest:
      type: object
      properties:
        email:
          type: string
          format: email
        name:
          type: string
          minLength: 3
        password:
          type: string
          minLength: 8
      required:
        - email
        - name
        - password

    PaginationMeta:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        total_pages:
          type: integer

  responses:
    BadRequest:
      description: 请求错误
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    NotFound:
      description: 资源不存在
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    ValidationError:
      description: 验证失败
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ValidationError'

    Error:
      type: object
      properties:
        error:
          type: object
          properties:
            message:
              type: string
            code:
              type: string

    ValidationError:
      type: object
      properties:
        error:
          type: object
          properties:
            message:
              type: string
            errors:
              type: object
              additionalProperties:
                type: array
                items:
                  type: string

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

## API 实现

### 路由处理

```php
<?php
declare(strict_types=1);

namespace App\API;

use App\Application\UserService;
use App\Infrastructure\UserRepository;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {}
    
    public function listUsers(array $query): array
    {
        $page = (int) ($query['page'] ?? 1);
        $limit = (int) ($query['limit'] ?? 20);
        $status = $query['status'] ?? null;
        
        $result = $this->userService->listUsers($page, $limit, $status);
        
        return [
            'data' => $result['users'],
            'meta' => [
                'page' => $page,
                'limit' => $limit,
                'total' => $result['total'],
                'total_pages' => (int) ceil($result['total'] / $limit),
            ],
        ];
    }
    
    public function createUser(array $data): array
    {
        // 验证数据
        $errors = $this->validateCreateUser($data);
        if (!empty($errors)) {
            http_response_code(422);
            return [
                'error' => [
                    'message' => 'Validation failed',
                    'errors' => $errors,
                ],
            ];
        }
        
        // 创建用户
        $user = $this->userService->createUser(
            $data['email'],
            $data['name'],
            $data['password']
        );
        
        http_response_code(201);
        return $this->userToArray($user);
    }
    
    public function getUser(string $id): array
    {
        $user = $this->userService->getUser($id);
        
        if ($user === null) {
            http_response_code(404);
            return [
                'error' => [
                    'message' => 'User not found',
                    'code' => 'USER_NOT_FOUND',
                ],
            ];
        }
        
        return $this->userToArray($user);
    }
    
    private function validateCreateUser(array $data): array
    {
        $errors = [];
        
        if (empty($data['email']) || !filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors['email'] = ['Email is required and must be valid'];
        }
        
        if (empty($data['name']) || strlen($data['name']) < 3) {
            $errors['name'] = ['Name is required and must be at least 3 characters'];
        }
        
        if (empty($data['password']) || strlen($data['password']) < 8) {
            $errors['password'] = ['Password is required and must be at least 8 characters'];
        }
        
        return $errors;
    }
    
    private function userToArray($user): array
    {
        return [
            'id' => $user->getId(),
            'email' => $user->getEmail(),
            'name' => $user->getName(),
            'status' => $user->getStatus(),
            'created_at' => $user->getCreatedAt()->toIso8601String(),
            'updated_at' => $user->getUpdatedAt()->toIso8601String(),
        ];
    }
}
```

## 自动生成 API 文档

### 使用 Swagger UI

```php
<?php
declare(strict_types=1);

// public/docs.php

require __DIR__ . '/../vendor/autoload.php';

use OpenApi\Generator;

// 扫描代码生成 OpenAPI 规范
$openapi = Generator::scan([__DIR__ . '/../src']);

// 输出 JSON
header('Content-Type: application/json');
echo $openapi->toJson();
```

### 使用 Swagger UI 展示

```html
<!DOCTYPE html>
<html>
<head>
    <title>API Documentation</title>
    <link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist@5/swagger-ui.css" />
</head>
<body>
    <div id="swagger-ui"></div>
    <script src="https://unpkg.com/swagger-ui-dist@5/swagger-ui-bundle.js"></script>
    <script>
        SwaggerUIBundle({
            url: '/api/openapi.json',
            dom_id: '#swagger-ui',
        });
    </script>
</body>
</html>
```

## SDK 自动生成

### 使用 openapi-generator

```bash
# 安装 openapi-generator
npm install @openapitools/openapi-generator-cli -g

# 生成 PHP SDK
openapi-generator-cli generate \
  -i openapi.yaml \
  -g php \
  -o ./sdk/php \
  --additional-properties=packageName=ExampleSDK

# 生成 JavaScript SDK
openapi-generator-cli generate \
  -i openapi.yaml \
  -g javascript \
  -o ./sdk/javascript
```

### PHP SDK 使用示例

```php
<?php
require 'vendor/autoload.php';

use ExampleSDK\Api\UsersApi;
use ExampleSDK\Configuration;

$config = Configuration::getDefaultConfiguration()
    ->setHost('https://api.example.com/v1')
    ->setAccessToken('your-token');

$api = new UsersApi(null, $config);

// 获取用户列表
$users = $api->listUsers(['page' => 1, 'limit' => 20]);

// 创建用户
$user = $api->createUser([
    'email' => 'alice@example.com',
    'name' => 'Alice',
    'password' => 'password123',
]);
```

## 测试体系

### 单元测试

```php
<?php
declare(strict_types=1);

namespace Tests\Unit\API;

use PHPUnit\Framework\TestCase;
use App\API\UserController;
use App\Application\UserService;

class UserControllerTest extends TestCase
{
    private UserController $controller;
    private UserService $userService;
    
    protected function setUp(): void
    {
        $this->userService = $this->createMock(UserService::class);
        $this->controller = new UserController($this->userService);
    }
    
    public function testListUsers(): void
    {
        $this->userService->expects($this->once())
            ->method('listUsers')
            ->with(1, 20, null)
            ->willReturn([
                'users' => [],
                'total' => 0,
            ]);
        
        $result = $this->controller->listUsers(['page' => 1, 'limit' => 20]);
        
        $this->assertArrayHasKey('data', $result);
        $this->assertArrayHasKey('meta', $result);
    }
}
```

### 集成测试

```php
<?php
declare(strict_types=1);

namespace Tests\Integration\API;

use PHPUnit\Framework\TestCase;
use GuzzleHttp\Client;

class UserApiTest extends TestCase
{
    private Client $client;
    
    protected function setUp(): void
    {
        $this->client = new Client([
            'base_uri' => 'http://localhost:8000',
            'headers' => [
                'Authorization' => 'Bearer test-token',
            ],
        ]);
    }
    
    public function testCreateUser(): void
    {
        $response = $this->client->post('/api/v1/users', [
            'json' => [
                'email' => 'test@example.com',
                'name' => 'Test User',
                'password' => 'password123',
            ],
        ]);
        
        $this->assertEquals(201, $response->getStatusCode());
        
        $data = json_decode($response->getBody()->getContents(), true);
        $this->assertArrayHasKey('id', $data);
        $this->assertEquals('test@example.com', $data['email']);
    }
}
```

### E2E 测试

```php
<?php
declare(strict_types=1);

namespace Tests\E2E;

use PHPUnit\Framework\TestCase;
use GuzzleHttp\Client;

class UserE2ETest extends TestCase
{
    private Client $client;
    private ?string $userId = null;
    
    protected function setUp(): void
    {
        $this->client = new Client([
            'base_uri' => 'https://api.example.com/v1',
            'headers' => [
                'Authorization' => 'Bearer ' . getenv('API_TOKEN'),
            ],
        ]);
    }
    
    public function testUserLifecycle(): void
    {
        // 1. 创建用户
        $response = $this->client->post('/users', [
            'json' => [
                'email' => 'e2e-test@example.com',
                'name' => 'E2E Test',
                'password' => 'password123',
            ],
        ]);
        
        $this->assertEquals(201, $response->getStatusCode());
        $user = json_decode($response->getBody()->getContents(), true);
        $this->userId = $user['id'];
        
        // 2. 获取用户
        $response = $this->client->get("/users/{$this->userId}");
        $this->assertEquals(200, $response->getStatusCode());
        
        // 3. 更新用户
        $response = $this->client->patch("/users/{$this->userId}", [
            'json' => ['name' => 'Updated Name'],
        ]);
        $this->assertEquals(200, $response->getStatusCode());
        
        // 4. 删除用户
        $response = $this->client->delete("/users/{$this->userId}");
        $this->assertEquals(204, $response->getStatusCode());
    }
    
    protected function tearDown(): void
    {
        // 清理测试数据
        if ($this->userId !== null) {
            try {
                $this->client->delete("/users/{$this->userId}");
            } catch (\Exception $e) {
                // 忽略删除失败
            }
        }
    }
}
```

## CI/CD 集成

### GitHub Actions 配置

```yaml
name: API Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install
      - run: composer test
      
  generate-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate OpenAPI
        run: |
          php public/docs.php > openapi.json
      - name: Upload docs
        uses: actions/upload-artifact@v3
        with:
          name: openapi-spec
          path: openapi.json
```

## 练习

1. **设计完整的 RESTful API**
   - 定义所有资源端点
   - 设计请求和响应格式
   - 实现错误处理

2. **使用 OpenAPI 定义 API 规范**
   - 编写完整的 OpenAPI 规范文件
   - 定义所有 schemas 和 responses
   - 添加认证和安全配置

3. **自动生成 API 文档**
   - 集成 Swagger UI
   - 自动扫描代码生成文档
   - 部署文档到生产环境

4. **实现 SDK 生成**
   - 配置 openapi-generator
   - 生成多语言 SDK（PHP、JavaScript、Python）
   - 发布 SDK 到包管理器

5. **建立 E2E 测试体系**
   - 编写单元测试
   - 编写集成测试
   - 编写 E2E 测试
   - 集成到 CI/CD 流程

6. **API 版本管理**
   - 实现 API 版本控制
   - 处理向后兼容性
   - 版本迁移策略

7. **API 监控和日志**
   - 记录所有 API 请求
   - 监控 API 性能
   - 设置告警机制

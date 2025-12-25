# 4.6.3 API 文档与测试

## 概述

API 文档和测试是确保 API 质量的关键。本节详细介绍 OpenAPI 规范、API 文档生成、API 测试工具、Mock 数据，以及最佳实践。

## OpenAPI 规范

### 基础结构

```yaml
openapi: 3.1.0
info:
  title: User Management API
  version: 1.0.0
  description: 用户管理 API
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: 获取用户列表
      operationId: listUsers
      parameters:
        - name: page
          in: query
          schema:
            type: integer
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
```

### PHP 注解生成

```php
<?php
declare(strict_types=1);

use OpenApi\Attributes as OA;

#[OA\Get(
    path: '/api/v1/users',
    summary: '获取用户列表',
    operationId: 'listUsers',
    tags: ['Users'],
    responses: [
        new OA\Response(response: 200, description: '成功'),
    ]
)]
class UserController
{
    public function index(): JsonResponse
    {
        // 实现
    }
}
```

## API 文档生成

### Swagger UI

```php
<?php
// 使用 L5-Swagger
// 自动生成 Swagger 文档
// 访问 /api/documentation 查看文档
```

### ReDoc

```html
<!-- 使用 ReDoc 展示 OpenAPI 文档 -->
<script src="https://cdn.jsdelivr.net/npm/redoc/bundles/redoc.standalone.js"></script>
<script>
  Redoc.init('/api/openapi.yaml', {}, document.getElementById('redoc'));
</script>
```

## API 测试

### Postman 集合

```json
{
  "info": {
    "name": "User API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Get Users",
      "request": {
        "method": "GET",
        "url": "{{base_url}}/api/v1/users"
      }
    }
  ]
}
```

### PHPUnit API 测试

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class ApiTest extends TestCase
{
    public function testGetUsers(): void
    {
        $response = $this->makeRequest('GET', '/api/v1/users');
        
        $this->assertEquals(200, $response->getStatusCode());
        $data = json_decode($response->getBody(), true);
        $this->assertArrayHasKey('data', $data);
    }

    private function makeRequest(string $method, string $path): Response
    {
        // 实现请求逻辑
    }
}
```

## Mock 数据

### 使用 Mock 服务

```php
<?php
declare(strict_types=1);

class MockApi
{
    public function getUsers(): array
    {
        return [
            ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'],
            ['id' => 2, 'name' => 'Bob', 'email' => 'bob@example.com'],
        ];
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ApiDocumentation
{
    public function generateOpenApiSpec(): array
    {
        return [
            'openapi' => '3.1.0',
            'info' => [
                'title' => 'User API',
                'version' => '1.0.0',
            ],
            'paths' => [
                '/users' => [
                    'get' => [
                        'summary' => '获取用户列表',
                        'responses' => [
                            '200' => [
                                'description' => '成功',
                            ],
                        ],
                    ],
                ],
            ],
        ];
    }
}
```

## 注意事项

1. **保持文档更新**：代码变更时同步更新文档
2. **提供示例**：文档中包含请求和响应示例
3. **测试覆盖**：为所有 API 端点编写测试
4. **Mock 数据**：使用 Mock 数据进行开发和测试

## 练习

1. 为你的 API 编写 OpenAPI 规范文档。

2. 使用 Swagger UI 或 ReDoc 展示 API 文档。

3. 编写 API 测试用例，覆盖所有端点。

4. 创建 Mock 数据服务，用于开发和测试。

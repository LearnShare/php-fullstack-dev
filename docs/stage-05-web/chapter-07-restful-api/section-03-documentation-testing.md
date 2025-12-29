# 5.7.3 API 文档与测试

## 概述

API 文档和测试是 API 开发的重要组成部分。完整的 API 文档帮助开发者理解和使用 API，全面的测试确保 API 的可靠性和稳定性。理解 API 文档的重要性、掌握文档编写方法、了解 API 测试的基本方法，对于构建高质量的 API 至关重要。本节详细介绍 API 文档的重要性、OpenAPI/Swagger 规范、API 文档编写、API 测试方法、测试用例设计等内容，帮助零基础学员掌握 API 文档和测试技术。

API 文档是 API 的"说明书"，好的文档可以大大提高 API 的易用性。API 测试则确保 API 的功能正确、性能稳定、安全可靠。

**主要内容**：
- API 文档重要性（为什么需要文档、文档的作用、文档的受众）
- OpenAPI/Swagger 规范（OpenAPI 规范、Swagger 工具、文档格式、文档生成）
- API 文档编写（接口描述、参数说明、响应格式、示例代码）
- API 测试方法（测试方法、测试工具、测试用例、自动化测试）
- 测试用例设计（功能测试、边界测试、错误测试、性能测试）
- 实际应用示例和最佳实践

## 特性

- **标准化**：使用 OpenAPI 标准规范
- **自动化**：支持文档自动生成
- **交互式**：支持交互式文档
- **全面测试**：覆盖功能、性能、安全
- **持续更新**：文档与代码同步更新

## API 文档重要性

### 为什么需要文档

1. **降低使用门槛**：帮助开发者快速理解和使用 API
2. **减少沟通成本**：减少开发者和 API 提供者之间的沟通
3. **提高开发效率**：提供清晰的示例和说明
4. **减少错误**：明确参数和响应格式，减少使用错误

### 文档的作用

1. **接口说明**：说明每个接口的用途和用法
2. **参数说明**：详细说明请求参数和响应格式
3. **示例代码**：提供完整的示例代码
4. **错误处理**：说明错误情况和处理方法

### 文档的受众

1. **前端开发者**：需要了解如何调用 API
2. **移动开发者**：需要了解 API 接口
3. **第三方开发者**：需要集成 API
4. **测试人员**：需要了解 API 功能进行测试

## OpenAPI/Swagger 规范

### OpenAPI 规范

OpenAPI（原 Swagger）是一个用于描述 RESTful API 的规范标准。

**OpenAPI 3.0 基本结构**：
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: User management API
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: Get user list
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
```

### Swagger 工具

**Swagger UI**：交互式 API 文档界面

**安装**：
```bash
composer require zircote/swagger-php
```

**使用**：
```php
<?php
declare(strict_types=1);

/**
 * @OA\Info(
 *     title="User API",
 *     version="1.0.0"
 * )
 * @OA\Server(
 *     url="https://api.example.com/v1"
 * )
 */
class UserController
{
    /**
     * @OA\Get(
     *     path="/users",
     *     summary="Get user list",
     *     @OA\Response(
     *         response=200,
     *         description="Success",
     *         @OA\JsonContent(
     *             type="array",
     *             @OA\Items(ref="#/components/schemas/User")
     *         )
     *     )
     * )
     */
    public function index(): void
    {
        // ...
    }
}
```

### 文档格式

**YAML 格式**（推荐）：
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: Get user list
```

**JSON 格式**：
```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "User API",
    "version": "1.0.0"
  },
  "paths": {
    "/users": {
      "get": {
        "summary": "Get user list"
      }
    }
  }
}
```

### 文档生成

**从代码注释生成**：
```bash
vendor/bin/openapi app/ -o public/api-docs/openapi.yaml
```

**从代码生成**（使用 swagger-php）：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

$openapi = \OpenApi\scan(__DIR__ . '/app');
header('Content-Type: application/json');
echo $openapi->toJson();
```

## API 文档编写

### 接口描述

**示例**：
```yaml
paths:
  /users:
    get:
      summary: 获取用户列表
      description: 获取系统中的所有用户列表，支持分页和筛选
      operationId: getUsers
      tags:
        - Users
      parameters:
        - name: page
          in: query
          description: 页码
          required: false
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          description: 每页数量
          required: false
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: 成功返回用户列表
```

### 参数说明

**示例**：
```yaml
parameters:
  - name: id
    in: path
    description: 用户 ID
    required: true
    schema:
      type: integer
      example: 123
  - name: name
    in: query
    description: 用户名称（模糊搜索）
    required: false
    schema:
      type: string
      example: "John"
requestBody:
  required: true
  content:
    application/json:
      schema:
        type: object
        required:
          - name
          - email
        properties:
          name:
            type: string
            description: 用户名称
            example: "John Doe"
          email:
            type: string
            format: email
            description: 用户邮箱
            example: "john@example.com"
```

### 响应格式

**示例**：
```yaml
responses:
  '200':
    description: 成功
    content:
      application/json:
        schema:
          type: object
          properties:
            success:
              type: boolean
              example: true
            data:
              type: array
              items:
                $ref: '#/components/schemas/User'
  '400':
    description: 请求参数错误
    content:
      application/json:
        schema:
          type: object
          properties:
            success:
              type: boolean
              example: false
            message:
              type: string
              example: "Invalid request parameters"
  '404':
    description: 资源不存在
    content:
      application/json:
        schema:
          type: object
          properties:
            success:
              type: boolean
              example: false
            message:
              type: string
              example: "Resource not found"
```

### 示例代码

**示例**：
```yaml
examples:
  curl:
    summary: cURL 示例
    value: |
      curl -X GET "https://api.example.com/v1/users" \
        -H "Authorization: Bearer {token}"
  
  php:
    summary: PHP 示例
    value: |
      <?php
      $ch = curl_init('https://api.example.com/v1/users');
      curl_setopt($ch, CURLOPT_HTTPHEADER, [
          'Authorization: Bearer {token}'
      ]);
      $response = curl_exec($ch);
      curl_close($ch);
  
  javascript:
    summary: JavaScript 示例
    value: |
      fetch('https://api.example.com/v1/users', {
          headers: {
              'Authorization': 'Bearer {token}'
          }
      })
      .then(response => response.json())
      .then(data => console.log(data));
```

## API 测试方法

### 测试方法

**测试类型**：
1. **功能测试**：测试 API 功能是否正确
2. **边界测试**：测试边界条件
3. **错误测试**：测试错误处理
4. **性能测试**：测试 API 性能
5. **安全测试**：测试 API 安全性

### 测试工具

**常用工具**：
- **Postman**：图形化 API 测试工具
- **curl**：命令行工具
- **PHPUnit**：PHP 单元测试框架
- **Guzzle**：PHP HTTP 客户端

### 测试用例

**示例**：
```php
<?php
declare(strict_types=1);

class UserApiTest
{
    public function testGetUsers(): void
    {
        // 测试获取用户列表
        $response = $this->get('/api/v1/users');
        $this->assertEquals(200, $response->getStatusCode());
        
        $data = json_decode($response->getBody(), true);
        $this->assertArrayHasKey('data', $data);
        $this->assertIsArray($data['data']);
    }
    
    public function testCreateUser(): void
    {
        // 测试创建用户
        $response = $this->post('/api/v1/users', [
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);
        
        $this->assertEquals(201, $response->getStatusCode());
        
        $data = json_decode($response->getBody(), true);
        $this->assertArrayHasKey('data', $data);
        $this->assertEquals('Test User', $data['data']['name']);
    }
    
    public function testGetUserNotFound(): void
    {
        // 测试获取不存在的用户
        $response = $this->get('/api/v1/users/99999');
        $this->assertEquals(404, $response->getStatusCode());
    }
}
```

### 自动化测试

**示例**：
```php
<?php
declare(strict_types=1);

class ApiTestSuite
{
    public function runAllTests(): void
    {
        $tests = [
            'testGetUsers',
            'testCreateUser',
            'testUpdateUser',
            'testDeleteUser',
        ];
        
        foreach ($tests as $test) {
            $this->runTest($test);
        }
    }
    
    private function runTest(string $testName): void
    {
        echo "Running {$testName}...\n";
        // 执行测试
    }
}
```

## 测试用例设计

### 功能测试

**测试内容**：
- 正常流程测试
- 参数验证测试
- 业务逻辑测试

**示例**：
```php
<?php
declare(strict_types=1);

public function testGetUsersWithPagination(): void
{
    // 测试分页功能
    $response = $this->get('/api/v1/users?page=1&limit=10');
    $this->assertEquals(200, $response->getStatusCode());
    
    $data = json_decode($response->getBody(), true);
    $this->assertCount(10, $data['data']);
}
```

### 边界测试

**测试内容**：
- 最大值测试
- 最小值测试
- 空值测试
- 特殊字符测试

**示例**：
```php
<?php
declare(strict_types=1);

public function testCreateUserWithMaxLengthName(): void
{
    // 测试最大长度名称
    $longName = str_repeat('a', 255);
    $response = $this->post('/api/v1/users', [
        'name' => $longName,
        'email' => 'test@example.com',
    ]);
    
    $this->assertEquals(201, $response->getStatusCode());
}

public function testCreateUserWithEmptyName(): void
{
    // 测试空名称
    $response = $this->post('/api/v1/users', [
        'name' => '',
        'email' => 'test@example.com',
    ]);
    
    $this->assertEquals(422, $response->getStatusCode());
}
```

### 错误测试

**测试内容**：
- 参数错误测试
- 认证错误测试
- 权限错误测试
- 资源不存在测试

**示例**：
```php
<?php
declare(strict_types=1);

public function testCreateUserWithInvalidEmail(): void
{
    // 测试无效邮箱
    $response = $this->post('/api/v1/users', [
        'name' => 'Test User',
        'email' => 'invalid-email',
    ]);
    
    $this->assertEquals(422, $response->getStatusCode());
    
    $data = json_decode($response->getBody(), true);
    $this->assertArrayHasKey('errors', $data);
    $this->assertArrayHasKey('email', $data['errors']);
}

public function testGetUserWithoutAuth(): void
{
    // 测试未认证请求
    $response = $this->get('/api/v1/users', [
        'headers' => []  // 无认证头
    ]);
    
    $this->assertEquals(401, $response->getStatusCode());
}
```

### 性能测试

**测试内容**：
- 响应时间测试
- 并发测试
- 负载测试

**示例**：
```php
<?php
declare(strict_types=1);

public function testGetUsersPerformance(): void
{
    $startTime = microtime(true);
    
    $response = $this->get('/api/v1/users');
    
    $endTime = microtime(true);
    $duration = $endTime - $startTime;
    
    $this->assertEquals(200, $response->getStatusCode());
    $this->assertLessThan(1.0, $duration, 'Response time should be less than 1 second');
}
```

## 使用场景

### API 开发

- 开发过程中编写文档
- 测试 API 功能
- 验证 API 行为

### 团队协作

- 前后端协作
- 移动端协作
- 第三方集成

### 客户端集成

- 前端应用集成
- 移动应用集成
- 第三方服务集成

### API 维护

- 版本更新
- 功能扩展
- 问题排查

## 注意事项

### 文档的及时更新

- **同步更新**：代码更新时同步更新文档
- **版本管理**：为每个版本维护文档
- **变更记录**：记录文档变更历史

### 文档的准确性

- **验证示例**：确保示例代码可以运行
- **参数说明**：确保参数说明准确
- **响应格式**：确保响应格式正确

### 示例的完整性

- **完整示例**：提供完整的示例代码
- **多种语言**：提供多种编程语言的示例
- **错误处理**：包含错误处理示例

### 测试的覆盖度

- **功能覆盖**：覆盖所有功能
- **边界覆盖**：覆盖边界条件
- **错误覆盖**：覆盖错误情况

## 常见问题

### 如何编写 API 文档？

1. **使用 OpenAPI 规范**：使用标准规范编写文档
2. **提供完整信息**：包含接口、参数、响应、示例
3. **保持更新**：及时更新文档

### OpenAPI 和 Swagger 的关系？

- **OpenAPI**：规范标准（原 Swagger 规范）
- **Swagger**：工具集（Swagger UI、Swagger Editor 等）
- **关系**：Swagger 工具实现 OpenAPI 规范

### 如何测试 API？

1. **使用 Postman**：手动测试
2. **使用 curl**：命令行测试
3. **使用 PHPUnit**：自动化测试

### 如何保持文档更新？

1. **代码注释**：在代码中添加注释
2. **自动生成**：使用工具自动生成文档
3. **定期审查**：定期审查和更新文档

## 最佳实践

### 使用 OpenAPI 规范

- 使用标准 OpenAPI 规范
- 保持文档格式统一
- 使用工具生成文档

### 提供完整示例

- 提供多种语言的示例
- 包含错误处理示例
- 确保示例可以运行

### 及时更新文档

- 代码更新时同步更新文档
- 定期审查文档准确性
- 记录文档变更历史

### 编写测试用例

- 覆盖所有功能
- 覆盖边界条件
- 覆盖错误情况
- 自动化测试

## 相关章节

- **[5.7.4 API 测试工具](section-04-api-testing.md)**：了解 API 测试工具的详细内容
- **[5.7.5 API 文档](section-05-api-documentation.md)**：了解 API 文档的详细内容
- **[5.5 请求体解析](../chapter-05-json-requests/readme.md)**：了解 API 请求的处理

## 练习任务

1. **编写 OpenAPI 文档**
   - 定义 API 结构
   - 编写接口文档
   - 生成交互式文档

2. **实现 API 测试**
   - 编写功能测试
   - 编写边界测试
   - 编写错误测试

3. **实现自动化测试**
   - 使用 PHPUnit 测试 API
   - 实现测试套件
   - 集成 CI/CD

4. **实现文档生成**
   - 从代码注释生成文档
   - 自动更新文档
   - 部署文档站点

5. **实现完整的 API 文档和测试系统**
   - 文档编写和维护
   - 测试用例设计
   - 自动化测试和文档生成

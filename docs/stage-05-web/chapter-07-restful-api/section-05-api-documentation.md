# 5.7.5 API 文档

## 概述

API 文档是 API 使用的重要参考，高质量的文档可以大大提高 API 的易用性和采用率。理解 API 文档的规范、掌握文档编写方法、使用工具生成和维护文档，对于构建成功的 API 至关重要。本节详细介绍 API 文档的编写规范、格式要求、Swagger/OpenAPI 规范、文档生成工具、文档维护方法等内容，帮助零基础学员编写高质量的 API 文档。

API 文档应该清晰、完整、准确、易于理解。好的文档应该包含 API 概述、认证说明、端点列表、请求/响应格式、错误码说明、示例代码等。

**主要内容**：
- API 文档规范（文档结构、必需内容、格式要求、示例要求）
- Swagger/OpenAPI 规范（规范版本、文档格式 YAML/JSON、工具支持、文档生成）
- 文档结构（API 概述、认证说明、端点列表、请求/响应格式、错误码说明、示例代码）
- 文档生成工具（代码注释生成、自动生成工具、文档托管、文档版本）
- 文档维护（文档更新、版本管理、变更记录、文档审查）
- 实际应用示例和最佳实践

## 特性

- **标准化**：使用 OpenAPI 标准规范
- **自动化**：支持文档自动生成
- **交互式**：支持交互式文档界面
- **多格式**：支持多种文档格式
- **易维护**：文档易于更新和维护

## API 文档规范

### 文档结构

**标准文档结构**：
1. **API 概述**：API 简介、版本信息、基础 URL
2. **认证说明**：认证方式、Token 获取、使用示例
3. **端点列表**：所有 API 端点的详细说明
4. **请求格式**：请求方法、参数、请求体格式
5. **响应格式**：响应结构、状态码、数据格式
6. **错误码说明**：错误码列表、错误处理
7. **示例代码**：多种语言的示例代码
8. **变更记录**：版本变更历史

### 必需内容

**每个端点必须包含**：
- 端点 URL
- HTTP 方法
- 功能描述
- 请求参数（路径、查询、请求体）
- 响应格式
- 状态码
- 示例请求
- 示例响应

### 格式要求

**格式要求**：
- **清晰**：使用清晰的标题和结构
- **完整**：包含所有必要信息
- **准确**：信息准确无误
- **一致**：格式保持一致

### 示例要求

**示例要求**：
- **完整**：提供完整的示例代码
- **可运行**：示例代码可以运行
- **多语言**：提供多种编程语言的示例
- **真实**：使用真实的数据示例

## Swagger/OpenAPI 规范

### 规范版本

**OpenAPI 版本**：
- **OpenAPI 3.0**：当前推荐版本
- **OpenAPI 2.0**（Swagger 2.0）：旧版本，仍在使用

### 文档格式

**YAML 格式**（推荐）：
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

### 工具支持

**OpenAPI 工具**：
- **Swagger UI**：交互式文档界面
- **Swagger Editor**：在线编辑器
- **swagger-php**：PHP 注释生成工具
- **OpenAPI Generator**：代码生成工具

### 文档生成

**从代码注释生成**（swagger-php）：
```php
<?php
declare(strict_types=1);

/**
 * @OA\Info(
 *     title="User API",
 *     version="1.0.0",
 *     description="User management API"
 * )
 * @OA\Server(
 *     url="https://api.example.com/v1",
 *     description="Production server"
 * )
 */
class UserController
{
    /**
     * @OA\Get(
     *     path="/users",
     *     summary="Get user list",
     *     tags={"Users"},
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

**生成文档**：
```bash
vendor/bin/openapi app/ -o public/api-docs/openapi.yaml
```

## 文档结构

### API 概述

**示例**：
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: |
    User management API provides endpoints for managing users.
    This API supports user creation, retrieval, update, and deletion.
  contact:
    name: API Support
    email: support@example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT
servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server
```

### 认证说明

**示例**：
```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
security:
  - bearerAuth: []

paths:
  /users:
    get:
      security:
        - bearerAuth: []
      summary: Get user list
      description: |
        Get a list of users. Requires authentication.
        Include the JWT token in the Authorization header.
      responses:
        '200':
          description: Success
        '401':
          description: Unauthorized
```

### 端点列表

**示例**：
```yaml
paths:
  /users:
    get:
      summary: Get user list
      description: Retrieve a list of users with pagination
      operationId: getUsers
      tags:
        - Users
      parameters:
        - name: page
          in: query
          description: Page number
          required: false
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          description: Items per page
          required: false
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
  
  /users/{id}:
    get:
      summary: Get user by ID
      description: Retrieve a specific user by ID
      operationId: getUser
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          description: User ID
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: '#/components/schemas/User'
        '404':
          description: User not found
```

### 请求/响应格式

**请求格式示例**：
```yaml
paths:
  /users:
    post:
      summary: Create user
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
                  description: User name
                  example: "John Doe"
                email:
                  type: string
                  format: email
                  description: User email
                  example: "john@example.com"
                age:
                  type: integer
                  description: User age
                  minimum: 0
                  maximum: 120
                  example: 30
```

**响应格式示例**：
```yaml
responses:
  '201':
    description: User created successfully
    content:
      application/json:
        schema:
          type: object
          properties:
            success:
              type: boolean
              example: true
            message:
              type: string
              example: "User created successfully"
            data:
              $ref: '#/components/schemas/User'
  '422':
    description: Validation error
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
              example: "Validation failed"
            errors:
              type: object
              properties:
                email:
                  type: array
                  items:
                    type: string
                  example: ["The email field is required."]
```

### 错误码说明

**示例**：
```yaml
components:
  schemas:
    Error:
      type: object
      properties:
        success:
          type: boolean
          example: false
        message:
          type: string
          example: "Error message"
        errors:
          type: object
          description: Validation errors

paths:
  /users:
    get:
      responses:
        '400':
          description: Bad request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                success: false
                message: "Invalid request parameters"
        '401':
          description: Unauthorized
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                success: false
                message: "Authentication required"
        '404':
          description: Not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                success: false
                message: "Resource not found"
        '500':
          description: Internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                success: false
                message: "Internal server error"
```

### 示例代码

**示例**：
```yaml
paths:
  /users:
    get:
      summary: Get user list
      x-code-samples:
        - lang: 'curl'
          source: |
            curl -X GET "https://api.example.com/v1/users" \
              -H "Authorization: Bearer {token}"
        - lang: 'php'
          source: |
            <?php
            $ch = curl_init('https://api.example.com/v1/users');
            curl_setopt($ch, CURLOPT_HTTPHEADER, [
                'Authorization: Bearer {token}'
            ]);
            $response = curl_exec($ch);
            curl_close($ch);
        - lang: 'javascript'
          source: |
            fetch('https://api.example.com/v1/users', {
                headers: {
                    'Authorization': 'Bearer {token}'
                }
            })
            .then(response => response.json())
            .then(data => console.log(data));
```

## 文档生成工具

### 代码注释生成

**使用 swagger-php**：
```php
<?php
declare(strict_types=1);

/**
 * @OA\Schema(
 *     schema="User",
 *     type="object",
 *     required={"id", "name", "email"},
 *     @OA\Property(property="id", type="integer", example=1),
 *     @OA\Property(property="name", type="string", example="John Doe"),
 *     @OA\Property(property="email", type="string", format="email", example="john@example.com")
 * )
 */
class User
{
    // ...
}
```

### 自动生成工具

**使用 OpenAPI Generator**：
```bash
# 安装
npm install @openapitools/openapi-generator-cli -g

# 生成 PHP 客户端
openapi-generator-cli generate \
  -i openapi.yaml \
  -g php \
  -o ./generated/php-client
```

### 文档托管

**Swagger UI**：
```html
<!DOCTYPE html>
<html>
<head>
    <title>API Documentation</title>
    <link rel="stylesheet" type="text/css" href="swagger-ui.css">
</head>
<body>
    <div id="swagger-ui"></div>
    <script src="swagger-ui-bundle.js"></script>
    <script>
        SwaggerUIBundle({
            url: "openapi.yaml",
            dom_id: '#swagger-ui'
        });
    </script>
</body>
</html>
```

### 文档版本

**版本管理**：
```yaml
info:
  title: User API
  version: 1.0.0
  description: |
    Version 1.0.0 of the User API.
    
    ## Changelog
    
    ### Version 1.0.0 (2024-01-01)
    - Initial release
    - User CRUD operations
```

## 文档维护

### 文档更新

**更新原则**：
- **同步更新**：代码更新时同步更新文档
- **及时更新**：及时更新文档内容
- **版本管理**：为每个版本维护文档

### 版本管理

**版本管理策略**：
```yaml
info:
  title: User API
  version: 2.0.0
  description: |
    ## Version History
    
    ### Version 2.0.0 (2024-06-01)
    - Added user avatar support
    - Updated response format
    - Deprecated v1.0.0
    
    ### Version 1.0.0 (2024-01-01)
    - Initial release
```

### 变更记录

**变更记录示例**：
```markdown
# API Changelog

## Version 2.0.0 (2024-06-01)

### Added
- User avatar upload endpoint
- User profile update endpoint

### Changed
- Response format updated to include metadata
- Pagination format changed

### Deprecated
- GET /api/v1/users (use v2 instead)

### Removed
- None

## Version 1.0.0 (2024-01-01)

### Added
- Initial API release
- User CRUD operations
```

### 文档审查

**审查要点**：
- **准确性**：文档内容是否准确
- **完整性**：是否包含所有必要信息
- **清晰性**：是否清晰易懂
- **一致性**：格式是否一致

## 使用场景

### API 开发

- 开发过程中编写文档
- 团队协作参考
- 客户端集成指南

### 团队协作

- 前后端协作
- 移动端协作
- 第三方集成

### 客户端开发

- 前端应用开发
- 移动应用开发
- 第三方服务集成

### API 维护

- 版本更新
- 功能扩展
- 问题排查

## 注意事项

### 文档的准确性

- **验证示例**：确保示例代码可以运行
- **参数说明**：确保参数说明准确
- **响应格式**：确保响应格式正确

### 示例的完整性

- **完整示例**：提供完整的示例代码
- **多种语言**：提供多种编程语言的示例
- **错误处理**：包含错误处理示例

### 文档的及时更新

- **同步更新**：代码更新时同步更新文档
- **定期审查**：定期审查文档准确性
- **版本管理**：为每个版本维护文档

### 版本管理

- **版本标识**：明确标识文档版本
- **变更记录**：记录版本变更历史
- **废弃通知**：提前通知版本废弃

## 常见问题

### 如何编写 API 文档？

1. **使用 OpenAPI 规范**：使用标准规范编写文档
2. **提供完整信息**：包含接口、参数、响应、示例
3. **保持更新**：及时更新文档

### OpenAPI 规范如何使用？

1. **编写 YAML/JSON**：使用 OpenAPI 格式编写文档
2. **使用工具生成**：使用工具从代码生成文档
3. **使用 Swagger UI**：使用 Swagger UI 展示文档

### 如何生成文档？

1. **代码注释**：在代码中添加注释
2. **自动生成**：使用工具自动生成文档
3. **手动编写**：手动编写 OpenAPI 文档

### 如何维护文档？

1. **同步更新**：代码更新时同步更新文档
2. **版本管理**：为每个版本维护文档
3. **定期审查**：定期审查文档准确性

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

### 使用工具生成文档

- 从代码注释生成文档
- 自动更新文档
- 使用 Swagger UI 展示文档

## 相关章节

- **[5.7.3 API 文档与测试](section-03-documentation-testing.md)**：了解 API 文档和测试的基础内容
- **[5.7.1 RESTful 设计原则](section-01-restful-principles.md)**：了解 RESTful API 的设计原则

## 练习任务

1. **编写 OpenAPI 文档**
   - 定义 API 结构
   - 编写接口文档
   - 添加示例代码

2. **使用工具生成文档**
   - 从代码注释生成文档
   - 使用 Swagger UI 展示文档
   - 部署文档站点

3. **实现文档版本管理**
   - 为每个版本维护文档
   - 记录版本变更历史
   - 管理文档版本

4. **实现文档自动化**
   - 自动生成文档
   - 自动更新文档
   - 集成到 CI/CD

5. **实现完整的 API 文档系统**
   - 文档编写和维护
   - 文档生成和部署
   - 文档版本管理

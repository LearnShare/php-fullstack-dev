# 5.7.4 API 测试工具

## 概述

API 测试工具能够提高 API 测试的效率和准确性。选择合适的测试工具、掌握工具的使用方法、实现自动化测试，对于确保 API 的质量和稳定性至关重要。本节详细介绍常用的 API 测试工具，包括 Postman、curl、PHPUnit 等，帮助零基础学员掌握 API 测试方法。

API 测试工具可以分为手动测试工具和自动化测试工具。手动测试工具适合开发过程中的快速测试，自动化测试工具适合持续集成和回归测试。

**主要内容**：
- API 测试工具概述（工具分类、工具选择、测试场景）
- Postman 使用（Postman 基础、请求构建、测试脚本、集合管理）
- curl 命令行工具（curl 命令基础、请求发送、参数设置、脚本化测试）
- PHPUnit API 测试（API 测试类、请求模拟、响应断言、测试套件）
- 测试自动化（自动化测试框架、CI/CD 集成、测试报告）
- 实际应用示例和最佳实践

## 特性

- **多种工具**：支持多种测试工具
- **自动化支持**：支持自动化测试
- **易于使用**：工具简单易用
- **功能强大**：支持复杂测试场景
- **集成友好**：易于集成到 CI/CD

## API 测试工具概述

### 工具分类

**手动测试工具**：
- **Postman**：图形化 API 测试工具
- **Insomnia**：API 客户端工具
- **HTTPie**：命令行 HTTP 客户端

**自动化测试工具**：
- **PHPUnit**：PHP 单元测试框架
- **Guzzle**：PHP HTTP 客户端
- **Codeception**：PHP 测试框架

**命令行工具**：
- **curl**：命令行 HTTP 客户端
- **wget**：命令行下载工具

### 工具选择

**选择原则**：
- **开发阶段**：使用 Postman 进行快速测试
- **自动化测试**：使用 PHPUnit 进行自动化测试
- **脚本测试**：使用 curl 进行脚本化测试
- **CI/CD**：使用 PHPUnit 集成到 CI/CD

### 测试场景

**不同场景使用不同工具**：
- **快速测试**：Postman
- **自动化测试**：PHPUnit
- **脚本测试**：curl
- **性能测试**：专用性能测试工具

## Postman 使用

### Postman 基础

**Postman 功能**：
- 发送 HTTP 请求
- 保存请求集合
- 编写测试脚本
- 环境变量管理
- 团队协作

### 请求构建

**基本请求**：
1. 选择 HTTP 方法（GET、POST 等）
2. 输入 URL
3. 设置请求头
4. 设置请求体（如需要）
5. 发送请求

**示例**：
```
GET https://api.example.com/v1/users
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json
```

### 测试脚本

**Postman 测试脚本**（使用 JavaScript）：
```javascript
// 测试响应状态码
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 测试响应时间
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// 测试响应体
pm.test("Response has data", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.be.an('array');
});

// 保存响应数据到环境变量
pm.test("Save user ID", function () {
    var jsonData = pm.response.json();
    pm.environment.set("userId", jsonData.data[0].id);
});
```

### 集合管理

**创建集合**：
1. 创建新的 Collection
2. 添加请求到集合
3. 组织请求结构
4. 设置集合变量

**集合变量**：
```javascript
// 在集合中设置变量
{
  "baseUrl": "https://api.example.com/v1",
  "token": "your-token-here"
}

// 在请求中使用变量
{{baseUrl}}/users
Authorization: Bearer {{token}}
```

### 环境变量

**环境管理**：
- **开发环境**：`dev-api.example.com`
- **测试环境**：`test-api.example.com`
- **生产环境**：`api.example.com`

**使用环境变量**：
```
{{baseUrl}}/users
```

## curl 命令行工具

### curl 命令基础

**基本语法**：
```bash
curl [options] <url>
```

**常用选项**：
- `-X`：指定 HTTP 方法
- `-H`：设置请求头
- `-d`：设置请求体
- `-u`：设置认证信息
- `-v`：显示详细信息

### 请求发送

**GET 请求**：
```bash
curl https://api.example.com/v1/users
```

**POST 请求**：
```bash
curl -X POST https://api.example.com/v1/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'
```

**带认证的请求**：
```bash
curl -X GET https://api.example.com/v1/users \
  -H "Authorization: Bearer your-token-here"
```

**PUT 请求**：
```bash
curl -X PUT https://api.example.com/v1/users/123 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com"}'
```

**DELETE 请求**：
```bash
curl -X DELETE https://api.example.com/v1/users/123 \
  -H "Authorization: Bearer your-token-here"
```

### 参数设置

**查询参数**：
```bash
curl "https://api.example.com/v1/users?page=1&limit=10"
```

**请求头**：
```bash
curl -X GET https://api.example.com/v1/users \
  -H "Authorization: Bearer token" \
  -H "Content-Type: application/json" \
  -H "X-Custom-Header: value"
```

**请求体（JSON）**：
```bash
curl -X POST https://api.example.com/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com"
  }'
```

**请求体（文件）**：
```bash
curl -X POST https://api.example.com/v1/upload \
  -F "file=@/path/to/file.jpg"
```

### 脚本化测试

**Bash 脚本示例**：
```bash
#!/bin/bash

BASE_URL="https://api.example.com/v1"
TOKEN="your-token-here"

# 测试获取用户列表
echo "Testing GET /users"
response=$(curl -s -w "\n%{http_code}" -X GET "$BASE_URL/users" \
  -H "Authorization: Bearer $TOKEN")
http_code=$(echo "$response" | tail -n1)
body=$(echo "$response" | sed '$d')

if [ "$http_code" -eq 200 ]; then
    echo "✓ GET /users: Success"
else
    echo "✗ GET /users: Failed (HTTP $http_code)"
    echo "$body"
    exit 1
fi

# 测试创建用户
echo "Testing POST /users"
response=$(curl -s -w "\n%{http_code}" -X POST "$BASE_URL/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com"}')
http_code=$(echo "$response" | tail -n1)
body=$(echo "$response" | sed '$d')

if [ "$http_code" -eq 201 ]; then
    echo "✓ POST /users: Success"
    user_id=$(echo "$body" | grep -o '"id":[0-9]*' | grep -o '[0-9]*')
    echo "Created user ID: $user_id"
else
    echo "✗ POST /users: Failed (HTTP $http_code)"
    echo "$body"
    exit 1
fi
```

## PHPUnit API 测试

### API 测试类

**示例**：
```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserApiTest extends TestCase
{
    private string $baseUrl = 'https://api.example.com/v1';
    private string $token = 'your-token-here';

    public function testGetUsers(): void
    {
        $ch = curl_init($this->baseUrl . '/users');
        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => [
                'Authorization: Bearer ' . $this->token,
            ],
        ]);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);
        
        $this->assertEquals(200, $httpCode);
        
        $data = json_decode($response, true);
        $this->assertIsArray($data);
        $this->assertArrayHasKey('data', $data);
    }
}
```

### 请求模拟

**使用 Guzzle**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use PHPUnit\Framework\TestCase;

class UserApiTest extends TestCase
{
    private Client $client;

    protected function setUp(): void
    {
        $this->client = new Client([
            'base_uri' => 'https://api.example.com/v1',
            'headers' => [
                'Authorization' => 'Bearer your-token-here',
                'Content-Type' => 'application/json',
            ],
        ]);
    }

    public function testGetUsers(): void
    {
        $response = $this->client->get('/users');
        
        $this->assertEquals(200, $response->getStatusCode());
        
        $data = json_decode($response->getBody()->getContents(), true);
        $this->assertArrayHasKey('data', $data);
    }

    public function testCreateUser(): void
    {
        $response = $this->client->post('/users', [
            'json' => [
                'name' => 'Test User',
                'email' => 'test@example.com',
            ],
        ]);
        
        $this->assertEquals(201, $response->getStatusCode());
        
        $data = json_decode($response->getBody()->getContents(), true);
        $this->assertArrayHasKey('data', $data);
        $this->assertEquals('Test User', $data['data']['name']);
    }
}
```

### 响应断言

**示例**：
```php
<?php
declare(strict_types=1);

public function testGetUserResponse(): void
{
    $response = $this->client->get('/users/123');
    
    // 断言状态码
    $this->assertEquals(200, $response->getStatusCode());
    
    // 断言响应头
    $this->assertStringContainsString('application/json', $response->getHeaderLine('Content-Type'));
    
    // 断言响应体
    $data = json_decode($response->getBody()->getContents(), true);
    $this->assertIsArray($data);
    $this->assertArrayHasKey('data', $data);
    $this->assertArrayHasKey('id', $data['data']);
    $this->assertArrayHasKey('name', $data['data']);
    $this->assertArrayHasKey('email', $data['data']);
    
    // 断言数据类型
    $this->assertIsInt($data['data']['id']);
    $this->assertIsString($data['data']['name']);
    $this->assertIsString($data['data']['email']);
}
```

### 测试套件

**示例**：
```php
<?php
declare(strict_types=1);

class ApiTestSuite
{
    private Client $client;

    public function __construct()
    {
        $this->client = new Client([
            'base_uri' => 'https://api.example.com/v1',
            'headers' => [
                'Authorization' => 'Bearer your-token-here',
            ],
        ]);
    }

    public function runAllTests(): void
    {
        $this->testGetUsers();
        $this->testCreateUser();
        $this->testGetUser();
        $this->testUpdateUser();
        $this->testDeleteUser();
    }

    private function testGetUsers(): void
    {
        $response = $this->client->get('/users');
        assert($response->getStatusCode() === 200);
        echo "✓ GET /users: Passed\n";
    }

    private function testCreateUser(): void
    {
        $response = $this->client->post('/users', [
            'json' => ['name' => 'Test', 'email' => 'test@example.com'],
        ]);
        assert($response->getStatusCode() === 201);
        echo "✓ POST /users: Passed\n";
    }

    // ... 其他测试方法
}
```

## 测试自动化

### 自动化测试框架

**使用 PHPUnit**：
```php
<?php
declare(strict_types=1);

namespace Tests\Api;

use GuzzleHttp\Client;
use PHPUnit\Framework\TestCase;

class UserApiTest extends TestCase
{
    private Client $client;

    protected function setUp(): void
    {
        $this->client = new Client([
            'base_uri' => getenv('API_BASE_URL') ?: 'https://api.example.com/v1',
            'timeout' => 5.0,
        ]);
    }

    public function testGetUsers(): void
    {
        $response = $this->client->get('/users');
        $this->assertEquals(200, $response->getStatusCode());
    }

    public function testCreateUser(): void
    {
        $response = $this->client->post('/users', [
            'json' => [
                'name' => 'Test User',
                'email' => 'test@example.com',
            ],
        ]);
        $this->assertEquals(201, $response->getStatusCode());
    }
}
```

### CI/CD 集成

**GitHub Actions 示例**：
```yaml
name: API Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.1'
      
      - name: Install dependencies
        run: composer install
      
      - name: Run API tests
        run: vendor/bin/phpunit tests/Api
        env:
          API_BASE_URL: ${{ secrets.API_BASE_URL }}
          API_TOKEN: ${{ secrets.API_TOKEN }}
```

### 测试报告

**生成测试报告**：
```bash
# 生成 HTML 报告
vendor/bin/phpunit --testdox-html report.html

# 生成 JUnit XML 报告
vendor/bin/phpunit --log-junit report.xml

# 生成覆盖率报告
vendor/bin/phpunit --coverage-html coverage/
```

## 使用场景

### API 开发测试

- 开发过程中快速测试
- 验证 API 功能
- 调试 API 问题

### 集成测试

- 测试 API 集成
- 验证数据流
- 测试端到端流程

### 回归测试

- 确保新功能不影响现有功能
- 自动化回归测试
- 持续集成测试

### 性能测试

- 测试 API 性能
- 负载测试
- 压力测试

## 注意事项

### 测试环境配置

- **环境隔离**：测试环境与生产环境隔离
- **测试数据**：使用测试数据，不影响生产数据
- **环境变量**：使用环境变量配置测试环境

### 测试数据管理

- **数据准备**：准备测试数据
- **数据清理**：测试后清理数据
- **数据隔离**：每个测试使用独立数据

### 测试隔离

- **独立测试**：每个测试独立运行
- **不依赖顺序**：测试不依赖执行顺序
- **清理状态**：测试后清理状态

### 测试覆盖率

- **功能覆盖**：覆盖所有功能
- **边界覆盖**：覆盖边界条件
- **错误覆盖**：覆盖错误情况

## 常见问题

### 如何选择测试工具？

- **开发阶段**：使用 Postman 进行快速测试
- **自动化测试**：使用 PHPUnit 进行自动化测试
- **脚本测试**：使用 curl 进行脚本化测试

### 如何使用 Postman？

1. 安装 Postman
2. 创建请求
3. 设置请求参数
4. 发送请求
5. 查看响应
6. 编写测试脚本

### 如何编写 API 测试？

1. 使用 PHPUnit 创建测试类
2. 使用 Guzzle 发送 HTTP 请求
3. 断言响应状态码和内容
4. 运行测试

### 如何自动化测试？

1. 使用 PHPUnit 编写测试
2. 集成到 CI/CD 流程
3. 自动运行测试
4. 生成测试报告

## 最佳实践

### 使用 Postman 进行手动测试

- 开发过程中使用 Postman 快速测试
- 保存常用请求到集合
- 使用环境变量管理不同环境

### 使用 PHPUnit 进行自动化测试

- 编写完整的测试用例
- 覆盖所有功能
- 集成到 CI/CD

### 编写完整的测试用例

- 功能测试
- 边界测试
- 错误测试
- 性能测试

### 定期运行测试

- 每次代码提交运行测试
- 定期运行完整测试套件
- 监控测试结果

## 相关章节

- **[5.7.3 API 文档与测试](section-03-documentation-testing.md)**：了解 API 文档和测试的基础内容
- **[5.12 HTTP 客户端](../chapter-12-http-client/readme.md)**：了解 HTTP 客户端的详细内容

## 练习任务

1. **使用 Postman 测试 API**
   - 创建请求集合
   - 编写测试脚本
   - 使用环境变量

2. **使用 curl 测试 API**
   - 发送各种 HTTP 请求
   - 编写测试脚本
   - 自动化测试

3. **使用 PHPUnit 测试 API**
   - 创建测试类
   - 编写测试用例
   - 运行测试

4. **实现自动化测试**
   - 集成到 CI/CD
   - 生成测试报告
   - 监控测试结果

5. **实现完整的 API 测试系统**
   - 测试框架搭建
   - 测试用例编写
   - 自动化测试和报告

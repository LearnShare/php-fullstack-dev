# 6.8.1 测试金字塔

## 概述

测试金字塔是测试策略的指导原则。本节介绍测试金字塔的概念、测试类型、测试策略等内容。

## 测试金字塔概念

### 三层结构

```
        /\
       /  \  E2E 测试（少量）
      /____\
     /      \  集成测试（中等）
    /________\
   /          \  单元测试（大量）
  /____________\
```

### 测试比例

- **单元测试**：70% - 快速、隔离、大量
- **集成测试**：20% - 中等速度、组件交互
- **E2E 测试**：10% - 慢速、完整系统

## 测试类型

### 单元测试

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAdd(): void
    {
        $calc = new Calculator();
        $this->assertEquals(5, $calc->add(2, 3));
    }
}
```

### 集成测试

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserServiceTest extends TestCase
{
    public function testCreateUser(): void
    {
        $service = new UserService($this->pdo);
        $user = $service->createUser('John', 'john@example.com');
        $this->assertNotNull($user->id);
    }
}
```

### E2E 测试

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserRegistrationTest extends TestCase
{
    public function testUserCanRegister(): void
    {
        $client = new Client();
        $response = $client->post('/register', [
            'name' => 'John',
            'email' => 'john@example.com',
        ]);
        $this->assertEquals(200, $response->getStatusCode());
    }
}
```

## 测试策略

### 1. 优先单元测试

- 快速执行
- 易于维护
- 覆盖核心逻辑

### 2. 适量集成测试

- 测试组件交互
- 验证接口契约
- 检查数据流

### 3. 少量 E2E 测试

- 验证关键流程
- 检查用户体验
- 回归测试

## 完整示例

```php
<?php
declare(strict_types=1);

// 单元测试
class UserTest extends TestCase
{
    public function testUserCreation(): void
    {
        $user = new User('John', 'john@example.com');
        $this->assertEquals('John', $user->getName());
    }
}

// 集成测试
class UserRepositoryTest extends TestCase
{
    public function testSaveUser(): void
    {
        $repo = new UserRepository($this->pdo);
        $user = new User('John', 'john@example.com');
        $repo->save($user);
        $this->assertNotNull($user->getId());
    }
}

// E2E 测试
class UserApiTest extends TestCase
{
    public function testCreateUserEndpoint(): void
    {
        $response = $this->post('/api/users', [
            'name' => 'John',
            'email' => 'john@example.com',
        ]);
        $this->assertEquals(201, $response->getStatusCode());
    }
}
```

## 最佳实践

1. **遵循金字塔**：大量单元测试，适量集成测试，少量 E2E 测试
2. **快速反馈**：优先快速测试
3. **隔离测试**：单元测试应该独立
4. **持续集成**：自动化测试执行

## 注意事项

1. 不要过度依赖 E2E 测试
2. 保持测试快速执行
3. 测试应该独立可重复
4. 定期审查测试覆盖率

## 练习

1. 为一个功能编写单元测试、集成测试和 E2E 测试。

2. 分析现有测试，评估是否符合测试金字塔原则。

3. 优化测试结构，提升测试执行速度。

4. 实现测试覆盖率监控。

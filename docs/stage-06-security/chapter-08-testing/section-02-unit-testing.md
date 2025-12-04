# 6.8.2 单元测试

## 概述

单元测试是测试金字塔的基础。本节介绍 PHPUnit 基础、测试编写、断言使用、测试组织等内容。

## PHPUnit 基础

### 安装

```bash
composer require --dev phpunit/phpunit
```

### 基础测试

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAdd(): void
    {
        $calc = new Calculator();
        $result = $calc->add(2, 3);
        $this->assertEquals(5, $result);
    }
}
```

## 断言使用

### 常用断言

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class AssertionsTest extends TestCase
{
    public function testAssertions(): void
    {
        // 相等
        $this->assertEquals(5, 5);
        $this->assertSame(5, 5); // 严格相等
        
        // 不相等
        $this->assertNotEquals(5, 4);
        $this->assertNotSame(5, '5');
        
        // 真值
        $this->assertTrue(true);
        $this->assertFalse(false);
        
        // 空值
        $this->assertNull(null);
        $this->assertNotNull('value');
        
        // 类型
        $this->assertIsArray([]);
        $this->assertIsString('string');
        $this->assertIsInt(123);
        
        // 包含
        $this->assertContains('value', ['value', 'other']);
        $this->assertStringContainsString('sub', 'substring');
        
        // 异常
        $this->expectException(InvalidArgumentException::class);
        throw new InvalidArgumentException();
    }
}
```

## 测试组织

### 测试类结构

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserServiceTest extends TestCase
{
    private UserService $service;
    
    protected function setUp(): void
    {
        parent::setUp();
        $this->service = new UserService();
    }
    
    protected function tearDown(): void
    {
        parent::tearDown();
        // 清理资源
    }
    
    public function testCreateUser(): void
    {
        $user = $this->service->createUser('John', 'john@example.com');
        $this->assertNotNull($user);
        $this->assertEquals('John', $user->getName());
    }
}
```

### 数据提供者

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd(int $a, int $b, int $expected): void
    {
        $calc = new Calculator();
        $this->assertEquals($expected, $calc->add($a, $b));
    }
    
    public function additionProvider(): array
    {
        return [
            [2, 3, 5],
            [0, 0, 0],
            [-1, 1, 0],
        ];
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserTest extends TestCase
{
    private User $user;
    
    protected function setUp(): void
    {
        $this->user = new User('John', 'john@example.com');
    }
    
    public function testGetName(): void
    {
        $this->assertEquals('John', $this->user->getName());
    }
    
    public function testGetEmail(): void
    {
        $this->assertEquals('john@example.com', $this->user->getEmail());
    }
    
    public function testSetName(): void
    {
        $this->user->setName('Jane');
        $this->assertEquals('Jane', $this->user->getName());
    }
}
```

## 最佳实践

1. **测试命名**：使用描述性名称
2. **单一职责**：每个测试只测试一个功能
3. **独立性**：测试之间应该独立
4. **快速执行**：保持测试快速

## 注意事项

1. 测试应该可重复执行
2. 避免测试之间的依赖
3. 使用 setUp 和 tearDown 管理资源
4. 保持测试代码简洁

## 练习

1. 为一个类编写完整的单元测试。

2. 使用数据提供者测试多个场景。

3. 组织测试类，使用 setUp 和 tearDown。

4. 实现测试覆盖率报告。

# 6.8 测试体系建设

## 目标

- 理解测试金字塔的概念。
- 掌握 PHPUnit 和 Pest 的使用。
- 了解单元测试、集成测试、功能测试的区别。
- 熟悉 Mock、Stub、Fakes 的使用。

## 测试金字塔

### 测试层次

```
        /\
       /E2E\       少量端到端测试
      /------\
     /Feature\      一些功能测试
    /----------\
   /Integration\    更多集成测试
  /--------------\
 /    Unit        \ 大量单元测试
/------------------\
```

### 测试类型

| 类型           | 说明                     | 数量 | 速度 | 成本 |
| :------------- | :----------------------- | :--- | :--- | :--- |
| 单元测试       | 测试单个函数/方法        | 多   | 快   | 低   |
| 集成测试       | 测试组件交互             | 中   | 中   | 中   |
| 功能测试       | 测试完整功能流程         | 少   | 慢   | 高   |
| 端到端测试     | 测试整个系统             | 极少 | 最慢 | 最高 |

## PHPUnit

### 与 Jest 对比

如果你熟悉 JavaScript 的 Jest，PHPUnit 是 PHP 的测试框架，功能类似：

| 功能 | Jest (JavaScript) | PHPUnit (PHP) |
| :--- | :---------------- | :------------ |
| 测试运行器 | `jest` | `phpunit` 或 `vendor/bin/phpunit` |
| 测试文件 | `*.test.js` | `*Test.php` |
| 断言 | `expect().toBe()` | `$this->assertEquals()` |
| 测试套件 | `describe()` | `class TestCase extends TestCase` |
| 前置/后置 | `beforeEach()` / `afterEach()` | `setUp()` / `tearDown()` |
| Mock | `jest.mock()` | `Mockery` 或 `createMock()` |
| 快照测试 | `toMatchSnapshot()` | Pest 支持 |
| 覆盖率 | `--coverage` | `--coverage-html` |

**关键差异**：

1. **语法风格**：
   - Jest：函数式（`test()`, `describe()`）
   - PHPUnit：类式（`class TestCase extends TestCase`）

2. **断言方法**：
   - Jest：链式调用（`expect().toBe()`）
   - PHPUnit：方法调用（`$this->assertEquals()`）

3. **Mock 库**：
   - Jest：内置 Mock
   - PHPUnit：需要 Mockery 或 PHPUnit 内置 Mock

**示例对比**：

```javascript
// Jest
describe('Calculator', () => {
    test('adds two numbers', () => {
        const calc = new Calculator();
        expect(calc.add(2, 3)).toBe(5);
    });
    
    test('subtracts two numbers', () => {
        const calc = new Calculator();
        expect(calc.subtract(3, 2)).toBe(1);
    });
});
```

```php
// PHPUnit
class CalculatorTest extends TestCase
{
    public function testAddsTwoNumbers(): void
    {
        $calc = new Calculator();
        $this->assertEquals(5, $calc->add(2, 3));
    }
    
    public function testSubtractsTwoNumbers(): void
    {
        $calc = new Calculator();
        $this->assertEquals(1, $calc->subtract(3, 2));
    }
}
```

### 版本要求

- **PHPUnit 11.x**：最新版本，要求 PHP 8.2+
- **PHPUnit 10.x**：支持 PHP 8.2+
- **PHPUnit 9.x**：支持 PHP 7.3-8.1（已不再维护）

### 基础测试

```php
<?php
declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAdd(): void
    {
        $calc = new Calculator();
        $this->assertEquals(5, $calc->add(2, 3));
    }
    
    public function testSubtract(): void
    {
        $calc = new Calculator();
        $this->assertEquals(1, $calc->subtract(3, 2));
    }
}
```

### 数据提供者

```php
<?php
/**
 * @dataProvider additionProvider
 */
public function testAdd(int $a, int $b, int $expected): void
{
    $this->assertEquals($expected, $a + $b);
}

public function additionProvider(): array
{
    return [
        [2, 3, 5],
        [0, 0, 0],
        [-1, 1, 0],
    ];
}
```

## Pest

### 与 Jest 对比

Pest 的语法更接近 Jest，如果你熟悉 Jest，Pest 会更容易上手：

| 特性 | Jest | Pest |
| :--- | :--- | :--- |
| 语法风格 | 函数式 | 函数式 |
| 测试定义 | `test()` | `test()` 或 `it()` |
| 断言 | `expect().toBe()` | `expect()->toBe()` |
| 分组 | `describe()` | `describe()` |
| 前置/后置 | `beforeEach()` | `beforeEach()` |
| 快照测试 | `toMatchSnapshot()` | `toMatchSnapshot()` |

**示例对比**：

```javascript
// Jest
describe('Calculator', () => {
    test('adds two numbers', () => {
        const calc = new Calculator();
        expect(calc.add(2, 3)).toBe(5);
    });
});
```

```php
// Pest
describe('Calculator', function () {
    test('adds two numbers', function () {
        $calc = new Calculator();
        expect($calc->add(2, 3))->toBe(5);
    });
});
```

### 版本说明

- **Pest 3.x**：最新版本，提供更好的性能和功能
- **Pest 2.x**：稳定版本，广泛使用
- 基于 PHPUnit，提供更简洁的语法（类似 Jest）

### 基础语法

```php
<?php
declare(strict_types=1);

use Tests\TestCase;

test('calculates sum correctly', function () {
    $calc = new Calculator();
    expect($calc->add(2, 3))->toBe(5);
});

it('subtracts numbers', function () {
    $calc = new Calculator();
    expect($calc->subtract(3, 2))->toBe(1);
});
```

### 快照测试

```php
<?php
test('generates correct output', function () {
    $output = generateReport();
    expect($output)->toMatchSnapshot();
});
```

## Mock 和 Stub

### Mockery

```php
<?php
use Mockery;

$mock = Mockery::mock(UserRepository::class);
$mock->shouldReceive('find')
    ->once()
    ->with(1)
    ->andReturn(new User(['id' => 1, 'name' => 'Alice']));

$service = new UserService($mock);
$user = $service->getUser(1);
```

### Pest Fakes

```php
<?php
use Illuminate\Support\Facades\Mail;

test('sends welcome email', function () {
    Mail::fake();
    
    $user = createUser();
    
    Mail::assertSent(WelcomeEmail::class);
});
```

## 测试覆盖率

### 生成覆盖率报告

```bash
# 使用 PHPUnit
phpunit --coverage-html coverage/

# 使用 Pest
pest --coverage

# 使用 Xdebug
phpunit --coverage-text
```

### 覆盖率目标

- **单元测试**：目标 80%+
- **集成测试**：目标 60%+
- **功能测试**：目标 40%+

## 持续集成测试

### GitHub Actions 示例

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.3'
        coverage: xdebug
    
    - name: Install dependencies
      run: composer install
    
    - name: Run tests
      run: phpunit --coverage-clover=coverage.xml
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
```

## 练习

1. 为现有代码编写单元测试，覆盖核心业务逻辑。

2. 创建集成测试，测试数据库操作和外部服务交互。

3. 实现 Mock 对象，隔离外部依赖进行测试。

4. 编写功能测试，测试完整的用户流程。

5. 配置测试覆盖率报告，确保达到目标覆盖率。

6. 建立 CI/CD 测试流程，自动运行测试。

7. 使用 Pest 创建快照测试，验证输出格式。

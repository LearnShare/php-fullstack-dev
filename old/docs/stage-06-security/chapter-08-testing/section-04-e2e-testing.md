# 6.8.4 E2E 测试

## 概述

E2E（端到端）测试验证完整系统的功能。本节介绍端到端测试、浏览器测试、E2E 测试工具等内容。

## 端到端测试

### 基础 E2E 测试

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use GuzzleHttp\Client;

class UserRegistrationE2ETest extends TestCase
{
    private Client $client;
    
    protected function setUp(): void
    {
        $this->client = new Client(['base_uri' => 'http://localhost:8000']);
    }
    
    public function testUserCanRegisterAndLogin(): void
    {
        // 1. 注册用户
        $registerResponse = $this->client->post('/register', [
            'json' => [
                'name' => 'John',
                'email' => 'john@example.com',
                'password' => 'password123',
            ],
        ]);
        $this->assertEquals(201, $registerResponse->getStatusCode());
        
        // 2. 登录
        $loginResponse = $this->client->post('/login', [
            'json' => [
                'email' => 'john@example.com',
                'password' => 'password123',
            ],
        ]);
        $this->assertEquals(200, $loginResponse->getStatusCode());
        
        $token = json_decode($loginResponse->getBody(), true)['token'];
        
        // 3. 访问受保护资源
        $profileResponse = $this->client->get('/api/user/profile', [
            'headers' => ['Authorization' => "Bearer {$token}"],
        ]);
        $this->assertEquals(200, $profileResponse->getStatusCode());
    }
}
```

## 浏览器测试

### 使用 Selenium

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use Facebook\WebDriver\Remote\RemoteWebDriver;
use Facebook\WebDriver\Remote\DesiredCapabilities;

class BrowserE2ETest extends TestCase
{
    private RemoteWebDriver $driver;
    
    protected function setUp(): void
    {
        $capabilities = DesiredCapabilities::chrome();
        $this->driver = RemoteWebDriver::create('http://localhost:4444/wd/hub', $capabilities);
    }
    
    protected function tearDown(): void
    {
        $this->driver->quit();
    }
    
    public function testUserRegistration(): void
    {
        $this->driver->get('http://localhost:8000/register');
        
        $this->driver->findElement(\Facebook\WebDriver\WebDriverBy::name('name'))
            ->sendKeys('John');
        $this->driver->findElement(\Facebook\WebDriver\WebDriverBy::name('email'))
            ->sendKeys('john@example.com');
        $this->driver->findElement(\Facebook\WebDriver\WebDriverBy::name('password'))
            ->sendKeys('password123');
        
        $this->driver->findElement(\Facebook\WebDriver\WebDriverBy::cssSelector('button[type="submit"]'))
            ->click();
        
        $this->assertStringContainsString('Welcome', $this->driver->getPageSource());
    }
}
```

## E2E 测试工具

### Codeception

```php
<?php
// tests/acceptance/UserRegistrationCest.php

class UserRegistrationCest
{
    public function testUserRegistration(AcceptanceTester $I)
    {
        $I->amOnPage('/register');
        $I->fillField('name', 'John');
        $I->fillField('email', 'john@example.com');
        $I->fillField('password', 'password123');
        $I->click('Register');
        $I->see('Welcome');
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class E2ETest extends TestCase
{
    private Client $client;
    
    protected function setUp(): void
    {
        $this->client = new Client(['base_uri' => 'http://localhost:8000']);
    }
    
    public function testCompleteUserFlow(): void
    {
        // 1. 注册
        $register = $this->client->post('/register', [
            'json' => [
                'name' => 'John',
                'email' => 'john@example.com',
                'password' => 'password123',
            ],
        ]);
        $this->assertEquals(201, $register->getStatusCode());
        
        // 2. 登录
        $login = $this->client->post('/login', [
            'json' => [
                'email' => 'john@example.com',
                'password' => 'password123',
            ],
        ]);
        $token = json_decode($login->getBody(), true)['token'];
        
        // 3. 创建资源
        $create = $this->client->post('/api/posts', [
            'headers' => ['Authorization' => "Bearer {$token}"],
            'json' => ['title' => 'Test Post', 'content' => 'Content'],
        ]);
        $this->assertEquals(201, $create->getStatusCode());
        
        // 4. 读取资源
        $get = $this->client->get('/api/posts', [
            'headers' => ['Authorization' => "Bearer {$token}"],
        ]);
        $this->assertEquals(200, $get->getStatusCode());
    }
}
```

## 最佳实践

1. **关键流程**：只测试关键业务流程
2. **独立环境**：使用独立的测试环境
3. **数据清理**：测试后清理数据
4. **稳定执行**：确保测试稳定可重复

## 注意事项

1. E2E 测试执行较慢
2. 需要完整的测试环境
3. 测试可能不稳定
4. 维护成本较高

## 练习

1. 编写一个完整的用户注册和登录 E2E 测试。

2. 使用浏览器自动化工具测试 Web 界面。

3. 实现 E2E 测试的数据清理机制。

4. 创建 E2E 测试的 CI/CD 流程。

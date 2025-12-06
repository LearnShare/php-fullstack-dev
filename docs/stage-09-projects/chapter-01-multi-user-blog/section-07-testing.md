# 9.1.7 测试策略与实现

## 一、测试策略

### 1.1 测试金字塔

```
        /\
       /  \
      / E2E \         10% - 端到端测试
     /______\
    /        \
   /Integration\      20% - 集成测试
  /____________\
 /              \
/   Unit Tests   \   70% - 单元测试
/________________\
```

**测试分布**：
- **单元测试**（70%）：测试领域层和应用层的独立单元
- **集成测试**（20%）：测试数据库、缓存、队列等基础设施集成
- **E2E 测试**（10%）：测试完整 API 流程和用户场景

### 1.2 测试原则

1. **测试隔离**：每个测试独立运行，不依赖其他测试
2. **快速执行**：单元测试应该快速执行（< 100ms）
3. **可重复性**：测试结果应该一致，不依赖外部状态
4. **清晰命名**：测试方法名应该清晰描述测试内容
5. **AAA 模式**：Arrange（准备）→ Act（执行）→ Assert（断言）

### 1.3 测试工具

- **PHPUnit**：PHP 标准测试框架
- **Pest**：现代化的测试框架，语法简洁
- **Laravel Testing**：Laravel 测试工具和辅助方法
- **Mockery**：Mock 对象框架
- **Laravel Dusk**：浏览器自动化测试（可选）

## 二、单元测试

### 2.1 测试范围

**领域层单元测试**：
- 领域实体（Entities）
- 值对象（Value Objects）
- 领域服务（Domain Services）
- 业务规则验证

**应用层单元测试**：
- 应用服务（Application Services）
- 用例逻辑
- DTOs 转换

### 2.2 领域实体测试示例

**测试文件**：`tests/Unit/Domain/Article/ArticleTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Unit\Domain\Article;

use App\Domain\Article\Article;
use App\Domain\Article\ArticleId;
use App\Domain\Article\ArticleStatus;
use App\Domain\User\UserId;
use PHPUnit\Framework\TestCase;

class ArticleTest extends TestCase
{
    public function test_can_create_article(): void
    {
        // Arrange
        $articleId = ArticleId::generate();
        $authorId = UserId::generate();
        $title = 'Test Article';
        $content = 'Article content';

        // Act
        $article = new Article(
            id: $articleId,
            authorId: $authorId,
            title: $title,
            content: $content
        );

        // Assert
        $this->assertEquals($articleId, $article->id());
        $this->assertEquals($authorId, $article->authorId());
        $this->assertEquals($title, $article->title());
        $this->assertEquals($content, $article->content());
        $this->assertEquals(ArticleStatus::DRAFT, $article->status());
    }

    public function test_can_publish_article(): void
    {
        // Arrange
        $article = $this->createDraftArticle();

        // Act
        $article->publish();

        // Assert
        $this->assertEquals(ArticleStatus::PUBLISHED, $article->status());
        $this->assertNotNull($article->publishedAt());
    }

    public function test_cannot_publish_article_without_title(): void
    {
        // Arrange
        $article = $this->createDraftArticle();
        $article->changeTitle('');

        // Act & Assert
        $this->expectException(\DomainException::class);
        $this->expectExceptionMessage('文章标题不能为空');
        $article->publish();
    }

    public function test_can_increment_view_count(): void
    {
        // Arrange
        $article = $this->createPublishedArticle();
        $initialCount = $article->viewCount();

        // Act
        $article->incrementViewCount();

        // Assert
        $this->assertEquals($initialCount + 1, $article->viewCount());
    }

    private function createDraftArticle(): Article
    {
        return new Article(
            id: ArticleId::generate(),
            authorId: UserId::generate(),
            title: 'Test Article',
            content: 'Content'
        );
    }

    private function createPublishedArticle(): Article
    {
        $article = $this->createDraftArticle();
        $article->publish();
        return $article;
    }
}
```

### 2.3 值对象测试示例

**测试文件**：`tests/Unit/Domain/User/EmailTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Unit\Domain\User;

use App\Domain\User\Email;
use InvalidArgumentException;
use PHPUnit\Framework\TestCase;

class EmailTest extends TestCase
{
    public function test_can_create_valid_email(): void
    {
        // Arrange & Act
        $email = new Email('user@example.com');

        // Assert
        $this->assertEquals('user@example.com', $email->value());
    }

    public function test_cannot_create_invalid_email(): void
    {
        // Act & Assert
        $this->expectException(InvalidArgumentException::class);
        new Email('invalid-email');
    }

    public function test_email_is_case_(): void
    {
        // Arrange & Act
        $email = new Email('User@Example.COM');

        // Assert
        $this->assertEquals('user@example.com', $email->value());
    }

    public function test_email_equality(): void
    {
        // Arrange
        $email1 = new Email('user@example.com');
        $email2 = new Email('user@example.com');
        $email3 = new Email('other@example.com');

        // Assert
        $this->assertTrue($email1->equals($email2));
        $this->assertFalse($email1->equals($email3));
    }
}
```

### 2.4 应用服务测试示例

**测试文件**：`tests/Unit/Application/Article/CreateArticleServiceTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Unit\Application\Article;

use App\Application\Article\CreateArticleService;
use App\Domain\Article\Article;
use App\Domain\Article\ArticleId;
use App\Domain\Article\ArticleRepository;
use App\Domain\User\UserId;
use Mockery;
use PHPUnit\Framework\TestCase;

class CreateArticleServiceTest extends TestCase
{
    private ArticleRepository $articleRepository;
    private CreateArticleService $service;

    protected function setUp(): void
    {
        parent::setUp();
        $this->articleRepository = Mockery::mock(ArticleRepository::class);
        $this->service = new CreateArticleService($this->articleRepository);
    }

    public function test_can_create_article(): void
    {
        // Arrange
        $authorId = UserId::generate();
        $title = 'Test Article';
        $content = 'Article content';

        $this->articleRepository
            ->shouldReceive('save')
            ->once()
            ->with(Mockery::on(function (Article $article) use ($authorId, $title, $content) {
                return $article->authorId()->equals($authorId)
                    && $article->title() === $title
                    && $article->content() === $content;
            }));

        // Act
        $result = $this->service->execute([
            'author_id' => $authorId->value(),
            'title' => $title,
            'content' => $content,
        ]);

        // Assert
        $this->assertInstanceOf(ArticleId::class, $result);
    }

    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }
}
```

## 三、集成测试

### 3.1 测试范围

**基础设施层集成测试**：
- Repository 实现（数据库操作）
- 缓存服务
- 队列服务
- 外部服务集成

### 3.2 Repository 集成测试示例

**测试文件**：`tests/Integration/Infrastructure/Persistence/Eloquent/ArticleEloquentRepositoryTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Integration\Infrastructure\Persistence\Eloquent;

use App\Domain\Article\Article;
use App\Domain\Article\ArticleId;
use App\Domain\Article\ArticleRepository;
use App\Domain\User\UserId;
use App\Infrastructure\Persistence\Eloquent\ArticleEloquentRepository;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ArticleEloquentRepositoryTest extends TestCase
{
    use RefreshDatabase;

    private ArticleRepository $repository;

    protected function setUp(): void
    {
        parent::setUp();
        $this->repository = new ArticleEloquentRepository();
    }

    public function test_can_save_and_find_article(): void
    {
        // Arrange
        $article = new Article(
            id: ArticleId::generate(),
            authorId: UserId::generate(),
            title: 'Test Article',
            content: 'Content'
        );

        // Act
        $this->repository->save($article);
        $found = $this->repository->findById($article->id());

        // Assert
        $this->assertNotNull($found);
        $this->assertEquals($article->id()->value(), $found->id()->value());
        $this->assertEquals($article->title(), $found->title());
    }

    public function test_can_find_articles_by_author(): void
    {
        // Arrange
        $authorId = UserId::generate();
        $article1 = new Article(
            id: ArticleId::generate(),
            authorId: $authorId,
            title: 'Article 1',
            content: 'Content 1'
        );
        $article2 = new Article(
            id: ArticleId::generate(),
            authorId: $authorId,
            title: 'Article 2',
            content: 'Content 2'
        );

        $this->repository->save($article1);
        $this->repository->save($article2);

        // Act
        $result = $this->repository->findByAuthor($authorId, 1, 10);

        // Assert
        $this->assertCount(2, $result->items());
    }

    public function test_can_delete_article(): void
    {
        // Arrange
        $article = new Article(
            id: ArticleId::generate(),
            authorId: UserId::generate(),
            title: 'Test Article',
            content: 'Content'
        );
        $this->repository->save($article);

        // Act
        $this->repository->delete($article->id());

        // Assert
        $found = $this->repository->findById($article->id());
        $this->assertNull($found);
    }
}
```

### 3.3 缓存服务集成测试示例

**测试文件**：`tests/Integration/Infrastructure/Cache/RedisCacheServiceTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Integration\Infrastructure\Cache;

use App\Infrastructure\Cache\RedisCacheService;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Redis;
use Tests\TestCase;

class RedisCacheServiceTest extends TestCase
{
    use RefreshDatabase;

    private RedisCacheService $cacheService;

    protected function setUp(): void
    {
        parent::setUp();
        $this->cacheService = new RedisCacheService();
        Redis::flushdb();
    }

    public function test_can_set_and_get_cache(): void
    {
        // Arrange
        $key = 'test:key';
        $value = ['data' => 'test'];

        // Act
        $this->cacheService->set($key, $value, 60);
        $result = $this->cacheService->get($key);

        // Assert
        $this->assertEquals($value, $result);
    }

    public function test_cache_expires(): void
    {
        // Arrange
        $key = 'test:key';
        $value = 'test';

        // Act
        $this->cacheService->set($key, $value, 1);
        sleep(2);
        $result = $this->cacheService->get($key);

        // Assert
        $this->assertNull($result);
    }

    public function test_can_delete_cache(): void
    {
        // Arrange
        $key = 'test:key';
        $value = 'test';
        $this->cacheService->set($key, $value, 60);

        // Act
        $this->cacheService->delete($key);
        $result = $this->cacheService->get($key);

        // Assert
        $this->assertNull($result);
    }
}
```

## 四、API 测试（E2E 测试）

### 4.1 测试范围

**API E2E 测试**：
- 完整的 API 请求流程
- 认证授权流程
- 错误处理
- 数据验证

### 4.2 用户 API 测试示例

**测试文件**：`tests/Feature/Api/User/UserApiTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Feature\Api\User;

use App\Domain\User\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Hash;
use Tests\TestCase;

class UserApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_register_user(): void
    {
        // Arrange
        $userData = [
            'username' => 'testuser',
            'email' => 'test@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
            'nickname' => 'Test User',
        ];

        // Act
        $response = $this->postJson('/api/v1/auth/register', $userData);

        // Assert
        $response->assertStatus(201)
            ->assertJsonStructure([
                'success',
                'data' => [
                    'user' => [
                        'id',
                        'username',
                        'email',
                        'nickname',
                    ],
                ],
            ]);

        $this->assertDatabaseHas('users', [
            'username' => 'testuser',
            'email' => 'test@example.com',
        ]);
    }

    public function test_cannot_register_with_duplicate_email(): void
    {
        // Arrange
        User::factory()->create(['email' => 'existing@example.com']);
        $userData = [
            'username' => 'newuser',
            'email' => 'existing@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ];

        // Act
        $response = $this->postJson('/api/v1/auth/register', $userData);

        // Assert
        $response->assertStatus(422)
            ->assertJsonValidationErrors(['email']);
    }

    public function test_can_login_with_valid_credentials(): void
    {
        // Arrange
        $user = User::factory()->create([
            'email' => 'test@example.com',
            'password' => Hash::make('password123'),
        ]);

        $loginData = [
            'email' => 'test@example.com',
            'password' => 'password123',
        ];

        // Act
        $response = $this->postJson('/api/v1/auth/login', $loginData);

        // Assert
        $response->assertStatus(200)
            ->assertJsonStructure([
                'success',
                'data' => [
                    'user',
                    'access_token',
                    'token_type',
                    'expires_in',
                ],
            ]);
    }

    public function test_cannot_login_with_invalid_credentials(): void
    {
        // Arrange
        $loginData = [
            'email' => 'test@example.com',
            'password' => 'wrongpassword',
        ];

        // Act
        $response = $this->postJson('/api/v1/auth/login', $loginData);

        // Assert
        $response->assertStatus(401)
            ->assertJson([
                'success' => false,
            ]);
    }

    public function test_can_get_current_user(): void
    {
        // Arrange
        $user = User::factory()->create();
        $token = $user->createToken('test-token')->plainTextToken;

        // Act
        $response = $this->withHeader('Authorization', "Bearer {$token}")
            ->getJson('/api/v1/auth/me');

        // Assert
        $response->assertStatus(200)
            ->assertJson([
                'success' => true,
                'data' => [
                    'id' => $user->id,
                    'username' => $user->username,
                    'email' => $user->email,
                ],
            ]);
    }
}
```

### 4.3 文章 API 测试示例

**测试文件**：`tests/Feature/Api/Article/ArticleApiTest.php`

```php
<?php

declare(strict_types=1);

namespace Tests\Feature\Api\Article;

use App\Domain\User\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ArticleApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_get_article_list(): void
    {
        // Act
        $response = $this->getJson('/api/v1/articles');

        // Assert
        $response->assertStatus(200)
            ->assertJsonStructure([
                'success',
                'data' => [
                    '*' => [
                        'id',
                        'title',
                        'slug',
                        'author',
                        'category',
                        'stats',
                    ],
                ],
                'meta' => [
                    'pagination',
                ],
            ]);
    }

    public function test_can_create_article(): void
    {
        // Arrange
        $user = User::factory()->create();
        $token = $user->createToken('test-token')->plainTextToken;

        $articleData = [
            => 'Test Article',
            'summary' => 'Test Summary',
            'content' => '# Test Content',
            'category_id' => 1,
            'status' => 'draft',
        ];

        // Act
        $response = $this->withHeader('Authorization', "Bearer {$token}")
            ->postJson('/api/v1/articles', $articleData);

        // Assert
        $response->assertStatus(201)
            ->assertJsonStructure([
                'success',
                'data' => [
                    'id',
                    'title',
                    'slug',
                ],
            ]);

        $this->assertDatabaseHas('articles', [
            'title' => 'Test Article',
            'author_id' => $user->id,
        ]);
    }

    public function test_cannot_create_article_without_authentication(): void
    {
        // Arrange
        $articleData = [
            'title' => 'Test Article',
            'content' => 'Content',
        ];

        // Act
        $response = $this->postJson('/api/v1/articles', $articleData);

        // Assert
        $response->assertStatus(401);
    }

    public function test_can_update_own_article(): void
    {
        // Arrange
        $user = User::factory()->create();
        $token = $user->createToken('test-token')->plainTextToken;
        $article = \App\Domain\Article\Article::factory()->create([
            'author_id' => $user->id,
        ]);

        $updateData = [
            'title' => 'Updated Title',
        ];

        // Act
        $response = $this->withHeader('Authorization', "Bearer {$token}")
            ->putJson("/api/v1/articles/{$article->id}", $updateData);

        // Assert
        $response->assertStatus(200);
        $this->assertDatabaseHas('articles', [
            'id' => $article->id,
            'title' => 'Updated Title',
        ]);
    }

    public function test_cannot_update_other_user_article(): void
    {
        // Arrange
        $user1 = User::factory()->create();
        $user2 = User::factory()->create();
        $token = $user1->createToken('test-token')->plainTextToken;
        $article = \App\Domain\Article\Article::factory()->create([
            'author_id' => $user2->id,
        ]);

        // Act
        $response = $this->withHeader('Authorization', "Bearer {$token}")
            ->putJson("/api/v1/articles/{$article->id}", [
                'title' => 'Updated Title',
            ]);

        // Assert
        $response->assertStatus(403);
    }
}
```

## 五、测试配置

### 5.1 PHPUnit 配置

**文件**：`phpunit.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         processIsolation="false"
         stopOnFailure="false">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory>tests/Integration</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>app</directory>
        </include>
        <exclude>
            <directory>app/Infrastructure/Persistence/Eloquent/Migrations</directory>
        </exclude>
    </source>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
    </php>
</phpunit>
```

### 5.2 测试辅助方法

**文件**：`tests/TestCase.php`

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;

    protected function setUp(): void
    {
        parent::setUp();
        // 测试前的准备工作
    }

    protected function tearDown(): void
    {
        // 测试后的清理工作
        parent::tearDown();
    }

    /**
     * 创建认证用户并返回 Token
     */
    protected function createAuthenticatedUser(): array
    {
        $user = \App\Domain\User\User::factory()->create();
        $token = $user->createToken('test-token')->plainTextToken;

        return [
            'user' => $user,
            'token' => $token,
        ];
    }
}
```

## 六、测试覆盖率

### 6.1 覆盖率目标

- **总体覆盖率**：> 80%
- **领域层覆盖率**：> 90%
- **应用层覆盖率**：> 85%
- **基础设施层覆盖率**：> 75%
- **表现层覆盖率**：> 70%

### 6.2 生成覆盖率报告

**命令**：
```bash
# 生成 HTML 覆盖率报告
php artisan test --coverage-html coverage

# 生成 Clover XML 报告（用于 CI）
php artisan test --coverage-clover coverage.xml

# 查看覆盖率摘要
php artisan test --coverage
```

### 6.3 覆盖率检查（CI）

**GitHub Actions 配置**：
```yaml
- name: Run Tests with Coverage
  run: |
    php artisan test --coverage-clover coverage.xml

- name: Upload Coverage
  uses: codecov/codecov-action@v3
  with:
    file: ./coverage.xml
```

## 七、测试最佳实践

### 7.1 测试命名

- **单元测试**：`test_方法名_场景_预期结果`
- **集成测试**：`test_can_执行操作`
- **API 测试**：`test_can_操作资源`

### 7.2 测试组织

```
tests/
├── Unit/                    # 单元测试
│   ├── Domain/
│   └── Application/
├── Integration/             # 集成测试
│   └── Infrastructure/
└── Feature/                 # E2E 测试
    └── Api/
```

### 7.3 Mock 使用原则

1. **只 Mock 外部依赖**：数据库、缓存、外部 API
2. **不 Mock 领域对象**：直接使用真实的领域对象
3. **使用真实数据**：尽可能使用真实数据进行测试

### 7.4 测试数据管理

1. **使用 Factory**：创建测试数据
2. **使用 Seeder**：初始化测试数据
3. **使用 RefreshDatabase**：每次测试重置数据库

## 八、持续集成测试

### 8.1 GitHub Actions 配置

**文件**：`.github/workflows/tests.yml`

```yaml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: testing
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: actions/checkout@v3

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: mbstring, pdo_mysql, redis
          coverage: xdebug

      - name: Install Dependencies
        run: composer install --prefer-dist --no-progress

      - name: Copy Environment File
        run: cp .env.testing .env

      - name: Generate Application Key
        run: php artisan key:generate

      - name: Run Migrations
        run: php artisan migrate --force

      - name: Run Tests
        run: php artisan test --coverage-clover coverage.xml

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

## 九、测试命令

### 9.1 运行测试

```bash
# 运行所有测试
php artisan test

# 运行单元测试
php artisan test --testsuite=Unit

# 运行集成测试
php artisan test --testsuite=Integration

# 运行 API 测试
php artisan test --testsuite=Feature

# 运行特定测试文件
php artisan test tests/Unit/Domain/Article/ArticleTest.php

# 运行特定测试方法
php artisan test --filter test_can_create_article

# 并行运行测试（加速）
php artisan test --parallel
```

### 9.2 测试调试

```bash
# 详细输出
php artisan test --verbose

# 停止在第一个失败
php artisan test --stop-on-failure

# 显示代码覆盖率
php artisan test --coverage
```

## 十、下一步

完成测试策略后，继续学习：
- [部署与运维](section-08-deployment.md)：Docker 部署和 CI/CD 配置

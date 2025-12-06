# 9.1.3 技术栈选型

## 一、技术选型原则

### 1.1 选型标准

1. **成熟稳定**：选择经过生产环境验证的技术
2. **社区活跃**：有活跃的社区支持和持续更新
3. **性能优异**：满足高并发和高性能需求
4. **易于维护**：文档完善，学习成本低
5. **生态丰富**：有丰富的第三方库和工具

### 1.2 技术栈概览

| 类别 | 技术选型 | 版本要求 |
|------|----------|----------|
| **后端框架** | Laravel | 11.x |
| **编程语言** | PHP | 8.2+ |
| **数据库** | MySQL | 8.0+ |
| **缓存** | Redis | 7.0+ |
| **消息队列** | Redis Queue / RabbitMQ | - |
| **搜索引擎** | Laravel Scout + MySQL Full-Text | - |
| **测试框架** | PHPUnit + Pest | - |
| **容器化** | Docker + Docker Compose | - |
| **CI/CD** | GitHub Actions | - |

## 二、后端技术栈

### 2.1 PHP 8.2+

**选择理由**：
- 性能大幅提升（JIT 编译器）
- 类型系统完善（联合类型、交集类型）
- 现代语法特性（枚举、只读属性、属性钩子）
- 长期支持版本

**关键特性使用**：
- 严格类型声明（`declare(strict_types=1)`）
- 类型声明（参数、返回值、属性）
- 枚举类型（`ArticleStatus`、`UserRole`）
- 只读属性（值对象）
- 命名参数

**示例**：
```php
<?php
declare(strict_types=1);

enum ArticleStatus: string
{
    case DRAFT = 'draft';
    case PUBLISHED = 'published';
    case DELETED = 'deleted';
}

class Article
{
    public function __construct(
        public readonly ArticleId $id,
        public readonly UserId $authorId,
        public string $title,
        public ArticleStatus $status = ArticleStatus::DRAFT
    ) {}
}
```

### 2.2 Laravel 11.x

**选择理由**：
- 最流行的 PHP 框架
- 丰富的功能组件（认证、授权、队列、缓存等）
- 优雅的语法和开发体验
- 完善的文档和社区支持
- 与 DDD 架构兼容

**核心组件使用**：

1. **路由系统**
   - RESTful API 路由
   - 路由缓存优化
   - API 版本控制

2. **中间件**
   - 认证中间件（JWT）
   - 授权中间件（权限检查）
   - 限流中间件（Rate Limiting）
   - CORS 中间件

3. **Eloquent ORM**
   - 实现 Repository 模式
   - 模型关联
   - 查询构建器
   - 数据迁移

4. **认证系统**
   - JWT 认证（Laravel Sanctum / Passport）
   - 用户认证
   - 权限管理

5. **队列系统**
   - Redis Queue
   - 异步任务处理
   - 失败重试机制

6. **缓存系统**
   - Redis 缓存
   - 缓存标签
   - 缓存预热

7. **事件系统**
   - 领域事件
   - 事件监听器
   - 事件订阅

**项目结构**：
```
app/
├── Domain/              # 领域层
├── Application/         # 应用层
├── Infrastructure/       # 基础设施层
└── Presentation/        # 表现层
```

### 2.3 MySQL 8.0+

**选择理由**：
- 成熟稳定的关系型数据库
- 高性能和可靠性
- 丰富的功能（JSON 字段、窗口函数、CTE）
- 良好的索引支持
- 与 Laravel 完美集成

**关键特性使用**：

1. **JSON 字段**
   - 存储文章元数据
   - 存储用户扩展信息

2. **全文搜索**
   - MySQL Full-Text Index
   - 文章内容搜索

3. **窗口函数**
   - 统计查询
   - 排名查询

4. **事务支持**
   - ACID 特性
   - 事务隔离级别

**数据库配置**：
```php
// config/database.php
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'blog'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'options' => [
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ],
],
```

### 2.4 Redis 7.0+

**选择理由**：
- 高性能内存数据库
- 丰富的数据结构（String、Hash、List、Set、Sorted Set）
- 支持持久化
- 与 Laravel 完美集成

**使用场景**：

1. **缓存**
   - 文章详情缓存
   - 用户信息缓存
   - 热门文章列表缓存
   - 查询结果缓存

2. **会话存储**
   - Session 存储
   - JWT Token 黑名单

3. **计数器**
   - 文章浏览数
   - 点赞数
   - 评论数

4. **限流**
   - API 限流
   - 登录限流

5. **消息队列**
   - 异步任务队列
   - 事件队列

**Redis 配置**：
```php
// config/cache.php
'redis' => [
    'driver' => 'redis',
    'connection' => 'cache',
    'lock_connection' => 'default',
],

// config/database.php
'redis' => [
    'client' => env('REDIS_CLIENT', 'phpredis'),
    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],
    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],
    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],
],
```

## 三、开发工具

### 3.1 Composer

**用途**：
- PHP 依赖管理
- 自动加载（PSR-4）
- 脚本执行

**关键依赖**：
```json
{
    "require": {
        "php": "^8.2",
        "laravel/framework": "^11.0",
        "laravel/sanctum": "^4.0",
        "predis/predis": "^2.0",
        "intervention/image": "^3.0"
    },
    "require-dev": {
        "pestphp/pest": "^2.0",
        "phpunit/phpunit": "^11.0",
        "laravel/pint": "^1.0"
    }
}
```

### 3.2 PHPUnit + Pest

**选择理由**：
- PHPUnit：PHP 标准测试框架
- Pest：现代化的测试框架，语法简洁

**测试配置**：
```php
// phpunit.xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>app</directory>
        </include>
    </source>
</phpunit>
```

### 3.3 Laravel Pint

**用途**：
- 代码格式化
- PSR-12 代码风格检查

**配置**：
```json
// pint.json
{
    "preset": "laravel",
    "rules": {
        "array_syntax": {"syntax": "short"}
    }
}
```

## 四、部署工具

### 4.1 Docker + Docker Compose

**选择理由**：
- 容器化部署，环境一致
- 易于扩展和维护
- 支持多服务编排

**Dockerfile 示例**：
```dockerfile
FROM php:8.2-fpm

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www

# 复制应用文件
COPY . /var/www

# 安装依赖
RUN composer install --optimize-autoloader --no-dev

# 设置权限
RUN chown -R www-data:www-data /var/www
```

**Docker Compose 配置**：
```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/var/www
    networks:
      - blog-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./docker/nginx:/etc/nginx/conf.d
      - .:/var/www
    networks:
      - blog-network
    depends_on:
      - app

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: blog
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - blog-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - blog-network

volumes:
  mysql-data:

networks:
  blog-network:
    driver: bridge
```

### 4.2 GitHub Actions

**用途**：
- 自动化测试
- 自动化部署
- 代码质量检查

**工作流示例**：
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install Dependencies
        run: composer install
      - name: Run Tests
        run: php artisan test
```

## 五、第三方服务

### 5.1 图片处理

**Intervention Image**
- 图片上传处理
- 图片裁剪和缩放
- 图片格式转换

### 5.2 邮件服务

**Laravel Mail**
- SMTP 邮件发送
- 邮件模板
- 队列异步发送

**可选服务**：
- SendGrid
- Mailgun
- AWS SES

### 5.3 文件存储

**Laravel Filesystem**
- 本地存储
- S3 存储（AWS）
- 阿里云 OSS（可选）

## 六、监控和日志

### 6.1 日志系统

**Laravel Logging**
- 文件日志
- 结构化日志（JSON）
- 日志轮转

**配置**：
```php
// config/logging.php
'channels' => [
    'daily' => [
        'driver' => 'daily',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'days' => 14,
    ],
],
```

### 6.2 性能监控

**可选工具**：
- Laravel Telescope（开发环境）
- New Relic（生产环境）
- Sentry（错误追踪）

## 七、安全工具

### 7.1 认证授权

**Laravel Sanctum**
- API Token 认证
- SPA 认证
- 轻量级，易于使用

**Laravel Passport**（可选）
- OAuth2 服务器
- 支持第三方登录

### 7.2 安全防护

**内置功能**：
- CSRF 防护
- XSS 防护
- SQL 注入防护（Eloquent）
- Rate Limiting

**第三方工具**：
- Laravel Security Headers
- Laravel Rate Limiter

## 八、技术栈总结

### 8.1 核心栈

- **后端**：PHP 8.2+ + Laravel 11.x
- **数据库**：MySQL 8.0+
- **缓存**：Redis 7.0+
- **队列**：Redis Queue

### 8.2 开发工具

- **依赖管理**：Composer
- **测试**：PHPUnit + Pest
- **代码风格**：Laravel Pint

### 8.3 部署工具

- **容器化**：Docker + Docker Compose
- **CI/CD**：GitHub Actions

### 8.4 优势

1. **技术成熟**：所有技术都经过生产环境验证
2. **生态丰富**：Laravel 生态完善，第三方库丰富
3. **性能优异**：PHP 8.2+ 性能大幅提升，Redis 缓存加速
4. **易于维护**：代码结构清晰，文档完善
5. **可扩展性**：支持水平扩展和微服务拆分

## 九、下一步

完成技术栈选型后，继续学习：
- [数据库设计](section-04-database-design.md)：设计数据库表结构

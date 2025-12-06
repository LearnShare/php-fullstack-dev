# 9.1.2 技术架构设计

## 一、架构设计原则

### 1.1 设计目标

1. **可维护性**：代码结构清晰，易于理解和修改
2. **可扩展性**：支持功能扩展和性能扩展
3. **可测试性**：代码易于测试，支持单元测试和集成测试
4. **高性能**：支持高并发，响应速度快
5. **安全性**：保障数据安全和系统安全

### 1.2 架构模式选择

本项目采用**六边形架构（Hexagonal Architecture）**结合**领域驱动设计（DDD）**：

- **六边形架构**：将业务逻辑与外部依赖隔离，提高可测试性和可维护性
- **DDD**：以业务领域为核心，构建清晰的领域模型

## 二、整体架构

### 2.1 架构分层

系统采用四层架构：

```
┌─────────────────────────────────────────┐
│        表现层（Presentation Layer）        │
│  HTTP Controllers / API Controllers     │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│        应用层（Application Layer）        │
│      Use Cases / Application Services   │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│        领域层（Domain Layer）            │
│    Entities / Value Objects / Services │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│     基础设施层（Infrastructure Layer）    │
│  Database / Cache / Queue / External   │
└─────────────────────────────────────────┘
```

### 2.2 层次职责

#### 2.2.1 表现层（Presentation Layer）

**职责**：
- 接收 HTTP 请求
- 参数验证和转换
- 调用应用层服务
- 返回响应（JSON/HTML）

**技术实现**：
- Laravel Controllers
- API Resources（数据转换）
- Form Requests（参数验证）
- Middleware（认证、授权、限流）

**示例结构**：
```
app/Presentation/
├── Http/
│   ├── Controllers/
│   │   ├── Api/
│   │   │   ├── ArticleController.php
│   │   │   ├── CommentController.php
│   │   │   └── UserController.php
│   │   └── Web/
│   │       └── HomeController.php
│   ├── Requests/
│   │   ├── CreateArticleRequest.php
│   │   └── UpdateArticleRequest.php
│   └── Resources/
│       ├── ArticleResource.php
│       └── UserResource.php
└── Middleware/
    ├── Authenticate.php
    └── RateLimit.php
```

#### 2.2.2 应用层（Application Layer）

**职责**：
- 实现用例（Use Cases）
- 协调领域层服务
- 事务管理
- 调用基础设施层

**技术实现**：
- Application Services
- DTOs（Data Transfer Objects）
- Event Handlers

**示例结构**：
```
app/Application/
├── Article/
│   ├── CreateArticleService.php
│   ├── UpdateArticleService.php
│   ├── DeleteArticleService.php
│   └── GetArticleService.php
├── Comment/
│   ├── CreateCommentService.php
│   └── DeleteCommentService.php
└── User/
    ├── RegisterUserService.php
    └── UpdateProfileService.php
```

#### 2.2.3 领域层（Domain Layer）

**职责**：
- 业务逻辑核心
- 领域模型（Entities、Value Objects）
- 领域服务（Domain Services）
- 业务规则验证

**技术实现**：
- Domain Entities
- Value Objects
- Domain Services
- Domain Events
- Repository Interfaces

**示例结构**：
```
app/Domain/
├── Article/
│   ├── Article.php              # Entity
│   ├── ArticleId.php            # Value Object
│   ├── ArticleStatus.php        # Value Object (Enum)
│   ├── ArticleRepository.php    # Repository Interface
│   └── ArticleService.php       # Domain Service
├── User/
│   ├── User.php
│   ├── UserId.php
│   ├── Email.php
│   └── UserRepository.php
└── Comment/
    ├── Comment.php
    ├── CommentId.php
    └── CommentRepository.php
```

#### 2.2.4 基础设施层（Infrastructure Layer）

**职责**：
- 数据持久化（Database）
- 缓存（Redis）
- 消息队列
- 外部服务集成

**技术实现**：
- Eloquent Models（实现 Repository）
- Redis Cache
- Queue Jobs
- External API Clients

**示例结构**：
```
app/Infrastructure/
├── Persistence/
│   ├── Eloquent/
│   │   ├── ArticleEloquentRepository.php
│   │   ├── UserEloquentRepository.php
│   │   └── CommentEloquentRepository.php
│   └── Migrations/
├── Cache/
│   └── RedisCacheService.php
├── Queue/
│   └── Jobs/
│       ├── SendEmailJob.php
│       └── ProcessNotificationJob.php
└── External/
    └── EmailService.php
```

## 三、领域模型设计

### 3.1 核心领域

#### 3.1.1 用户领域（User Domain）

**实体**：`User`

**属性**：
- `UserId`：用户 ID（值对象）
- `Email`：邮箱（值对象）
- `Username`：用户名
- `PasswordHash`：密码哈希
- `Nickname`：昵称
- `Avatar`：头像 URL
- `Bio`：个人简介
- `Status`：状态（正常/禁用/删除）
- `EmailVerifiedAt`：邮箱验证时间
- `CreatedAt`：创建时间
- `UpdatedAt`：更新时间

**值对象**：
- `UserId`：用户 ID
- `Email`：邮箱（包含验证逻辑）
- `Password`：密码（包含加密逻辑）

**领域服务**：
- `UserRegistrationService`：用户注册服务
- `UserAuthenticationService`：用户认证服务

**仓储接口**：
```php
interface UserRepository
{
    public function save(User $user): void;
    public function findById(UserId $id): ?User;
    public function findByEmail(Email $email): ?User;
    public function findByUsername(string $username): ?User;
    public function delete(UserId $id): void;
}
```

#### 3.1.2 文章领域（Article Domain）

**实体**：`Article`

**属性**：
- `ArticleId`：文章 ID（值对象）
- `AuthorId`：作者 ID（值对象）
- `Title`：标题
- `Slug`：URL 友好标识
- `Summary`：摘要
- `Content`：正文（Markdown）
- `CoverImage`：封面图片
- `CategoryId`：分类 ID
- `Status`：状态（草稿/已发布/已删除）
- `Visibility`：可见性（公开/私密/仅关注者）
- `ViewCount`：浏览次数
- `LikeCount`：点赞数
- `CommentCount`：评论数
- `PublishedAt`：发布时间
- `CreatedAt`：创建时间
- `UpdatedAt`：更新时间

**值对象**：
- `ArticleId`：文章 ID
- `ArticleStatus`：文章状态（枚举）
- `ArticleVisibility`：可见性（枚举）
- `Slug`：URL 友好标识（包含生成逻辑）

**领域服务**：
- `ArticlePublishingService`：文章发布服务
- `ArticleSlugGenerator`：Slug 生成服务

**仓储接口**：
```php
interface ArticleRepository
{
    public function save(Article $article): void;
    public function findById(ArticleId $id): ?Article;
    public function findBySlug(string $slug): ?Article;
    public function findByAuthor(UserId $authorId, int $page, int $perPage): PaginatedResult;
    public function findPublished(int $page, int $perPage): PaginatedResult;
    public function search(string $keyword, int $page, int $perPage): PaginatedResult;
    public function delete(ArticleId $id): void;
}
```

#### 3.1.3 评论领域（Comment Domain）

**实体**：`Comment`

**属性**：
- `CommentId`：评论 ID（值对象）
- `ArticleId`：文章 ID
- `AuthorId`：作者 ID
- `ParentId`：父评论 ID（支持嵌套）
- `Content`：评论内容
- `Status`：状态（待审核/已审核/已删除）
- `LikeCount`：点赞数
- `CreatedAt`：创建时间
- `UpdatedAt`：更新时间

**值对象**：
- `CommentId`：评论 ID
- `CommentStatus`：评论状态（枚举）

**领域服务**：
- `CommentModerationService`：评论审核服务

**仓储接口**：
```php
interface CommentRepository
{
    public function save(Comment $comment): void;
    public function findById(CommentId $id): ?Comment;
    public function findByArticle(ArticleId $articleId): array;
    public function findByAuthor(UserId $authorId, int $page, int $perPage): PaginatedResult;
    public function delete(CommentId $id): void;
}
```

#### 3.1.4 标签领域（Tag Domain）

**实体**：`Tag`

**属性**：
- `TagId`：标签 ID
- `Name`：标签名称
- `Slug`：URL 友好标识
- `Description`：描述
- `Color`：颜色
- `UsageCount`：使用次数
- `CreatedAt`：创建时间

**仓储接口**：
```php
interface TagRepository
{
    public function save(Tag $tag): void;
    public function findById(TagId $id): ?Tag;
    public function findByName(string $name): ?Tag;
    public function findBySlug(string $slug): ?Tag;
    public function findPopular(int $limit): array;
}
```

### 3.2 领域事件（Domain Events）

**事件设计**：

1. **UserRegistered**：用户注册事件
   - 触发：用户注册成功
   - 处理：发送验证邮件

2. **ArticlePublished**：文章发布事件
   - 触发：文章发布成功
   - 处理：发送通知、更新索引

3. **CommentCreated**：评论创建事件
   - 触发：评论创建成功
   - 处理：发送通知、审核

4. **UserFollowed**：用户关注事件
   - 触发：用户关注成功
   - 处理：发送通知

**事件实现**：
```php
// Domain Event Interface
interface DomainEvent
{
    public function occurredOn(): DateTimeImmutable;
}

// Example: ArticlePublished Event
class ArticlePublished implements DomainEvent
{
    public function __construct(
        private ArticleId $articleId,
        private UserId $authorId,
        private DateTimeImmutable $occurredOn
    ) {}

    public function articleId(): ArticleId
    {
        return $this->articleId;
    }

    public function authorId(): UserId
    {
        return $this->authorId;
    }

    public function occurredOn(): DateTimeImmutable
    {
        return $this->occurredOn;
    }
}
```

## 四、技术架构图

### 4.1 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端层                                │
│  Web Browser / Mobile App / API Client                       │
└─────────────────────────────────────────────────────────────┘
                            ↓ HTTP/HTTPS
┌─────────────────────────────────────────────────────────────┐
│                     表现层（Laravel）                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Web MVC    │  │   API REST   │  │  Middleware  │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     应用层（Use Cases）                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Article      │  │ Comment      │  │ User         │    │
│  │ Services     │  │ Services     │  │ Services     │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     领域层（Domain）                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Entities     │  │ Value        │  │ Domain       │    │
│  │              │  │ Objects      │  │ Services     │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│  ┌──────────────┐  ┌──────────────┐                       │
│  │ Repository  │  │ Domain       │                       │
│  │ Interfaces  │  │ Events       │                       │
│  └──────────────┘  └──────────────┘                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  基础设施层（Infrastructure）                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Eloquent     │  │ Redis        │  │ Queue        │    │
│  │ Repository   │  │ Cache        │  │ Jobs         │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      数据存储层                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ MySQL        │  │ Redis        │  │ File         │    │
│  │ Database     │  │ Cache        │  │ Storage      │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 数据流图

**文章发布流程**：

```
用户请求 → Controller → Application Service → Domain Service
                                              ↓
                                    Domain Entity (Article)
                                              ↓
                                    Repository Interface
                                              ↓
                                    Eloquent Repository
                                              ↓
                                    MySQL Database
                                              ↓
                                    Domain Event (ArticlePublished)
                                              ↓
                                    Event Handler
                                              ↓
                                    Queue Job (SendNotification)
```

## 五、模块划分

### 5.1 功能模块

1. **用户模块（User Module）**
   - 用户注册、登录、认证
   - 用户资料管理
   - 用户权限管理

2. **文章模块（Article Module）**
   - 文章 CRUD
   - 文章发布流程
   - 文章搜索

3. **评论模块（Comment Module）**
   - 评论 CRUD
   - 嵌套评论
   - 评论审核

4. **互动模块（Interaction Module）**
   - 点赞、收藏
   - 用户关注
   - 通知系统

5. **内容管理模块（Content Module）**
   - 分类管理
   - 标签管理
   - 内容审核

6. **管理后台模块（Admin Module）**
   - 用户管理
   - 内容管理
   - 数据统计

### 5.2 共享模块

1. **认证授权模块（Auth Module）**
   - JWT 认证
   - 权限检查
   - 角色管理

2. **缓存模块（Cache Module）**
   - Redis 缓存服务
   - 缓存策略

3. **队列模块（Queue Module）**
   - 异步任务处理
   - 消息队列

4. **通知模块（Notification Module）**
   - 站内通知
   - 邮件通知
   - 推送通知

## 六、设计模式应用

### 6.1 仓储模式（Repository Pattern）

**目的**：隔离领域层和数据访问层

**实现**：
- 领域层定义 Repository 接口
- 基础设施层实现 Repository（使用 Eloquent）

**优势**：
- 领域层不依赖具体的数据访问技术
- 易于测试（可以 Mock Repository）
- 易于切换数据源

### 6.2 服务模式（Service Pattern）

**应用层服务**：
- 实现用例逻辑
- 协调多个领域对象
- 管理事务

**领域服务**：
- 实现跨实体的业务逻辑
- 封装复杂的业务规则

### 6.3 事件驱动模式（Event-Driven Pattern）

**领域事件**：
- 解耦业务逻辑
- 支持异步处理
- 提高系统可扩展性

**实现**：
- Laravel Events
- Queue Jobs

### 6.4 策略模式（Strategy Pattern）

**应用场景**：
- 内容审核策略（自动/人工）
- 缓存策略（不同缓存时间）
- 通知策略（站内/邮件/推送）

## 七、安全架构

### 7.1 认证授权

1. **JWT Token 认证**
   - Access Token（短期，15 分钟）
   - Refresh Token（长期，7 天）
   - Token 刷新机制

2. **权限控制**
   - 基于角色的访问控制（RBAC）
   - 中间件权限检查
   - 资源级权限控制

### 7.2 数据安全

1. **输入验证**
   - Form Request 验证
   - 参数类型检查
   - SQL 注入防护（PDO 预处理）

2. **输出过滤**
   - XSS 防护（内容转义）
   - CSRF 防护（Token 验证）

3. **敏感数据保护**
   - 密码加密存储（bcrypt）
   - 敏感信息脱敏

### 7.3 安全措施

1. **Rate Limiting**
   - API 限流
   - 登录限流
   - 操作频率限制

2. **内容安全**
   - 敏感词过滤
   - 内容审核
   - 用户行为监控

## 八、性能架构

### 8.1 缓存策略

1. **数据缓存**
   - 文章详情缓存（Redis）
   - 用户信息缓存
   - 热门文章列表缓存

2. **查询缓存**
   - 数据库查询结果缓存
   - 统计信息缓存

3. **页面缓存**
   - 静态页面缓存（可选）
   - CDN 缓存

### 8.2 数据库优化

1. **索引设计**
   - 主键索引
   - 外键索引
   - 查询字段索引
   - 全文搜索索引

2. **查询优化**
   - 避免 N+1 查询（Eager Loading）
   - 分页查询优化
   - 慢查询监控

### 8.3 异步处理

1. **消息队列**
   - 邮件发送（异步）
   - 通知发送（异步）
   - 内容审核（异步）

2. **后台任务**
   - 数据统计
   - 内容索引更新
   - 缓存预热

## 九、扩展性设计

### 9.1 水平扩展

1. **无状态设计**
   - 应用服务器无状态
   - Session 存储在 Redis
   - 支持多实例部署

2. **数据库扩展**
   - 读写分离
   - 分表分库（未来）

### 9.2 微服务拆分（未来）

**潜在拆分**：
- 用户服务（User Service）
- 文章服务（Article Service）
- 评论服务（Comment Service）
- 通知服务（Notification Service）

**通信方式**：
- REST API
- 消息队列（事件驱动）

## 十、下一步

完成架构设计后，继续学习：
- [技术栈选型](section-03-technology-stack.md)：选择具体的技术和工具

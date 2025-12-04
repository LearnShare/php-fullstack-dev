# 9.1 SaaS 平台完整方案设计

## 一、方案设计

### 1.1 业务目标

构建一个多租户 SaaS 平台，支持：
- 用户注册、登录、权限管理
- 多租户数据隔离（基于数据库/表级别）
- 订阅计费系统（支持多种计费模式）
- 工作空间（Workspace）管理
- 模块化扩展能力

### 1.2 架构设计

#### 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                     负载均衡器 (Nginx)                    │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
┌───────▼──────┐ ┌────▼─────┐ ┌──────▼──────┐
│  Web 服务器   │ │ Web 服务器│ │ Web 服务器  │
│  (PHP-FPM)   │ │ (PHP-FPM)│ │ (PHP-FPM)   │
└───────┬──────┘ └────┬─────┘ └──────┬──────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
┌───────▼──────┐ ┌────▼─────┐ ┌──────▼──────┐
│   MySQL      │ │  Redis   │ │  队列服务   │
│  (主从)      │ │ (集群)   │ │ (RabbitMQ)  │
└──────────────┘ └──────────┘ └─────────────┘
```

#### 模块化单体架构

```
src/
├── Modules/
│   ├── Auth/                    # 认证模块
│   │   ├── Domain/
│   │   │   ├── User.php         # 用户实体
│   │   │   ├── Role.php         # 角色实体
│   │   │   └── Permission.php   # 权限实体
│   │   ├── Application/
│   │   │   ├── AuthService.php  # 认证服务
│   │   │   └── UserService.php   # 用户服务
│   │   └── Infrastructure/
│   │       ├── UserRepository.php
│   │       └── TokenRepository.php
│   │
│   ├── Billing/                 # 计费模块
│   │   ├── Domain/
│   │   │   ├── Subscription.php # 订阅实体
│   │   │   ├── Plan.php         # 套餐实体
│   │   │   └── Invoice.php      # 发票实体
│   │   ├── Application/
│   │   │   ├── BillingService.php
│   │   │   └── PaymentService.php
│   │   └── Infrastructure/
│   │       ├── SubscriptionRepository.php
│   │       └── PaymentGatewayAdapter.php
│   │
│   ├── Workspaces/              # 工作空间模块
│   │   ├── Domain/
│   │   │   ├── Workspace.php    # 工作空间实体
│   │   │   └── Member.php       # 成员实体
│   │   └── Application/
│   │       └── WorkspaceService.php
│   │
│   └── Tenant/                  # 租户模块
│       ├── Domain/
│       │   └── Tenant.php       # 租户实体
│       └── Application/
│           └── TenantService.php
│
├── Shared/                      # 共享模块
│   ├── Kernel/
│   │   ├── TenantResolver.php   # 租户解析器
│   │   └── TenantMiddleware.php  # 租户中间件
│   ├── Infrastructure/
│   │   ├── Database/
│   │   │   └── TenantConnection.php
│   │   └── Events/
│   │       └── EventDispatcher.php
│   └── Contracts/
│       └── RepositoryInterface.php
│
└── public/
    └── index.php
```

#### 多租户实现策略

**方案选择：共享数据库、独立 Schema**

- **优点**：数据隔离清晰，易于备份恢复，支持租户级扩展
- **实现**：每个租户使用独立的数据库 Schema
- **识别方式**：基于子域名（`tenant1.app.com`）或 Header（`X-Tenant-ID`）

### 1.3 核心模块设计

#### 租户识别与路由

```php
<?php
declare(strict_types=1);

namespace App\Shared\Kernel;

class TenantResolver
{
    public function resolveFromRequest(array $request): ?Tenant
    {
        // 方式1：从子域名识别
        $host = $request['HTTP_HOST'] ?? '';
        $subdomain = $this->extractSubdomain($host);
        
        // 方式2：从 Header 识别
        $tenantId = $request['HTTP_X_TENANT_ID'] ?? null;
        
        // 方式3：从路径识别
        $path = $request['REQUEST_URI'] ?? '';
        $tenantId = $this->extractFromPath($path);
        
        return $this->findTenant($subdomain ?? $tenantId);
    }
}
```

#### 数据隔离实现

```php
<?php
declare(strict_types=1);

namespace App\Shared\Infrastructure\Database;

class TenantConnection
{
    public function switchDatabase(string $tenantId): void
    {
        $tenant = $this->tenantRepository->find($tenantId);
        $dsn = sprintf(
            'mysql:host=%s;dbname=%s;charset=utf8mb4',
            $tenant->getDatabaseHost(),
            $tenant->getDatabaseName()
        );
        
        $this->pdo = new PDO($dsn, $tenant->getDatabaseUser(), $tenant->getDatabasePassword());
    }
}
```

## 二、技术要求

### 2.1 技术栈

#### 后端
- **PHP**: 8.3+
- **框架**: Laravel 11+ 或 自建微框架
- **数据库**: MySQL 8.0+（主从复制）
- **缓存**: Redis 7.0+（集群模式）
- **队列**: RabbitMQ 或 Redis Stream
- **搜索**: Elasticsearch（可选）

#### 前端
- **Web**: Vue 3 + TypeScript 或 React 18+
- **移动端**: React Native 或 Flutter（可选）

#### 基础设施
- **Web 服务器**: Nginx 1.24+
- **应用服务器**: PHP-FPM 或 RoadRunner
- **容器**: Docker + Docker Compose
- **CI/CD**: GitHub Actions 或 GitLab CI
- **监控**: Prometheus + Grafana

### 2.2 依赖清单

```json
{
  "require": {
    "php": "^8.3",
    "laravel/framework": "^11.0",
    "laravel/sanctum": "^4.0",
    "stripe/stripe-php": "^13.0",
    "predis/predis": "^2.2",
    "monolog/monolog": "^3.0",
    "guzzlehttp/guzzle": "^7.8"
  },
  "require-dev": {
    "phpunit/phpunit": "^11.0",
    "laravel/pint": "^1.13",
    "phpstan/phpstan": "^1.10"
  }
}
```

### 2.3 环境要求

- **开发环境**: Docker Desktop / Docker Compose
- **生产环境**: 
  - 最低配置：2 CPU, 4GB RAM, 50GB SSD
  - 推荐配置：4 CPU, 8GB RAM, 100GB SSD
  - 数据库：独立服务器，8GB+ RAM

## 三、开发迭代部署方案

### 3.1 MVP（最小可行产品）阶段

#### 功能范围（4 周）

**Week 1: 基础架构**
- [ ] 项目初始化与模块划分
- [ ] 数据库设计与迁移
- [ ] 租户识别与路由
- [ ] 基础认证系统

**Week 2: 核心功能**
- [ ] 用户注册/登录
- [ ] 工作空间创建与管理
- [ ] 基础权限系统（RBAC）

**Week 3: 计费系统**
- [ ] 套餐管理
- [ ] 订阅创建与更新
- [ ] 支付集成（Stripe/PayPal）

**Week 4: 测试与部署**
- [ ] 单元测试与集成测试
- [ ] 性能测试
- [ ] 部署到 Staging 环境

#### 部署流程

```bash
# 1. 代码部署
git clone https://github.com/your-org/saas-platform.git
cd saas-platform
composer install --no-dev --optimize-autoloader

# 2. 环境配置
cp .env.example .env
php artisan key:generate
php artisan config:cache
php artisan route:cache

# 3. 数据库迁移
php artisan migrate --force

# 4. 队列启动
php artisan queue:work --daemon

# 5. 服务重启
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```

### 3.2 迭代计划

#### 迭代 1：MVP（4 周）
- 基础功能上线
- 支持 10 个测试租户

#### 迭代 2：增强功能（3 周）
- [ ] 多租户数据统计
- [ ] 邮件通知系统
- [ ] 日志与审计

#### 迭代 3：性能优化（2 周）
- [ ] 缓存策略优化
- [ ] 数据库查询优化
- [ ] CDN 集成

#### 迭代 4：高级功能（4 周）
- [ ] API 开放平台
- [ ] Webhook 支持
- [ ] 第三方集成

### 3.3 部署策略

#### 蓝绿部署

```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  app-green:
    build: .
    environment:
      - APP_ENV=production
    deploy:
      replicas: 3
  
  app-blue:
    build: .
    environment:
      - APP_ENV=production
    deploy:
      replicas: 0  # 初始为 0，切换时调整
```

#### 部署脚本

```bash
#!/bin/bash
# deploy.sh

# 1. 构建新版本
docker build -t saas-platform:v${VERSION} .

# 2. 健康检查
docker run --rm saas-platform:v${VERSION} php artisan health:check

# 3. 切换到新版本（蓝绿切换）
./switch-deployment.sh green blue

# 4. 监控 5 分钟
sleep 300
if [ $(check-error-rate) -lt 0.01 ]; then
    echo "部署成功"
else
    echo "回滚"
    ./switch-deployment.sh blue green
fi
```

## 四、持续迭代方案

### 4.1 监控体系

#### 应用监控

```php
<?php
// app/Http/Middleware/MonitoringMiddleware.php

class MonitoringMiddleware
{
    public function handle($request, Closure $next)
    {
        $startTime = microtime(true);
        $startMemory = memory_get_usage();
        
        $response = $next($request);
        
        // 记录指标
        $this->recordMetrics([
            'duration' => microtime(true) - $startTime,
            'memory' => memory_get_usage() - $startMemory,
            'status' => $response->getStatusCode(),
            'tenant_id' => $request->tenant()->id,
        ]);
        
        return $response;
    }
}
```

#### 监控指标

- **性能指标**: 响应时间、吞吐量、错误率
- **业务指标**: 租户数量、订阅数、收入
- **基础设施**: CPU、内存、磁盘、网络

### 4.2 优化策略

#### 数据库优化

```sql
-- 租户表分区（按月）
ALTER TABLE events PARTITION BY RANGE (MONTH(created_at)) (
    PARTITION p202401 VALUES LESS THAN (2),
    PARTITION p202402 VALUES LESS THAN (3),
    -- ...
);

-- 索引优化
CREATE INDEX idx_tenant_created ON events(tenant_id, created_at);
```

#### 缓存策略

```php
<?php
// 租户级缓存
Cache::tags(['tenant:' . $tenantId])->put('key', $value);

// 全局缓存
Cache::remember('global:stats', 3600, function() {
    return $this->calculateStats();
});
```

### 4.3 扩展方案

#### 水平扩展

- **应用层**: 无状态设计，支持多实例部署
- **数据库层**: 读写分离，分库分表（按租户）
- **缓存层**: Redis 集群模式

#### 功能扩展

- **插件系统**: 支持第三方模块开发
- **API 开放**: 提供 RESTful API 和 GraphQL
- **Webhook**: 支持事件通知

### 4.4 迭代节奏

#### 双周迭代

- **Week 1**: 需求评审、开发
- **Week 2**: 测试、部署、复盘

#### 发布流程

1. **开发分支**: `feature/xxx`
2. **测试分支**: `staging`（自动部署到测试环境）
3. **生产分支**: `main`（手动审批后部署）

### 4.5 持续改进

#### 技术债务管理

- 每迭代预留 20% 时间处理技术债务
- 定期代码审查与重构
- 性能测试与优化

#### 用户反馈循环

- 用户反馈收集（支持系统、问卷）
- 数据分析（用户行为、功能使用率）
- 产品迭代决策（基于数据驱动）

## 五、风险评估与应对

### 5.1 技术风险

| 风险 | 影响 | 应对措施 |
| :--- | :--- | :--- |
| 数据库性能瓶颈 | 高 | 读写分离、分库分表、缓存优化 |
| 单点故障 | 高 | 多实例部署、负载均衡、故障转移 |
| 数据安全 | 高 | 加密存储、访问控制、审计日志 |

### 5.2 业务风险

| 风险 | 影响 | 应对措施 |
| :--- | :--- | :--- |
| 用户增长过快 | 中 | 弹性扩容、限流策略 |
| 计费系统故障 | 高 | 备用支付通道、人工对账 |
| 合规要求 | 中 | GDPR 合规、数据备份 |

## 六、成功指标

### 6.1 技术指标

- **可用性**: 99.9%+
- **响应时间**: P95 < 200ms
- **错误率**: < 0.1%

### 6.2 业务指标

- **用户增长**: 月增长 20%+
- **留存率**: 30 天留存 > 60%
- **收入**: MRR（月度经常性收入）持续增长

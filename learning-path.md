---
author: Gemini, ChatGPT
createdAt: 2025-11-28
version: 2.0
---

# PHP + MySQL 全栈开发指南

**适用对象：具有网页开发基础（HTML/CSS/JS/TS），但缺乏 PHP/后端经验的工程师**
**目标：从零开始掌握 PHP 8.2+、现代运行时、OOP 架构，为后续进阶到 Laravel / 企业级开发奠定基础**

-----

### 🚀 阶段一：环境、运行时与工具链（基础设施）

#### 1.1 PHP 安装与运行基础

  * 如何选择 PHP 版本（PHP 8.2/8.3 推荐）
  * 各系统的现代安装方式（Laragon / Herd / Homebrew / Docker）
  * 配置切换 CLI 与 FPM 的版本

#### 1.2 MySQL 环境搭建与工具

  * MySQL 版本选择（8.0+ 必须）
  * 本地安装与 Docker 容器运行（推荐）
  * 数据库客户端工具（TablePlus / DataGrip）

#### 1.3 核心工具链（现代 PHP 必备）

  * Composer（依赖管理、`vendor` 目录、自动加载的根本意义）
  * IDE 选择与配置（VS Code + 扩展 / PhpStorm）

#### 1.4 配置、扩展与调试

  * `php.ini` 基础（找出真正生效的配置）
  * 常用配置讲解：`display_errors`、`memory_limit` 等
  * PHP 扩展（mysqli/pdo\_mysql, curl, mbstring 等）
  * **Xdebug（断点调试必学）**：原理、如何在 IDE 中打断点

#### 1.5 PHP 执行模式与 Web 架构

  * CLI（命令行模式）与内置 Web 服务器
  * **Nginx + PHP-FPM 工作原理（必学）**：FastCGI、FPM Pool、Worker
  * Shared Nothing 架构（请求生命周期、高并发、无状态服务）

#### 1.6 现代 PHP 运行时与新生态

  * RoadRunner（Go 驱动的常驻运行时）
  * FrankenPHP（内置 HTTP Server，Worker 持久化）
  * Open Swoole（协程 Event Loop）
  * Laravel Octane（框架级常驻内存加速）

#### 1.7 Worker、协程、Event Loop

  * Worker 重复处理请求
  * 什么是协程（Coroutine）
  * PHP 8 的 Fibers 基础能力

-----

### 📘 阶段二：PHP 基础语法·零基础完全体（语言精通）

#### 2.1 PHP 基本语法结构

  * PHP 标签、文件结构
  * 语句、分号、注释
  * Whitespace 与 UTF-8 编码

#### 2.2 输出与调试

  * `echo` vs `print`
  * `var_dump` 的作用与示例
  * `print_r` 的使用场景
  * `printf / sprintf` 格式化输出

#### 2.3 变量与常量

  * 变量命名规则、动态类型
  * 变量的引用（`&`）
  * `const` vs `define()`
  * 魔术常量：`__FILE__`, `__DIR__`, `__LINE__`

#### 2.4 数据类型

  * 标量类型（布尔、整数、浮点数）
  * 字符串（单双引号差异、Nowdoc/Heredoc）
  * 数组（索引/关联/混合）、NULL
  * 类型检测：`is_*()` 系列

#### 2.5 类型转换与比较

  * 隐式转换规则（Type Juggling）
  * 强制转换（`(int)`、`(float)` 等）
  * `intval`/`floatval`
  * 严格比较 `===` vs 宽松比较 `==`

#### 2.6 表达式与运算符

  * 算术/比较/逻辑/赋值运算符
  * 字符串连接（`.`）
  * Null 合并运算符（`??`）
  * 三元表达式

#### 2.7 字符串操作

  * 转义规则
  * 常用函数：`substr`、`strpos`、`explode`、`implode`
  * **UTF-8 与 `mb_*` 函数**（多字节处理）

#### 2.8 数组完整指南

  * 数组创建与解构（`list`, `[]`）
  * 高阶函数：`array_map`, `array_filter`, `array_reduce`
  * 排序函数：`sort`, `asort`, `ksort`
  * 与 JSON 相互转换（`json_encode`/`json_decode`）

#### 2.9 控制结构

  * `if / else / elseif`
  * `switch` 与 **`match` 表达式**
  * `while / do…while / for`
  * `foreach`（键值、引用遍历）
  * `break` / `continue`

#### 2.10 函数与作用域

  * 函数定义、参数默认值、可变参数（`...$args`）
  * **按引用传参（`&`）**
  * 局部、全局、`global` 关键字
  * **`static` 静态变量** 与生命周期

#### 2.11 匿名函数与闭包

  * 匿名函数作为回调
  * 闭包与 `use` 关键字
  * 返回值与参数类型声明

#### 2.12 超级全局变量

  * `$_GET`, `$_POST` 接收数据
  * `$_SERVER`, `$_COOKIE`, `$_SESSION`, `$_FILES`
  * 典型使用场景与安全基础

#### 2.13 文件引入与模块化

  * `include` vs `require`
  * `_once` 版本的作用（防止重复定义）
  * Composer autoload 快速介绍（现代模块化基础）

#### 2.14 文件系统操作

  * 路径魔术常量（`__DIR__`）
  * `file_get_contents` / `file_put_contents`
  * 底层 I/O（`fopen/fread/fwrite`）
  * 文件检查（`file_exists` / `is_dir`）

#### 2.15 时间与日期处理

  * Unix 时间戳 `time()`
  * `date()` 函数格式化
  * **`DateTime` / `DateTimeZone`** (面向对象 API)
  * 时区设置与国际化

#### 2.16 isset / empty / Null 体系

  * `isset` 的本质（存在且非 NULL）
  * `empty` 的规则（零值、空字符串、空数组）
  * Null 合并运算符（`??`）

#### 2.17 错误与异常处理

  * 错误类型（E\_ERROR, E\_WARNING, E\_NOTICE）
  * `error_reporting` 配置
  * **`try/catch/finally`**
  * `Error` vs `Exception` vs **`Throwable`**

#### 2.18 代码规范

  * PHPDoc（`@var`, `@param`, `@return`）
  * PSR-1 基本原则（命名风格）
  * PSR-12 代码风格（缩进、括号）

#### 2.19 本章小结与练习

-----

### 🚀 阶段三：面向对象、架构与设计模式（高级工程）

#### 3.1 类、对象与基础 OOP

  * 类与对象的概念、属性与类型声明
  * 方法与可见性（public/protected/private）
  * 构造函数 `__construct()`、析构函数 `__destruct()`
  * `$this` 的意义与 `clone`

#### 3.2 现代 PHP 8+ OOP 能力

  * **构造器属性提升**（减少重复代码）
  * **Readonly** 属性与类（不可变对象）
  * **Enums**（枚举）在状态管理中的应用
  * 静态属性与方法（滥用风险）

#### 3.3 OOP 三大特性

  * 继承 `extends` 与 `final` 关键字
  * **接口 `interface`**（强类型契约）
  * 抽象类 `abstract`（模板模式）

#### 3.4 代码模块化与元编程

  * **Traits**（混入能力）与场景
  * **Attributes**（现代 PHP 注解系统）创建与应用

#### 3.5 命名空间与自动加载

  * **Namespace** 完整讲解（命名冲突、类隔离）
  * **Composer 自动加载（PSR-4）**：路径映射原理，构建组件

#### 3.6 异常体系与框架级设计

  * `Throwable` 捕获（现代写法）
  * 自定义异常
  * 设计框架级异常体系

#### 3.7 应用架构模式

  * **MVC 与 ADR**（Action-Domain-Responder）
  * 模块化单体（Domain/Infrastructure/Application 组织）

#### 3.8 六边形架构

  * Hexagonal Architecture / Ports & Adapters 原理
  * 适配器模式在 PHP 框架中的体现

#### 3.9 DDD 领域驱动设计初级入门

  * Entity 与 Value Object（immutable）
  * Aggregate Root
  * Repository（仓储）接口

#### 3.10 数据流设计进阶

  * 命令（Command）与查询（Query）
  * 事件溯源（Event Store）与事件总线（Event Bus）

#### 3.11 阶段总结

  * 具备框架级结构理解、模块化设计能力。

-----

### 🌐 阶段四：Web 服务与 API-First 开发 (Web Essentials · 教材加强版)

#### 4.1 Web 交互基础：从请求到响应

  * HTTP 请求到达 Web Server（Nginx/Apache）
  * Server 将请求交给 PHP-FPM / 运行时
  * PHP 处理并返回纯文本（HTML/JSON/文件）
  * 响应通过 Server 返回浏览器
    ➡ 这一流程要配图讲，便于理解。

#### 4.2 HTML 渲染与输出控制

  * PHP 的三种页面渲染方式
    1.  纯 PHP 输出 HTML
    2.  PHP 模板嵌入（最常见）
    3.  模板引擎（Twig / Blade）
  * 输出控制：`echo` 输出 HTML
  * 输出缓冲：`ob_start()`, `ob_get_clean()`, 模板渲染场景使用
  * 为什么模板引擎能提高安全性（自动转义 XSS）

#### 4.3 超全局变量：Web 输入的核心

  * 深入讲解：`$_GET`、`$_POST`：表单数据、查询参数
  * `$_REQUEST`：为什么不建议使用
  * `$_SERVER`：获取 Header、Method、路径
  * `$_COOKIE`：会话管理必备
  * 输入过滤：`filter_input`、`htmlspecialchars`
  * 要强调安全处理、类型转换、编码问题。

#### 4.4 请求体解析：处理 JSON / API 请求

  * 为什么 AJAX / API 传 JSON 时 `$_POST` 是空的
  * 如何使用：

<!-- end list -->

```php
$json = file_get_contents("php://input");
$data = json_decode($json, true);
```

  * 需要讲解：Content-Type: application/json，JSON 序列化/反序列化安全问题

#### 4.5 文件上传处理

  * 表单编码 `multipart/form-data`
  * `$_FILES` 数据结构详解（数组形式）
  * 常见风险：MIME 欺骗、文件名伪造、图片木马
  * 安全策略：`mime_content_type()`、随机文件名、独立文件目录

#### 4.6 RESTful API 设计与接口规范

  * HTTP 方法的语义
  * 资源路径的规范设计
  * 错误响应格式
  * JSON API 标准

#### 4.7 响应处理与跨域（CORS）

  * 设置状态码：`http_response_code()`
  * 设置 Header：`header()`
  * CORS 三步：Access-Control-Allow-Origin, Preflight, Credentials

#### 4.8 会话与状态管理

  * Web 的无状态特性
  * 会话方式比较：Cookie，Session（服务器存），Token / JWT（无状态 API），Redis session store
  * Session 原理（session\_id cookie），JWT 不适合存敏感信息，Token 失效策略

#### 4.9 鉴权与授权模型（AuthN & AuthZ）

  * 四种企业级方式：基础 Token 鉴权，JWT 鉴权，OAuth2（第三方登录），RBAC/ABAC 授权控制
  * 企业级内容：Access Token / Refresh Token 机制，权限模型设计（Role-Permission 关系）

#### 4.10 流量治理与安全

  * API Rate Limiting：固定窗口、滑动窗口、Redis 实现方式
  * 防止重复提交（幂等性 Token）
  * 请求签名（Timestamp + HMAC）

-----

### 💾 阶段五：数据持久化与日志管理 (Data Persistence & Logging · 教材加强版)

#### 5.1 数据库设计基础

  * ER 图（示例），一、二、三范式，反范式适用场景
  * 字符集：`utf8` vs `utf8mb4`，排序规则：`utf8mb4_unicode_ci`

#### 5.2 PDO 入门与高安全模式

  * DSN 格式，连接选项（error mode、fetch mode）
  * **预处理语句防注入**，CRUD 示例
  * 强调：SQL 注入原理，`bindParam` vs `bindValue`，返回行数影响

#### 5.3 MySQL 事务处理

  * ACID 四特性
  * `beginTransaction()` → `commit()` → `rollBack()`
  * 事务隔离级别，死锁与重试策略

#### 5.4 现代 MySQL 高级特性

  * JSON 字段（查询、索引、存储结构）
  * Window Functions（排名、累计统计）
  * Fulltext 索引 vs LIKE

#### 5.5 并发控制与 MVCC

  * 快照读、当前读
  * Undo Log
  * 为什么 InnoDB 是主流
  * Phantom Read

#### 5.6 ORM 框架与数据迁移

  * Eloquent：简洁、Laravel 风格
  * Doctrine：强 DDD、Repository 模式
  * 用迁移版本化数据库
  * Seeder 填充数据

#### 5.7 Redis 缓存策略与性能优化

  * 缓存模式
  * 缓存一致性
  * 击穿 / 穿透 / 雪崩处理
  * SetNX 分布式锁

#### 5.8 文件系统流式操作

  * 读写文件
  * 处理大文件（流式处理）
  * 文件句柄与缓冲区
  * 避免竞争条件

#### 5.9 日志体系与监控

  * PSR-3 标准
  * Monolog 设计与频道（Channel）
  * 结构化日志（JSON 日志）
  * 请求链路追踪（Trace ID）
  * 错误通知（Email/Slack/Sentry）

-----

### 🛡 阶段六：安全、性能与可观测性 (Production Ready)

#### 6.1 OWASP Top 10（2025）风险模型与防御

  * XSS（含 DOM-Based XSS 处理, HTML Purifier）
  * CSRF（含双提交 Cookie、SameSite 策略）
  * SQL Injection（PDO 预处理 + ORM 防注入）
  * Broken Authentication（登录防爆破、2FA）
  * Security Misconfig（HTTP header、CSP）
  * SSRF / RCE / Template Injection 说明与防护

#### 6.2 密码与加密安全

  * `password_hash`、`password_verify`、Argon2id 参数最佳实践
  * Sodium 密码库：对称/非对称加密、签名
  * 敏感数据脱敏（Masking）

#### 6.3 Secrets 管理

  * `.env` 安全
  * Vault / AWS Secret Manager / Doppler
  * 密钥轮换（key rotation）

#### 6.4 安全 HTTP 头

  * CSP（内容安全策略）
  * HSTS、X-Frame-Options、防点击劫持
  * Referrer-Policy、Permissions-Policy（硬化浏览器能力）

#### 6.5 PHP 性能优化

  * OPcache 配置（JIT on/off 场景）
  * FPM worker 优化（pm.max\_children）
  * Runtime：FrankenPHP / OpenSwoole 的性能差异分析

#### 6.6 数据库深度优化

  * N+1 问题识别与消除（Eloquent / Doctrine）
  * 慢查询分析流程：`EXPLAIN`、profiling、索引设计
  * 分库分表基础、读写分离

#### 6.7 异步与分布式处理

  * 消息队列 MQ 对比：Redis Stream / RabbitMQ / Kafka
  * 事件驱动架构（EDA）
  * WebSocket / SSE / MQTT 对比

#### 6.8 测试体系建设 (Testing Pyramid)

  * Unit / Integration / Feature / End-to-End 测试关系
  * Pest vs PHPUnit：现代语法、快照测试 (Snapshot Testing)

#### 6.9 Mock、Stub、Fakes 深度讲解

  * Mock 服务层、Repository
  * 使用 Mockery、Pest fakes

#### 6.10 可观测性体系（Observability）

  * 三大支柱：日志、指标、链路追踪
  * OpenTelemetry（自动埋点 + 自定义 spans）
  * Prometheus：指标暴露 + Grafana 可视化
  * Sentry：错误监控（release + source map）

-----

### 🚀 阶段七：现代框架深度应用 (Framework Mastery)

#### 7.1 IoC 与 DI 设计模式深度理解

  * 服务容器的生命周期
  * Provider、Binding、Singleton
  * 自动注入（autowire）与 constructor vs property 注入

#### 7.2 路由、中间件、Pipeline

  * 任意框架通用的“洋葱模型（Onion Model）”
  * HTTP Kernel（Laravel）分析
  * Symfony HttpFoundation 请求/响应模型

#### 7.3 Laravel（2025 最新特性）

  * Livewire（服务器端 UI 组件）
  * Inertia（SPA without API）
  * Octane（Swoole/RoadRunner 常驻内存）
  * Reverb（WebSocket 全双工实时通信）
  * Horizon（队列监控）
  * Telescope（调试框架）

#### 7.4 Symfony

  * Messenger：队列 + 总线（CommandBus/EventBus）
  * Workflow：业务流程（状态机）
  * API Platform：HATEOAS + GraphQL + JSON-LD
  * Symfony Runtime + Fiber 支持

-----

### ☁️ 阶段八：部署、云原生与 DevOps (Cloud Native)

#### 8.1 专业 Dockerfile

  * FPM + Nginx 分离容器
  * multi-stage 构建（Builder → Production）
  * 优化镜像大小（`--no-dev`, Alpine vs Debian）

#### 8.2 docker-compose 与本地开发环境

  * PHP / MySQL / Redis / Mailpit
  * 生产 vs 本地不同 compose 文件策略

#### 8.3 镜像安全

  * Trivy 扫描漏洞
  * 镜像签名（cosign）

#### 8.4 部署选择

  * AWS EC2 / ECS / ECR
  * Laravel Forge / Vapor（Serverless）
  * DigitalOcean Apps

#### 8.5 Kubernetes (K8s)

  * Deployment / StatefulSet / DaemonSet
  * Service、Ingress、HorizontalPodAutoscaler（HPA）
  * ConfigMap / Secrets / PVC（持久化）

#### 8.6 Serverless PHP

  * Bref（AWS Lambda）
  * 冷启动优化策略
  * 无服务器事件驱动架构

#### 8.7 CI/CD 与 GitHub Actions

  * Lint → Test → Build → Deploy pipeline
  * 缓存 Composer 依赖
  * 自动化测试覆盖率报告

#### 8.8 Deploy 策略

  * Blue/Green, Canary 部署
  * Zero Downtime 部署策略
  * Database migration 安全策略（Atomic Migration）

#### 8.9 GitOps

  * ArgoCD 应用交付
  * 全自动化同步与回滚

-----

### 🏁 阶段九：高质量实战项目 (Capstone)

#### 9.1 SaaS 平台（核心项目）

  * Modular Monolith 模块化架构
  * DDD 分层（Domain / Application / Infra）
  * 租户系统（Multi-tenancy）
  * Billing（Stripe/LemonSqueezy）

#### 9.2 高并发实时应用

  * WebSocket + Reverb + Redis Stream
  * 聊天系统、白板协作、实时仪表板

#### 9.3 API-First 企业服务

  * OpenAPI 3.1 + API 文档自动生成
  * SDK 自动生成
  * PHPUnit + Pest E2E 测试

#### 9.4 生产级部署

  * Docker + GitHub Actions + ECS
  * 监控：Prometheus + Grafana
  * 日志：ELK / Loki

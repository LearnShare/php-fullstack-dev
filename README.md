# PHP + MySQL 全栈开发指南

**适用对象**：具有网页开发基础（HTML/CSS/JS/TS），但缺乏 PHP/后端经验的工程师

**目标**：从零开始掌握 PHP 8.2+、现代运行时、OOP 架构，为后续进阶到 Laravel / 企业级开发奠定基础

## 文档说明

本指南采用分阶段、分章节的结构，每个阶段和章节都是独立的文档文件，便于学习和查阅。所有内容面向零基础学员设计，提供详细的概念解释、语法说明、参数列表、完整示例代码和练习任务。

## 目录结构

### 阶段一：环境、运行时与工具链（基础设施）

- [阶段一总览](docs/stage-01-foundation/README.md)
- [1.1 PHP 安装与运行基础](docs/stage-01-foundation/chapter-01-runtime/README.md)
- [1.2 MySQL 环境搭建与工具](docs/stage-01-foundation/chapter-02-mysql/README.md)
- [1.3 核心工具链](docs/stage-01-foundation/chapter-03-toolchain/README.md)
- [1.4 配置、扩展与调试](docs/stage-01-foundation/chapter-04-config-debug/README.md)
- [1.5 PHP 执行模式与 Web 架构](docs/stage-01-foundation/chapter-05-web-architecture/README.md)
- [1.6 现代 PHP 运行时与新生态](docs/stage-01-foundation/chapter-06-runtime/README.md)
- [1.7 Worker、协程、Event Loop](docs/stage-01-foundation/chapter-07-workers/README.md)

### 阶段二：PHP 基础语法·零基础完全体（语言精通）

- [阶段二总览](docs/stage-02-language/README.md)
- [2.1 PHP 基本语法结构](docs/stage-02-language/chapter-01-syntax/README.md)
  - [2.1.1 第一个 PHP 程序：Hello World](docs/stage-02-language/chapter-01-syntax/section-01-hello-world.md)
  - [2.1.2 文件结构与编码](docs/stage-02-language/chapter-01-syntax/section-02-file-structure.md)
  - [2.1.3 语句与注释](docs/stage-02-language/chapter-01-syntax/section-03-statements-comments.md)
  - [2.1.4 文件执行方式](docs/stage-02-language/chapter-01-syntax/section-04-execution.md)
  - [2.1.5 常见错误与解决方案](docs/stage-02-language/chapter-01-syntax/section-05-common-errors.md)
  - [2.1.6 实践建议与示例](docs/stage-02-language/chapter-01-syntax/section-06-best-practices.md)
- [2.2 输出与调试](docs/stage-02-language/chapter-02-output/README.md)
  - [2.2.1 输出 API](docs/stage-02-language/chapter-02-output/section-01-output-api.md)
  - [2.2.2 格式化占位符详解](docs/stage-02-language/chapter-02-output/section-02-format-placeholders.md)
  - [2.2.3 调试函数和策略](docs/stage-02-language/chapter-02-output/section-03-debugging.md)
- [2.3 变量与常量](docs/stage-02-language/chapter-03-variables/README.md)
- [2.4 数据类型](docs/stage-02-language/chapter-04-types/README.md)
- [2.5 类型转换与比较](docs/stage-02-language/chapter-05-type-casting/README.md)
- [2.6 表达式与运算符](docs/stage-02-language/chapter-06-expressions/README.md)
- [2.7 字符串操作](docs/stage-02-language/chapter-07-strings/README.md)
- [2.8 数组完整指南](docs/stage-02-language/chapter-08-arrays/README.md)
- [2.9 控制结构](docs/stage-02-language/chapter-09-control-flow/README.md)
- [2.10 函数与作用域](docs/stage-02-language/chapter-10-functions/README.md)
- [2.11 匿名函数与闭包](docs/stage-02-language/chapter-11-closures/README.md)
- [2.12 超级全局变量](docs/stage-02-language/chapter-12-superglobals/README.md)
- [2.13 文件引入与模块化](docs/stage-02-language/chapter-13-modularity/README.md)
- [2.14 文件系统操作](docs/stage-02-language/chapter-14-filesystem/README.md)
- [2.15 时间与日期处理](docs/stage-02-language/chapter-15-datetime/README.md)
- [2.16 isset / empty / Null 体系](docs/stage-02-language/chapter-16-null-system/README.md)
- [2.17 错误与异常处理](docs/stage-02-language/chapter-17-errors/README.md)
- [2.18 代码规范](docs/stage-02-language/chapter-18-standards/README.md)
- [2.19 本章小结与练习](docs/stage-02-language/chapter-19-summary/README.md)
- [2.20 PHP 版本新特性](docs/stage-02-language/chapter-20-php-versions/README.md)
- [2.21 常见错误与调试技巧](docs/stage-02-language/chapter-21-debugging/README.md)

### 阶段三：面向对象、架构与设计模式（高级工程）

- [阶段三总览](docs/stage-03-oop/README.md)
- [3.1 类、对象与基础 OOP](docs/stage-03-oop/chapter-01-classes/README.md)
- [3.2 现代 PHP 8+ OOP 能力](docs/stage-03-oop/chapter-02-modern-oop/README.md)
- [3.3 OOP 三大特性](docs/stage-03-oop/chapter-03-oop-features/README.md)
- [3.4 代码模块化与元编程](docs/stage-03-oop/chapter-04-metaprogramming/README.md)
- [3.5 命名空间与自动加载](docs/stage-03-oop/chapter-05-namespaces/README.md)
- [3.6 异常体系与框架级设计](docs/stage-03-oop/chapter-06-exceptions/README.md)
- [3.7 应用架构模式](docs/stage-03-oop/chapter-07-architecture/README.md)
- [3.8 六边形架构](docs/stage-03-oop/chapter-08-hexagonal/README.md)
- [3.9 DDD 领域驱动设计初级入门](docs/stage-03-oop/chapter-09-ddd/README.md)
- [3.10 数据流设计进阶](docs/stage-03-oop/chapter-10-dataflow/README.md)
- [3.11 阶段总结](docs/stage-03-oop/chapter-11-summary/README.md)
- [3.12 微服务架构](docs/stage-03-oop/chapter-12-microservices/README.md)

### 阶段四：Web 服务与 API-First 开发（Web Essentials）

- [阶段四总览](docs/stage-04-web/README.md)
- [4.1 Web 交互基础：从请求到响应](docs/stage-04-web/chapter-01-request-response/README.md)
- [4.2 HTML 渲染与输出控制](docs/stage-04-web/chapter-02-html-rendering/README.md)
- [4.3 超全局变量：Web 输入的核心](docs/stage-04-web/chapter-03-superglobals/README.md)
- [4.4 请求体解析：处理 JSON / API 请求](docs/stage-04-web/chapter-04-json-requests/README.md)
- [4.5 文件上传处理](docs/stage-04-web/chapter-05-file-upload/README.md)
- [4.6 RESTful API 设计与接口规范](docs/stage-04-web/chapter-06-restful-api/README.md)
- [4.7 响应处理与跨域（CORS）](docs/stage-04-web/chapter-07-response-cors/README.md)
- [4.8 会话与状态管理](docs/stage-04-web/chapter-08-session/README.md)
- [4.9 鉴权与授权模型（AuthN & AuthZ）](docs/stage-04-web/chapter-09-auth/README.md)
- [4.10 流量治理与安全](docs/stage-04-web/chapter-10-traffic-security/README.md)
- [4.11 HTTP 客户端](docs/stage-04-web/chapter-11-http-client/README.md)

### 阶段五：数据持久化与日志管理（Data Persistence & Logging）

- [阶段五总览](docs/stage-05-data/README.md)
- [5.1 数据库设计基础](docs/stage-05-data/chapter-01-database-design/README.md)
- [5.2 PDO 入门与高安全模式](docs/stage-05-data/chapter-02-pdo/README.md)
- [5.3 MySQL 事务处理](docs/stage-05-data/chapter-03-transactions/README.md)
- [5.4 现代 MySQL 高级特性](docs/stage-05-data/chapter-04-mysql-advanced/README.md)
- [5.5 并发控制与 MVCC](docs/stage-05-data/chapter-05-concurrency/README.md)
- [5.6 ORM 框架与数据迁移](docs/stage-05-data/chapter-06-orm/README.md)
- [5.7 Redis 缓存策略与性能优化](docs/stage-05-data/chapter-07-redis/README.md)
- [5.8 文件系统流式操作](docs/stage-05-data/chapter-08-filesystem/README.md)
- [5.9 日志体系与监控](docs/stage-05-data/chapter-09-logging/README.md)

### 阶段六：安全、性能与可观测性（Production Ready）

- [阶段六总览](docs/stage-06-security/README.md)
- [6.1 OWASP Top 10（2025）风险模型与防御](docs/stage-06-security/chapter-01-owasp/README.md)
- [6.2 密码与加密安全](docs/stage-06-security/chapter-02-encryption/README.md)
- [6.3 Secrets 管理](docs/stage-06-security/chapter-03-secrets/README.md)
- [6.4 安全 HTTP 头](docs/stage-06-security/chapter-04-http-headers/README.md)
- [6.5 PHP 性能优化](docs/stage-06-security/chapter-05-performance/README.md)
- [6.6 数据库深度优化](docs/stage-06-security/chapter-06-db-optimization/README.md)
- [6.7 异步与分布式处理](docs/stage-06-security/chapter-07-async/README.md)
- [6.8 测试体系建设（Testing Pyramid）](docs/stage-06-security/chapter-08-testing/README.md)
- [6.9 Mock、Stub、Fakes 深度讲解](docs/stage-06-security/chapter-09-mocking/README.md)
- [6.10 可观测性体系（Observability）](docs/stage-06-security/chapter-10-observability/README.md)

### 阶段七：现代框架深度应用（Framework Mastery）

- [阶段七总览](docs/stage-07-frameworks/README.md)
- [7.1 IoC 与 DI 设计模式深度理解](docs/stage-07-frameworks/chapter-01-ioc-di/README.md)
- [7.2 路由、中间件、Pipeline](docs/stage-07-frameworks/chapter-02-middleware/README.md)
- [7.3 Laravel（2025 最新特性）](docs/stage-07-frameworks/chapter-03-laravel/README.md)
- [7.4 Symfony](docs/stage-07-frameworks/chapter-04-symfony/README.md)

### 阶段八：部署、云原生与 DevOps（Cloud Native）

- [阶段八总览](docs/stage-08-devops/README.md)
- [8.1 专业 Dockerfile](docs/stage-08-devops/chapter-01-dockerfile/README.md)
- [8.2 docker-compose 与本地开发环境](docs/stage-08-devops/chapter-02-docker-compose/README.md)
- [8.3 镜像安全](docs/stage-08-devops/chapter-03-security/README.md)
- [8.4 部署选择](docs/stage-08-devops/chapter-04-deployment/README.md)
- [8.5 Kubernetes (K8s)](docs/stage-08-devops/chapter-05-kubernetes/README.md)
- [8.6 Serverless PHP](docs/stage-08-devops/chapter-06-serverless/README.md)
- [8.7 CI/CD 与 GitHub Actions](docs/stage-08-devops/chapter-07-cicd/README.md)
- [8.8 Deploy 策略](docs/stage-08-devops/chapter-08-strategies/README.md)
- [8.9 GitOps](docs/stage-08-devops/chapter-09-gitops/README.md)

### 阶段九：高质量实战项目（Capstone）

- [阶段九总览](docs/stage-09-projects/README.md)
- [9.1 SaaS 平台（核心项目）](docs/stage-09-projects/chapter-01-saas/README.md)
- [9.2 高并发实时应用](docs/stage-09-projects/chapter-02-realtime/README.md)
- [9.3 API-First 企业服务](docs/stage-09-projects/chapter-03-api-first/README.md)
- [9.4 生产级部署](docs/stage-09-projects/chapter-04-production/README.md)

### 阶段十：附言

- [阶段十总览](docs/stage-10-appendix/README.md)
- [10.1 PSR 标准规范](docs/stage-10-appendix/chapter-01-psr-standards/README.md)
  - [10.1.1 PSR 标准概述](docs/stage-10-appendix/chapter-01-psr-standards/section-01-introduction.md)
  - [10.1.2 PSR-1 基础编码标准](docs/stage-10-appendix/chapter-01-psr-standards/section-02-psr-1.md)
  - [10.1.3 PSR-12 扩展编码风格指南](docs/stage-10-appendix/chapter-01-psr-standards/section-03-psr-12.md)
  - [10.1.4 PSR-4 自动加载标准](docs/stage-10-appendix/chapter-01-psr-standards/section-04-psr-4.md)
  - [10.1.5 PSR-3 日志接口](docs/stage-10-appendix/chapter-01-psr-standards/section-05-psr-3.md)
  - [10.1.6 PSR-7 HTTP 消息接口](docs/stage-10-appendix/chapter-01-psr-standards/section-06-psr-7.md)
  - [10.1.7 PSR-11 容器接口](docs/stage-10-appendix/chapter-01-psr-standards/section-07-psr-11.md)
  - [10.1.8 其他 PSR 标准](docs/stage-10-appendix/chapter-01-psr-standards/section-08-other-psr.md)
  - [10.1.9 PSR 标准实施](docs/stage-10-appendix/chapter-01-psr-standards/section-09-implementation.md)

## 学习路径建议

### 初学者路径

1. **阶段一**：搭建开发环境，掌握基础工具
2. **阶段二**：系统学习 PHP 语言基础
3. **阶段三**：理解面向对象和架构设计
4. **阶段四**：学习 Web 开发和 API 设计
5. **阶段五**：掌握数据库和持久化
6. **阶段六**：了解安全和性能优化
7. **阶段七**：深入学习现代框架
8. **阶段八**：掌握部署和 DevOps
9. **阶段九**：完成实战项目
10. **阶段十**：参考附言内容（PSR 标准规范等），作为补充学习资料

### 有经验开发者路径

- 可直接跳转到感兴趣的阶段
- 建议重点学习阶段三（架构设计）、阶段六（安全与性能）、阶段七（框架应用）
- **阶段十**：建议深入学习 PSR 标准规范，提升代码质量和团队协作能力

## 文档特点

- **面向零基础**：所有内容从基础概念开始，循序渐进
- **内容详实**：提供完整的语法、参数说明和使用示例
- **示例丰富**：每个知识点都配有完整的代码示例
- **实践导向**：每章都包含练习任务，帮助巩固学习
- **结构清晰**：按阶段、章节、小节分层组织，便于查阅
- **标准规范**：附言阶段包含 PSR 标准规范等补充内容，帮助提升代码质量

## 贡献与反馈

本指南持续更新中，如有问题或建议，欢迎提出反馈。

## 版本信息

- **版本**：2.0
- **创建日期**：2025-11-28
- **适用 PHP 版本**：PHP 8.2+

# PHP + MySQL 全栈开发指南

**适用对象**：具有网页开发基础（HTML/CSS/JS/TS），但缺乏 PHP/后端经验的工程师

**目标**：从零开始掌握 PHP 8.2+、现代运行时、OOP 架构，为后续进阶到 Laravel / 企业级开发奠定基础

---

## 目录

### 🚀 阶段一：环境、运行时与工具链（基础设施）

1. [PHP 安装与运行基础](docs/stage-01-foundation/chapter-01-runtime/README.md)
2. [MySQL 环境搭建与工具](docs/stage-01-foundation/chapter-02-mysql/README.md)
3. [核心工具链（现代 PHP 必备）](docs/stage-01-foundation/chapter-03-toolchain/README.md)
4. [配置、扩展与调试](docs/stage-01-foundation/chapter-04-config-debug/README.md)
5. [PHP 执行模式与 Web 架构](docs/stage-01-foundation/chapter-05-web-architecture/README.md)
6. [现代 PHP 运行时与新生态](docs/stage-01-foundation/chapter-06-runtime/README.md)
7. [Worker、协程、Event Loop](docs/stage-01-foundation/chapter-07-workers/README.md)

[阶段一总览](docs/stage-01-foundation/README.md)

---

### 📘 阶段二：PHP 基础语法·零基础完全体（语言精通）

1. [PHP 基本语法结构](docs/stage-02-language/chapter-01-syntax/README.md)
2. [输出与调试](docs/stage-02-language/chapter-02-output/README.md)
3. [变量与常量](docs/stage-02-language/chapter-03-variables/README.md)
4. [数据类型](docs/stage-02-language/chapter-04-types/README.md)
5. [类型转换与比较](docs/stage-02-language/chapter-05-type-casting/README.md)
6. [表达式与运算符](docs/stage-02-language/chapter-06-expressions/README.md)
7. [字符串操作](docs/stage-02-language/chapter-07-strings/README.md)
8. [数组完整指南](docs/stage-02-language/chapter-08-arrays/README.md)
9. [控制结构](docs/stage-02-language/chapter-09-control-flow/README.md)
10. [函数与作用域](docs/stage-02-language/chapter-10-functions/README.md)
11. [匿名函数与闭包](docs/stage-02-language/chapter-11-closures/README.md)
12. [超级全局变量](docs/stage-02-language/chapter-12-superglobals/README.md)
13. [文件引入与模块化](docs/stage-02-language/chapter-13-modularity/README.md)
14. [文件系统与 I/O](docs/stage-02-language/chapter-14-filesystem/README.md)
15. [时间日期 API 与国际化](docs/stage-02-language/chapter-15-datetime/README.md)
16. [isset、empty 与 Null 合并运算](docs/stage-02-language/chapter-16-null-system/README.md)
17. [错误与异常处理](docs/stage-02-language/chapter-17-errors/README.md)
18. [代码规范、PHPDoc 与 PSR](docs/stage-02-language/chapter-18-standards/README.md)
19. [阶段总结与练习指引](docs/stage-02-language/chapter-19-summary/README.md)
20. [PHP 8.2-8.5 版本新特性](docs/stage-02-language/chapter-20-php-versions/README.md)
21. [常见错误与调试技巧](docs/stage-02-language/chapter-21-debugging/README.md)

[阶段二总览](docs/stage-02-language/README.md)

---

### 🏗️ 阶段三：面向对象、架构与设计模式

1. [类、对象与基础 OOP 概念](docs/stage-03-oop/chapter-01-classes/README.md)
2. [现代 PHP 8+ OOP 能力](docs/stage-03-oop/chapter-02-modern-oop/README.md)
3. [OOP 三大特性（继承、接口、抽象类）](docs/stage-03-oop/chapter-03-oop-features/README.md)
4. [代码模块化与元编程（Traits、Attributes）](docs/stage-03-oop/chapter-04-metaprogramming/README.md)
5. [命名空间与自动加载（PSR-4）](docs/stage-03-oop/chapter-05-namespaces/README.md)
6. [异常体系与框架级设计](docs/stage-03-oop/chapter-06-exceptions/README.md)
7. [应用架构模式（MVC、ADR）](docs/stage-03-oop/chapter-07-architecture/README.md)
8. [六边形架构（Ports & Adapters）](docs/stage-03-oop/chapter-08-hexagonal/README.md)
9. [DDD 领域驱动设计初级入门](docs/stage-03-oop/chapter-09-ddd/README.md)
10. [数据流设计进阶（CQRS、事件溯源）](docs/stage-03-oop/chapter-10-dataflow/README.md)
11. [微服务架构](docs/stage-03-oop/chapter-12-microservices/README.md)
12. [阶段总结与练习指引](docs/stage-03-oop/chapter-11-summary/README.md)

[阶段三总览](docs/stage-03-oop/README.md)

---

### 🌐 阶段四：Web 服务与 API-First 开发

1. [Web 交互基础：从请求到响应](docs/stage-04-web/chapter-01-request-response/README.md)
2. [HTML 渲染与输出控制](docs/stage-04-web/chapter-02-html-rendering/README.md)
3. [超全局变量：Web 输入的核心](docs/stage-04-web/chapter-03-superglobals/README.md)
4. [请求体解析：处理 JSON / API 请求](docs/stage-04-web/chapter-04-json-requests/README.md)
5. [文件上传处理](docs/stage-04-web/chapter-05-file-upload/README.md)
6. [RESTful API 设计与接口规范](docs/stage-04-web/chapter-06-restful-api/README.md)
7. [响应处理与跨域（CORS）](docs/stage-04-web/chapter-07-response-cors/README.md)
8. [会话与状态管理](docs/stage-04-web/chapter-08-session/README.md)
9. [鉴权与授权模型（AuthN & AuthZ）](docs/stage-04-web/chapter-09-auth/README.md)
10. [流量治理与安全](docs/stage-04-web/chapter-10-traffic-security/README.md)
11. [HTTP 客户端：Guzzle 详细使用](docs/stage-04-web/chapter-11-http-client/README.md)

[阶段四总览](docs/stage-04-web/README.md)

---

### 💾 阶段五：数据持久化与日志管理

1. [数据库设计基础](docs/stage-05-data/chapter-01-database-design/README.md)
2. [PDO 入门与高安全模式](docs/stage-05-data/chapter-02-pdo/README.md)
3. [MySQL 事务处理](docs/stage-05-data/chapter-03-transactions/README.md)
4. [现代 MySQL 高级特性](docs/stage-05-data/chapter-04-mysql-advanced/README.md)
5. [并发控制与 MVCC](docs/stage-05-data/chapter-05-concurrency/README.md)
6. [ORM 框架与数据迁移](docs/stage-05-data/chapter-06-orm/README.md)
7. [Redis 缓存策略与性能优化](docs/stage-05-data/chapter-07-redis/README.md)
8. [文件系统流式操作](docs/stage-05-data/chapter-08-filesystem/README.md)
9. [日志体系与监控](docs/stage-05-data/chapter-09-logging/README.md)

[阶段五总览](docs/stage-05-data/README.md)

---

### 🔒 阶段六：安全、性能与可观测性

1. [OWASP Top 10（2025）风险模型与防御](docs/stage-06-security/chapter-01-owasp/README.md)
2. [密码与加密安全](docs/stage-06-security/chapter-02-encryption/README.md)
3. [Secrets 管理](docs/stage-06-security/chapter-03-secrets/README.md)
4. [安全 HTTP 头](docs/stage-06-security/chapter-04-http-headers/README.md)
5. [PHP 性能优化](docs/stage-06-security/chapter-05-performance/README.md)
6. [数据库深度优化](docs/stage-06-security/chapter-06-db-optimization/README.md)
7. [异步与分布式处理](docs/stage-06-security/chapter-07-async/README.md)
8. [测试体系建设](docs/stage-06-security/chapter-08-testing/README.md)
9. [Mock、Stub、Fakes 深度讲解](docs/stage-06-security/chapter-09-mocking/README.md)
10. [可观测性体系](docs/stage-06-security/chapter-10-observability/README.md)

[阶段六总览](docs/stage-06-security/README.md)

---

### 🎯 阶段七：现代框架深度应用

1. [IoC 与 DI 设计模式深度理解](docs/stage-07-frameworks/chapter-01-ioc-di/README.md)
2. [路由、中间件、Pipeline](docs/stage-07-frameworks/chapter-02-middleware/README.md)
3. [Laravel（2025 最新特性）](docs/stage-07-frameworks/chapter-03-laravel/README.md)
4. [Symfony 深度应用](docs/stage-07-frameworks/chapter-04-symfony/README.md)

[阶段七总览](docs/stage-07-frameworks/README.md)

---

### 🚢 阶段八：部署、云原生与 DevOps

1. [专业 Dockerfile](docs/stage-08-devops/chapter-01-dockerfile/README.md)
2. [docker-compose 与本地开发环境](docs/stage-08-devops/chapter-02-docker-compose/README.md)
3. [镜像安全](docs/stage-08-devops/chapter-03-security/README.md)
4. [部署选择](docs/stage-08-devops/chapter-04-deployment/README.md)
5. [Kubernetes (K8s)](docs/stage-08-devops/chapter-05-kubernetes/README.md)
6. [Serverless PHP](docs/stage-08-devops/chapter-06-serverless/README.md)
7. [CI/CD 与 GitHub Actions](docs/stage-08-devops/chapter-07-cicd/README.md)
8. [Deploy 策略](docs/stage-08-devops/chapter-08-strategies/README.md)
9. [GitOps](docs/stage-08-devops/chapter-09-gitops/README.md)

[阶段八总览](docs/stage-08-devops/README.md)

---

### 🎓 阶段九：高质量实战项目

1. [SaaS 平台（核心项目）](docs/stage-09-projects/chapter-01-saas/README.md)
2. [高并发实时应用](docs/stage-09-projects/chapter-02-realtime/README.md)
3. [API-First 企业服务](docs/stage-09-projects/chapter-03-api-first/README.md)
4. [生产级部署](docs/stage-09-projects/chapter-04-production/README.md)

[阶段九总览](docs/stage-09-projects/README.md)

---

## 学习路径

本教材按照从基础到高级的顺序组织，建议按阶段顺序学习：

1. **阶段一**：搭建开发环境，熟悉工具链
2. **阶段二**：掌握 PHP 语言基础
3. **阶段三**：学习面向对象和架构设计
4. **阶段四**：掌握 Web 开发和 API 设计
5. **阶段五**：学习数据持久化和缓存
6. **阶段六**：掌握安全、性能和可观测性
7. **阶段七**：深入学习现代框架
8. **阶段八**：掌握部署和 DevOps
9. **阶段九**：通过实战项目巩固知识

## 技术栈

- **PHP**：8.2-8.5（最新版本）
- **MySQL**：8.0+
- **框架**：Laravel 11、Symfony 7.0
- **工具**：Composer、PHPUnit 11、Pest 3、PHPStan、Psalm
- **运行时**：RoadRunner、FrankenPHP、OpenSwoole、Octane
- **DevOps**：Docker、Kubernetes、GitHub Actions、CI/CD

## 特色

- ✅ **面向零基础**：适合有前端/Node.js/TypeScript 背景的初学者
- ✅ **对比说明**：关键章节提供与 JavaScript/TypeScript/Node.js 的对比
- ✅ **详细完整**：每个概念都有详细说明和完整示例
- ✅ **最新技术**：覆盖 PHP 8.2-8.5、Laravel 11、Symfony 7.0 等最新特性
- ✅ **实战项目**：提供完整的实战项目示例
- ✅ **生产级**：包含生产环境部署和运维实践

## 快速开始

1. 阅读[阶段一：环境、运行时与工具链](docs/stage-01-foundation/README.md)
2. 按照章节顺序学习，每完成一章进行练习
3. 完成所有阶段后，通过[阶段九：高质量实战项目](docs/stage-09-projects/README.md)巩固知识

---

**最后更新**：2025年11月28日

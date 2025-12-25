# 阶段八：性能优化与安全（Performance & Security）

本阶段深入讲解 PHP 应用的性能优化、安全防护和测试体系建设。从安全防护开始，逐步掌握性能优化、现代 PHP 运行时、Worker 模式、异步处理、测试体系、TDD/BDD、代码覆盖率、可观测性等。每章提供完整示例、最佳实践与常见陷阱说明，帮助你构建高性能、安全、可测试的应用。

## 定位

性能优化、安全防护、测试体系

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握（阶段一、阶段二、阶段三、阶段五、阶段六、阶段七）

- **阶段一**：环境搭建、工具配置
- **阶段二**：PHP 语言基础
- **阶段三**：面向对象编程
- **阶段五**：Web 开发基础
- **阶段六**：数据库操作、缓存系统
- **阶段七**：架构设计、设计模式

### 不需要掌握

以下知识**不需要**提前掌握，会在学习过程中逐步学习：

- 性能优化（从零开始学习）
- 安全防护（从零开始学习）
- 测试体系（从零开始学习）
- 可观测性（从零开始学习）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **基础检查**
   - [ ] 能够开发完整的 Web 应用
   - [ ] 理解架构设计和设计模式
   - [ ] 能够使用数据库和缓存

2. **理解检查**
   - [ ] 理解 Web 应用的基本架构
   - [ ] 能够阅读和理解复杂的 PHP 代码
   - [ ] 理解性能和安全的基本概念

3. **实践检查**
   - [ ] 完成阶段七的所有练习
   - [ ] 能够独立开发中等复杂度的应用

**如果以上检查点都通过，可以开始阶段八的学习。**

**如果某些检查点未通过，建议：**
- 复习阶段七的相关章节
- 完成阶段七的练习
- 确保理解架构设计后再继续

## 学习时间估算

**建议学习时间：** 8-10 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含练习和实践时间

**时间分配建议：**
- 安全防护（8.1）：2 周
- 性能优化（8.2）：2 周
- 现代 PHP 运行时（8.3）：1-2 周
- Worker、协程、Event Loop（8.4）：1-2 周
- 异步与分布式处理（8.5）：1 周
- 测试体系建设（8.6-8.10）：2-3 周
- 可观测性体系（8.11）：1 周
- 阶段总结与实践：1 周

## 练习建议

### 基础练习（每章完成后）

每完成一章，建议完成 3-5 个小练习，例如：

1. **安全防护**
   - 实现 OWASP Top 10 防护
   - 实现密码加密和安全存储
   - 实现 API 安全机制

2. **性能优化**
   - 优化 PHP 代码性能
   - 配置 OPcache
   - 优化数据库查询

3. **测试体系**
   - 编写单元测试
   - 实现 TDD 开发流程
   - 实现代码覆盖率报告

### 综合练习（阶段完成后）

4. **综合项目**
   - 优化现有应用的性能
   - 实现安全防护机制
   - 建立完整的测试体系
   - 实现可观测性监控

**项目要求：**
- 使用本阶段学到的所有知识点
- 代码规范、注释清晰
- 完善的测试覆盖
- 性能和安全最佳实践

## 章节内容

1. **[8.1 安全防护](chapter-01-security/readme.md)**：OWASP Top 10（2025）风险模型与防御、密码与加密安全、Secrets 管理、安全 HTTP 头、API 安全、安全审计与日志、依赖安全。
   - [8.1.1 OWASP Top 10（2025）风险模型与防御](chapter-01-security/section-01-owasp.md)
   - [8.1.2 密码与加密安全](chapter-01-security/section-02-encryption.md)
   - [8.1.3 Secrets 管理](chapter-01-security/section-03-secrets.md)
   - [8.1.4 安全 HTTP 头](chapter-01-security/section-04-http-headers.md)
   - [8.1.5 API 安全](chapter-01-security/section-05-api-security.md)
   - [8.1.6 安全审计与日志](chapter-01-security/section-06-audit.md)
   - [8.1.7 依赖安全](chapter-01-security/section-07-dependency-security.md)

2. **[8.2 性能优化](chapter-02-performance/readme.md)**：PHP 性能优化、OPcache 配置、代码优化技巧、数据库深度优化、性能监控、性能测试。
   - [8.2.1 PHP 性能优化](chapter-02-performance/section-01-php-optimization.md)
   - [8.2.2 OPcache 配置](chapter-02-performance/section-02-opcache.md)
   - [8.2.3 代码优化技巧](chapter-02-performance/section-03-code-optimization.md)
   - [8.2.4 数据库深度优化](chapter-02-performance/section-04-db-optimization.md)
   - [8.2.5 性能监控](chapter-02-performance/section-05-monitoring.md)
   - [8.2.6 性能测试](chapter-02-performance/section-06-performance-testing.md)

3. **[8.3 现代 PHP 运行时](chapter-03-runtime/readme.md)**：RoadRunner、FrankenPHP、OpenSwoole、Laravel Octane、运行时对比与选择。
   - [8.3.1 RoadRunner](chapter-03-runtime/section-01-roadrunner.md)
   - [8.3.2 FrankenPHP](chapter-03-runtime/section-02-frankenphp.md)
   - [8.3.3 OpenSwoole](chapter-03-runtime/section-03-openswoole.md)
   - [8.3.4 Laravel Octane](chapter-03-runtime/section-04-laravel-octane.md)
   - [8.3.5 运行时对比与选择](chapter-03-runtime/section-05-comparison.md)

4. **[8.4 Worker、协程、Event Loop](chapter-04-workers/readme.md)**：Worker 模式、协程（Coroutine）、Event Loop（事件循环）、PHP 8 Fibers、实际应用与最佳实践。
   - [8.4.1 Worker 模式](chapter-04-workers/section-01-worker-mode.md)
   - [8.4.2 协程（Coroutine）](chapter-04-workers/section-02-coroutines.md)
   - [8.4.3 Event Loop（事件循环）](chapter-04-workers/section-03-event-loop.md)
   - [8.4.4 PHP 8 Fibers](chapter-04-workers/section-04-fibers.md)
   - [8.4.5 实际应用与最佳实践](chapter-04-workers/section-05-best-practices.md)

5. **[8.5 异步与分布式处理](chapter-05-async/readme.md)**：异步处理基础、消息队列、分布式处理。
   - [8.5.1 异步处理基础](chapter-05-async/section-01-async-basics.md)
   - [8.5.2 消息队列](chapter-05-async/section-02-message-queue.md)
   - [8.5.3 分布式处理](chapter-05-async/section-03-distributed.md)

6. **[8.6 测试体系建设](chapter-06-testing/readme.md)**：测试金字塔、单元测试、集成测试、E2E 测试。
   - [8.6.1 测试金字塔](chapter-06-testing/section-01-testing-pyramid.md)
   - [8.6.2 单元测试](chapter-06-testing/section-02-unit-testing.md)
   - [8.6.3 集成测试](chapter-06-testing/section-03-integration-testing.md)
   - [8.6.4 E2E 测试](chapter-06-testing/section-04-e2e-testing.md)

7. **[8.7 Mock、Stub、Fakes 深度讲解](chapter-07-mocking/readme.md)**：Mock 基础、Stub 与 Fakes、测试替身实践。
   - [8.7.1 Mock 基础](chapter-07-mocking/section-01-mock-basics.md)
   - [8.7.2 Stub 与 Fakes](chapter-07-mocking/section-02-stub-fakes.md)
   - [8.7.3 测试替身实践](chapter-07-mocking/section-03-test-doubles.md)

8. **[8.8 TDD（测试驱动开发）](chapter-08-tdd/readme.md)**：TDD 概述、TDD 工作流程、TDD 实践。
   - [8.8.1 TDD 概述](chapter-08-tdd/section-01-overview.md)
   - [8.8.2 TDD 工作流程](chapter-08-tdd/section-02-workflow.md)
   - [8.8.3 TDD 实践](chapter-08-tdd/section-03-practice.md)

9. **[8.9 BDD（行为驱动开发）](chapter-09-bdd/readme.md)**：BDD 概述、BDD 工具（Behat、Codeception）、BDD 实践。
   - [8.9.1 BDD 概述](chapter-09-bdd/section-01-overview.md)
   - [8.9.2 BDD 工具（Behat、Codeception）](chapter-09-bdd/section-02-tools.md)
   - [8.9.3 BDD 实践](chapter-09-bdd/section-03-practice.md)

10. **[8.10 代码覆盖率深入](chapter-10-coverage/readme.md)**：代码覆盖率概述、PHPUnit 覆盖率、Xdebug 覆盖率、覆盖率报告与分析。
    - [8.10.1 代码覆盖率概述](chapter-10-coverage/section-01-overview.md)
    - [8.10.2 PHPUnit 覆盖率](chapter-10-coverage/section-02-phpunit.md)
    - [8.10.3 Xdebug 覆盖率](chapter-10-coverage/section-03-xdebug.md)
    - [8.10.4 覆盖率报告与分析](chapter-10-coverage/section-04-reports.md)

11. **[8.11 可观测性体系](chapter-11-observability/readme.md)**：可观测性概述、日志、指标、追踪、监控与告警。
    - [8.11.1 可观测性概述](chapter-11-observability/section-01-overview.md)
    - [8.11.2 日志、指标、追踪](chapter-11-observability/section-02-logs-metrics-traces.md)
    - [8.11.3 监控与告警](chapter-11-observability/section-03-monitoring-alerts.md)

## 完成本阶段后，你将具备：

- 扎实的安全防护能力，能够防范常见安全漏洞
- 掌握性能优化方法，能够优化应用性能
- 理解现代 PHP 运行时，能够选择合适的运行时
- 掌握 Worker 模式和协程，能够实现高性能应用
- 建立完整的测试体系，能够编写高质量的测试
- 掌握 TDD/BDD 开发方法，能够进行测试驱动开发
- 建立可观测性体系，能够监控和诊断应用问题

## 相关章节

- **阶段四：系统编程**：错误处理、调试、日志
- **阶段五：Web/API 开发**：Web 开发中的应用
- **阶段六：数据库与缓存系统**：数据库优化
- **阶段七：高级架构与设计模式**：架构设计
- **阶段九：现代框架深度应用**：框架性能优化
- **阶段十：部署、云原生与 DevOps**：部署和监控

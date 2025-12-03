# PHP + MySQL 全栈开发指南

**适用对象**：具有网页开发基础（HTML/CSS/JS/TS），但缺乏 PHP/后端经验的工程师

**目标**：从零开始掌握 PHP 8.2+、现代运行时、OOP 架构，为后续进阶到 Laravel / 企业级开发奠定基础

## 文档说明

本指南采用分阶段、分章节的结构，每个阶段和章节都是独立的文档文件，便于学习和查阅。所有内容面向零基础学员设计，提供详细的概念解释、语法说明、参数列表、完整示例代码和练习任务。

## 目录结构

### 阶段一：环境、运行时与工具链（基础设施）

- [阶段一总览](docs/stage-01-foundation/README.md)
- [1.1 PHP 安装与运行基础](docs/stage-01-foundation/chapter-01-runtime/README.md)
  - [1.1.1 PHP 安装与版本管理](docs/stage-01-foundation/chapter-01-runtime/section-01-installation.md)
  - [1.1.2 运行模式与验证](docs/stage-01-foundation/chapter-01-runtime/section-02-execution-modes.md)
- [1.2 MySQL 环境搭建与工具](docs/stage-01-foundation/chapter-02-mysql/README.md)
  - [1.2.1 MySQL 安装与配置](docs/stage-01-foundation/chapter-02-mysql/section-01-installation.md)
  - [1.2.2 客户端工具与命令](docs/stage-01-foundation/chapter-02-mysql/section-02-tools-commands.md)
- [1.3 核心工具链](docs/stage-01-foundation/chapter-03-toolchain/README.md)
  - [1.3.1 Composer 依赖管理](docs/stage-01-foundation/chapter-03-toolchain/section-01-composer.md)
  - [1.3.2 IDE 与扩展配置](docs/stage-01-foundation/chapter-03-toolchain/section-02-ide-extensions.md)
  - [1.3.3 Git 与任务管理](docs/stage-01-foundation/chapter-03-toolchain/section-03-git-tasks.md)
  - [1.3.4 容器与环境管理](docs/stage-01-foundation/chapter-03-toolchain/section-04-docker-compose.md)
  - [1.3.5 自动化脚本](docs/stage-01-foundation/chapter-03-toolchain/section-05-automation.md)
- [1.4 配置、扩展与调试](docs/stage-01-foundation/chapter-04-config-debug/README.md)
  - [1.4.1 php.ini 配置](docs/stage-01-foundation/chapter-04-config-debug/section-01-php-ini.md)
  - [1.4.2 扩展管理](docs/stage-01-foundation/chapter-04-config-debug/section-02-extensions.md)
  - [1.4.3 Xdebug 调试配置](docs/stage-01-foundation/chapter-04-config-debug/section-03-xdebug.md)
  - [1.4.4 调试技巧与实践](docs/stage-01-foundation/chapter-04-config-debug/section-04-debugging-tips.md)
- [1.5 PHP 执行模式与 Web 架构](docs/stage-01-foundation/chapter-05-web-architecture/README.md)
  - [1.5.1 PHP 执行模式](docs/stage-01-foundation/chapter-05-web-architecture/section-01-execution-modes.md)
  - [1.5.2 Nginx + PHP-FPM 架构](docs/stage-01-foundation/chapter-05-web-architecture/section-02-nginx-fpm.md)
  - [1.5.3 Shared Nothing 架构](docs/stage-01-foundation/chapter-05-web-architecture/section-03-shared-nothing.md)
  - [1.5.4 配置与优化实践](docs/stage-01-foundation/chapter-05-web-architecture/section-04-optimization.md)
- [1.6 现代 PHP 运行时与新生态](docs/stage-01-foundation/chapter-06-runtime/README.md)
  - [1.6.1 RoadRunner](docs/stage-01-foundation/chapter-06-runtime/section-01-roadrunner.md)
  - [1.6.2 FrankenPHP](docs/stage-01-foundation/chapter-06-runtime/section-02-frankenphp.md)
  - [1.6.3 OpenSwoole](docs/stage-01-foundation/chapter-06-runtime/section-03-openswoole.md)
  - [1.6.4 Laravel Octane](docs/stage-01-foundation/chapter-06-runtime/section-04-laravel-octane.md)
  - [1.6.5 运行时对比与选择](docs/stage-01-foundation/chapter-06-runtime/section-05-comparison.md)
- [1.7 Worker、协程、Event Loop](docs/stage-01-foundation/chapter-07-workers/README.md)
  - [1.7.1 Worker 模式](docs/stage-01-foundation/chapter-07-workers/section-01-worker-mode.md)
  - [1.7.2 协程（Coroutine）](docs/stage-01-foundation/chapter-07-workers/section-02-coroutines.md)
  - [1.7.3 Event Loop（事件循环）](docs/stage-01-foundation/chapter-07-workers/section-03-event-loop.md)
  - [1.7.4 PHP 8 Fibers](docs/stage-01-foundation/chapter-07-workers/section-04-fibers.md)
  - [1.7.5 实际应用与最佳实践](docs/stage-01-foundation/chapter-07-workers/section-05-best-practices.md)

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
  - [2.3.1 变量基础](docs/stage-02-language/chapter-03-variables/section-01-variable-basics.md)
  - [2.3.2 引用与可变变量](docs/stage-02-language/chapter-03-variables/section-02-references.md)
  - [2.3.3 常量](docs/stage-02-language/chapter-03-variables/section-03-constants.md)
  - [2.3.4 全局与静态](docs/stage-02-language/chapter-03-variables/section-04-scope-static.md)
  - [2.3.5 最佳实践与综合练习](docs/stage-02-language/chapter-03-variables/section-05-best-practices.md)
- [2.4 数据类型](docs/stage-02-language/chapter-04-types/README.md)
  - [2.4.1 标量类型](docs/stage-02-language/chapter-04-types/section-01-scalar-types.md)
  - [2.4.2 复合类型](docs/stage-02-language/chapter-04-types/section-02-composite-types.md)
  - [2.4.3 特殊类型](docs/stage-02-language/chapter-04-types/section-03-special-types.md)
  - [2.4.4 类型检测](docs/stage-02-language/chapter-04-types/section-04-type-detection.md)
  - [2.4.5 类型声明与联合类型](docs/stage-02-language/chapter-04-types/section-05-type-declarations.md)
  - [2.4.6 JSON 与数组互转](docs/stage-02-language/chapter-04-types/section-06-json.md)
- [2.5 类型转换与比较](docs/stage-02-language/chapter-05-type-casting/README.md)
  - [2.5.1 隐式转换](docs/stage-02-language/chapter-05-type-casting/section-01-implicit-conversion.md)
  - [2.5.2 显式转换](docs/stage-02-language/chapter-05-type-casting/section-02-explicit-conversion.md)
  - [2.5.3 转换函数](docs/stage-02-language/chapter-05-type-casting/section-03-conversion-functions.md)
  - [2.5.4 比较运算符与函数](docs/stage-02-language/chapter-05-type-casting/section-04-comparison.md)
- [2.6 表达式与运算符](docs/stage-02-language/chapter-06-expressions/README.md)
  - [2.6.1 算术运算符](docs/stage-02-language/chapter-06-expressions/section-01-arithmetic-operators.md)
  - [2.6.2 赋值运算符](docs/stage-02-language/chapter-06-expressions/section-02-assignment-operators.md)
  - [2.6.3 逻辑运算符](docs/stage-02-language/chapter-06-expressions/section-03-logical-operators.md)
  - [2.6.4 三元运算符与空合并运算符](docs/stage-02-language/chapter-06-expressions/section-04-ternary-null-coalescing.md)
  - [2.6.5 match 表达式](docs/stage-02-language/chapter-06-expressions/section-05-match-expression.md)
  - [2.6.6 运算符优先级与结合性](docs/stage-02-language/chapter-06-expressions/section-06-operator-precedence.md)
- [2.7 字符串操作](docs/stage-02-language/chapter-07-strings/README.md)
  - [2.7.1 字符串创建方式](docs/stage-02-language/chapter-07-strings/section-01-string-creation.md)
  - [2.7.2 常用字符串函数](docs/stage-02-language/chapter-07-strings/section-02-string-functions.md)
- [2.8 数组完整指南](docs/stage-02-language/chapter-08-arrays/README.md)
  - [2.8.1 数组基础](docs/stage-02-language/chapter-08-arrays/section-01-array-basics.md)
  - [2.8.2 数组遍历](docs/stage-02-language/chapter-08-arrays/section-02-array-iteration.md)
  - [2.8.3 数组操作函数](docs/stage-02-language/chapter-08-arrays/section-03-array-functions.md)
  - [2.8.4 数组排序](docs/stage-02-language/chapter-08-arrays/section-04-array-sorting.md)
  - [2.8.5 数组解构](docs/stage-02-language/chapter-08-arrays/section-05-array-destructuring.md)
- [2.9 控制结构](docs/stage-02-language/chapter-09-control-flow/README.md)
  - [2.9.1 条件语句](docs/stage-02-language/chapter-09-control-flow/section-01-conditional-statements.md)
  - [2.9.2 循环结构](docs/stage-02-language/chapter-09-control-flow/section-02-loops.md)
  - [2.9.3 跳转语句](docs/stage-02-language/chapter-09-control-flow/section-03-jump-statements.md)
- [2.10 函数与作用域](docs/stage-02-language/chapter-10-functions/README.md)
  - [2.10.1 函数基础](docs/stage-02-language/chapter-10-functions/section-01-function-basics.md)
  - [2.10.2 可变参数](docs/stage-02-language/chapter-10-functions/section-02-variable-arguments.md)
  - [2.10.3 引用、箭头函数与闭包](docs/stage-02-language/chapter-10-functions/section-03-references-closures.md)
  - [2.10.4 可调用类型与内置函数](docs/stage-02-language/chapter-10-functions/section-04-callable-types.md)
- [2.11 匿名函数与闭包](docs/stage-02-language/chapter-11-closures/README.md)
  - [2.11.1 匿名函数基础](docs/stage-02-language/chapter-11-closures/section-01-anonymous-functions.md)
  - [2.11.2 闭包与作用域](docs/stage-02-language/chapter-11-closures/section-02-closures-scope.md)
- [2.12 超级全局变量](docs/stage-02-language/chapter-12-superglobals/README.md)
  - [2.12.1 $_GET、$_POST 与 $_REQUEST](docs/stage-02-language/chapter-12-superglobals/section-01-get-post-request.md)
  - [2.12.2 $_SERVER、$_SESSION 与 $_COOKIE](docs/stage-02-language/chapter-12-superglobals/section-02-server-session-cookie.md)
  - [2.12.3 $_FILES 与安全处理](docs/stage-02-language/chapter-12-superglobals/section-03-files-security.md)
- [2.13 文件引入与模块化](docs/stage-02-language/chapter-13-modularity/README.md)
  - [2.13.1 include 与 require](docs/stage-02-language/chapter-13-modularity/section-01-include-require.md)
  - [2.13.2 Composer Autoload](docs/stage-02-language/chapter-13-modularity/section-02-composer-autoload.md)
- [2.14 文件系统操作](docs/stage-02-language/chapter-14-filesystem/README.md)
  - [2.14.1 文件操作](docs/stage-02-language/chapter-14-filesystem/section-01-file-operations.md)
  - [2.14.2 目录操作](docs/stage-02-language/chapter-14-filesystem/section-02-directory-operations.md)
- [2.15 时间与日期处理](docs/stage-02-language/chapter-15-datetime/README.md)
  - [2.15.1 DateTime 基础](docs/stage-02-language/chapter-15-datetime/section-01-datetime-basics.md)
- [2.16 isset / empty / Null 体系](docs/stage-02-language/chapter-16-null-system/README.md)
  - [2.16.1 isset、empty 与 is_null](docs/stage-02-language/chapter-16-null-system/section-01-isset-empty.md)
  - [2.16.2 空合并运算符](docs/stage-02-language/chapter-16-null-system/section-02-null-coalescing.md)
- [2.17 错误与异常处理](docs/stage-02-language/chapter-17-errors/README.md)
  - [2.17.1 错误处理](docs/stage-02-language/chapter-17-errors/section-01-error-handling.md)
  - [2.17.2 异常处理](docs/stage-02-language/chapter-17-errors/section-02-exceptions.md)
- [2.18 代码规范](docs/stage-02-language/chapter-18-standards/README.md)
  - [2.18.1 PSR 标准](docs/stage-02-language/chapter-18-standards/section-01-psr-standards.md)
  - [2.18.2 PHPDoc](docs/stage-02-language/chapter-18-standards/section-02-phpdoc.md)
- [2.19 常见错误与调试技巧](docs/stage-02-language/chapter-21-debugging/README.md)
- [2.20 PHP 版本新特性](docs/stage-02-language/chapter-20-php-versions/README.md)
- [2.21 本章小结与练习](docs/stage-02-language/chapter-19-summary/README.md)

### 阶段三：面向对象、架构与设计模式（高级工程）

- [阶段三总览](docs/stage-03-oop/README.md)
- [3.1 类、对象与基础 OOP](docs/stage-03-oop/chapter-01-classes/README.md)
  - [3.1.1 类与对象基础](docs/stage-03-oop/chapter-01-classes/section-01-basics.md)
  - [3.1.2 可见性修饰符](docs/stage-03-oop/chapter-01-classes/section-02-visibility.md)
  - [3.1.3 构造函数与析构函数](docs/stage-03-oop/chapter-01-classes/section-03-constructors-destructors.md)
  - [3.1.4 对象克隆与引用](docs/stage-03-oop/chapter-01-classes/section-04-clone-references.md)
  - [3.1.5 静态属性与方法、魔术方法](docs/stage-03-oop/chapter-01-classes/section-05-static-magic.md)
- [3.2 现代 PHP 8+ OOP 能力](docs/stage-03-oop/chapter-02-modern-oop/README.md)
  - [3.2.1 构造器属性提升](docs/stage-03-oop/chapter-02-modern-oop/section-01-constructor-promotion.md)
  - [3.2.2 Readonly 属性](docs/stage-03-oop/chapter-02-modern-oop/section-02-readonly.md)
  - [3.2.3 枚举（Enums）](docs/stage-03-oop/chapter-02-modern-oop/section-03-enums.md)
  - [3.2.4 属性钩子与静态方法](docs/stage-03-oop/chapter-02-modern-oop/section-04-property-hooks.md)
- [3.3 OOP 三大特性](docs/stage-03-oop/chapter-03-oop-features/README.md)
  - [3.3.1 继承（Inheritance）](docs/stage-03-oop/chapter-03-oop-features/section-01-inheritance.md)
  - [3.3.2 接口（Interface）](docs/stage-03-oop/chapter-03-oop-features/section-02-interfaces.md)
  - [3.3.3 抽象类（Abstract Class）](docs/stage-03-oop/chapter-03-oop-features/section-03-abstract-classes.md)
  - [3.3.4 多态（Polymorphism）](docs/stage-03-oop/chapter-03-oop-features/section-04-polymorphism.md)
- [3.4 代码模块化与元编程](docs/stage-03-oop/chapter-04-metaprogramming/README.md)
  - [3.4.1 Traits](docs/stage-03-oop/chapter-04-metaprogramming/section-01-traits.md)
  - [3.4.2 Attributes（注解）](docs/stage-03-oop/chapter-04-metaprogramming/section-02-attributes.md)
- [3.5 命名空间与自动加载](docs/stage-03-oop/chapter-05-namespaces/README.md)
  - [3.5.1 命名空间基础](docs/stage-03-oop/chapter-05-namespaces/section-01-basics.md)
  - [3.5.2 use 语句](docs/stage-03-oop/chapter-05-namespaces/section-02-use-statements.md)
  - [3.5.3 自动加载基础](docs/stage-03-oop/chapter-05-namespaces/section-03-autoloading-basics.md)
  - [3.5.4 PSR-4 自动加载标准](docs/stage-03-oop/chapter-05-namespaces/section-04-psr4.md)
- [3.6 异常体系与框架级设计](docs/stage-03-oop/chapter-06-exceptions/README.md)
  - [3.6.1 异常体系层次](docs/stage-03-oop/chapter-06-exceptions/section-01-exception-hierarchy.md)
  - [3.6.2 自定义异常](docs/stage-03-oop/chapter-06-exceptions/section-02-custom-exceptions.md)
  - [3.6.3 框架级异常设计](docs/stage-03-oop/chapter-06-exceptions/section-03-framework-design.md)
- [3.7 应用架构模式](docs/stage-03-oop/chapter-07-architecture/README.md)
  - [3.7.1 MVC 架构模式](docs/stage-03-oop/chapter-07-architecture/section-01-mvc.md)
  - [3.7.2 ADR 架构模式](docs/stage-03-oop/chapter-07-architecture/section-02-adr.md)
  - [3.7.3 模块化单体架构](docs/stage-03-oop/chapter-07-architecture/section-03-modular-monolith.md)
- [3.8 六边形架构](docs/stage-03-oop/chapter-08-hexagonal/README.md)
  - [3.8.1 六边形架构概述与 Ports](docs/stage-03-oop/chapter-08-hexagonal/section-01-overview-ports.md)
  - [3.8.2 Adapters（适配器）](docs/stage-03-oop/chapter-08-hexagonal/section-02-adapters.md)
  - [3.8.3 六边形架构实践](docs/stage-03-oop/chapter-08-hexagonal/section-03-practice.md)
- [3.9 DDD 领域驱动设计初级入门](docs/stage-03-oop/chapter-09-ddd/README.md)
  - [3.9.1 DDD 概述与分层](docs/stage-03-oop/chapter-09-ddd/section-01-overview.md)
  - [3.9.2 实体与值对象](docs/stage-03-oop/chapter-09-ddd/section-02-entities-value-objects.md)
  - [3.9.3 聚合根（Aggregate Root）](docs/stage-03-oop/chapter-09-ddd/section-03-aggregates.md)
  - [3.9.4 仓储模式](docs/stage-03-oop/chapter-09-ddd/section-04-repositories.md)
- [3.10 数据流设计进阶](docs/stage-03-oop/chapter-10-dataflow/README.md)
  - [3.10.1 CQRS 模式](docs/stage-03-oop/chapter-10-dataflow/section-01-cqrs.md)
  - [3.10.2 事件溯源（Event Sourcing）](docs/stage-03-oop/chapter-10-dataflow/section-02-event-sourcing.md)
  - [3.10.3 事件总线（Event Bus）](docs/stage-03-oop/chapter-10-dataflow/section-03-event-bus.md)
- [3.11 阶段总结](docs/stage-03-oop/chapter-11-summary/README.md)
- [3.12 微服务架构](docs/stage-03-oop/chapter-12-microservices/README.md)
  - [3.12.1 微服务概述与服务拆分](docs/stage-03-oop/chapter-12-microservices/section-01-overview-splitting.md)
  - [3.12.2 服务间通信](docs/stage-03-oop/chapter-12-microservices/section-02-communication.md)
  - [3.12.3 微服务挑战与解决方案](docs/stage-03-oop/chapter-12-microservices/section-03-challenges-solutions.md)

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

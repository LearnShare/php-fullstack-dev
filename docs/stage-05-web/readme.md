# 阶段五：Web/API 开发（Web Development）

本阶段深入讲解 PHP 在 Web 开发中的核心应用，从 HTTP 请求响应流程开始，逐步掌握 HTML 渲染、表单处理、文件上传、RESTful API 设计、会话管理、鉴权授权等 Web 开发必备技能。每章提供完整示例、安全最佳实践与常见陷阱说明，帮助你构建安全、可扩展的 Web 应用。

## 定位

Web 开发、API 设计、HTTP 协议、会话管理等

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握（阶段一、阶段二、阶段三、阶段四）

- **阶段一**：PHP 安装配置、MySQL 安装配置、开发工具
- **阶段二**：PHP 语言基础（语法、函数、数组、字符串）
- **阶段三**：面向对象编程、命名空间、自动加载
- **阶段四**：文件系统操作、错误处理、调试

### 不需要掌握

以下知识**不需要**提前掌握，会在学习过程中逐步学习：

- Web 开发（从零开始学习）
- HTTP 协议（从零开始学习）
- API 设计（从零开始学习）
- 会话管理（从零开始学习）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **基础检查**
   - [ ] 能够编写 PHP 类和对象
   - [ ] 理解继承和接口
   - [ ] 能够使用命名空间组织代码
   - [ ] 理解 Web 服务器的基本工作原理

2. **理解检查**
   - [ ] 理解 HTTP 协议的基本概念（请求、响应、方法、状态码）
   - [ ] 理解客户端和服务器的交互过程
   - [ ] 能够阅读和理解面向对象的 PHP 代码

3. **实践检查**
   - [ ] 完成阶段四的所有练习
   - [ ] 能够使用文件系统操作处理文件

**如果以上检查点都通过，可以开始阶段五的学习。**

**如果某些检查点未通过，建议：**
- 复习阶段三的相关章节（特别是 OOP 和架构模式）
- 完成阶段四的练习
- 确保理解面向对象编程后再继续

## 学习时间估算

**建议学习时间：** 6-8 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含练习和实践时间

**时间分配建议：**
- Web 交互基础（5.1-5.2）：1 周
- 超全局变量与 URL 处理（5.3-5.4）：1 周
- 请求处理（5.5-5.6）：1 周
- RESTful API（5.7-5.8）：1 周
- 会话与鉴权（5.9-5.10）：1 周
- 流量治理与安全（5.11）：1 周
- HTTP 客户端与中间件（5.12-5.13）：1 周
- 路由与验证（5.14-5.15）：1 周
- 错误处理与高级特性（5.16-5.18）：1 周
- 阶段总结与实践：1 周

## 练习建议

### 基础练习（每章完成后）

每完成一章，建议完成 3-5 个小练习，例如：

1. **Web 交互基础**
   - 创建简单的 HTTP 请求处理程序
   - 实现 HTML 渲染
   - 练习输出缓冲

2. **超全局变量**
   - 处理 GET 和 POST 请求
   - 实现 Session 管理
   - 处理文件上传

3. **API 设计**
   - 设计 RESTful API
   - 实现 API 路由
   - 实现 API 文档

### 综合练习（阶段完成后）

4. **综合项目**
   - 开发一个完整的 Web 应用（如：博客系统）
   - 实现用户认证和授权
   - 实现 RESTful API
   - 实现文件上传功能

**项目要求：**
- 使用本阶段学到的所有知识点
- 代码规范、注释清晰
- 完善的错误处理和日志记录
- 安全最佳实践

## 章节内容

1. **[5.1 Web 交互基础：从请求到响应](chapter-01-request-response/readme.md)**：HTTP 请求响应流程、Web Server 与 PHP-FPM 协作、Shared Nothing 架构。
   - [5.1.1 HTTP 请求响应流程](chapter-01-request-response/section-01-http-flow.md)
   - [5.1.2 Web Server 与 PHP-FPM 协作](chapter-01-request-response/section-02-web-server-fpm.md)
   - [5.1.3 Shared Nothing 架构](chapter-01-request-response/section-03-shared-nothing.md)

2. **[5.2 HTML 渲染与输出控制](chapter-02-html-rendering/readme.md)**：HTML 渲染与模板引擎、输出缓冲与控制。
   - [5.2.1 HTML 渲染与模板引擎](chapter-02-html-rendering/section-01-rendering-templates.md)
   - [5.2.2 输出缓冲与控制](chapter-02-html-rendering/section-02-output-buffering.md)

3. **[5.3 超全局变量：Web 输入的核心](chapter-03-superglobals/readme.md)**：$_GET、$_POST 与 $_REQUEST、$_SERVER、$_SESSION 与 $_COOKIE、$_FILES 与安全处理。
   - [5.3.1 $_GET、$_POST 与 $_REQUEST](chapter-03-superglobals/section-01-get-post-request.md)
   - [5.3.2 $_SERVER、$_SESSION 与 $_COOKIE](chapter-03-superglobals/section-02-server-session-cookie.md)
   - [5.3.3 $_FILES 与安全处理](chapter-03-superglobals/section-03-files-security.md)

4. **[5.4 URL 处理](chapter-04-url-handling/readme.md)**：URL 解析与构建、URL 编码与解码、查询字符串处理、URL 路由与重写。
   - [5.4.1 URL 解析与构建](chapter-04-url-handling/section-01-url-parsing.md)
   - [5.4.2 URL 编码与解码](chapter-04-url-handling/section-02-url-encoding.md)
   - [5.4.3 查询字符串处理](chapter-04-url-handling/section-03-query-string.md)
   - [5.4.4 URL 路由与重写](chapter-04-url-handling/section-04-url-routing.md)

5. **[5.5 请求体解析：处理 JSON / API 请求](chapter-05-json-requests/readme.md)**：JSON 请求解析、API 请求处理。
   - [5.5.1 JSON 请求解析](chapter-05-json-requests/section-01-json-parsing.md)
   - [5.5.2 API 请求处理](chapter-05-json-requests/section-02-api-requests.md)

6. **[5.6 文件上传处理](chapter-06-file-upload/readme.md)**：文件上传基础、文件验证与安全、文件存储策略。
   - [5.6.1 文件上传基础](chapter-06-file-upload/section-01-upload-basics.md)
   - [5.6.2 文件验证与安全](chapter-06-file-upload/section-02-validation-security.md)
   - [5.6.3 文件存储策略](chapter-06-file-upload/section-03-storage-strategy.md)

7. **[5.7 RESTful API 设计与接口规范](chapter-07-restful-api/readme.md)**：RESTful 设计原则、API 路由与版本控制、API 文档与测试、API 测试工具、API 文档。
   - [5.7.1 RESTful 设计原则](chapter-07-restful-api/section-01-restful-principles.md)
   - [5.7.2 API 路由与版本控制](chapter-07-restful-api/section-02-routing-versioning.md)
   - [5.7.3 API 文档与测试](chapter-07-restful-api/section-03-documentation-testing.md)
   - [5.7.4 API 测试工具](chapter-07-restful-api/section-04-api-testing.md)
   - [5.7.5 API 文档](chapter-07-restful-api/section-05-api-documentation.md)

8. **[5.8 响应处理与跨域（CORS）](chapter-08-response-cors/readme.md)**：响应处理、CORS 跨域处理。
   - [5.8.1 响应处理](chapter-08-response-cors/section-01-response-handling.md)
   - [5.8.2 CORS 跨域处理](chapter-08-response-cors/section-02-cors.md)

9. **[5.9 会话与状态管理](chapter-09-session/readme.md)**：Session 基础、Cookie 管理、状态管理最佳实践。
   - [5.9.1 Session 基础](chapter-09-session/section-01-session-basics.md)
   - [5.9.2 Cookie 管理](chapter-09-session/section-02-cookie-management.md)
   - [5.9.3 状态管理最佳实践](chapter-09-session/section-03-state-management.md)

10. **[5.10 鉴权与授权模型（AuthN & AuthZ）](chapter-10-auth/readme.md)**：认证基础（AuthN）、授权模型（AuthZ）、JWT 与 Token、OAuth2 实现。
    - [5.10.1 认证基础（AuthN）](chapter-10-auth/section-01-authentication.md)
    - [5.10.2 授权模型（AuthZ）](chapter-10-auth/section-02-authorization.md)
    - [5.10.3 JWT 与 Token](chapter-10-auth/section-03-jwt-token.md)
    - [5.10.4 OAuth2 实现](chapter-10-auth/section-04-oauth2.md)

11. **[5.11 流量治理与安全](chapter-11-traffic-security/readme.md)**：Rate Limiting、请求签名与验证、安全最佳实践。
    - [5.11.1 Rate Limiting](chapter-11-traffic-security/section-01-rate-limiting.md)
    - [5.11.2 请求签名与验证](chapter-11-traffic-security/section-02-request-signing.md)
    - [5.11.3 安全最佳实践](chapter-11-traffic-security/section-03-security-best-practices.md)

12. **[5.12 HTTP 客户端](chapter-12-http-client/readme.md)**：Guzzle 基础、异步请求、错误处理与重试。
    - [5.12.1 Guzzle 基础](chapter-12-http-client/section-01-guzzle-basics.md)
    - [5.12.2 异步请求](chapter-12-http-client/section-02-async-requests.md)
    - [5.12.3 错误处理与重试](chapter-12-http-client/section-03-error-handling-retry.md)

13. **[5.13 中间件深入](chapter-13-middleware/readme.md)**：中间件概述、中间件模式、编写自定义中间件、中间件最佳实践。
    - [5.13.1 中间件概述](chapter-13-middleware/section-01-overview.md)
    - [5.13.2 中间件模式](chapter-13-middleware/section-02-patterns.md)
    - [5.13.3 编写自定义中间件](chapter-13-middleware/section-03-custom.md)
    - [5.13.4 中间件最佳实践](chapter-13-middleware/section-04-best-practices.md)

14. **[5.14 路由设计](chapter-14-routing/readme.md)**：路由设计概述、路由组织与模块化、动态路由与参数、路由守卫与权限控制。
    - [5.14.1 路由设计概述](chapter-14-routing/section-01-overview.md)
    - [5.14.2 路由组织与模块化](chapter-14-routing/section-02-organization.md)
    - [5.14.3 动态路由与参数](chapter-14-routing/section-03-dynamic.md)
    - [5.14.4 路由守卫与权限控制](chapter-14-routing/section-04-guards.md)

15. **[5.15 请求验证与数据校验](chapter-15-validation/readme.md)**：请求验证概述、Symfony Validator、Respect/Validation、自定义验证规则。
    - [5.15.1 请求验证概述](chapter-15-validation/section-01-overview.md)
    - [5.15.2 Symfony Validator](chapter-15-validation/section-02-symfony-validator.md)
    - [5.15.3 Respect/Validation](chapter-15-validation/section-03-respect-validation.md)
    - [5.15.4 自定义验证规则](chapter-15-validation/section-04-custom-rules.md)

16. **[5.16 错误处理最佳实践（Web 层面）](chapter-16-error-handling/readme.md)**：错误处理概述、错误分类与自定义错误、错误中间件、错误响应格式。
    - [5.16.1 错误处理概述](chapter-16-error-handling/section-01-overview.md)
    - [5.16.2 错误分类与自定义错误](chapter-16-error-handling/section-02-error-types.md)
    - [5.16.3 错误中间件](chapter-16-error-handling/section-03-error-middleware.md)
    - [5.16.4 错误响应格式](chapter-16-error-handling/section-04-response-format.md)

17. **[5.17 WebSocket 服务器](chapter-17-websocket/readme.md)**：WebSocket 概述、Ratchet、ReactPHP WebSocket、WebSocket 最佳实践。
    - [5.17.1 WebSocket 概述](chapter-17-websocket/section-01-overview.md)
    - [5.17.2 Ratchet](chapter-17-websocket/section-02-ratchet.md)
    - [5.17.3 ReactPHP WebSocket](chapter-17-websocket/section-03-reactphp.md)
    - [5.17.4 WebSocket 最佳实践](chapter-17-websocket/section-04-best-practices.md)

18. **[5.18 GraphQL API](chapter-18-graphql/readme.md)**：GraphQL 概述、GraphQLite、Lighthouse、GraphQL 最佳实践。
    - [5.18.1 GraphQL 概述](chapter-18-graphql/section-01-overview.md)
    - [5.18.2 GraphQLite](chapter-18-graphql/section-02-graphqlite.md)
    - [5.18.3 Lighthouse](chapter-18-graphql/section-03-lighthouse.md)
    - [5.18.4 GraphQL 最佳实践](chapter-18-graphql/section-04-best-practices.md)

## 完成本阶段后，你将具备：

- 扎实的 Web 开发能力，能够构建完整的 Web 应用
- 理解 HTTP 协议和 Web 架构，能够设计 RESTful API
- 掌握会话管理和鉴权授权机制
- 掌握安全最佳实践，能够防范常见安全漏洞
- 能够使用中间件、路由等现代 Web 开发模式
- 能够实现 WebSocket 和 GraphQL 等高级特性

## 相关章节

- **阶段一：基础入门**：环境搭建和工具配置
- **阶段二：PHP 语言特性**：PHP 基础语法
- **阶段三：面向对象编程基础**：OOP 基础
- **阶段四：系统编程**：文件系统、错误处理
- **阶段六：数据库与缓存系统**：数据持久化
- **阶段八：性能优化与安全**：性能优化和安全防护

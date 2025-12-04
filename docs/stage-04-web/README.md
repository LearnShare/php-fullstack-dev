# 阶段四：Web 服务与 API-First 开发

本阶段深入讲解 PHP 在 Web 开发中的核心应用，从 HTTP 请求响应流程开始，逐步掌握 HTML 渲染、表单处理、文件上传、RESTful API 设计、会话管理、鉴权授权等 Web 开发必备技能。每章提供完整示例、安全最佳实践与常见陷阱说明，帮助你构建安全、可扩展的 Web 应用。

1. **[Web 交互基础：从请求到响应](chapter-01-request-response/README.md)**：Web 交互基础：从请求到响应。
   - [4.1.1 HTTP 请求响应流程](chapter-01-request-response/section-01-http-flow.md)
   - [4.1.2 Web Server 与 PHP-FPM 协作](chapter-01-request-response/section-02-web-server-fpm.md)
   - [4.1.3 Shared Nothing 架构](chapter-01-request-response/section-03-shared-nothing.md)
2. **[HTML 渲染与输出控制](chapter-02-html-rendering/README.md)**：HTML 渲染与输出控制。
   - [4.2.1 HTML 渲染与模板引擎](chapter-02-html-rendering/section-01-rendering-templates.md)
   - [4.2.2 输出缓冲与控制](chapter-02-html-rendering/section-02-output-buffering.md)
3. **[超全局变量：Web 输入的核心](chapter-03-superglobals/README.md)**：超全局变量：Web 输入的核心。
   - [4.3.1 $_GET、$_POST 与 $_REQUEST](chapter-03-superglobals/section-01-get-post-request.md)
   - [4.3.2 $_SERVER、$_SESSION 与 $_COOKIE](chapter-03-superglobals/section-02-server-session-cookie.md)
   - [4.3.3 $_FILES 与安全处理](chapter-03-superglobals/section-03-files-security.md)
4. **[请求体解析：处理 JSON / API 请求](chapter-04-json-requests/README.md)**：请求体解析：处理 JSON / API 请求。
   - [4.4.1 JSON 请求解析](chapter-04-json-requests/section-01-json-parsing.md)
   - [4.4.2 API 请求处理](chapter-04-json-requests/section-02-api-requests.md)
5. **[文件上传处理](chapter-05-file-upload/README.md)**：文件上传处理。
   - [4.5.1 文件上传基础](chapter-05-file-upload/section-01-upload-basics.md)
   - [4.5.2 文件验证与安全](chapter-05-file-upload/section-02-validation-security.md)
   - [4.5.3 文件存储策略](chapter-05-file-upload/section-03-storage-strategy.md)
6. **[RESTful API 设计与接口规范](chapter-06-restful-api/README.md)**：RESTful API 设计与接口规范。
   - [4.6.1 RESTful 设计原则](chapter-06-restful-api/section-01-restful-principles.md)
   - [4.6.2 API 路由与版本控制](chapter-06-restful-api/section-02-routing-versioning.md)
   - [4.6.3 API 文档与测试](chapter-06-restful-api/section-03-documentation-testing.md)
7. **[响应处理与跨域（CORS）](chapter-07-response-cors/README.md)**：响应处理与跨域（CORS）。
   - [4.7.1 响应处理](chapter-07-response-cors/section-01-response-handling.md)
   - [4.7.2 CORS 跨域处理](chapter-07-response-cors/section-02-cors.md)
8. **[会话与状态管理](chapter-08-session/README.md)**：会话与状态管理。
   - [4.8.1 Session 基础](chapter-08-session/section-01-session-basics.md)
   - [4.8.2 Cookie 管理](chapter-08-session/section-02-cookie-management.md)
   - [4.8.3 状态管理最佳实践](chapter-08-session/section-03-state-management.md)
9. **[鉴权与授权模型（AuthN & AuthZ）](chapter-09-auth/README.md)**：鉴权与授权模型（AuthN & AuthZ）。
   - [4.9.1 认证基础（AuthN）](chapter-09-auth/section-01-authentication.md)
   - [4.9.2 授权模型（AuthZ）](chapter-09-auth/section-02-authorization.md)
   - [4.9.3 JWT 与 Token](chapter-09-auth/section-03-jwt-token.md)
   - [4.9.4 OAuth2 实现](chapter-09-auth/section-04-oauth2.md)
10. **[流量治理与安全](chapter-10-traffic-security/README.md)**：流量治理与安全。
    - [4.10.1 Rate Limiting](chapter-10-traffic-security/section-01-rate-limiting.md)
    - [4.10.2 请求签名与验证](chapter-10-traffic-security/section-02-request-signing.md)
    - [4.10.3 安全最佳实践](chapter-10-traffic-security/section-03-security-best-practices.md)
11. **[HTTP 客户端：Guzzle 详细使用](chapter-11-http-client/README.md)**：HTTP 客户端：Guzzle 详细使用。
    - [4.11.1 Guzzle 基础](chapter-11-http-client/section-01-guzzle-basics.md)
    - [4.11.2 异步请求](chapter-11-http-client/section-02-async-requests.md)
    - [4.11.3 错误处理与重试](chapter-11-http-client/section-03-error-handling-retry.md)

完成阶段四后，你将具备：

- 深入理解 HTTP 协议与 Web 请求响应流程，能够处理各种类型的 Web 请求。
- 掌握 HTML 渲染、模板引擎使用、输出缓冲等 Web 输出技术。
- 熟练使用超全局变量处理表单数据、查询参数、文件上传等 Web 输入。
- 能够设计符合 RESTful 规范的 API，处理 JSON 请求响应。
- 理解会话管理、Cookie、Session、JWT 等状态管理机制。
- 掌握鉴权授权、OAuth2、RBAC 等安全机制。
- 具备 API 安全防护能力：CORS、Rate Limiting、请求签名等。

# 6.4 安全 HTTP 头

## 目标

- 理解安全 HTTP 头的作用。
- 掌握 CSP（内容安全策略）的配置。
- 熟悉 HSTS、X-Frame-Options 等安全头。
- 了解 Referrer-Policy、Permissions-Policy 的配置。

## 章节内容

本章分为两个独立小节，每节提供详细的概念解释、代码示例和最佳实践：

1. **[安全 HTTP 头](section-01-security-headers.md)**：CSP、HSTS、X-Frame-Options、X-Content-Type-Options、Referrer-Policy、完整示例。

2. **[安全配置](section-02-security-config.md)**：安全头配置、中间件实现、Nginx 配置、安全测试、完整示例。

## 核心概念

- **CSP**：内容安全策略
- **HSTS**：HTTP 严格传输安全
- **安全头**：HTTP 安全响应头
- **安全配置**：统一安全配置

## 学习建议

1. **重点掌握**：
   - CSP 配置
   - 安全头设置
   - 安全配置

2. **实践练习**：
   - 完成每小节后的练习题目
   - 配置安全 HTTP 头
   - 进行安全测试

## 完成本章后

- 能够配置安全 HTTP 头。
- 理解 CSP，能够有效防护 XSS。
- 掌握安全配置，能够提升应用安全性。
- 具备构建安全 Web 应用的能力。

## 相关章节

- **6.1 OWASP Top 10**：安全风险
- **4.7 响应处理与跨域**：响应处理

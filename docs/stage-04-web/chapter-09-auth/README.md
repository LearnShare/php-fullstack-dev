# 4.9 鉴权与授权模型（AuthN & AuthZ）

## 目标

- 理解鉴权（Authentication）与授权（Authorization）的区别。
- 掌握基础 Token 鉴权、JWT 鉴权的实现。
- 了解 OAuth2 流程与第三方登录。
- 熟悉 RBAC/ABAC 权限模型的设计与实现。

## 章节内容

本章分为四个独立小节，每节提供详细的概念解释、代码示例和最佳实践：

1. **[认证基础（AuthN）](section-01-authentication.md)**：鉴权与授权区别、基础 Token 鉴权、密码验证、Session 认证、完整示例。

2. **[授权模型（AuthZ）](section-02-authorization.md)**：RBAC（基于角色的访问控制）、ABAC（基于属性的访问控制）、权限检查、完整示例。

3. **[JWT 与 Token](section-03-jwt-token.md)**：JWT 结构、JWT 实现、Token 刷新、Token 撤销、完整示例。

4. **[OAuth2 实现](section-04-oauth2.md)**：OAuth2 流程、授权码模式、客户端凭证模式、第三方登录、完整示例。

## 核心概念

- **Authentication（认证）**：验证用户身份
- **Authorization（授权）**：验证用户权限
- **JWT**：JSON Web Token，无状态认证方案
- **OAuth2**：授权框架，支持第三方登录

## 学习建议

1. **重点掌握**：
   - 认证和授权的区别
   - JWT 的使用和安全
   - RBAC 权限模型

2. **实践练习**：
   - 完成每小节后的练习题目
   - 实现完整的认证授权系统
   - 实现 JWT 认证

## 完成本章后

- 能够实现完整的认证和授权系统。
- 理解 JWT 的工作原理，能够安全使用。
- 掌握 RBAC 权限模型的设计和实现。
- 具备实现 OAuth2 第三方登录的能力。

## 相关章节

- **4.8 会话与状态管理**：Session 和 Cookie 管理
- **阶段六：安全、性能与可观测性**：安全最佳实践

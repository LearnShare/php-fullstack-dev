# 5.10 鉴权与授权模型（AuthN & AuthZ）

## 目标

- 理解认证（Authentication）和授权（Authorization）的区别，掌握鉴权授权的基本概念。
- 掌握 JWT 和 Token 的使用方法，能够实现基于 Token 的认证。
- 理解 OAuth2 协议，能够实现第三方登录功能。

## 章节内容

本章分为四个独立小节，每节提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[5.10.1 认证基础（AuthN）](section-01-authentication.md)**：认证概念、密码验证、Session 认证、Token 认证、完整示例。

2. **[5.10.2 授权模型（AuthZ）](section-02-authorization.md)**：授权概念、RBAC（基于角色的访问控制）、ACL（访问控制列表）、权限检查、完整示例。

3. **[5.10.3 JWT 与 Token](section-03-jwt-token.md)**：JWT 概念、JWT 结构、JWT 生成和验证、Token 刷新、完整示例。

4. **[5.10.4 OAuth2 实现](section-04-oauth2.md)**：OAuth2 概念、OAuth2 流程、授权码模式、客户端模式、完整示例。

## 核心概念

- **认证（Authentication）**：验证用户身份
- **授权（Authorization）**：控制访问权限
- **JWT**：JSON Web Token
- **OAuth2**：授权协议

## 学习建议

1. **按顺序学习**：按顺序学习四个小节，理解鉴权授权的完整体系。

2. **重点掌握**：
   - 认证和授权的区别
   - JWT 的使用
   - OAuth2 的实现
   - 权限控制模型

3. **实践练习**：
   - 完成每小节后的练习题目
   - 实现用户认证系统
   - 实现权限控制系统

## 完成本章后

- 能够实现用户认证功能。
- 能够实现权限控制系统。
- 能够使用 JWT 进行 Token 认证。
- 能够实现 OAuth2 第三方登录。

## 相关章节

- **5.9 会话与状态管理**：Session 在认证中的应用。
- **8.2 安全防护**：安全相关的最佳实践。

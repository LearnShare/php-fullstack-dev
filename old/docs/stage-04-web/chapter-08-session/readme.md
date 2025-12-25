# 4.8 会话与状态管理

## 目标

- 理解 Web 的无状态特性与状态管理需求。
- 掌握 Cookie、Session、Token/JWT 等状态管理机制。
- 熟悉 Session 的工作原理、配置与安全设置。
- 了解 Redis Session Store 的使用场景与实现。

## 章节内容

本章分为三个独立小节，每节提供详细的概念解释、代码示例和最佳实践：

1. **[Session 基础](section-01-session-basics.md)**：Web 无状态特性、Session 工作原理、Session 配置、Session 存储（文件/Redis）、Session 安全设置及完整示例。

2. **[Cookie 管理](section-02-cookie-management.md)**：Cookie 基础、Cookie 参数详解、SameSite 属性、安全 Cookie 设置、Cookie 与 Session 配合使用及完整示例。

3. **[状态管理最佳实践](section-03-state-management.md)**：Token/JWT 使用、状态管理方案对比、分布式 Session、安全最佳实践、完整示例及选择指南。

## 核心概念

- **无状态协议**：HTTP 本身无状态
- **Session**：服务器端状态存储
- **Cookie**：客户端状态存储
- **Token/JWT**：无状态认证方案

## 学习建议

1. **重点掌握**：
   - Session 的工作原理和配置
   - Cookie 的安全设置
   - 不同状态管理方案的适用场景

2. **实践练习**：
   - 完成每小节后的练习题目
   - 实现 Session 管理类
   - 实现 Token 认证系统

## 完成本章后

- 能够选择合适的状态管理方案。
- 理解 Session 和 Cookie 的工作原理。
- 掌握安全的状态管理配置。
- 具备构建分布式状态管理系统的能力。

## 相关章节

- **4.3.2 $_SERVER、$_SESSION 与 $_COOKIE**：超全局变量基础
- **4.9 鉴权与授权模型**：认证与授权

# 5.11.3 安全最佳实践

## 概述

Web 安全是应用开发的重要方面。本节总结 Web 安全的最佳实践，包括 OWASP Top 10、常见漏洞防护等，帮助零基础学员掌握安全防护技术。

**章节类型**：最佳实践章节

**主要内容**：
- Web 安全概述
- OWASP Top 10
- XSS 防护
- CSRF 防护
- SQL 注入防护
- 其他安全措施
- 完整示例

## 核心内容

### Web 安全概述

- 安全的重要性
- 常见攻击类型
- 防护原则

### OWASP Top 10

- 注入攻击
- 失效的身份认证
- 敏感数据泄露
- XML 外部实体
- 失效的访问控制
- 安全配置错误
- 跨站脚本（XSS）
- 不安全的反序列化
- 使用含有已知漏洞的组件
- 不足的日志记录和监控

### XSS 防护

- XSS 类型
- 输出转义
- Content Security Policy
- 输入验证

### CSRF 防护

- CSRF 攻击原理
- Token 验证
- SameSite Cookie
- Referer 检查

## 基本用法

### 安全防护示例

```php
<?php
declare(strict_types=1);

// XSS 防护：转义输出
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');

// CSRF 防护：Token 验证
if (!hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
    die('CSRF token mismatch');
}
```

## 使用场景

- 所有 Web 应用
- 用户输入处理
- 敏感操作
- 数据展示

## 注意事项

- 多层防护
- 安全配置
- 定期更新
- 安全审计

## 常见问题

- 如何防护 XSS？
- 如何防护 CSRF？
- 如何防护 SQL 注入？
- OWASP Top 10 是什么？

## 最佳实践

- 始终验证和转义输入
- 使用参数化查询
- 实现 CSRF 防护
- 定期安全审计

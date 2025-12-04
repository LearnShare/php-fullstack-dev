# 4.4 请求体解析：处理 JSON / API 请求

## 目标

- 理解为什么 AJAX / API 传 JSON 时 `$_POST` 是空的。
- 掌握使用 `php://input` 读取原始请求体。
- 熟悉 JSON 序列化与反序列化的安全处理。
- 能够正确处理各种 Content-Type 的 API 请求。

## 章节内容

本章分为两个独立小节，每节提供详细的概念解释、代码示例和最佳实践：

1. **[JSON 请求解析](section-01-json-parsing.md)**：为什么 `$_POST` 是空的、`php://input` 使用、`json_validate()` 函数（PHP 8.3+）、JSON 解码安全处理、性能优化及完整示例。

2. **[API 请求处理](section-02-api-requests.md)**：不同 Content-Type 处理、请求体解析封装、错误处理、API 响应格式、完整示例及最佳实践。

## 核心概念

- **Content-Type**：决定数据如何解析
- **php://input**：读取原始请求体的流
- **JSON 验证**：先验证再解码，提升性能
- **错误处理**：统一的错误响应格式

## 学习建议

1. **重点掌握**：
   - `php://input` 的使用
   - JSON 请求的安全处理
   - 不同 Content-Type 的处理方式

2. **实践练习**：
   - 完成每小节后的练习题目
   - 实现一个 API 请求处理类
   - 处理各种 Content-Type 的请求

## 完成本章后

- 能够正确处理 JSON API 请求。
- 理解不同 Content-Type 的处理方式。
- 掌握 JSON 验证和解码的安全方法。
- 具备构建 API 请求处理系统的能力。

## 相关章节

- **2.4.6 JSON 与数组互转**：JSON 基础
- **4.3 超全局变量**：Web 输入的核心

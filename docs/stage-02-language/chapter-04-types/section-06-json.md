# 2.4.6 JSON 与数组互转

## 概述

JSON 是数据交换的常用格式，PHP 提供了 `json_encode()` 和 `json_decode()` 函数进行 JSON 编码和解码。本节详细介绍这两个函数的语法、常用选项标志、错误处理、实际应用场景。

**章节类型**：工具性章节

**主要内容**：
- `json_encode()` 的语法和用法
- `json_decode()` 的语法和用法
- 常用选项标志（`JSON_PRETTY_PRINT`、`JSON_UNESCAPED_UNICODE` 等）
- 错误处理和检测
- JSON 与数组的互转
- 实际应用场景

## 特性

- **编码**：将 PHP 值转换为 JSON 字符串
- **解码**：将 JSON 字符串转换为 PHP 值
- **选项控制**：通过选项标志控制编码/解码行为
- **错误处理**：提供错误检测和处理机制

## 基本用法/命令

### json_encode()

**语法**：`json_encode(mixed $value, int $flags = 0, int $depth = 512): string|false`
**功能**：将 PHP 值编码为 JSON 字符串
**返回值**：成功返回 JSON 字符串，失败返回 `false`

### json_decode()

**语法**：`json_decode(string $json, ?bool $associative = null, int $depth = 512, int $flags = 0): mixed`
**功能**：将 JSON 字符串解码为 PHP 值
**返回值**：成功返回 PHP 值，失败返回 `null`

### 错误检测

**json_last_error()**：获取最后一次 JSON 操作的错误代码
**json_last_error_msg()**：获取最后一次 JSON 操作的错误消息

## 使用场景

- **API 数据交换**：与前端或其他服务交换数据
- **配置文件**：存储和读取配置
- **数据持久化**：将数据保存为 JSON 格式
- **日志记录**：结构化日志记录

## 注意事项

- **编码问题**：确保使用 UTF-8 编码
- **深度限制**：注意 `$depth` 参数的限制
- **错误处理**：始终检查返回值和处理错误
- **选项标志**：根据需求选择合适的选项

## 常见问题

### 问题 1：JSON 编码失败
- **原因**：包含无法编码的值（如资源）
- **解决**：检查数据，移除无法编码的值

### 问题 2：JSON 解码失败
- **原因**：JSON 格式错误
- **解决**：使用 `json_last_error()` 检查错误

## 最佳实践

- 始终检查返回值
- 使用 `JSON_THROW_ON_ERROR` 标志（PHP 7.3+）
- 使用 `JSON_PRETTY_PRINT` 提高可读性
- 使用 `JSON_UNESCAPED_UNICODE` 处理中文

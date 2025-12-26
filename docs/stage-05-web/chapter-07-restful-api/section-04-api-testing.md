# 5.7.4 API 测试工具

## 概述

API 测试工具能够提高 API 测试的效率。本节介绍常用的 API 测试工具，包括 Postman、curl、PHPUnit 等，帮助零基础学员掌握 API 测试方法。

**章节类型**：工具性章节

**主要内容**：
- API 测试工具概述
- Postman 使用
- curl 命令行工具
- PHPUnit API 测试
- 测试自动化
- 完整示例

## 核心内容

### API 测试工具概述

- 工具分类
- 工具选择
- 测试场景

### Postman

- Postman 基础
- 请求构建
- 测试脚本
- 集合管理

### curl

- curl 命令基础
- 请求发送
- 参数设置
- 脚本化测试

### PHPUnit API 测试

- API 测试类
- 请求模拟
- 响应断言
- 测试套件

## 基本用法

### curl 示例

```bash
# GET 请求
curl http://api.example.com/users

# POST 请求
curl -X POST http://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John"}'
```

### PHPUnit 示例

```php
<?php
declare(strict_types=1);

public function testGetUsers(): void {
    $response = $this->get('/api/users');
    $response->assertStatus(200);
}
```

## 使用场景

- API 开发测试
- 集成测试
- 回归测试
- 性能测试

## 注意事项

- 测试环境配置
- 测试数据管理
- 测试隔离
- 测试覆盖率

## 常见问题

- 如何选择测试工具？
- 如何使用 Postman？
- 如何编写 API 测试？
- 如何自动化测试？

## 最佳实践

- 使用 Postman 进行手动测试
- 使用 PHPUnit 进行自动化测试
- 编写完整的测试用例
- 定期运行测试

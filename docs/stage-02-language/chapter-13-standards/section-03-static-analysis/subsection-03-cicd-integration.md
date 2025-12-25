# 2.13.3.3 静态分析与 CI/CD 集成

## 概述

将静态分析集成到 CI/CD 流程可以自动化代码质量检查。本节介绍如何将 PHPStan 和 Psalm 集成到 GitHub Actions、GitLab CI 等 CI/CD 系统中。

**章节类型**：实践性章节

**主要内容**：
- GitHub Actions 集成
- GitLab CI 集成
- 其他 CI/CD 系统集成
- 配置示例
- 最佳实践

## 核心内容

### CI/CD 集成

**目的**：自动化代码质量检查
**好处**：及早发现问题、统一代码标准
**工具**：GitHub Actions、GitLab CI、Jenkins

### GitHub Actions 示例

```yaml
name: Static Analysis

on: [push, pull_request]

jobs:
  phpstan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: shivammathur/setup-php@v2
      - run: composer install
      - run: vendor/bin/phpstan analyse
```

## 使用场景

- **持续集成**：每次提交自动检查
- **代码审查**：作为代码审查的一部分
- **质量保证**：确保代码质量

## 注意事项

- **性能考虑**：分析可能需要时间
- **缓存使用**：使用缓存提高性能
- **失败处理**：合理设置失败条件

## 常见问题

### 问题 1：CI 运行时间过长
- **原因**：分析时间过长
- **解决**：使用缓存、增量分析

### 问题 2：误报导致失败
- **原因**：分析级别过高
- **解决**：调整级别或添加注释

## 最佳实践

- 集成到 CI/CD 流程
- 使用缓存提高性能
- 合理设置失败条件
- 定期审查分析结果

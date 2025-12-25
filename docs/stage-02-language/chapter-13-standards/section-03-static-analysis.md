# 2.13.3 静态代码分析

## 概述

静态代码分析是提高代码质量的重要工具。本节介绍 PHPStan 配置与使用、Psalm 配置与使用、静态分析与 CI/CD 集成、代码质量提升实践。

**章节类型**：工具性章节

**主要内容**：
- 静态代码分析的概念
- PHPStan 的配置和使用
- Psalm 的配置和使用
- 静态分析与 CI/CD 集成
- 代码质量提升实践
- 工具对比和选择

## 核心内容

### 静态代码分析

**概念**：在不运行代码的情况下分析代码
**作用**：发现错误、提高代码质量
**工具**：PHPStan、Psalm、PHP_CodeSniffer

### PHPStan

**特点**：功能强大、配置灵活
**级别**：支持多个分析级别
**集成**：支持 IDE 和 CI/CD

### Psalm

**特点**：由 Vimeo 开发、性能优秀
**功能**：类型检查、错误检测
**集成**：支持 IDE 和 CI/CD

## 基本用法/命令

### PHPStan 安装

```bash
composer require --dev phpstan/phpstan
```

### PHPStan 配置

```neon
# phpstan.neon
parameters:
    level: 5
    paths:
        - src
```

### PHPStan 运行

```bash
vendor/bin/phpstan analyse
```

### Psalm 安装

```bash
composer require --dev vimeo/psalm
```

### Psalm 配置

```xml
<!-- psalm.xml -->
<psalm>
    <projectFiles>
        <directory name="src" />
    </projectFiles>
</psalm>
```

### Psalm 运行

```bash
vendor/bin/psalm
```

## 使用场景

- **代码质量**：提高代码质量
- **错误检测**：及早发现错误
- **团队协作**：统一代码标准

## 注意事项

- **配置级别**：根据项目选择合适的级别
- **性能考虑**：大型项目可能需要较长时间
- **持续集成**：集成到 CI/CD 流程

## 常见问题

### 问题 1：误报过多
- **原因**：分析级别过高
- **解决**：降低分析级别或添加注释

### 问题 2：性能问题
- **原因**：项目过大
- **解决**：使用缓存、增量分析

## 最佳实践

- 选择合适的分析级别
- 集成到 CI/CD 流程
- 逐步提高代码质量
- 使用工具提供的修复建议

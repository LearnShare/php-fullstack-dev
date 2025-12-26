# 10.5.4 配置与存储

## 概述

配置和存储是 Kubernetes 应用管理的重要方面。本节介绍 ConfigMap、Secret、Volume、持久化存储，帮助零基础学员理解如何管理配置和存储。

**章节类型**：实践性章节

**主要内容**：
- ConfigMap
- Secret
- Volume
- 持久化存储
- 完整示例

## 核心内容

### ConfigMap

- ConfigMap 概述
- ConfigMap 创建
- ConfigMap 使用
- ConfigMap 更新

### Secret

- Secret 概述
- Secret 创建
- Secret 使用
- Secret 管理

### Volume

- Volume 概述
- Volume 类型
- Volume 挂载
- Volume 使用

### 持久化存储

- 持久化存储概述
- 存储类（StorageClass）
- 持久卷（PV）
- 持久卷声明（PVC）

## 基本用法

### 配置和存储示例

```yaml
# 配置和存储示例
```

## 完整示例

### 示例：配置和存储使用

```yaml
# 完整示例代码
```

## 注意事项

- 安全管理 Secret
- 合理使用 ConfigMap
- 选择合适的存储类型
- 管理存储生命周期

## 常见问题

### Q: ConfigMap 和 Secret 有什么区别？

A: ConfigMap 存储非敏感配置，Secret 存储敏感信息（如密码、密钥）。

### Q: 如何实现持久化存储？

A: 使用 PV 和 PVC，或使用动态存储类自动创建存储。

## 最佳实践

- 安全管理 Secret
- 合理使用 ConfigMap
- 选择合适的存储类型
- 管理存储生命周期

## 对比分析

### 不同存储类型对比

- 临时存储 vs 持久化存储
- 适用场景对比

## 实践任务

1. 创建 ConfigMap 和 Secret
2. 配置 Volume
3. 实现持久化存储

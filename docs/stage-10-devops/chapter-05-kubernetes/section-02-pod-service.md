# 10.5.2 Pod 与 Service

## 概述

Pod 和 Service 是 Kubernetes 的核心资源。本节介绍 Pod 和 Service 的概念、配置方法、使用场景，帮助零基础学员理解如何使用 Pod 和 Service。

**章节类型**：语法性章节

**主要内容**：
- Pod 概述
- Service 概述
- Pod 配置
- Service 配置
- 完整示例

## 核心内容

### Pod 概述

- Pod 的作用
- Pod 特点
- Pod 生命周期

### Service 概述

- Service 的作用
- Service 类型
- Service 原理

### Pod 配置

- Pod 定义
- 容器配置
- 资源限制
- 环境变量

### Service 配置

- Service 定义
- 服务类型
- 端口配置
- 选择器配置

## 基本用法

### Pod 和 Service 示例

```yaml
# Pod 和 Service 示例
```

## 完整示例

### 示例：Pod 和 Service 配置

```yaml
# 完整示例代码
```

## 注意事项

- 合理配置 Pod 资源
- 选择合适的 Service 类型
- 配置健康检查
- 管理 Pod 生命周期

## 常见问题

### Q: Pod 和容器有什么区别？

A: Pod 是 Kubernetes 的最小部署单元，可以包含一个或多个容器。

### Q: Service 如何实现服务发现？

A: Service 通过标签选择器选择 Pod，提供稳定的网络访问。

## 最佳实践

- 合理配置 Pod 资源
- 选择合适的 Service 类型
- 配置健康检查
- 管理 Pod 生命周期

## 对比分析

### 不同 Service 类型对比

- ClusterIP vs NodePort vs LoadBalancer
- 适用场景对比

## 实践任务

1. 创建 Pod 配置
2. 创建 Service 配置
3. 测试服务访问

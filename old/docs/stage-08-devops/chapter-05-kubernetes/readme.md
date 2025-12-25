# 8.5 Kubernetes (K8s)

## 目标

- 理解 Kubernetes 的基本概念。
- 掌握 Deployment、Service、Ingress 的配置。
- 了解 HPA（水平自动扩展）的使用。
- 熟悉 ConfigMap 和 Secret 的使用。

## 章节内容

本章分为四个独立小节，每节提供详细的概念解释、代码示例和最佳实践：

1. **[K8s 基础](section-01-k8s-basics.md)**：Kubernetes 概念、集群架构、核心组件、完整示例。

2. **[Pod 与 Service](section-02-pod-service.md)**：Pod 配置、Service 类型、服务发现、负载均衡、完整示例。

3. **[Deployment 与 Scaling](section-03-deployment-scaling.md)**：Deployment 配置、滚动更新、HPA、自动扩展、完整示例。

4. **[配置与存储](section-04-config-storage.md)**：ConfigMap、Secret、Volume、持久化存储、完整示例。

## 核心概念

- **Kubernetes**：容器编排平台
- **Pod**：最小部署单元
- **Service**：服务发现和负载均衡
- **Deployment**：应用部署管理

## 学习建议

1. **重点掌握**：
   - Kubernetes 基础概念
   - Deployment 配置
   - 服务发现

2. **实践练习**：
   - 完成每小节后的练习题目
   - 部署应用到 K8s
   - 配置自动扩展

## 完成本章后

- 能够使用 Kubernetes 部署应用。
- 理解 K8s 架构，能够配置资源。
- 掌握自动扩展，能够处理流量变化。
- 具备构建生产级 K8s 应用的能力。

## 相关章节

- **8.1 专业 Dockerfile**：容器镜像
- **8.4 部署选择**：部署方案

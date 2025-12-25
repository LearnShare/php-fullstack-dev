# 8.5.1 K8s 基础

## 概述

Kubernetes (K8s) 是容器编排的标准平台。本节介绍 Kubernetes 的基本概念、集群架构、核心组件。

## Kubernetes 概念

### 核心概念

- **Pod**：最小部署单元，包含一个或多个容器
- **Service**：服务发现和负载均衡
- **Deployment**：管理 Pod 的副本和更新
- **Namespace**：资源隔离和分组

### 集群架构

```
Master Node
  - API Server
  - etcd
  - Controller Manager
  - Scheduler

Worker Nodes
  - kubelet
  - kube-proxy
  - Container Runtime
```

## 核心组件

### API Server

Kubernetes 的入口，处理所有 API 请求。

### etcd

分布式键值存储，保存集群状态。

### Controller Manager

管理各种控制器，确保期望状态。

### Scheduler

调度 Pod 到合适的节点。

## 基础操作

### kubectl 命令

```bash
# 查看集群信息
kubectl cluster-info

# 查看节点
kubectl get nodes

# 查看 Pod
kubectl get pods

# 查看服务
kubectl get services

# 查看部署
kubectl get deployments
```

## 完整示例

```yaml
# 基础 Pod 定义
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    ports:
    - containerPort: 80
```

## 最佳实践

1. **理解概念**：深入理解 K8s 核心概念
2. **使用 YAML**：使用 YAML 定义资源
3. **命名空间**：合理使用命名空间
4. **资源限制**：设置资源限制

## 注意事项

1. K8s 学习曲线较陡
2. 需要理解分布式系统
3. 注意资源消耗
4. 定期更新集群

## 练习

1. 安装本地 K8s 集群（minikube 或 kind）。

2. 创建第一个 Pod。

3. 使用 kubectl 管理资源。

4. 理解 K8s 架构和组件。

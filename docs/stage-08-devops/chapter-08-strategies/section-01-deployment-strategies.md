# 8.8.1 部署策略

## 概述

部署策略决定了如何将新版本部署到生产环境。本节介绍部署策略对比、滚动更新、零停机部署等内容。

## 部署策略对比

### 滚动更新

逐步替换旧实例，保持服务可用。

```yaml
# Kubernetes 滚动更新
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### 蓝绿部署

维护两套环境，切换流量。

### 金丝雀发布

逐步将流量切换到新版本。

## 滚动更新

### Kubernetes 滚动更新

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 最多新增 1 个 Pod
      maxUnavailable: 0  # 最多不可用 0 个 Pod
  template:
    spec:
      containers:
      - name: app
        image: my-app:v2
```

## 零停机部署

### 实现方式

1. **健康检查**：确保新实例健康
2. **流量切换**：逐步切换流量
3. **回滚准备**：准备快速回滚

### 完整示例

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      containers:
      - name: app
        image: my-app:v2
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 最佳实践

1. **健康检查**：配置健康检查
2. **逐步更新**：逐步替换实例
3. **监控**：监控部署过程
4. **回滚**：准备快速回滚

## 注意事项

1. 测试部署策略
2. 注意数据库迁移
3. 监控部署过程
4. 准备回滚方案

## 练习

1. 实现滚动更新部署。

2. 配置零停机部署。

3. 实现自动回滚机制。

4. 测试部署策略。

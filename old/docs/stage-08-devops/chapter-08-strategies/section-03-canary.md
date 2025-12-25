# 8.8.3 金丝雀发布

## 概述

金丝雀发布是逐步将流量切换到新版本的部署策略。本节介绍金丝雀发布原理、流量分配、监控告警等内容。

## 金丝雀发布原理

### 工作原理

1. **部署新版本**：部署少量新版本实例
2. **分配流量**：将少量流量（如 10%）切换到新版本
3. **监控指标**：监控错误率、响应时间等
4. **逐步增加**：如无问题，逐步增加流量
5. **完全切换**：最终 100% 流量切换到新版本

### 流量分配

```
100% 流量
  ├─ 90% → 旧版本
  └─ 10% → 新版本（金丝雀）
```

## 实现方式

### Kubernetes + Istio

```yaml
# VirtualService
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
    - my-app
  http:
    - match:
        - headers:
            canary:
              exact: "true"
      route:
        - destination:
            host: my-app
            subset: v2
          weight: 100
    - route:
        - destination:
            host: my-app
            subset: v1
          weight: 90
        - destination:
            host: my-app
            subset: v2
          weight: 10
```

### Nginx 实现

```nginx
# nginx.conf
upstream backend {
    server app-v1:80 weight=90;
    server app-v2:80 weight=10;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

## 流量分配

### 逐步增加流量

```yaml
# 阶段 1: 10% 流量
weight: 10

# 阶段 2: 25% 流量
weight: 25

# 阶段 3: 50% 流量
weight: 50

# 阶段 4: 100% 流量
weight: 100
```

## 监控告警

### 关键指标

- **错误率**：新版本错误率
- **响应时间**：响应时间对比
- **吞吐量**：请求处理能力

### 告警配置

```yaml
# Prometheus 告警规则
groups:
  - name: canary
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        annotations:
          summary: "High error rate in canary deployment"
```

## 完整示例

```yaml
# 金丝雀部署配置
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
spec:
  replicas: 9
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: app
        image: my-app:v1

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: app
        image: my-app:v2

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
```

## 最佳实践

1. **逐步增加**：逐步增加流量比例
2. **监控指标**：监控关键指标
3. **快速回滚**：准备快速回滚
4. **自动化**：自动化流量切换

## 注意事项

1. 监控新版本表现
2. 设置合理的流量比例
3. 准备回滚方案
4. 测试金丝雀流程

## 练习

1. 实现金丝雀发布架构。

2. 配置流量分配，逐步增加流量。

3. 设置监控告警。

4. 实现自动回滚机制。

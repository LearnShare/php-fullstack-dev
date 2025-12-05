# 8.5.2 Pod 与 Service

## 概述

Pod 和 Service 是 Kubernetes 的基础资源。本节介绍 Pod 配置、Service 类型、服务发现、负载均衡等内容。

## Pod 配置

### 基础 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    ports:
    - containerPort: 80
    env:
    - name: APP_ENV
      value: "production"
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 多容器 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
  - name: sidecar
    image: sidecar:latest
```

## Service 类型

### ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
```

### NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

### LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
```

## 服务发现

### DNS 服务发现

```yaml
# Pod 可以通过服务名访问
# my-app-service.default.svc.cluster.local
```

### 环境变量

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    env:
    - name: DB_HOST
      value: "db-service"
```

## 完整示例

```yaml
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    ports:
    - containerPort: 80

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

## 最佳实践

1. **使用标签**：合理使用标签选择器
2. **服务类型**：根据需求选择服务类型
3. **健康检查**：配置就绪和存活探针
4. **资源限制**：设置资源请求和限制

## 注意事项

1. Pod 是临时资源
2. Service 提供稳定访问
3. 注意端口映射
4. 使用标签选择器

## 练习

1. 创建一个 Pod 和对应的 Service。

2. 配置不同 Service 类型，测试访问。

3. 实现服务发现。

4. 配置负载均衡。

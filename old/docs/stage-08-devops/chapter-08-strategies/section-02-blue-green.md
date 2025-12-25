# 8.8.2 蓝绿部署

## 概述

蓝绿部署维护两套完全相同的环境，通过切换流量实现零停机部署。本节介绍蓝绿部署原理、实现方式、流量切换、回滚机制。

## 蓝绿部署原理

### 工作原理

1. **蓝色环境**：当前生产环境
2. **绿色环境**：新版本环境
3. **流量切换**：从蓝色切换到绿色
4. **回滚**：如有问题，切回蓝色

### 架构图

```
负载均衡器
    ↓
  [蓝色环境] ← 当前流量
  [绿色环境] ← 新版本（待切换）
```

## 实现方式

### Kubernetes 实现

```yaml
# 蓝色环境（当前）
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: app
        image: my-app:v1

---
# 绿色环境（新版本）
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: app
        image: my-app:v2

---
# Service（切换版本标签）
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue  # 切换到 green 即可切换环境
  ports:
  - port: 80
```

### 流量切换脚本

```bash
#!/bin/bash
# switch-blue-green.sh

CURRENT_VERSION=$(kubectl get svc my-app-service -o jsonpath='{.spec.selector.version}')

if [ "$CURRENT_VERSION" == "blue" ]; then
    NEW_VERSION="green"
else
    NEW_VERSION="blue"
fi

echo "Switching from $CURRENT_VERSION to $NEW_VERSION"

# 更新 Service 选择器
kubectl patch service my-app-service -p "{\"spec\":{\"selector\":{\"version\":\"$NEW_VERSION\"}}}"

# 等待新环境就绪
kubectl wait --for=condition=ready pod -l app=my-app,version=$NEW_VERSION --timeout=300s

echo "Switched to $NEW_VERSION"
```

## 完整示例

```yaml
# 蓝绿部署完整配置
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: app
        image: my-app:v1
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /health
            port: 80
        readinessProbe:
          httpGet:
            path: /ready
            port: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: app
        image: my-app:v2
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /health
            port: 80
        readinessProbe:
          httpGet:
            path: /ready
            port: 80

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue  # 默认指向蓝色
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

## 最佳实践

1. **环境一致性**：确保蓝绿环境一致
2. **数据库迁移**：谨慎处理数据库迁移
3. **监控**：监控两个环境
4. **快速切换**：实现快速切换机制

## 注意事项

1. 需要双倍资源
2. 数据库迁移需要特别处理
3. 确保环境一致性
4. 测试切换流程

## 练习

1. 实现蓝绿部署架构。

2. 创建流量切换脚本。

3. 测试蓝绿切换流程。

4. 实现自动回滚机制。

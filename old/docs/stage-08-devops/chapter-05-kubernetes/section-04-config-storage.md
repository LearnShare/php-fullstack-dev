# 8.5.4 配置与存储

## 概述

Kubernetes 提供了 ConfigMap 和 Secret 管理配置，以及 Volume 管理存储。本节介绍 ConfigMap、Secret、Volume、持久化存储等内容。

## ConfigMap

### 创建 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_DEBUG: "false"
  database.yml: |
    host: db
    port: 3306
```

### 使用 ConfigMap

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

## Secret

### 创建 Secret

```bash
# 命令行创建
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### Secret YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 编码
  password: c2VjcmV0MTIz
```

### 使用 Secret

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

## Volume

### 临时卷

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

### 持久化卷

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    volumeMounts:
    - name: storage
      mountPath: /var/www/html/storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: app-storage
```

## 完整示例

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  db-password: cGFzc3dvcmQxMjM=

---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

---
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    envFrom:
    - configMapRef:
        name: app-config
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: db-password
    volumeMounts:
    - name: storage
      mountPath: /var/www/html/storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: app-storage
```

## 最佳实践

1. **配置分离**：使用 ConfigMap 管理配置
2. **敏感信息**：使用 Secret 管理敏感信息
3. **持久化存储**：重要数据使用持久化存储
4. **加密传输**：Secret 传输时加密

## 注意事项

1. Secret 不是完全加密的
2. ConfigMap 有大小限制
3. 注意存储类型和访问模式
4. 定期轮换 Secret

## 练习

1. 创建 ConfigMap，管理应用配置。

2. 创建 Secret，管理数据库密码。

3. 配置持久化存储，保存应用数据。

4. 在 Pod 中使用 ConfigMap 和 Secret。

# 8.5 Kubernetes (K8s)

## 目标

- 理解 Kubernetes 的基本概念。
- 掌握 Deployment、Service、Ingress 的配置。
- 了解 HPA（水平自动扩展）的使用。

## 基础概念

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: php-app
  template:
    metadata:
      labels:
        app: php-app
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 9000
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: php-app-service
spec:
  selector:
    app: php-app
  ports:
  - port: 80
    targetPort: 9000
  type: LoadBalancer
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: php-app-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: php-app-service
            port:
              number: 80
```

## 练习

1. 创建 Kubernetes 部署配置。

2. 配置 Service 和 Ingress。

3. 实现 HPA 自动扩展。

4. 配置 ConfigMap 和 Secrets。

5. 实现滚动更新策略。

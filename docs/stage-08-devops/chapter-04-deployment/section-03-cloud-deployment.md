# 8.4.3 云平台部署

## 概述

云平台部署提供了弹性和可扩展性。本节介绍 AWS ECS、Google Cloud Run、Azure Container Apps 等云平台部署方案。

## AWS ECS

### ECS 任务定义

```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "my-app",
      "image": "my-app:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "APP_ENV",
          "value": "production"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-east-1"
        }
      }
    }
  ]
}
```

### ECS 服务配置

```json
{
  "serviceName": "my-app-service",
  "cluster": "my-cluster",
  "taskDefinition": "my-app",
  "desiredCount": 2,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-xxx"],
      "securityGroups": ["sg-xxx"],
      "assignPublicIp": "ENABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:...",
      "containerName": "my-app",
      "containerPort": 80
    }
  ]
}
```

## Google Cloud Run

### Cloud Run 配置

```yaml
# cloud-run.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "10"
        run.googleapis.com/cpu-throttling: "true"
    spec:
      containerConcurrency: 80
      containers:
      - image: gcr.io/my-project/my-app:latest
        ports:
        - containerPort: 8080
        env:
        - name: APP_ENV
          value: "production"
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
```

### 部署命令

```bash
# 构建镜像
gcloud builds submit --tag gcr.io/my-project/my-app

# 部署到 Cloud Run
gcloud run deploy my-app \
  --image gcr.io/my-project/my-app \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

## Azure Container Apps

### 容器应用配置

```yaml
# container-app.yaml
apiVersion: 2022-03-01
kind: ContainerApp
properties:
  configuration:
    ingress:
      external: true
      targetPort: 80
  template:
    containers:
    - name: my-app
      image: myregistry.azurecr.io/my-app:latest
      env:
      - name: APP_ENV
        value: "production"
    scale:
      minReplicas: 1
      maxReplicas: 10
```

## 完整示例

```bash
#!/bin/bash
# 云平台部署脚本

# AWS ECS 部署
deploy_aws_ecs() {
    # 构建镜像
    docker build -t my-app:latest .
    docker tag my-app:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
    
    # 推送镜像
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
    docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
    
    # 更新服务
    aws ecs update-service --cluster my-cluster --service my-app-service --force-new-deployment
}

# Google Cloud Run 部署
deploy_gcp() {
    # 构建镜像
    gcloud builds submit --tag gcr.io/my-project/my-app
    
    # 部署
    gcloud run deploy my-app \
        --image gcr.io/my-project/my-app \
        --platform managed \
        --region us-central1
}

# Azure Container Apps 部署
deploy_azure() {
    # 构建镜像
    az acr build --registry myregistry --image my-app:latest .
    
    # 部署
    az containerapp update \
        --name my-app \
        --resource-group my-resource-group \
        --image myregistry.azurecr.io/my-app:latest
}
```

## 最佳实践

1. **使用托管服务**：利用云平台托管服务
2. **自动扩展**：配置自动扩展
3. **健康检查**：配置健康检查
4. **日志监控**：集成日志和监控

## 注意事项

1. 注意云平台成本
2. 配置安全组和网络
3. 使用密钥管理服务
4. 监控资源使用

## 练习

1. 在 AWS ECS 上部署应用。

2. 使用 Google Cloud Run 部署应用。

3. 配置自动扩展和健康检查。

4. 实现多区域部署。

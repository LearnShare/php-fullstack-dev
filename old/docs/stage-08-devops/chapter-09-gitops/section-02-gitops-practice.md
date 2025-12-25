# 8.9.2 GitOps 实践

## 概述

GitOps 实践需要合适的工具和流程。本节介绍 ArgoCD 使用、应用定义、同步策略、基础设施即代码等内容。

## ArgoCD 使用

### 安装 ArgoCD

```bash
# 安装 ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 获取管理员密码
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 创建应用

```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## 应用定义

### 目录结构

```
repo/
  k8s/
    app/
      deployment.yaml
      service.yaml
      configmap.yaml
    base/
      kustomization.yaml
```

### Kustomize 配置

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml

images:
  - name: my-app
    newTag: v1.2.3
```

## 同步策略

### 自动同步

```yaml
syncPolicy:
  automated:
    prune: true      # 删除 Git 中不存在的资源
    selfHeal: true   # 自动修复集群中的变更
```

### 手动同步

```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
```

## 基础设施即代码

### Terraform 集成

```hcl
# terraform/main.tf
resource "kubernetes_deployment" "app" {
  metadata {
    name = "my-app"
  }
  spec {
    replicas = 3
    template {
      spec {
        container {
          name  = "app"
          image = "my-app:latest"
        }
      }
    }
  }
}
```

## 完整示例

```yaml
# GitOps 完整实践
# 1. 应用定义
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo
    targetRevision: main
    path: k8s/app
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
```

## 最佳实践

1. **环境分离**：不同环境使用不同分支或路径
2. **审查流程**：使用 Pull Request 审查
3. **自动化**：自动化同步和修复
4. **监控**：监控同步状态

## 注意事项

1. 保护 Git 仓库访问
2. 测试配置变更
3. 监控同步状态
4. 准备回滚方案

## 练习

1. 安装和配置 ArgoCD。

2. 创建 GitOps 应用定义。

3. 配置自动同步策略。

4. 实现基础设施即代码。

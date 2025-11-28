# 8.9 GitOps

## 目标

- 理解 GitOps 的概念与优势。
- 了解 ArgoCD 的使用。
- 掌握基础设施即代码（IaC）。

## GitOps 概念

- 使用 Git 作为单一事实来源。
- 自动同步 Git 仓库与集群状态。
- 版本控制和审计。

## ArgoCD

### 应用定义

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 练习

1. 配置 ArgoCD 实现 GitOps 工作流。

2. 创建应用定义和同步策略。

3. 实现自动部署和回滚。

4. 设计多环境 GitOps 流程。

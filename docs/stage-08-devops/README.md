# 阶段八：部署、云原生与 DevOps

本阶段深入讲解生产环境的部署、容器化、云原生和 DevOps 实践。从 Docker 基础开始，逐步掌握 Dockerfile 优化、docker-compose 编排、Kubernetes 部署、Serverless 应用、CI/CD 流程等现代部署技术。每章提供完整示例、最佳实践与生产级配置，帮助你构建可扩展、可维护的部署体系。

1. **[专业 Dockerfile](chapter-01-dockerfile/README.md)**：专业 Dockerfile。
   - [8.1.1 Dockerfile 基础](chapter-01-dockerfile/section-01-dockerfile-basics.md)
   - [8.1.2 多阶段构建](chapter-01-dockerfile/section-02-multi-stage.md)
   - [8.1.3 优化技巧](chapter-01-dockerfile/section-03-optimization.md)
2. **[docker-compose 与本地开发环境](chapter-02-docker-compose/README.md)**：docker-compose 与本地开发环境。
   - [8.2.1 docker-compose 基础](chapter-02-docker-compose/section-01-compose-basics.md)
   - [8.2.2 本地开发环境](chapter-02-docker-compose/section-02-dev-environment.md)
3. **[镜像安全](chapter-03-security/README.md)**：镜像安全。
   - [8.3.1 镜像安全](chapter-03-security/section-01-image-security.md)
   - [8.3.2 容器安全](chapter-03-security/section-02-container-security.md)
4. **[部署选择](chapter-04-deployment/README.md)**：部署选择。
   - [8.4.1 部署选择](chapter-04-deployment/section-01-deployment-options.md)
   - [8.4.2 传统部署](chapter-04-deployment/section-02-traditional-deployment.md)
   - [8.4.3 云平台部署](chapter-04-deployment/section-03-cloud-deployment.md)
5. **[Kubernetes (K8s)](chapter-05-kubernetes/README.md)**：Kubernetes (K8s)。
   - [8.5.1 K8s 基础](chapter-05-kubernetes/section-01-k8s-basics.md)
   - [8.5.2 Pod 与 Service](chapter-05-kubernetes/section-02-pod-service.md)
   - [8.5.3 Deployment 与 Scaling](chapter-05-kubernetes/section-03-deployment-scaling.md)
   - [8.5.4 配置与存储](chapter-05-kubernetes/section-04-config-storage.md)
6. **[Serverless PHP](chapter-06-serverless/README.md)**：Serverless PHP。
   - [8.6.1 Serverless 概念](chapter-06-serverless/section-01-serverless-concept.md)
   - [8.6.2 PHP Serverless 实践](chapter-06-serverless/section-02-php-serverless.md)
7. **[CI/CD 与 GitHub Actions](chapter-07-cicd/README.md)**：CI/CD 与 GitHub Actions。
   - [8.7.1 CI/CD 基础](chapter-07-cicd/section-01-cicd-basics.md)
   - [8.7.2 GitHub Actions](chapter-07-cicd/section-02-github-actions.md)
   - [8.7.3 GitLab CI](chapter-07-cicd/section-03-gitlab-ci.md)
8. **[Deploy 策略](chapter-08-strategies/README.md)**：Deploy 策略。
   - [8.8.1 部署策略](chapter-08-strategies/section-01-deployment-strategies.md)
   - [8.8.2 蓝绿部署](chapter-08-strategies/section-02-blue-green.md)
   - [8.8.3 金丝雀发布](chapter-08-strategies/section-03-canary.md)
9. **[GitOps](chapter-09-gitops/README.md)**：GitOps。
   - [8.9.1 GitOps 概念](chapter-09-gitops/section-01-gitops-concept.md)
   - [8.9.2 GitOps 实践](chapter-09-gitops/section-02-gitops-practice.md)

完成阶段八后，你将具备：

- 能够编写优化的 Dockerfile，构建高效的容器镜像。
- 掌握 docker-compose 的使用，搭建完整的开发环境。
- 理解容器安全，能够扫描和加固镜像。
- 熟悉各种部署方案（ECS、Kubernetes、Serverless）。
- 建立完整的 CI/CD 流程，实现自动化部署。
- 掌握蓝绿部署、金丝雀部署等高级部署策略。
- 了解 GitOps 工作流，实现基础设施即代码。

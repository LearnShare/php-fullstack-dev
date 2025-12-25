# 阶段八：部署、云原生与 DevOps

本阶段深入讲解生产环境的部署、容器化、云原生和 DevOps 实践。从 Docker 基础开始，逐步掌握 Dockerfile 优化、docker-compose 编排、Kubernetes 部署、Serverless 应用、CI/CD 流程等现代部署技术。每章提供完整示例、最佳实践与生产级配置，帮助你构建可扩展、可维护的部署体系。

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握

- **阶段一至阶段七**：所有前面阶段的知识
- **Linux 基础**：基本的 Linux 命令操作（阶段一已涉及）
- **Git**：版本控制基础（阶段一）
- **应用开发**：能够开发完整的 Web 应用（阶段四至阶段七）

### 特别重要

- **应用部署经验**：理解应用部署的基本流程
- **服务器操作**：能够操作 Linux 服务器
- **网络基础**：理解基本的网络概念（端口、域名等）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **基础检查**
   - [ ] 能够开发完整的 Web 应用
   - [ ] 能够使用 Git 进行版本控制
   - [ ] 理解应用部署的基本流程
   - [ ] 能够操作 Linux 服务器（基本命令）

2. **理解检查**
   - [ ] 理解容器化的概念和好处
   - [ ] 理解 CI/CD 的基本概念
   - [ ] 理解云原生应用的特点

3. **实践检查**
   - [ ] 完成阶段七的所有练习
   - [ ] 能够将应用部署到服务器（传统方式）

**如果以上检查点都通过，可以开始阶段八的学习。**

**如果某些检查点未通过，建议：**
- 复习阶段一的 Linux 和 Git 基础
- 确保有应用部署的实践经验
- 学习基本的 Linux 服务器操作

## 学习时间估算

**建议学习时间：** 4-6 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含练习和实践时间

**时间分配建议：**
- Docker 基础（8.1-8.3）：1-2 周
- 部署方案（8.4-8.6）：1-2 周
- CI/CD（8.7-8.9）：1-2 周
- 实践项目：1 周

## 练习建议

### 基础练习（每章完成后）

1. **Docker 基础**
   - 编写 Dockerfile 构建镜像
   - 使用 docker-compose 编排服务
   - 优化镜像大小和构建速度

2. **部署实践**
   - 将应用部署到云平台
   - 配置域名和 SSL 证书
   - 实现应用监控

3. **CI/CD**
   - 配置 GitHub Actions
   - 实现自动化测试和部署
   - 配置部署策略

### 进阶练习（阶段中）

4. **Kubernetes**
   - 部署应用到 Kubernetes
   - 配置 Service 和 Ingress
   - 实现自动扩缩容

5. **高级部署**
   - 实现蓝绿部署
   - 实现金丝雀发布
   - 配置 GitOps 工作流

### 综合项目（阶段完成后）

6. **生产级部署**
   - 将完整应用容器化
   - 建立完整的 CI/CD 流程
   - 部署到生产环境
   - 实现监控和告警

**项目要求：**
- 使用 Docker 容器化应用
- 建立完整的 CI/CD 流程
- 部署到生产环境（云平台或 Kubernetes）
- 实现监控、日志、告警
- 支持自动扩缩容和滚动更新

## 章节内容

1. **[专业 Dockerfile](chapter-01-dockerfile/readme.md)**：专业 Dockerfile。
   - [8.1.1 Dockerfile 基础](chapter-01-dockerfile/section-01-dockerfile-basics.md)
   - [8.1.2 多阶段构建](chapter-01-dockerfile/section-02-multi-stage.md)
   - [8.1.3 优化技巧](chapter-01-dockerfile/section-03-optimization.md)
2. **[docker-compose 与本地开发环境](chapter-02-docker-compose/readme.md)**：docker-compose 与本地开发环境。
   - [8.2.1 docker-compose 基础](chapter-02-docker-compose/section-01-compose-basics.md)
   - [8.2.2 本地开发环境](chapter-02-docker-compose/section-02-dev-environment.md)
3. **[镜像安全](chapter-03-security/readme.md)**：镜像安全。
   - [8.3.1 镜像安全](chapter-03-security/section-01-image-security.md)
   - [8.3.2 容器安全](chapter-03-security/section-02-container-security.md)
4. **[部署选择](chapter-04-deployment/readme.md)**：部署选择。
   - [8.4.1 部署选择](chapter-04-deployment/section-01-deployment-options.md)
   - [8.4.2 传统部署](chapter-04-deployment/section-02-traditional-deployment.md)
   - [8.4.3 云平台部署](chapter-04-deployment/section-03-cloud-deployment.md)
5. **[Kubernetes (K8s)](chapter-05-kubernetes/readme.md)**：Kubernetes (K8s)。
   - [8.5.1 K8s 基础](chapter-05-kubernetes/section-01-k8s-basics.md)
   - [8.5.2 Pod 与 Service](chapter-05-kubernetes/section-02-pod-service.md)
   - [8.5.3 Deployment 与 Scaling](chapter-05-kubernetes/section-03-deployment-scaling.md)
   - [8.5.4 配置与存储](chapter-05-kubernetes/section-04-config-storage.md)
6. **[Serverless PHP](chapter-06-serverless/readme.md)**：Serverless PHP。
   - [8.6.1 Serverless 概念](chapter-06-serverless/section-01-serverless-concept.md)
   - [8.6.2 PHP Serverless 实践](chapter-06-serverless/section-02-php-serverless.md)
7. **[CI/CD 与 GitHub Actions](chapter-07-cicd/readme.md)**：CI/CD 与 GitHub Actions。
   - [8.7.1 CI/CD 基础](chapter-07-cicd/section-01-cicd-basics.md)
   - [8.7.2 GitHub Actions](chapter-07-cicd/section-02-github-actions.md)
   - [8.7.3 GitLab CI](chapter-07-cicd/section-03-gitlab-ci.md)
8. **[Deploy 策略](chapter-08-strategies/readme.md)**：Deploy 策略。
   - [8.8.1 部署策略](chapter-08-strategies/section-01-deployment-strategies.md)
   - [8.8.2 蓝绿部署](chapter-08-strategies/section-02-blue-green.md)
   - [8.8.3 金丝雀发布](chapter-08-strategies/section-03-canary.md)
9. **[GitOps](chapter-09-gitops/readme.md)**：GitOps。
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

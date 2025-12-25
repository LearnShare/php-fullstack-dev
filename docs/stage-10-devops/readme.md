# 阶段十：部署、云原生与 DevOps（DevOps）

本阶段深入讲解 PHP 应用的部署、容器化、CI/CD、监控等 DevOps 相关技能。从 Dockerfile 开始，逐步掌握 docker-compose、镜像安全、部署选择、Kubernetes、Serverless、CI/CD、部署策略、GitOps、基础设施即代码、回滚与灾难恢复、监控与告警、API 网关、负载均衡、CDN 等。每章提供完整示例、最佳实践与常见陷阱说明，帮助你掌握现代 DevOps 实践。

## 定位

部署、容器、CI/CD、监控等

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握（阶段一、阶段二、阶段三、阶段五、阶段六、阶段九）

- **阶段一**：环境搭建、工具配置、Docker Compose 基础
- **阶段二**：PHP 语言基础
- **阶段三**：面向对象编程
- **阶段五**：Web 开发基础
- **阶段六**：数据库操作
- **阶段九**：框架应用

### 不需要掌握

以下知识**不需要**提前掌握，会在学习过程中逐步学习：

- Docker 高级应用（从零开始学习）
- Kubernetes（从零开始学习）
- CI/CD（从零开始学习）
- 云原生（从零开始学习）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **基础检查**
   - [ ] 能够使用框架开发应用
   - [ ] 理解 Docker 基本概念
   - [ ] 能够使用 Git 进行版本控制

2. **理解检查**
   - [ ] 理解应用部署的基本概念
   - [ ] 能够阅读和理解配置文件
   - [ ] 理解 CI/CD 的基本概念

3. **实践检查**
   - [ ] 完成阶段九的所有练习
   - [ ] 能够独立开发完整的应用

**如果以上检查点都通过，可以开始阶段十的学习。**

**如果某些检查点未通过，建议：**
- 复习阶段一的相关章节（特别是 Docker Compose）
- 完成阶段九的练习
- 确保理解框架应用后再继续

## 学习时间估算

**建议学习时间：** 6-8 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含练习和实践时间

**时间分配建议：**
- Docker 与容器（10.1-10.3）：1-2 周
- 部署选择（10.4）：1 周
- Kubernetes（10.5）：1-2 周
- Serverless（10.6）：1 周
- CI/CD（10.7-10.9）：1-2 周
- 基础设施即代码（10.10）：1 周
- 回滚与灾难恢复（10.11）：1 周
- 监控与告警（10.12）：1 周
- API 网关与负载均衡（10.13-10.14）：1 周
- CDN（10.15）：1 周
- 阶段总结与实践：1 周

## 练习建议

### 基础练习（每章完成后）

每完成一章，建议完成 3-5 个小练习，例如：

1. **Docker**
   - 编写优化的 Dockerfile
   - 配置 docker-compose
   - 实现多阶段构建

2. **CI/CD**
   - 配置 GitHub Actions
   - 实现自动化部署
   - 实现自动化测试

3. **Kubernetes**
   - 部署应用到 K8s
   - 配置 Service 和 Deployment
   - 实现自动扩缩容

### 综合练习（阶段完成后）

4. **综合项目**
   - 实现完整的 DevOps 流程
   - 部署应用到生产环境
   - 实现监控和告警
   - 实现自动化运维

**项目要求：**
- 使用本阶段学到的所有知识点
- 代码规范、注释清晰
- 完善的 CI/CD 流程
- 生产环境最佳实践

## 章节内容

1. **[10.1 专业 Dockerfile](chapter-01-dockerfile/readme.md)**：Dockerfile 基础、多阶段构建、优化技巧。
   - [10.1.1 Dockerfile 基础](chapter-01-dockerfile/section-01-dockerfile-basics.md)
   - [10.1.2 多阶段构建](chapter-01-dockerfile/section-02-multi-stage.md)
   - [10.1.3 优化技巧](chapter-01-dockerfile/section-03-optimization.md)

2. **[10.2 docker-compose 与本地开发环境](chapter-02-docker-compose/readme.md)**：docker-compose 基础、本地开发环境。
   - [10.2.1 docker-compose 基础](chapter-02-docker-compose/section-01-compose-basics.md)
   - [10.2.2 本地开发环境](chapter-02-docker-compose/section-02-dev-environment.md)

3. **[10.3 镜像安全](chapter-03-security/readme.md)**：镜像安全、容器安全。
   - [10.3.1 镜像安全](chapter-03-security/section-01-image-security.md)
   - [10.3.2 容器安全](chapter-03-security/section-02-container-security.md)

4. **[10.4 部署选择](chapter-04-deployment/readme.md)**：部署选择、传统部署、云平台部署。
   - [10.4.1 部署选择](chapter-04-deployment/section-01-deployment-options.md)
   - [10.4.2 传统部署](chapter-04-deployment/section-02-traditional-deployment.md)
   - [10.4.3 云平台部署](chapter-04-deployment/section-03-cloud-deployment.md)

5. **[10.5 Kubernetes (K8s)](chapter-05-kubernetes/readme.md)**：K8s 基础、Pod 与 Service、Deployment 与 Scaling、配置与存储。
   - [10.5.1 K8s 基础](chapter-05-kubernetes/section-01-k8s-basics.md)
   - [10.5.2 Pod 与 Service](chapter-05-kubernetes/section-02-pod-service.md)
   - [10.5.3 Deployment 与 Scaling](chapter-05-kubernetes/section-03-deployment-scaling.md)
   - [10.5.4 配置与存储](chapter-05-kubernetes/section-04-config-storage.md)

6. **[10.6 Serverless PHP](chapter-06-serverless/readme.md)**：Serverless 概念、PHP Serverless 实践。
   - [10.6.1 Serverless 概念](chapter-06-serverless/section-01-serverless-concept.md)
   - [10.6.2 PHP Serverless 实践](chapter-06-serverless/section-02-php-serverless.md)

7. **[10.7 CI/CD 与 GitHub Actions](chapter-07-cicd/readme.md)**：CI/CD 基础、GitHub Actions、GitLab CI。
   - [10.7.1 CI/CD 基础](chapter-07-cicd/section-01-cicd-basics.md)
   - [10.7.2 GitHub Actions](chapter-07-cicd/section-02-github-actions.md)
   - [10.7.3 GitLab CI](chapter-07-cicd/section-03-gitlab-ci.md)

8. **[10.8 Deploy 策略](chapter-08-strategies/readme.md)**：部署策略、蓝绿部署、金丝雀发布。
   - [10.8.1 部署策略](chapter-08-strategies/section-01-deployment-strategies.md)
   - [10.8.2 蓝绿部署](chapter-08-strategies/section-02-blue-green.md)
   - [10.8.3 金丝雀发布](chapter-08-strategies/section-03-canary.md)

9. **[10.9 GitOps](chapter-09-gitops/readme.md)**：GitOps 概念、GitOps 实践。
   - [10.9.1 GitOps 概念](chapter-09-gitops/section-01-gitops-concept.md)
   - [10.9.2 GitOps 实践](chapter-09-gitops/section-02-gitops-practice.md)

10. **[10.10 基础设施即代码（IaC）](chapter-10-iac/readme.md)**：基础设施即代码概述、Terraform、Pulumi。
    - [10.10.1 基础设施即代码概述](chapter-10-iac/section-01-overview.md)
    - [10.10.2 Terraform](chapter-10-iac/section-02-terraform.md)
    - [10.10.3 Pulumi](chapter-10-iac/section-03-pulumi.md)

11. **[10.11 回滚与灾难恢复](chapter-11-rollback/readme.md)**：回滚策略、灾难恢复、备份与恢复流程。
    - [10.11.1 回滚策略](chapter-11-rollback/section-01-rollback.md)
    - [10.11.2 灾难恢复](chapter-11-rollback/section-02-disaster-recovery.md)
    - [10.11.3 备份与恢复流程](chapter-11-rollback/section-03-backup-restore.md)

12. **[10.12 监控与告警](chapter-12-monitoring/readme.md)**：监控与告警概述、应用监控、基础设施监控、告警系统。
    - [10.12.1 监控与告警概述](chapter-12-monitoring/section-01-overview.md)
    - [10.12.2 应用监控](chapter-12-monitoring/section-02-application.md)
    - [10.12.3 基础设施监控](chapter-12-monitoring/section-03-infrastructure.md)
    - [10.12.4 告警系统](chapter-12-monitoring/section-04-alerts.md)

13. **[10.13 API 网关](chapter-13-api-gateway/readme.md)**：API 网关概述、Kong、Traefik。
    - [10.13.1 API 网关概述](chapter-13-api-gateway/section-01-overview.md)
    - [10.13.2 Kong](chapter-13-api-gateway/section-02-kong.md)
    - [10.13.3 Traefik](chapter-13-api-gateway/section-03-traefik.md)

14. **[10.14 负载均衡](chapter-14-load-balancing/readme.md)**：负载均衡概述、负载均衡算法、Nginx 负载均衡、HAProxy。
    - [10.14.1 负载均衡概述](chapter-14-load-balancing/section-01-overview.md)
    - [10.14.2 负载均衡算法](chapter-14-load-balancing/section-02-algorithms.md)
    - [10.14.3 Nginx 负载均衡](chapter-14-load-balancing/section-03-nginx.md)
    - [10.14.4 HAProxy](chapter-14-load-balancing/section-04-haproxy.md)

15. **[10.15 CDN 深入](chapter-15-cdn/readme.md)**：CDN 原理与架构、CDN 缓存策略、CDN 性能优化。
    - [10.15.1 CDN 原理与架构](chapter-15-cdn/section-01-principles.md)
    - [10.15.2 CDN 缓存策略](chapter-15-cdn/section-02-cache-strategies.md)
    - [10.15.3 CDN 性能优化](chapter-15-cdn/section-03-optimization.md)

## 完成本阶段后，你将具备：

- 扎实的 DevOps 能力，能够部署和管理应用
- 掌握 Docker 和容器化技术，能够编写优化的 Dockerfile
- 理解 Kubernetes，能够部署应用到 K8s
- 掌握 CI/CD 流程，能够实现自动化部署
- 掌握部署策略，能够实现蓝绿部署和金丝雀发布
- 掌握基础设施即代码，能够使用 Terraform 或 Pulumi
- 建立监控和告警体系，能够监控和诊断应用问题
- 掌握 API 网关和负载均衡，能够构建高可用系统

## 相关章节

- **阶段一：基础入门**：Docker Compose 基础
- **阶段五：Web/API 开发**：Web 开发基础
- **阶段八：性能优化与安全**：性能优化和监控
- **阶段九：现代框架深度应用**：框架应用

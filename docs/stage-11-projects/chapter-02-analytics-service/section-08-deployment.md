# 11.2.8 部署与运维

## 概述

部署与运维是项目上线的关键步骤。本节介绍网站访问统计服务的部署与运维，包括部署流程、高可用架构、监控告警、数据备份等，帮助零基础学员理解如何部署和运维生产环境。

**章节类型**：实践性章节

**主要内容**：
- 部署概述
- 部署架构
- Docker 部署
- Kubernetes 部署
- 监控告警
- 数据备份
- 日志管理
- 部署流程
- 完整示例

## 核心内容

### 部署概述

- 部署的作用
- 部署方案：Docker Compose、Kubernetes、传统部署
- 高可用设计

### 部署架构

#### 高可用架构

- 架构图：CDN → Load Balancer → API Servers → Databases
- 服务部署：API 服务（多实例）、MySQL（主从）、Redis（集群）、RabbitMQ（集群）、InfluxDB（集群）
- 负载均衡：Nginx、HAProxy

### Docker 部署

#### Dockerfile

- PHP API 服务 Dockerfile
- 多阶段构建
- 优化技巧

#### docker-compose.yml

- 服务定义：nginx、api、mysql、redis、rabbitmq、influxdb
- 网络配置
- 数据卷配置

### Kubernetes 部署

#### Deployment

- API 服务 Deployment 配置
- 资源限制（CPU、内存）
- 健康检查（liveness、readiness）

#### Service

- Service 配置
- LoadBalancer 类型

#### HorizontalPodAutoscaler

- HPA 配置
- 自动扩缩容规则（CPU、内存）

### 监控告警

#### 监控指标

- 应用监控：请求量、响应时间、错误率
- 系统监控：CPU、内存、磁盘、网络
- 数据库监控：连接数、查询性能、慢查询
- 队列监控：队列长度、处理速度、失败率

#### Prometheus 配置

- 监控配置
- 指标收集

#### Grafana 仪表板

- 仪表板配置
- 图表展示

#### 告警规则

- 告警规则定义
- 告警通知

### 数据备份

#### MySQL 备份

- 备份脚本
- 自动备份（CronJob）
- 备份保留策略

#### InfluxDB 备份

- 备份脚本
- 自动备份
- 备份保留策略

### 日志管理

#### 日志收集

- Filebeat 配置
- 日志聚合

#### 日志分析

- ELK Stack
- Grafana Loki

### 部署流程

#### CI/CD 流程

- GitHub Actions / GitLab CI 配置
- 构建镜像
- 推送到镜像仓库
- 部署到 Kubernetes

## 基本用法

### 部署流程

- 准备环境
- 构建镜像
- 部署服务
- 监控运维

## 完整示例

### 部署脚本

- Docker Compose 部署脚本
- Kubernetes 部署配置
- CI/CD 配置文件

## 注意事项

- 高可用设计
- 性能优化
- 安全加固
- 监控告警

## 常见问题

**Q: 如何实现零停机部署？**

A: 使用蓝绿部署或金丝雀部署，逐步切换流量，实现零停机。

**Q: 如何处理数据库迁移？**

A: 使用数据库迁移工具（如 Laravel Migrations），在部署前执行迁移。

**Q: 如何实现自动扩缩容？**

A: 使用 Kubernetes HPA 或云服务的自动扩缩容功能，根据负载自动调整实例数量。

## 最佳实践

- 容器化部署
- 自动化部署
- 监控告警
- 数据备份

## 对比分析

### 部署方案对比

- Docker Compose vs Kubernetes vs 传统部署

## 实践任务

1. 部署练习：使用 Docker Compose 部署服务，使用 Kubernetes 部署服务，配置监控告警
2. 运维实践：配置自动备份，设置日志收集，进行性能优化
3. 故障处理：模拟故障场景，练习故障排查，优化故障处理流程

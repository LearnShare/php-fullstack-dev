# 10.2.1 docker-compose 基础

## 概述

docker-compose 是管理多容器应用的工具。本节介绍 docker-compose 的基础概念、配置文件、常用命令、服务编排，帮助零基础学员理解如何使用 docker-compose。

**章节类型**：工具性章节

**主要内容**：
- docker-compose 概述
- 配置文件
- 常用命令
- 服务编排
- 完整示例

## 核心内容

### docker-compose 概述

- docker-compose 的作用
- 工作原理
- 优势特点

### 配置文件

- docker-compose.yml
- 服务定义
- 网络配置
- 卷配置

### 常用命令

- up/down
- start/stop
- ps/logs
- exec/build

### 服务编排

- 服务依赖
- 服务启动顺序
- 服务通信
- 服务扩展

## 基本用法

### docker-compose 示例

```yaml
# docker-compose.yml 示例
```

## 完整示例

### 示例：docker-compose 配置

```yaml
# 完整示例代码
```

## 注意事项

- 合理组织服务
- 配置服务依赖
- 管理数据卷
- 优化网络配置

## 常见问题

### Q: docker-compose 和 Docker 有什么区别？

A: Docker 管理单个容器，docker-compose 管理多个容器的应用。

### Q: 如何配置服务依赖？

A: 使用 `depends_on` 配置服务依赖关系。

## 最佳实践

- 合理组织服务
- 配置服务依赖
- 管理数据卷
- 优化网络配置

## 对比分析

### docker-compose vs 手动管理容器

- 管理方式对比
- 适用场景对比

## 实践任务

1. 编写 docker-compose.yml
2. 启动多容器应用
3. 管理服务编排

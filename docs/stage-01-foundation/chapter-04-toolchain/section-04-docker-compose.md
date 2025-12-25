# 1.4.4 容器与环境管理

## 概述

Docker Compose 是定义和运行多容器 Docker 应用程序的工具。使用 Docker Compose 可以方便地管理开发环境，确保团队成员使用一致的环境配置。

**章节类型**：配置性章节

**主要内容**：
- Docker Compose 基础概念
- docker-compose.yml 配置文件结构
- 常用命令（up、down、ps、logs、exec）
- 环境变量配置
- 数据卷管理
- 网络管理
- 完整配置示例（PHP + MySQL + Redis）

## 特性

- **多容器管理**：可以同时管理多个容器
- **环境一致性**：确保开发、测试、生产环境一致
- **配置即代码**：使用 YAML 文件定义配置
- **快速启动**：一键启动整个开发环境

## 配置步骤/语法

### Docker Compose 基础

**安装 Docker Compose**：
- Windows/macOS：Docker Desktop 自带
- Linux：单独安装方法

**基本概念**：
- service：服务定义
- volume：数据卷
- network：网络
- environment：环境变量

### docker-compose.yml 配置

**基本结构**：
- version：Compose 文件版本
- services：服务定义
- volumes：数据卷定义
- networks：网络定义

**服务配置**：
- image：镜像名称
- build：构建配置
- ports：端口映射
- volumes：数据卷挂载
- environment：环境变量
- depends_on：依赖关系

### 常用命令

**启动和停止**：
- `docker-compose up`：启动服务
- `docker-compose up -d`：后台启动
- `docker-compose down`：停止并删除容器
- `docker-compose stop`：停止服务
- `docker-compose start`：启动服务

**查看和管理**：
- `docker-compose ps`：查看运行状态
- `docker-compose logs`：查看日志
- `docker-compose exec`：执行命令
- `docker-compose build`：构建镜像

### 环境变量配置

**方法 1：在 docker-compose.yml 中定义**
**方法 2：使用 .env 文件**
**方法 3：使用环境变量文件**

### 数据卷管理

**命名卷**：
- 定义和使用命名卷
- 数据持久化

**绑定挂载**：
- 挂载主机目录到容器
- 开发环境常用

### 网络管理

**默认网络**：
- Compose 自动创建网络
- 服务间通信

**自定义网络**：
- 定义自定义网络
- 网络配置选项

### 完整配置示例

**PHP + MySQL + Redis 环境**：
- 完整的 docker-compose.yml 示例
- 各服务配置说明
- 环境变量配置
- 数据卷配置

## 注意事项

- **端口冲突**：注意端口映射，避免冲突
- **数据持久化**：重要数据应使用数据卷
- **环境变量**：敏感信息不要硬编码，使用环境变量
- **资源限制**：合理配置资源限制
- **网络隔离**：使用网络隔离不同环境

## 常见问题

### 问题 1：端口被占用
- **原因**：主机端口已被占用
- **解决**：修改端口映射或关闭占用端口的程序

### 问题 2：容器无法启动
- **原因**：配置错误、镜像不存在、资源不足
- **解决**：检查配置、拉取镜像、检查资源

### 问题 3：数据丢失
- **原因**：未配置数据卷
- **解决**：配置数据卷，确保数据持久化

## 最佳实践

- 使用版本控制管理 docker-compose.yml
- 使用 .env 文件管理环境变量
- 为不同环境创建不同的 compose 文件
- 定期备份数据卷
- 使用命名卷而非绑定挂载（生产环境）
- 配置资源限制，避免资源耗尽

# 10.1.1 Dockerfile 基础

## 概述

Dockerfile 是构建 Docker 镜像的脚本文件。本节介绍 Dockerfile 的基础概念、语法、常用指令、构建方法，帮助零基础学员理解如何编写 Dockerfile。

**章节类型**：语法性章节

**主要内容**：
- Dockerfile 概述
- 基础语法
- 常用指令
- 构建镜像
- 完整示例

## 核心内容

### Dockerfile 概述

- Dockerfile 的作用
- Dockerfile 原理
- 镜像构建流程

### 基础语法

- 指令格式
- 注释
- 指令顺序
- 最佳实践

### 常用指令

- FROM
- RUN
- COPY/ADD
- WORKDIR
- ENV
- EXPOSE
- CMD/ENTRYPOINT

### 构建镜像

- 构建命令
- 构建上下文
- 构建优化
- 镜像标签

## 基本用法

### Dockerfile 示例

```dockerfile
# Dockerfile 示例
```

## 完整示例

### 示例：PHP 应用 Dockerfile

```dockerfile
# 完整示例代码
```

## 注意事项

- 合理使用缓存
- 减少镜像层数
- 使用 .dockerignore
- 优化构建速度

## 常见问题

### Q: Dockerfile 中的指令顺序重要吗？

A: 重要，应该将变化频率低的指令放在前面，利用层缓存。

### Q: COPY 和 ADD 有什么区别？

A: COPY 更简单直接，ADD 支持 URL 和自动解压，建议优先使用 COPY。

## 最佳实践

- 合理使用缓存
- 减少镜像层数
- 使用 .dockerignore
- 优化构建速度

## 对比分析

### 不同构建方法对比

- 单阶段构建 vs 多阶段构建
- 适用场景对比

## 实践任务

1. 编写基础 Dockerfile
2. 构建 Docker 镜像
3. 优化 Dockerfile

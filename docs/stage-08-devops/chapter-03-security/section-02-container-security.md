# 8.3.2 容器安全

## 概述

容器运行时安全是容器安全的重要组成部分。本节介绍容器运行时安全、权限控制、安全配置等内容。

## 运行时安全

### 只读文件系统

```yaml
# docker-compose.yml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
```

### 资源限制

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

## 权限控制

### 用户权限

```dockerfile
# 使用非 root 用户
FROM php:8.2-fpm-alpine

RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

USER appuser
```

### 能力限制

```yaml
services:
  app:
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
```

## 安全配置

### 网络隔离

```yaml
services:
  app:
    networks:
      - frontend
  
  db:
    networks:
      - backend
    # 不暴露端口到主机
    # ports: []  # 注释掉端口映射
```

### 环境变量安全

```yaml
services:
  app:
    env_file:
      - .env
    environment:
      - APP_ENV=${APP_ENV}
    # 不在 compose 文件中硬编码敏感信息
```

## 完整示例

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build: .
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
    user: "1000:1000"
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
    networks:
      - backend
    restart: unless-stopped

networks:
  backend:
    driver: bridge
```

## 最佳实践

1. **只读文件系统**：使用只读文件系统
2. **资源限制**：限制容器资源使用
3. **权限最小化**：使用最小权限
4. **网络隔离**：隔离容器网络

## 注意事项

1. 测试安全配置不影响功能
2. 注意权限问题
3. 监控容器资源使用
4. 定期审查安全配置

## 练习

1. 配置容器使用只读文件系统。

2. 设置资源限制，防止资源耗尽。

3. 配置网络隔离，保护后端服务。

4. 实现完整的容器安全配置。

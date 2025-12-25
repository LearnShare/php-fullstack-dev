# 8.3.1 镜像安全

## 概述

镜像安全是容器安全的基础。本节介绍镜像安全风险、安全扫描工具、漏洞修复、镜像优化等内容。

## 镜像安全风险

### 常见风险

1. **基础镜像漏洞**：基础镜像包含已知漏洞
2. **依赖漏洞**：应用依赖包含安全漏洞
3. **敏感信息泄露**：镜像中包含密钥等敏感信息
4. **权限过大**：使用 root 用户运行

## 安全扫描

### 使用 Trivy

```bash
# 安装 Trivy
brew install trivy

# 扫描镜像
trivy image my-app:latest

# 扫描 Dockerfile
trivy fs .

# 只显示高危漏洞
trivy image --severity HIGH,CRITICAL my-app:latest
```

### 集成到 CI/CD

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: my-app:latest
          format: 'sarif'
          output: 'trivy-results.sarif'
```

## 漏洞修复

### 更新基础镜像

```dockerfile
# 不推荐：使用旧版本
FROM php:8.0-fpm

# 推荐：使用最新版本
FROM php:8.2-fpm-alpine
```

### 定期更新

```bash
# 定期更新基础镜像
docker pull php:8.2-fpm-alpine
docker build --pull -t my-app:latest .
```

## 镜像优化

### 最小化攻击面

```dockerfile
# 只安装必要的包
FROM php:8.2-fpm-alpine

# 使用虚拟包，构建后删除
RUN apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
    && docker-php-ext-install pdo_mysql \
    && apk del .build-deps
```

### 使用非 root 用户

```dockerfile
FROM php:8.2-fpm-alpine

# 创建非 root 用户
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

USER appuser
```

## 完整示例

```dockerfile
# 安全优化的 Dockerfile
FROM php:8.2-fpm-alpine AS base

# 安装最小依赖
RUN apk add --no-cache \
    --virtual .build-deps \
    $PHPIZE_DEPS \
    libpng-dev \
    && docker-php-ext-install gd \
    && apk del .build-deps \
    && rm -rf /var/cache/apk/*

# 创建非 root 用户
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /var/www/html

# 复制文件
COPY --chown=appuser:appuser . .

# 切换到非 root 用户
USER appuser

EXPOSE 9000
CMD ["php-fpm"]
```

## 最佳实践

1. **定期扫描**：定期扫描镜像漏洞
2. **及时更新**：及时更新基础镜像
3. **最小权限**：使用非 root 用户
4. **最小攻击面**：只安装必要的包

## 注意事项

1. 扫描结果需要及时处理
2. 注意误报和漏报
3. 平衡安全性和功能性
4. 建立安全扫描流程

## 练习

1. 使用 Trivy 扫描镜像，找出安全漏洞。

2. 修复发现的安全漏洞。

3. 优化 Dockerfile，减小攻击面。

4. 集成安全扫描到 CI/CD 流程。

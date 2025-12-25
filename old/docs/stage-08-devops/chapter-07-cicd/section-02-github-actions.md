# 8.7.2 GitHub Actions

## 概述

GitHub Actions 是 GitHub 提供的 CI/CD 平台。本节介绍 GitHub Actions 基础、工作流配置、Actions 市场等内容。

## GitHub Actions 基础

### 工作流文件

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install dependencies
        run: composer install
      - name: Run tests
        run: phpunit
```

## 工作流配置

### 多作业工作流

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: phpunit

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t my-app:latest .

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: ./deploy.sh
```

### 矩阵构建

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php-version: ['8.1', '8.2', '8.3']
    steps:
      - uses: actions/checkout@v3
      - name: Setup PHP ${{ matrix.php-version }}
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
      - name: Run tests
        run: phpunit
```

## Actions 市场

### 常用 Actions

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: shivammathur/setup-php@v2
    with:
      php-version: '8.2'
  - uses: docker/login-action@v2
    with:
      username: ${{ secrets.DOCKER_USERNAME }}
      password: ${{ secrets.DOCKER_PASSWORD }}
  - uses: docker/build-push-action@v3
    with:
      context: .
      push: true
      tags: my-app:latest
```

## 完整示例

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install dependencies
        run: composer install
      - name: Run tests
        run: phpunit

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v3
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v3
        with:
          context: .
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          # 部署脚本
          ./deploy.sh
```

## 最佳实践

1. **使用缓存**：缓存依赖提升速度
2. **并行执行**：并行执行独立任务
3. **使用 Secrets**：保护敏感信息
4. **条件部署**：只在主分支部署

## 注意事项

1. 保护 Secrets
2. 注意 Actions 版本
3. 优化工作流速度
4. 监控工作流执行

## 练习

1. 创建一个 GitHub Actions 工作流。

2. 配置多作业工作流，实现测试、构建、部署。

3. 使用矩阵构建，测试多个 PHP 版本。

4. 集成 Docker 构建和推送。

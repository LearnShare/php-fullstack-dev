# 8.7.1 CI/CD 基础

## 概述

CI/CD（持续集成/持续部署）是 DevOps 的核心实践。本节介绍 CI/CD 概念、持续集成、持续部署、CI/CD 流程等内容。

## CI/CD 概念

### 持续集成（CI）

持续集成是频繁地将代码集成到主分支，自动运行测试和构建。

### 持续部署（CD）

持续部署是自动将代码部署到生产环境。

### CI/CD 流程

```
代码提交 → 自动构建 → 运行测试 → 部署到测试环境 → 部署到生产环境
```

## 持续集成

### CI 流程

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

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

## 持续部署

### CD 流程

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          # 部署脚本
          ./deploy.sh
```

## 完整示例

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
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
      - name: Code coverage
        run: phpunit --coverage-clover coverage.xml

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t my-app:${{ github.sha }} .
      - name: Push to registry
        run: docker push my-app:${{ github.sha }}

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          # 部署逻辑
          ./deploy.sh
```

## 最佳实践

1. **自动化测试**：每次提交运行测试
2. **快速反馈**：快速反馈构建结果
3. **部署自动化**：自动化部署流程
4. **回滚机制**：准备快速回滚

## 注意事项

1. 测试应该可靠
2. 保护生产环境
3. 使用环境变量管理配置
4. 监控部署过程

## 练习

1. 创建一个 CI 流程，自动运行测试。

2. 实现 CD 流程，自动部署到生产环境。

3. 配置多环境部署（开发、测试、生产）。

4. 实现部署回滚机制。

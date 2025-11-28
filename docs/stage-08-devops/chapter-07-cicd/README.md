# 8.7 CI/CD 与 GitHub Actions

## 目标

- 理解 CI/CD 的概念与流程。
- 掌握 GitHub Actions 的配置。
- 实现自动化测试、构建、部署流程。

## GitHub Actions

### 基础工作流

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.3'
        coverage: xdebug
    
    - name: Install dependencies
      run: composer install
    
    - name: Run tests
      run: composer test
    
    - name: Build Docker image
      run: docker build -t myapp:${{ github.sha }} .
    
    - name: Push to registry
      run: docker push myapp:${{ github.sha }}
```

## 练习

1. 配置完整的 CI/CD 流程。

2. 实现自动化测试和代码质量检查。

3. 创建多环境部署流程。

4. 配置部署通知和回滚机制。

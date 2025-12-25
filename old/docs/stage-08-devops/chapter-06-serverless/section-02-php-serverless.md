# 8.6.2 PHP Serverless 实践

## 概述

PHP Serverless 实践需要特殊考虑。本节介绍 Bref 的使用、AWS Lambda 部署、冷启动优化等内容。

## Bref

### 安装 Bref

```bash
composer require bref/bref
php vendor/bin/bref init
```

### serverless.yml 配置

```yaml
service: my-app

provider:
  name: aws
  runtime: provided.al2
  region: us-east-1

functions:
  web:
    handler: public/index.php
    runtime: provided.al2
    events:
      - httpApi: '*'

plugins:
  - ./vendor/bref/bref
```

### 部署

```bash
# 部署到 AWS Lambda
serverless deploy
```

## AWS Lambda

### Lambda 函数配置

```yaml
functions:
  api:
    handler: public/index.php
    runtime: provided.al2
    timeout: 30
    memorySize: 512
    events:
      - httpApi:
          path: /{proxy+}
          method: ANY
      - httpApi:
          path: /
          method: ANY
```

### 环境变量

```yaml
provider:
  environment:
    APP_ENV: production
    APP_DEBUG: false
    DB_HOST: ${env:DB_HOST}
```

## 冷启动优化

### 优化策略

1. **预热函数**：定期调用函数保持热状态
2. **减小包大小**：减少依赖和文件大小
3. **使用层**：将依赖放在 Lambda 层
4. **Provisioned Concurrency**：使用预置并发

### 使用 Lambda 层

```yaml
layers:
  php:
    path: layers/php
    compatibleRuntimes:
      - provided.al2

functions:
  api:
    layers:
      - ${self:provider.layers.php}
```

## 完整示例

```yaml
# serverless.yml
service: my-app

provider:
  name: aws
  runtime: provided.al2
  region: us-east-1
  environment:
    APP_ENV: ${env:APP_ENV, 'production'}
    DB_HOST: ${env:DB_HOST}

functions:
  web:
    handler: public/index.php
    runtime: provided.al2
    timeout: 30
    memorySize: 512
    events:
      - httpApi:
          path: /{proxy+}
          method: ANY
      - httpApi:
          path: /
          method: ANY

plugins:
  - ./vendor/bref/bref
```

## 最佳实践

1. **使用 Bref**：使用 Bref 简化部署
2. **优化冷启动**：减小包大小，使用层
3. **错误处理**：实现完善的错误处理
4. **监控**：配置 CloudWatch 监控

## 注意事项

1. PHP 冷启动较慢
2. 注意 Lambda 限制
3. 优化依赖大小
4. 使用 Provisioned Concurrency

## 练习

1. 使用 Bref 创建 Serverless 应用。

2. 部署到 AWS Lambda。

3. 优化冷启动时间。

4. 配置监控和日志。

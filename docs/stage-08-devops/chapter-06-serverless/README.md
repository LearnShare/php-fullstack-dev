# 8.6 Serverless PHP

## 目标

- 理解 Serverless 的概念与优势。
- 掌握 Bref 的使用（AWS Lambda）。
- 了解冷启动优化策略。

## Bref

### 基础配置

```yaml
# serverless.yml
service: myapp

provider:
  name: aws
  runtime: provided.al2
  region: us-east-1

functions:
  api:
    handler: public/index.php
    events:
      - httpApi: '*'

plugins:
  - ./vendor/bref/bref
```

### 优化

```php
<?php
// 优化冷启动
require __DIR__ . '/vendor/autoload.php';

// 预加载常用类
class_exists(\App\Service\UserService::class);
```

## 练习

1. 使用 Bref 部署 PHP 应用到 AWS Lambda。

2. 优化冷启动时间。

3. 配置 API Gateway 路由。

4. 实现 Serverless 数据库连接。

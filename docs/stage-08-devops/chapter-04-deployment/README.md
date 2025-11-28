# 8.4 部署选择

## 目标

- 了解各种部署方案的特点。
- 掌握 AWS ECS、Laravel Forge 等部署方式。
- 理解 Serverless 部署的优势。

## 部署方案对比

| 方案              | 特点                           | 适用场景           |
| :---------------- | :----------------------------- | :----------------- |
| 传统 VPS         | 完全控制，需要自己管理         | 小型项目           |
| ECS / EKS        | 容器编排，自动扩展             | 中大型项目         |
| Laravel Forge    | 简单易用，Laravel 优化         | Laravel 项目       |
| Serverless       | 按需付费，自动扩展             | 低流量、突发流量   |

## AWS ECS

### 任务定义

```json
{
  "family": "myapp",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "myapp:latest",
      "portMappings": [
        {
          "containerPort": 9000
        }
      ],
      "environment": [
        {
          "name": "DB_HOST",
          "value": "db.example.com"
        }
      ]
    }
  ]
}
```

## Laravel Forge

### 部署流程

1. 连接服务器
2. 创建站点
3. 配置环境变量
4. 部署代码

## 练习

1. 配置 AWS ECS 部署流程。

2. 使用 Laravel Forge 部署应用。

3. 对比不同部署方案的优缺点。

4. 实现自动化部署脚本。

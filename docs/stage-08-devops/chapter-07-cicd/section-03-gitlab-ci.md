# 8.7.3 GitLab CI

## 概述

GitLab CI 是 GitLab 提供的 CI/CD 平台。本节介绍 GitLab CI 基础、.gitlab-ci.yml 配置、Pipeline 配置等内容。

## GitLab CI 基础

### .gitlab-ci.yml

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  image: php:8.2
  script:
    - composer install
    - phpunit
  only:
    - main
    - develop

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t my-app:latest .
    - docker push my-app:latest
  only:
    - main

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

## Pipeline 配置

### 多阶段 Pipeline

```yaml
stages:
  - test
  - build
  - deploy-staging
  - deploy-production

test:
  stage: test
  image: php:8.2
  script:
    - composer install
    - phpunit

build:
  stage: build
  image: docker:latest
  script:
    - docker build -t my-app:$CI_COMMIT_SHA .
    - docker push my-app:$CI_COMMIT_SHA

deploy-staging:
  stage: deploy-staging
  script:
    - ./deploy.sh staging
  only:
    - develop

deploy-production:
  stage: deploy-production
  script:
    - ./deploy.sh production
  only:
    - main
  when: manual
```

## 完整示例

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: my-app
  DOCKER_REGISTRY: registry.example.com

test:
  stage: test
  image: php:8.2
  before_script:
    - composer install
  script:
    - phpunit --coverage-text
  coverage: '/^\s*Lines:\s*\d+.\d+\%/'
  artifacts:
    reports:
      junit: phpunit.xml
    paths:
      - coverage/
    expire_in: 1 week

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker build -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

deploy:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - ssh $DEPLOY_USER@$DEPLOY_HOST "cd /var/www/myapp && ./deploy.sh"
  only:
    - main
  when: manual
```

## 最佳实践

1. **使用缓存**：缓存依赖提升速度
2. **并行执行**：并行执行独立任务
3. **使用变量**：使用 CI/CD 变量
4. **条件执行**：使用 only/except 控制执行

## 注意事项

1. 保护 CI/CD 变量
2. 注意 Runner 配置
3. 优化 Pipeline 速度
4. 监控 Pipeline 执行

## 练习

1. 创建一个 GitLab CI Pipeline。

2. 配置多阶段 Pipeline。

3. 实现自动部署。

4. 配置手动部署到生产环境。

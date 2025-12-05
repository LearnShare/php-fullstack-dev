# 7.4.3 最佳实践

## 概述

Symfony 最佳实践帮助构建可维护的应用。本节介绍 Symfony 最佳实践、项目结构、配置管理等内容。

## 项目结构

### 推荐结构

```
src/
  Controller/        # 控制器
  Entity/           # 实体
  Repository/       # 仓库
  Service/          # 服务
  Form/             # 表单
  Event/            # 事件
  EventListener/    # 事件监听器
  Security/         # 安全相关
config/
  packages/         # 包配置
  routes.yaml       # 路由配置
  services.yaml     # 服务配置
templates/          # 模板
public/             # 公共文件
var/
  cache/            # 缓存
  log/              # 日志
```

## 配置管理

### 环境配置

```yaml
# config/packages/framework.yaml
framework:
    secret: '%env(APP_SECRET)%'
    default_locale: '%env(APP_LOCALE)%'
```

### 服务配置

```yaml
# config/services.yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true
    
    App\:
        resource: '../src/*'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'
    
    App\Service\EmailService:
        arguments:
            $smtpHost: '%env(SMTP_HOST)%'
```

## 代码组织

### 服务类

```php
<?php
declare(strict_types=1);

namespace App\Service;

class UserService
{
    public function __construct(
        private UserRepository $repository,
        private EmailService $emailService
    ) {}
    
    public function createUser(array $data): User
    {
        $user = new User($data);
        $this->repository->save($user);
        $this->emailService->sendWelcomeEmail($user);
        return $user;
    }
}
```

### 事件系统

```php
<?php
declare(strict_types=1);

namespace App\Event;

use App\Entity\User;
use Symfony\Contracts\EventDispatcher\Event;

class UserCreatedEvent extends Event
{
    public function __construct(
        public User $user
    ) {}
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use App\Service\UserService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class UserController extends AbstractController
{
    public function __construct(
        private UserService $userService
    ) {}
    
    #[Route('/users', name: 'user_list', methods: ['GET'])]
    public function index(): JsonResponse
    {
        $users = $this->userService->getAll();
        return $this->json($users);
    }
    
    #[Route('/users', name: 'user_create', methods: ['POST'])]
    public function create(Request $request): JsonResponse
    {
        $data = json_decode($request->getContent(), true);
        $user = $this->userService->createUser($data);
        return $this->json($user, 201);
    }
}
```

## 最佳实践

1. **服务分离**：业务逻辑放在服务类
2. **依赖注入**：使用依赖注入
3. **事件驱动**：使用事件解耦
4. **配置分离**：配置和代码分离

## 注意事项

1. 遵循 Symfony 约定
2. 使用服务容器管理依赖
3. 合理组织代码结构
4. 注意性能优化

## 练习

1. 重构代码，遵循 Symfony 最佳实践。

2. 组织项目结构，分离关注点。

3. 配置服务，使用依赖注入。

4. 实现事件系统，解耦代码。

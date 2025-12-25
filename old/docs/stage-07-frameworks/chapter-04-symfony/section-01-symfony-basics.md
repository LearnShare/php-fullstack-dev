# 7.4.1 Symfony 基础

## 概述

Symfony 是一个强大的 PHP 框架和组件库。本节介绍 Symfony 架构、核心组件、HttpFoundation、依赖注入等内容。

## Symfony 架构

### 组件化设计

```
Symfony = 独立组件 + 框架集成
```

### 目录结构

```
config/
  packages/
  routes.yaml
public/
  index.php
src/
  Controller/
  Entity/
  Repository/
templates/
var/
  cache/
  log/
```

## 核心组件

### HttpFoundation

```php
<?php
declare(strict_types=1);

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

// 创建请求
$request = Request::createFromGlobals();

// 获取请求数据
$name = $request->query->get('name');
$email = $request->request->get('email');

// 创建响应
$response = new Response('Hello World', 200);
$response->headers->set('Content-Type', 'text/html');

$response->send();
```

### 路由系统

```php
<?php
declare(strict_types=1);

use Symfony\Component\Routing\Route;
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Matcher\UrlMatcher;
use Symfony\Component\Routing\RequestContext;

$routes = new RouteCollection();
$routes->add('user_list', new Route('/users', [
    '_controller' => 'App\Controller\UserController::index',
]));

$routes->add('user_show', new Route('/users/{id}', [
    '_controller' => 'App\Controller\UserController::show',
    'id' => '\d+',
]));

$context = new RequestContext();
$matcher = new UrlMatcher($routes, $context);

$parameters = $matcher->match('/users/123');
```

## 依赖注入

### 服务配置

```yaml
# config/services.yaml
services:
    App\Service\EmailService:
        arguments:
            $smtpHost: '%env(SMTP_HOST)%'
    
    App\Service\UserService:
        arguments:
            $emailService: '@App\Service\EmailService'
```

### 自动装配

```php
<?php
declare(strict_types=1);

namespace App\Service;

class UserService
{
    public function __construct(
        private EmailService $emailService
    ) {}
}
```

## 控制器

### 基础控制器

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class UserController extends AbstractController
{
    #[Route('/users', name: 'user_list')]
    public function index(): Response
    {
        $users = []; // 获取用户列表
        return $this->render('users/index.html.twig', [
            'users' => $users,
        ]);
    }
    
    #[Route('/users/{id}', name: 'user_show')]
    public function show(int $id): Response
    {
        $user = []; // 获取用户
        return $this->json($user);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class UserController extends AbstractController
{
    public function __construct(
        private UserService $userService
    ) {}
    
    #[Route('/users', name: 'user_list', methods: ['GET'])]
    public function index(): Response
    {
        $users = $this->userService->getAll();
        return $this->json($users);
    }
    
    #[Route('/users', name: 'user_create', methods: ['POST'])]
    public function create(Request $request): Response
    {
        $data = json_decode($request->getContent(), true);
        $user = $this->userService->create($data);
        return $this->json($user, 201);
    }
}
```

## 最佳实践

1. **使用组件**：利用 Symfony 组件
2. **依赖注入**：使用依赖注入管理服务
3. **配置分离**：配置和代码分离
4. **遵循约定**：遵循 Symfony 约定

## 注意事项

1. Symfony 组件可以独立使用
2. 配置使用 YAML 或 PHP
3. 使用注解或 YAML 定义路由
4. 注意服务配置

## 练习

1. 创建一个 Symfony 项目，配置路由和控制器。

2. 使用 HttpFoundation 处理请求和响应。

3. 配置依赖注入，注册服务。

4. 创建一个完整的 RESTful API。

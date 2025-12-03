# 3.7.1 MVC 架构模式

## 概述

MVC（Model-View-Controller）是最经典的 Web 应用架构模式，将应用分为模型、视图和控制器三个部分，实现关注点分离。

## MVC 核心概念

### 三个组成部分

MVC 将应用分为三个主要部分：

- **Model（模型）**：处理数据和业务逻辑。
- **View（视图）**：负责展示数据，处理用户界面。
- **Controller（控制器）**：协调 Model 和 View，处理用户输入。

```
用户请求
    ↓
Controller（处理请求、调用 Model、选择 View）
    ↓
Model（业务逻辑、数据操作）
    ↓
View（渲染响应）
    ↓
用户响应
```

## 基础 MVC 实现

### Model 层

```php
namespace App\Models;

class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
    }

    public static function find(int $id): ?self
    {
        // 从数据库查找用户
        // 这里简化处理
        return new self($id, 'Alice', 'alice@example.com');
    }

    public function save(): void
    {
        // 保存到数据库
    }
}
```

### View 层

```php
namespace App\Views;

class UserView
{
    public function render(array $user): string
    {
        return "
            <h1>User Profile</h1>
            <p>ID: {$user['id']}</p>
            <p>Name: {$user['name']}</p>
            <p>Email: {$user['email']}</p>
        ";
    }
}
```

### Controller 层

```php
namespace App\Controllers;

use App\Models\User;
use App\Views\UserView;

class UserController
{
    public function show(int $id): string
    {
        $user = User::find($id);
        
        if ($user === null) {
            return "User not found";
        }

        $view = new UserView();
        return $view->render([
            'id' => $user->id,
            'name' => $user->name,
            'email' => $user->email,
        ]);
    }
}
```

## MVC 路由示例

### 简单路由实现

```php
class Router
{
    private array $routes = [];

    public function get(string $path, callable $handler): void
    {
        $this->routes['GET'][$path] = $handler;
    }

    public function dispatch(string $method, string $path): string
    {
        $handler = $this->routes[$method][$path] ?? null;
        
        if ($handler === null) {
            return "404 Not Found";
        }

        return $handler();
    }
}

// 使用
$router = new Router();
$router->get('/users/{id}', function ($id) {
    $controller = new UserController();
    return $controller->show((int) $id);
});
```

## MVC 的优缺点

### 优点

- 关注点分离，代码组织清晰。
- 易于测试（各层可独立测试）。
- 视图与业务逻辑解耦。

### 缺点

- Controller 可能变得臃肿（Fat Controller）。
- Model 可能承担过多责任。
- View 与 Controller 耦合可能过紧。

## 完整示例

```php
<?php
declare(strict_types=1);

// Model
namespace App\Models;

class Product
{
    public function __construct(
        public int $id,
        public string $name,
        public float $price
    ) {
    }

    public static function find(int $id): ?self
    {
        // 从数据库查找
        return new self($id, 'Laptop', 999.99);
    }

    public function save(): void
    {
        // 保存到数据库
    }
}

// View
namespace App\Views;

class ProductView
{
    public function render(array $product): string
    {
        return "
            <h1>{$product['name']}</h1>
            <p>Price: \${$product['price']}</p>
        ";
    }
}

// Controller
namespace App\Controllers;

use App\Models\Product;
use App\Views\ProductView;

class ProductController
{
    public function show(int $id): string
    {
        $product = Product::find($id);
        
        if ($product === null) {
            return "Product not found";
        }

        $view = new ProductView();
        return $view->render([
            'id' => $product->id,
            'name' => $product->name,
            'price' => $product->price,
        ]);
    }
}
```

## 注意事项

1. **职责分离**：Model 处理数据，View 处理展示，Controller 处理协调。

2. **避免 Fat Controller**：Controller 应该保持简洁，复杂逻辑应该放在 Model 或 Service 层。

3. **View 独立性**：View 不应该包含业务逻辑，只负责展示数据。

4. **测试友好**：各层应该可以独立测试。

5. **扩展性**：考虑使用 Service 层处理复杂业务逻辑。

## 练习

1. 实现一个完整的 MVC 应用，包含 User 模型、视图和控制器。

2. 创建一个路由系统，将请求分发到对应的控制器方法。

3. 实现一个 Product MVC 应用，包含列表、详情、创建功能。

4. 设计一个避免 Fat Controller 的方案，引入 Service 层。

5. 创建一个可测试的 MVC 结构，各层可以独立测试。

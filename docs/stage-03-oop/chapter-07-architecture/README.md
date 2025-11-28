# 3.7 应用架构模式

## 目标

- 理解 MVC（Model-View-Controller）架构模式的核心概念与实现方式。
- 掌握 ADR（Action-Domain-Responder）模式，理解其与 MVC 的区别。
- 熟悉模块化单体架构的组织方式（Domain/Infrastructure/Application 分层）。
- 能够在实际项目中应用这些架构模式，设计清晰的应用结构。

## MVC 架构模式

### MVC 核心概念

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

### 基础 MVC 实现

#### Model 层

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

#### View 层

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

#### Controller 层

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

### MVC 路由示例

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

### MVC 的优缺点

**优点：**
- 关注点分离，代码组织清晰。
- 易于测试（各层可独立测试）。
- 视图与业务逻辑解耦。

**缺点：**
- Controller 可能变得臃肿（Fat Controller）。
- Model 可能承担过多责任。
- View 与 Controller 耦合可能过紧。

## ADR 架构模式

### ADR 核心概念

ADR（Action-Domain-Responder）是 MVC 的改进版本：

- **Action（动作）**：处理单个 HTTP 请求，协调 Domain 和 Responder。
- **Domain（领域）**：包含业务逻辑，不依赖框架。
- **Responder（响应器）**：构建 HTTP 响应（JSON、HTML 等）。

```
HTTP 请求
    ↓
Action（处理请求、调用 Domain、使用 Responder）
    ↓
Domain（业务逻辑）
    ↓
Responder（构建响应）
    ↓
HTTP 响应
```

### ADR 实现示例

#### Domain 层

```php
namespace App\Domain;

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {
    }

    public function getUserById(int $id): User
    {
        $user = $this->repository->find($id);
        
        if ($user === null) {
            throw new UserNotFoundException($id);
        }

        return $user;
    }

    public function createUser(string $name, string $email): User
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($email);
        }

        $user = new User(null, $name, $email);
        $this->repository->save($user);
        
        return $user;
    }
}
```

#### Action 层

```php
namespace App\Actions;

use App\Domain\UserService;
use App\Responders\JsonResponder;
use App\Responders\HtmlResponder;

class ShowUserAction
{
    public function __construct(
        private UserService $userService,
        private JsonResponder $jsonResponder,
        private HtmlResponder $htmlResponder
    ) {
    }

    public function __invoke(int $id, string $format = 'json'): Response
    {
        try {
            $user = $this->userService->getUserById($id);
            
            return match ($format) {
                'json' => $this->jsonResponder->respond($user),
                'html' => $this->htmlResponder->respond($user),
                default => throw new InvalidArgumentException("Unsupported format: {$format}"),
            };
        } catch (UserNotFoundException $e) {
            return $this->jsonResponder->error($e->getMessage(), 404);
        }
    }
}
```

#### Responder 层

```php
namespace App\Responders;

class JsonResponder
{
    public function respond(mixed $data): Response
    {
        return new Response(
            json_encode($data, JSON_UNESCAPED_UNICODE),
            200,
            ['Content-Type' => 'application/json']
        );
    }

    public function error(string $message, int $code = 500): Response
    {
        return new Response(
            json_encode(['error' => $message], JSON_UNESCAPED_UNICODE),
            $code,
            ['Content-Type' => 'application/json']
        );
    }
}

class HtmlResponder
{
    public function respond(User $user): Response
    {
        $html = "
            <!DOCTYPE html>
            <html>
            <head><title>User Profile</title></head>
            <body>
                <h1>{$user->getName()}</h1>
                <p>Email: {$user->getEmail()}</p>
            </body>
            </html>
        ";

        return new Response($html, 200, ['Content-Type' => 'text/html']);
    }
}
```

### ADR vs MVC

| 特性           | MVC                    | ADR                        |
| :------------- | :--------------------- | :------------------------- |
| Controller     | 处理多个动作           | 每个 Action 处理单个请求   |
| 响应构建       | 通常在 Controller 中   | 独立的 Responder 层        |
| 路由映射       | 路由到 Controller 方法 | 路由到 Action 类           |
| 代码组织       | 按资源分组             | 按动作分组                 |
| 测试性         | 较好                   | 更好（更细粒度）           |

## 模块化单体架构

### 分层架构

将应用分为多个层次，每层有明确的职责：

```
┌─────────────────────────────────────┐
│   Presentation Layer (表现层)        │
│   - Controllers / Actions           │
│   - Views / Responders             │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│   Application Layer (应用层)          │
│   - Services                        │
│   - DTOs                            │
│   - Use Cases                       │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│   Domain Layer (领域层)               │
│   - Entities                        │
│   - Value Objects                   │
│   - Domain Services                 │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│   Infrastructure Layer (基础设施层)  │
│   - Repositories                    │
│   - Database                        │
│   - External APIs                   │
└─────────────────────────────────────┘
```

### 目录结构示例

```
src/
├── Domain/                    # 领域层
│   ├── User/
│   │   ├── User.php          # 实体
│   │   ├── UserId.php        # 值对象
│   │   ├── UserRepository.php # 仓储接口
│   │   └── UserService.php   # 领域服务
│   └── Product/
├── Application/               # 应用层
│   ├── Services/
│   │   └── UserApplicationService.php
│   └── DTOs/
│       └── CreateUserDTO.php
├── Infrastructure/            # 基础设施层
│   ├── Persistence/
│   │   └── DatabaseUserRepository.php
│   └── External/
│       └── EmailService.php
└── Http/                      # 表现层
    ├── Controllers/
    │   └── UserController.php
    └── Middleware/
```

### 领域层示例

```php
namespace App\Domain\User;

class User
{
    public function __construct(
        private ?UserId $id,
        private string $name,
        private Email $email
    ) {
        if (empty($name)) {
            throw new InvalidArgumentException('Name cannot be empty');
        }
    }

    public function getId(): ?UserId
    {
        return $this->id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function changeEmail(Email $newEmail): void
    {
        $this->email = $newEmail;
    }
}

class Email
{
    public function __construct(private string $value)
    {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($value);
        }
    }

    public function getValue(): string
    {
        return $this->value;
    }
}
```

### 应用层示例

```php
namespace App\Application;

use App\Domain\User\User;
use App\Domain\User\UserRepository;

class UserApplicationService
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function createUser(CreateUserDTO $dto): User
    {
        $email = new Email($dto->email);
        
        // 检查邮箱是否已存在
        if ($this->userRepository->existsByEmail($email)) {
            throw new EmailAlreadyExistsException($dto->email);
        }

        $user = new User(null, $dto->name, $email);
        $this->userRepository->save($user);
        
        return $user;
    }

    public function getUserById(int $id): User
    {
        $user = $this->userRepository->find(new UserId($id));
        
        if ($user === null) {
            throw new UserNotFoundException($id);
        }

        return $user;
    }
}
```

### 基础设施层示例

```php
namespace App\Infrastructure\Persistence;

use App\Domain\User\User;
use App\Domain\User\UserId;
use App\Domain\User\UserRepository;
use PDO;

class DatabaseUserRepository implements UserRepository
{
    public function __construct(private PDO $db)
    {
    }

    public function find(UserId $id): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id->getValue()]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($data === false) {
            return null;
        }

        return $this->mapToUser($data);
    }

    public function save(User $user): void
    {
        if ($user->getId() === null) {
            $this->insert($user);
        } else {
            $this->update($user);
        }
    }

    private function insert(User $user): void
    {
        $stmt = $this->db->prepare('INSERT INTO users (name, email) VALUES (?, ?)');
        $stmt->execute([$user->getName(), $user->getEmail()->getValue()]);
    }

    private function update(User $user): void
    {
        $stmt = $this->db->prepare('UPDATE users SET name = ?, email = ? WHERE id = ?');
        $stmt->execute([
            $user->getName(),
            $user->getEmail()->getValue(),
            $user->getId()->getValue(),
        ]);
    }

    private function mapToUser(array $data): User
    {
        return new User(
            new UserId($data['id']),
            $data['name'],
            new Email($data['email'])
        );
    }
}
```

### 表现层示例

```php
namespace App\Http\Controllers;

use App\Application\UserApplicationService;

class UserController
{
    public function __construct(
        private UserApplicationService $userService
    ) {
    }

    public function create(Request $request): Response
    {
        try {
            $dto = new CreateUserDTO(
                $request->get('name'),
                $request->get('email')
            );

            $user = $this->userService->createUser($dto);

            return new JsonResponse([
                'id' => $user->getId()->getValue(),
                'name' => $user->getName(),
                'email' => $user->getEmail()->getValue(),
            ], 201);
        } catch (EmailAlreadyExistsException $e) {
            return new JsonResponse(['error' => $e->getMessage()], 409);
        } catch (InvalidArgumentException $e) {
            return new JsonResponse(['error' => $e->getMessage()], 400);
        }
    }

    public function show(int $id): Response
    {
        try {
            $user = $this->userService->getUserById($id);
            
            return new JsonResponse([
                'id' => $user->getId()->getValue(),
                'name' => $user->getName(),
                'email' => $user->getEmail()->getValue(),
            ]);
        } catch (UserNotFoundException $e) {
            return new JsonResponse(['error' => $e->getMessage()], 404);
        }
    }
}
```

## 依赖方向

### 依赖倒置原则

- 高层模块不应依赖低层模块，两者都应依赖抽象。
- 领域层不依赖基础设施层，基础设施层实现领域层定义的接口。

```php
// 领域层定义接口
namespace App\Domain\User;

interface UserRepository
{
    public function find(UserId $id): ?User;
    public function save(User $user): void;
}

// 基础设施层实现接口
namespace App\Infrastructure\Persistence;

class DatabaseUserRepository implements UserRepository
{
    // 实现细节
}
```

## 架构选择建议

### 何时使用 MVC

- 中小型项目。
- 团队熟悉 MVC 模式。
- 需要快速开发。

### 何时使用 ADR

- 需要更细粒度的控制。
- API 和 Web 页面需要不同的响应格式。
- 希望每个动作独立测试。

### 何时使用分层架构

- 大型项目。
- 需要清晰的领域模型。
- 需要支持多种表现层（Web、API、CLI）。

## 练习

1. 实现一个简单的 MVC 框架，包含 Router、Controller、Model、View 基础类。

2. 将上述 MVC 实现重构为 ADR 架构，将 Controller 拆分为 Action 和 Responder。

3. 设计一个用户管理模块，使用分层架构，包含 Domain、Application、Infrastructure、Http 四层。

4. 创建一个 Repository 接口在领域层，然后在基础设施层实现数据库版本和内存版本。

5. 实现一个简单的依赖注入容器，能够自动解析构造函数依赖并创建对象实例。

6. 设计一个事件系统，允许领域层发布事件，应用层订阅并处理这些事件。

# 7.3.1 MVC 架构模式

## 概述

MVC（Model-View-Controller）是最流行且影响最深远的软件架构模式之一。它最初由 Trygve Reenskaug 在 1979 年提出，后经由 Smalltalk 社区发展完善，并在 Web 应用开发中得到广泛应用。几乎所有的现代 Web 框架都直接或间接地采用 MVC 作为其核心架构。

MVC 模式将应用程序分为三个核心组件：模型（Model）、视图（View）和控制器（Controller）。这种分离使得代码更加模块化、易于维护和测试。通过明确区分数据处理（Model）、用户界面（View）和业务流程控制（Controller），MVC 帮助开发者构建结构清晰、可扩展的应用。

理解 MVC 架构是学习现代 PHP 框架（如 Laravel、Symfony）的基础。几乎所有 PHP Web 框架都采用了 MVC 或其变体作为应用架构的核心模式。

**主要内容**：
- MVC 架构模式的定义和历史
- 模型层的职责和实现
- 视图层的职责和实现
- 控制器层的职责和实现
- 数据流向和交互
- MVC 的变体和现代演进

## MVC 架构详解

### MVC 概念

MVC 代表三个核心组件的首字母：
- **Model（模型）**：负责数据和业务逻辑
- **View（视图）**：负责用户界面的呈现
- **Controller（控制器）**：负责协调模型和视图，处理用户输入

MVC 的核心目标是实现关注点分离（Separation of Concerns），使每个组件只关注自己的职责，从而提高代码的可维护性和可测试性。

### MVC 的历史

MVC 模式的发展历程：
- 1979 年：Trygve Reenskaug 在 Smalltalk-76 中提出 MVC
- 1988 年：MVC 被正式定义和文档化
- 2000 年代：成为 Web 应用开发的主流架构
- 至今：几乎所有 Web 框架都采用 MVC 或其变体

## 模型层（Model）

模型层是应用程序的核心，负责：
- 数据的表示和结构
- 业务逻辑的实现
- 数据的验证
- 与数据存储的交互

### 模型的设计原则

```php
<?php
declare(strict_types=1);

// 用户实体
class User
{
    private ?int $id;
    private string $name;
    private string $email;
    private DateTime $createdAt;
    
    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
        $this->createdAt = new DateTime();
    }
    
    // Getter 方法
    public function getId(): ?int { return $this->id; }
    public function getName(): string { return $this->name; }
    public function getEmail(): string { return $this->email; }
    public function getCreatedAt(): DateTime { return $this->createdAt; }
    
    // Setter 方法
    public function setId(int $id): void { $this->id = $id; }
    public function setName(string $name): void { $this->name = $name; }
    public function setEmail(string $email): void { $this->email = $email; }
    
    // 业务方法
    public function isValid(): bool
    {
        return !empty($this->name) && filter_var($this->email, FILTER_VALIDATE_EMAIL);
    }
    
    public function toArray(): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->createdAt->format('Y-m-d H:i:s'),
        ];
    }
}

// 用户仓储（数据访问）
interface UserRepository
{
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function save(User $user): void;
    public function delete(int $id): void;
}

// MySQL 实现
class MySQLUserRepository implements UserRepository
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($row === false) {
            return null;
        }
        
        return $this->rowToUser($row);
    }
    
    public function findByEmail(string $email): ?User
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE email = ?");
        $stmt->execute([$email]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($row === false) {
            return null;
        }
        
        return $this->rowToUser($row);
    }
    
    public function save(User $user): void
    {
        if ($user->getId() === null) {
            // 插入
            $stmt = $this->pdo->prepare(
                "INSERT INTO users (name, email, created_at) VALUES (?, ?, ?)"
            );
            $stmt->execute([
                $user->getName(),
                $user->getEmail(),
                $user->getCreatedAt()->format('Y-m-d H:i:s')
            ]);
            $user->setId((int)$this->pdo->lastInsertId());
        } else {
            // 更新
            $stmt = $this->pdo->prepare(
                "UPDATE users SET name = ?, email = ? WHERE id = ?"
            );
            $stmt->execute([
                $user->getName(),
                $user->getEmail(),
                $user->getId()
            ]);
        }
    }
    
    public function delete(int $id): void
    {
        $stmt = $this->pdo->prepare("DELETE FROM users WHERE id = ?");
        $stmt->execute([$id]);
    }
    
    private function rowToUser(array $row): User
    {
        $user = new User($row['name'], $row['email']);
        $user->setId((int)$row['id']);
        return $user;
    }
}
```

## 视图层（View）

视图层负责将数据呈现给用户。在 Web 应用中，视图通常是 HTML 模板。视图应该只负责显示数据，不包含业务逻辑。

### 视图的设计

```php
<?php
declare(strict_types=1);

// 简单的模板引擎
class View
{
    private string $templateDir;
    private array $data = [];
    
    public function __construct(string $templateDir = 'templates/')
    {
        $this->templateDir = rtrim($templateDir, '/') . '/';
    }
    
    public function assign(string $key, mixed $value): self
    {
        $this->data[$key] = $value;
        return $this;
    }
    
    public function render(string $template): string
    {
        $templatePath = $this->templateDir . $template . '.php';
        
        if (!file_exists($templatePath)) {
            throw new RuntimeException("Template not found: {$templatePath}");
        }
        
        // 提取数据到局部变量
        extract($this->data);
        
        // 开始输出缓冲
        ob_start();
        
        // 包含模板
        include $templatePath;
        
        // 获取渲染内容
        return ob_get_clean();
    }
}

// 用户列表视图模板
class UserListView
{
    private View $view;
    
    public function __construct(View $view)
    {
        $this->view = $view;
    }
    
    public function render(array $users): string
    {
        return $this->view
            ->assign('users', $users)
            ->render('user/list');
    }
}

// 用户详情视图
class UserDetailView
{
    private View $view;
    
    public function __construct(View $view)
    {
        $this->view = $view;
    }
    
    public function render(User $user): string
    {
        return $this->view
            ->assign('user', $user)
            ->render('user/detail');
    }
}
```

### 视图模板示例

```php
<?php
// templates/user/list.php
?>
<h1>User List</h1>
<table>
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Created</th>
        </tr>
    </thead>
    <tbody>
        <?php foreach ($users as $user): ?>
        <tr>
            <td><?= $user->getId() ?></td>
            <td><?= htmlspecialchars($user->getName()) ?></td>
            <td><?= htmlspecialchars($user->getEmail()) ?></td>
            <td><?= $user->getCreatedAt()->format('Y-m-d') ?></td>
        </tr>
        <?php endforeach; ?>
    </tbody>
</table>
```

## 控制器层（Controller）

控制器是连接模型和视图的桥梁，负责：
- 接收用户请求
- 调用模型处理业务逻辑
- 选择合适的视图返回响应

### 控制器的设计

```php
<?php
declare(strict_types=1);

// HTTP 请求
class Request
{
    public function __construct(
        public readonly array $get,
        public readonly array $post,
        public readonly array $server,
        public readonly array $headers
    ) {}
    
    public function get(string $key, mixed $default = null): mixed
    {
        return $this->get[$key] ?? $default;
    }
    
    public function post(string $key, mixed $default = null): mixed
    {
        return $this->post[$key] ?? $default;
    }
    
    public function method(): string
    {
        return $this->server['REQUEST_METHOD'] ?? 'GET';
    }
    
    public function uri(): string
    {
        return $this->server['REQUEST_URI'] ?? '/';
    }
}

// HTTP 响应
class Response
{
    public function __construct(
        private string $content,
        private int $statusCode = 200,
        private array $headers = []
    ) {}
    
    public function send(): void
    {
        http_response_code($this->statusCode);
        
        foreach ($this->headers as $name => $value) {
            header("{$name}: {$value}");
        }
        
        echo $this->content;
    }
    
    public static function html(string $content, int $statusCode = 200): self
    {
        return new self($content, $statusCode, [
            'Content-Type' => 'text/html; charset=UTF-8'
        ]);
    }
    
    public static function json(mixed $data, int $statusCode = 200): self
    {
        return new self(
            json_encode($data),
            $statusCode,
            ['Content-Type' => 'application/json']
        );
    }
    
    public static function redirect(string $url, int $statusCode = 302): self
    {
        return new self('', $statusCode, ['Location' => $url]);
    }
}

// 用户控制器
class UserController
{
    private UserRepository $repository;
    private UserListView $listView;
    private UserDetailView $detailView;
    
    public function __construct(
        UserRepository $repository,
        UserListView $listView,
        UserDetailView $detailView
    ) {
        $this->repository = $repository;
        $this->listView = $listView;
        $this->detailView = $detailView;
    }
    
    // 用户列表
    public function index(Request $request): Response
    {
        $users = $this->repository->findAll();
        $html = $this->listView->render($users);
        
        return Response::html($html);
    }
    
    // 用户详情
    public function show(Request $request): Response
    {
        $id = (int)$request->get('id');
        
        if ($id <= 0) {
            return Response::json(['error' => 'Invalid user ID'], 400);
        }
        
        $user = $this->repository->findById($id);
        
        if ($user === null) {
            return Response::json(['error' => 'User not found'], 404);
        }
        
        $html = $this->detailView->render($user);
        
        return Response::html($html);
    }
    
    // 创建用户
    public function store(Request $request): Response
    {
        $name = $request->post('name');
        $email = $request->post('email');
        
        if (empty($name) || empty($email)) {
            return Response::json(['error' => 'Name and email are required'], 400);
        }
        
        $user = new User($name, $email);
        
        if (!$user->isValid()) {
            return Response::json(['error' => 'Invalid user data'], 400);
        }
        
        $this->repository->save($user);
        
        return Response::json($user->toArray(), 201);
    }
}
```

## 数据流向

### 请求-响应流程

```
用户 → 请求 → 控制器 → 模型 → 数据库
                ↓
              视图 ← 数据
                ↓
              响应 → 用户
```

### MVC 数据流详解

1. **用户发送请求**：用户通过浏览器发送 HTTP 请求
2. **控制器接收请求**：路由器将请求分发给对应的控制器
3. **控制器调用模型**：控制器调用模型处理业务逻辑
4. **模型处理数据**：模型与数据库交互，处理数据
5. **模型返回数据**：模型将处理结果返回给控制器
6. **控制器选择视图**：控制器选择合适的视图
7. **视图渲染**：视图使用模型数据渲染 HTML
8. **返回响应**：控制器返回响应给用户

## 简单 MVC 框架实现

```php
<?php
declare(strict_types=1);

// 简单的 MVC 框架
class Application
{
    private array $bindings = [];
    
    public function bind(string $abstract, mixed $concrete): void
    {
        $this->bindings[$abstract] = $concrete;
    }
    
    public function make(string $abstract): mixed
    {
        if (isset($this->bindings[$abstract])) {
            return $this->bindings[$abstract];
        }
        
        // 自动实例化
        $class = new ReflectionClass($abstract);
        $constructor = $class->getConstructor();
        
        if ($constructor === null) {
            return new $abstract();
        }
        
        $params = [];
        foreach ($constructor->getParameters() as $param) {
            $type = $param->getType();
            if ($type !== null && !$type->isBuiltin()) {
                $params[] = $this->make($type->getName());
            }
        }
        
        return $class->newInstanceArgs($params);
    }
    
    public function run(): void
    {
        $request = new Request($_GET, $_POST, $_SERVER, getallheaders());
        
        // 简单路由
        $uri = $request->uri();
        $method = $request->method();
        
        // 路由映射
        $routes = [
            'GET /users' => [UserController::class, 'index'],
            'GET /user' => [UserController::class, 'show'],
            'POST /users' => [UserController::class, 'store'],
        ];
        
        $key = "{$method} {$uri}";
        
        if (!isset($routes[$key])) {
            http_response_code(404);
            echo "404 Not Found";
            return;
        }
        
        [$controllerClass, $action] = $routes[$key];
        
        $controller = $this->make($controllerClass);
        $response = $controller->$action($request);
        
        $response->send();
    }
}

// 使用框架
$app = new Application();

// 绑定依赖
$app->bind(UserRepository::class, new MySQLUserRepository(
    new PDO('mysql:host=localhost;dbname=app', 'root', '')
));
$app->bind(View::class, new View(__DIR__ . '/templates/'));
$app->bind(UserListView::class, new UserListView($app->make(View::class)));
$app->bind(UserDetailView::class, new UserDetailView($app->make(View::class)));

// 运行
$app->run();
```

## MVC 的演进

### 变体模式

1. **MVP（Model-View-Presenter）**：控制器改为 Presenter，更强调视图的被动性
2. **MVVM（Model-View-ViewModel）**：引入 ViewModel，用于数据绑定
3. **MVC变体**：现代框架如 Laravel 的架构虽然仍称为 MVC，但有细微差别

### 现代 MVC 的特点

- **服务层**：在控制器和模型之间添加服务层
- **仓储模式**：统一数据访问
- **中间件**：请求/响应的预处理
- **依赖注入**：管理组件依赖

## 使用场景

MVC 适用于：
- Web 应用开发
- 需要清晰代码组织的项目
- 团队协作开发
- 需要长期维护的项目

## 常见问题

### Controller 应该包含多少逻辑？

控制器应该尽量轻薄，只负责协调工作。业务逻辑应该放在模型或服务层。

### Model 应该是贫血还是充血？

现代 PHP 开发倾向于"充血模型"，即模型包含业务逻辑。但也要避免过度。

### View 中是否可以包含逻辑？

视图应该只包含必要的显示逻辑。复杂的数据处理应该在控制器或模型中完成。

## 最佳实践

1. **保持控制器轻薄**：控制器只负责协调，不包含业务逻辑
2. **使用充血模型**：模型应该包含业务逻辑和验证
3. **视图保持简洁**：视图只负责显示，不处理业务逻辑
4. **使用依赖注入**：通过依赖注入管理组件依赖
5. **遵循 REST 原则**：设计 RESTful 风格的控制器

## 练习任务

1. **实现 MVC 框架**：使用 PHP 实现一个简单的 MVC 框架
2. **设计用户系统**：使用 MVC 架构设计一个完整的用户管理系统
3. **重构现有代码**：将一个 monolithic 代码重构为 MVC 架构
4. **分析框架源码**：分析 Laravel 或 Symfony 的 MVC 实现
5. **对比不同模式**：比较 MVC、MVP、MVVM 的区别

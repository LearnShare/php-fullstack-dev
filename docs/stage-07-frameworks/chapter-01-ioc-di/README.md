# 7.1 IoC 与 DI 设计模式深度理解

## 目标

- 深入理解 IoC（控制反转）和 DI（依赖注入）的概念。
- 掌握服务容器的生命周期管理。
- 熟悉 Provider、Binding、Singleton 的实现。
- 理解自动注入（autowire）的原理。

## IoC 与 DI 概念

### 控制反转（IoC）

- **传统方式**：对象自己创建依赖。
- **IoC 方式**：依赖由外部容器提供。

```php
<?php
// 传统方式
class UserService
{
    private UserRepository $repository;
    
    public function __construct()
    {
        $this->repository = new DatabaseUserRepository();  // 硬编码依赖
    }
}

// IoC 方式
class UserService
{
    public function __construct(
        private UserRepository $repository  // 依赖注入
    ) {
    }
}
```

### 依赖注入（DI）

- **构造函数注入**：通过构造函数注入依赖。
- **属性注入**：通过属性注入依赖（不推荐）。
- **方法注入**：通过方法参数注入依赖。

```php
<?php
declare(strict_types=1);

// 构造函数注入（推荐）
class UserService
{
    public function __construct(
        private UserRepository $repository,
        private EmailService $emailService
    ) {
    }
}

// 方法注入
class OrderController
{
    public function create(Request $request, OrderService $service): Response
    {
        // $service 由容器自动注入
    }
}
```

## 服务容器

### 简单容器实现

```php
<?php
declare(strict_types=1);

class Container
{
    private array $bindings = [];
    private array $singletons = [];
    private array $instances = [];
    
    public function bind(string $abstract, callable|string $concrete, bool $singleton = false): void
    {
        $this->bindings[$abstract] = [
            'concrete' => $concrete,
            'singleton' => $singleton,
        ];
    }
    
    public function singleton(string $abstract, callable|string $concrete): void
    {
        $this->bind($abstract, $concrete, true);
    }
    
    public function make(string $abstract): mixed
    {
        // 检查单例
        if (isset($this->instances[$abstract])) {
            return $this->instances[$abstract];
        }
        
        // 获取绑定
        if (!isset($this->bindings[$abstract])) {
            // 尝试自动解析
            return $this->resolve($abstract);
        }
        
        $binding = $this->bindings[$abstract];
        $concrete = $binding['concrete'];
        
        // 解析实例
        if (is_callable($concrete)) {
            $instance = $concrete($this);
        } elseif (is_string($concrete)) {
            $instance = $this->resolve($concrete);
        } else {
            $instance = $concrete;
        }
        
        // 单例缓存
        if ($binding['singleton']) {
            $this->instances[$abstract] = $instance;
        }
        
        return $instance;
    }
    
    private function resolve(string $class): object
    {
        $reflection = new ReflectionClass($class);
        $constructor = $reflection->getConstructor();
        
        if ($constructor === null) {
            return new $class();
        }
        
        $parameters = $constructor->getParameters();
        $dependencies = [];
        
        foreach ($parameters as $parameter) {
            $type = $parameter->getType();
            
            if ($type instanceof ReflectionNamedType && !$type->isBuiltin()) {
                $dependencies[] = $this->make($type->getName());
            } else {
                throw new RuntimeException("Cannot resolve dependency: {$parameter->getName()}");
            }
        }
        
        return $reflection->newInstanceArgs($dependencies);
    }
}

// 使用
$container = new Container();

// 绑定接口到实现
$container->bind(UserRepository::class, DatabaseUserRepository::class);

// 单例绑定
$container->singleton(EmailService::class, function ($container) {
    return new SmtpEmailService($container->make(Config::class));
});

// 解析
$userService = $container->make(UserService::class);
```

## Provider 模式

```php
<?php
declare(strict_types=1);

interface ServiceProvider
{
    public function register(Container $container): void;
    public function boot(Container $container): void;
}

class DatabaseServiceProvider implements ServiceProvider
{
    public function register(Container $container): void
    {
        $container->singleton(PDO::class, function ($container) {
            $config = $container->make(Config::class);
            return new PDO(
                "mysql:host={$config->dbHost};dbname={$config->dbName}",
                $config->dbUser,
                $config->dbPassword
            );
        });
        
        $container->bind(UserRepository::class, DatabaseUserRepository::class);
    }
    
    public function boot(Container $container): void
    {
        // 启动逻辑
    }
}

// 注册 Provider
$container = new Container();
$container->register(new DatabaseServiceProvider());
```

## 自动注入

### 反射解析

```php
<?php
declare(strict_types=1);

class AutowireResolver
{
    public function resolve(string $class, Container $container): object
    {
        $reflection = new ReflectionClass($class);
        $constructor = $reflection->getConstructor();
        
        if ($constructor === null) {
            return new $class();
        }
        
        $dependencies = $this->resolveDependencies($constructor, $container);
        return $reflection->newInstanceArgs($dependencies);
    }
    
    private function resolveDependencies(ReflectionMethod $constructor, Container $container): array
    {
        $dependencies = [];
        
        foreach ($constructor->getParameters() as $parameter) {
            $type = $parameter->getType();
            
            if ($type instanceof ReflectionNamedType && !$type->isBuiltin()) {
                $dependencies[] = $container->make($type->getName());
            } elseif ($parameter->isDefaultValueAvailable()) {
                $dependencies[] = $parameter->getDefaultValue();
            } else {
                throw new RuntimeException("Cannot resolve: {$parameter->getName()}");
            }
        }
        
        return $dependencies;
    }
}
```

## 练习

1. 实现一个完整的服务容器，支持绑定、单例、自动注入。

2. 创建一个 ServiceProvider 系统，支持服务的注册和启动。

3. 实现依赖注入的自动解析，支持构造函数和类型提示。

4. 设计一个容器作用域系统，支持请求级别的单例。

5. 创建一个依赖图分析工具，检测循环依赖。

6. 实现一个服务装饰器模式，支持中间件式的服务包装。

# 7.1.1 IoC 容器原理

## 概述

IoC（Inversion of Control，控制反转）容器是现代框架的核心。本节介绍控制反转的概念、依赖注入、服务容器的实现原理。

## 控制反转概念

### 传统依赖方式

```php
<?php
declare(strict_types=1);

// 传统方式：类自己创建依赖
class UserService
{
    private EmailService $emailService;
    
    public function __construct()
    {
        // 直接创建依赖，紧耦合
        $this->emailService = new EmailService();
    }
}
```

### 控制反转方式

```php
<?php
declare(strict_types=1);

// IoC 方式：依赖从外部注入
class UserService
{
    public function __construct(
        private EmailServiceInterface $emailService
    ) {}
}

// 容器负责创建和注入依赖
$container = new Container();
$container->bind(EmailServiceInterface::class, SmtpEmailService::class);
$userService = $container->make(UserService::class);
```

## 服务容器

### 基础容器实现

```php
<?php
declare(strict_types=1);

class Container
{
    private array $bindings = [];
    private array $instances = [];
    private array $resolved = [];
    
    /**
     * 绑定接口到实现
     */
    public function bind(string $abstract, mixed $concrete): void
    {
        $this->bindings[$abstract] = $concrete;
    }
    
    /**
     * 绑定单例
     */
    public function singleton(string $abstract, mixed $concrete): void
    {
        $this->bind($abstract, $concrete);
        $this->instances[$abstract] = null;
    }
    
    /**
     * 解析服务
     */
    public function make(string $abstract): mixed
    {
        // 如果是单例且已实例化，直接返回
        if (isset($this->instances[$abstract]) && $this->instances[$abstract] !== null) {
            return $this->instances[$abstract];
        }
        
        // 解析服务
        $instance = $this->resolve($abstract);
        
        // 如果是单例，保存实例
        if (array_key_exists($abstract, $this->instances)) {
            $this->instances[$abstract] = $instance;
        }
        
        return $instance;
    }
    
    /**
     * 解析服务实现
     */
    private function resolve(string $abstract): object
    {
        // 获取具体实现
        $concrete = $this->getConcrete($abstract);
        
        // 如果是闭包，执行闭包
        if ($concrete instanceof Closure) {
            return $concrete($this);
        }
        
        // 如果是类名，递归解析
        if (is_string($concrete)) {
            return $this->build($concrete);
        }
        
        return $concrete;
    }
    
    /**
     * 获取具体实现
     */
    private function getConcrete(string $abstract): mixed
    {
        // 如果已绑定，返回绑定
        if (isset($this->bindings[$abstract])) {
            return $this->bindings[$abstract];
        }
        
        // 如果没有绑定，假设抽象就是具体类
        return $abstract;
    }
    
    /**
     * 构建类实例
     */
    private function build(string $class): object
    {
        $reflection = new ReflectionClass($class);
        
        // 检查是否可以实例化
        if (!$reflection->isInstantiable()) {
            throw new RuntimeException("Class {$class} is not instantiable");
        }
        
        // 获取构造函数
        $constructor = $reflection->getConstructor();
        
        // 如果没有构造函数，直接实例化
        if ($constructor === null) {
            return new $class();
        }
        
        // 解析构造函数参数
        $parameters = $constructor->getParameters();
        $dependencies = $this->resolveDependencies($parameters);
        
        // 实例化类
        return $reflection->newInstanceArgs($dependencies);
    }
    
    /**
     * 解析依赖
     */
    private function resolveDependencies(array $parameters): array
    {
        $dependencies = [];
        
        foreach ($parameters as $parameter) {
            $type = $parameter->getType();
            
            // 如果没有类型提示，尝试使用默认值
            if ($type === null) {
                if ($parameter->isDefaultValueAvailable()) {
                    $dependencies[] = $parameter->getDefaultValue();
                } else {
                    throw new RuntimeException("Cannot resolve dependency: {$parameter->getName()}");
                }
                continue;
            }
            
            // 如果是内置类型，使用默认值
            if ($type->isBuiltin()) {
                if ($parameter->isDefaultValueAvailable()) {
                    $dependencies[] = $parameter->getDefaultValue();
                } else {
                    throw new RuntimeException("Cannot resolve builtin type: {$parameter->getName()}");
                }
                continue;
            }
            
            // 解析类型提示的类
            $typeName = $type->getName();
            $dependencies[] = $this->make($typeName);
        }
        
        return $dependencies;
    }
    
    /**
     * 检查是否已绑定
     */
    public function bound(string $abstract): bool
    {
        return isset($this->bindings[$abstract]);
    }
    
    /**
     * 获取所有绑定
     */
    public function getBindings(): array
    {
        return $this->bindings;
    }
}
```

## 容器使用示例

```php
<?php
declare(strict_types=1);

// 定义接口
interface EmailServiceInterface
{
    public function send(string $to, string $subject, string $body): bool;
}

// 实现接口
class SmtpEmailService implements EmailServiceInterface
{
    public function send(string $to, string $subject, string $body): bool
    {
        return mail($to, $subject, $body);
    }
}

// 使用接口的服务
class UserService
{
    public function __construct(
        private EmailServiceInterface $emailService
    ) {}
    
    public function sendWelcomeEmail(string $email): void
    {
        $this->emailService->send($email, 'Welcome', 'Welcome to our service');
    }
}

// 使用容器
$container = new Container();

// 绑定接口到实现
$container->bind(EmailServiceInterface::class, SmtpEmailService::class);

// 解析服务（自动注入依赖）
$userService = $container->make(UserService::class);
$userService->sendWelcomeEmail('user@example.com');
```

## 单例模式

```php
<?php
declare(strict_types=1);

// 绑定单例
$container->singleton(EmailServiceInterface::class, SmtpEmailService::class);

// 多次解析返回同一实例
$service1 = $container->make(EmailServiceInterface::class);
$service2 = $container->make(EmailServiceInterface::class);

var_dump($service1 === $service2); // true
```

## 闭包绑定

```php
<?php
declare(strict_types=1);

// 使用闭包绑定
$container->bind(EmailServiceInterface::class, function(Container $container) {
    $config = $container->make(Config::class);
    return new SmtpEmailService($config->get('smtp'));
});
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 完整的容器使用示例
interface DatabaseInterface
{
    public function query(string $sql): array;
}

class MySQLDatabase implements DatabaseInterface
{
    public function query(string $sql): array
    {
        // 执行查询
        return [];
    }
}

interface CacheInterface
{
    public function get(string $key): mixed;
    public function set(string $key, mixed $value): void;
}

class RedisCache implements CacheInterface
{
    public function get(string $key): mixed
    {
        // 从 Redis 获取
        return null;
    }
    
    public function set(string $key, mixed $value): void
    {
        // 设置到 Redis
    }
}

class UserRepository
{
    public function __construct(
        private DatabaseInterface $db,
        private CacheInterface $cache
    ) {}
    
    public function find(int $id): ?array
    {
        $cacheKey = "user:{$id}";
        $user = $this->cache->get($cacheKey);
        
        if ($user === null) {
            $user = $this->db->query("SELECT * FROM users WHERE id = {$id}");
            $this->cache->set($cacheKey, $user);
        }
        
        return $user;
    }
}

// 配置容器
$container = new Container();
$container->singleton(DatabaseInterface::class, MySQLDatabase::class);
$container->singleton(CacheInterface::class, RedisCache::class);

// 解析服务（自动注入所有依赖）
$userRepo = $container->make(UserRepository::class);
$user = $userRepo->find(1);
```

## 最佳实践

1. **接口抽象**：使用接口定义服务契约
2. **依赖注入**：通过构造函数注入依赖
3. **单例管理**：合理使用单例模式
4. **延迟解析**：只在需要时解析服务

## 注意事项

1. 避免循环依赖
2. 合理使用单例，避免状态污染
3. 保持容器简单，不要过度设计
4. 使用类型提示，便于自动解析

## 练习

1. 实现一个简单的 IoC 容器，支持依赖注入。

2. 实现单例绑定功能，确保同一服务返回同一实例。

3. 支持闭包绑定，允许使用闭包创建服务。

4. 处理循环依赖，提供清晰的错误信息。

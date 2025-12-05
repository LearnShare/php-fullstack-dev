# 7.1.2 依赖注入

## 概述

依赖注入（Dependency Injection）是实现控制反转的具体方式。本节介绍构造函数注入、属性注入、方法注入、接口注入，以及自动注入的原理。

## 构造函数注入

### 基础用法

```php
<?php
declare(strict_types=1);

class UserService
{
    public function __construct(
        private EmailServiceInterface $emailService,
        private UserRepositoryInterface $userRepository
    ) {}
    
    public function sendWelcomeEmail(int $userId): void
    {
        $user = $this->userRepository->find($userId);
        $this->emailService->send($user->getEmail(), 'Welcome', 'Welcome message');
    }
}
```

### 容器自动注入

```php
<?php
declare(strict_types=1);

// 容器自动解析构造函数参数
$container = new Container();
$container->bind(EmailServiceInterface::class, SmtpEmailService::class);
$container->bind(UserRepositoryInterface::class, UserRepository::class);

$userService = $container->make(UserService::class);
// 自动注入 EmailService 和 UserRepository
```

## 属性注入

### 使用属性注入

```php
<?php
declare(strict_types=1);

class UserService
{
    public EmailServiceInterface $emailService;
    public UserRepositoryInterface $userRepository;
    
    public function sendWelcomeEmail(int $userId): void
    {
        $user = $this->userRepository->find($userId);
        $this->emailService->send($user->getEmail(), 'Welcome', 'Welcome message');
    }
}

// 手动注入
$userService = new UserService();
$userService->emailService = new SmtpEmailService();
$userService->userRepository = new UserRepository();
```

### 使用属性注入的容器

```php
<?php
declare(strict_types=1);

class Container
{
    // ... 其他方法 ...
    
    /**
     * 通过属性注入解析
     */
    public function makeWithProperties(string $class): object
    {
        $instance = $this->build($class);
        
        $reflection = new ReflectionClass($class);
        $properties = $reflection->getProperties();
        
        foreach ($properties as $property) {
            $type = $property->getType();
            
            if ($type instanceof ReflectionNamedType && !$type->isBuiltin()) {
                $typeName = $type->getName();
                if ($this->bound($typeName)) {
                    $property->setAccessible(true);
                    $property->setValue($instance, $this->make($typeName));
                }
            }
        }
        
        return $instance;
    }
}
```

## 方法注入

### 使用方法注入

```php
<?php
declare(strict_types=1);

class UserService
{
    public function sendWelcomeEmail(
        int $userId,
        EmailServiceInterface $emailService
    ): void {
        $user = $this->userRepository->find($userId);
        $emailService->send($user->getEmail(), 'Welcome', 'Welcome message');
    }
}

// 手动注入
$userService = new UserService();
$userService->sendWelcomeEmail(1, new SmtpEmailService());
```

### 容器方法注入

```php
<?php
declare(strict_types=1);

class Container
{
    /**
     * 调用方法并注入依赖
     */
    public function call(object $instance, string $method, array $parameters = []): mixed
    {
        $reflection = new ReflectionClass($instance);
        $methodReflection = $reflection->getMethod($method);
        $methodParameters = $methodReflection->getParameters();
        
        $dependencies = [];
        foreach ($methodParameters as $parameter) {
            $name = $parameter->getName();
            
            // 如果参数已提供，使用提供的值
            if (isset($parameters[$name])) {
                $dependencies[] = $parameters[$name];
                continue;
            }
            
            // 尝试从容器解析
            $type = $parameter->getType();
            if ($type instanceof ReflectionNamedType && !$type->isBuiltin()) {
                $dependencies[] = $this->make($type->getName());
                continue;
            }
            
            // 使用默认值
            if ($parameter->isDefaultValueAvailable()) {
                $dependencies[] = $parameter->getDefaultValue();
                continue;
            }
            
            throw new RuntimeException("Cannot resolve parameter: {$name}");
        }
        
        return $methodReflection->invokeArgs($instance, $dependencies);
    }
}

// 使用
$container = new Container();
$userService = new UserService();
$container->call($userService, 'sendWelcomeEmail', ['userId' => 1]);
```

## 接口注入

### 使用接口注入

```php
<?php
declare(strict_types=1);

interface Injectable
{
    public function inject(Container $container): void;
}

class UserService implements Injectable
{
    private ?EmailServiceInterface $emailService = null;
    
    public function inject(Container $container): void
    {
        $this->emailService = $container->make(EmailServiceInterface::class);
    }
    
    public function sendWelcomeEmail(string $email): void
    {
        $this->emailService->send($email, 'Welcome', 'Welcome message');
    }
}

// 使用
$container = new Container();
$userService = new UserService();
$userService->inject($container);
```

## 自动注入

### 自动解析依赖

```php
<?php
declare(strict_types=1);

class Container
{
    /**
     * 自动解析所有依赖
     */
    public function make(string $abstract): mixed
    {
        return $this->resolve($abstract);
    }
    
    private function resolve(string $abstract): object
    {
        $concrete = $this->getConcrete($abstract);
        
        if ($concrete instanceof Closure) {
            return $concrete($this);
        }
        
        if (is_string($concrete)) {
            return $this->build($concrete);
        }
        
        return $concrete;
    }
    
    private function build(string $class): object
    {
        $reflection = new ReflectionClass($class);
        $constructor = $reflection->getConstructor();
        
        if ($constructor === null) {
            return new $class();
        }
        
        $parameters = $constructor->getParameters();
        $dependencies = $this->resolveDependencies($parameters);
        
        return $reflection->newInstanceArgs($dependencies);
    }
    
    private function resolveDependencies(array $parameters): array
    {
        $dependencies = [];
        
        foreach ($parameters as $parameter) {
            $type = $parameter->getType();
            
            if ($type === null || $type->isBuiltin()) {
                if ($parameter->isDefaultValueAvailable()) {
                    $dependencies[] = $parameter->getDefaultValue();
                } else {
                    throw new RuntimeException("Cannot resolve: {$parameter->getName()}");
                }
                continue;
            }
            
            $typeName = $type->getName();
            $dependencies[] = $this->make($typeName);
        }
        
        return $dependencies;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 定义服务接口
interface LoggerInterface
{
    public function log(string $message): void;
}

interface DatabaseInterface
{
    public function query(string $sql): array;
}

// 实现服务
class FileLogger implements LoggerInterface
{
    public function log(string $message): void
    {
        file_put_contents('app.log', $message . "\n", FILE_APPEND);
    }
}

class MySQLDatabase implements DatabaseInterface
{
    public function query(string $sql): array
    {
        // 执行查询
        return [];
    }
}

// 使用依赖注入的服务
class OrderService
{
    public function __construct(
        private LoggerInterface $logger,
        private DatabaseInterface $db
    ) {}
    
    public function createOrder(array $data): void
    {
        $this->logger->log('Creating order');
        $this->db->query("INSERT INTO orders ...");
        $this->logger->log('Order created');
    }
}

// 配置容器
$container = new Container();
$container->bind(LoggerInterface::class, FileLogger::class);
$container->bind(DatabaseInterface::class, MySQLDatabase::class);

// 自动注入依赖
$orderService = $container->make(OrderService::class);
$orderService->createOrder(['item' => 'Product']);
```

## 最佳实践

1. **优先构造函数注入**：最常用且最清晰
2. **使用接口**：依赖接口而非具体实现
3. **避免属性注入**：除非必要，避免使用属性注入
4. **自动注入**：利用容器自动解析依赖

## 注意事项

1. 避免循环依赖
2. 保持依赖关系清晰
3. 使用类型提示便于自动解析
4. 合理使用单例模式

## 练习

1. 实现构造函数注入，自动解析依赖。

2. 实现方法注入，支持方法参数自动解析。

3. 创建一个依赖注入示例，展示多种注入方式。

4. 处理循环依赖，提供清晰的错误信息。

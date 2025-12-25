# 7.1.3 服务提供者

## 概述

服务提供者是框架中注册和配置服务的机制。本节介绍服务提供者的概念、服务绑定、单例模式、服务解析等内容。

## 服务提供者概念

### 什么是服务提供者

服务提供者负责在容器中注册服务，是框架组织代码的重要方式。

```php
<?php
declare(strict_types=1);

interface ServiceProviderInterface
{
    public function register(Container $container): void;
    public function boot(Container $container): void;
}

abstract class ServiceProvider implements ServiceProviderInterface
{
    protected Container $container;
    
    public function __construct(Container $container)
    {
        $this->container = $container;
    }
    
    public function register(Container $container): void
    {
        // 注册服务
    }
    
    public function boot(Container $container): void
    {
        // 启动服务
    }
}
```

## 服务绑定

### 基础绑定

```php
<?php
declare(strict_types=1);

class EmailServiceProvider extends ServiceProvider
{
    public function register(Container $container): void
    {
        // 绑定接口到实现
        $container->bind(EmailServiceInterface::class, SmtpEmailService::class);
    }
    
    public function boot(Container $container): void
    {
        // 启动时的逻辑
    }
}
```

### 单例绑定

```php
<?php
declare(strict_types=1);

class DatabaseServiceProvider extends ServiceProvider
{
    public function register(Container $container): void
    {
        // 绑定单例
        $container->singleton(DatabaseInterface::class, function(Container $container) {
            $config = $container->make(Config::class);
            return new MySQLDatabase($config->get('database'));
        });
    }
}
```

### 条件绑定

```php
<?php
declare(strict_types=1);

class CacheServiceProvider extends ServiceProvider
{
    public function register(Container $container): void
    {
        // 根据环境绑定不同的实现
        if ($container->make(Config::class)->get('cache.driver') === 'redis') {
            $container->bind(CacheInterface::class, RedisCache::class);
        } else {
            $container->bind(CacheInterface::class, FileCache::class);
        }
    }
}
```

## 服务提供者注册

### 提供者管理器

```php
<?php
declare(strict_types=1);

class ServiceProviderManager
{
    private Container $container;
    private array $providers = [];
    private array $booted = [];
    
    public function __construct(Container $container)
    {
        $this->container = $container;
    }
    
    /**
     * 注册服务提供者
     */
    public function register(string $provider): void
    {
        if (in_array($provider, $this->providers)) {
            return;
        }
        
        $instance = new $provider($this->container);
        $instance->register($this->container);
        
        $this->providers[] = $provider;
    }
    
    /**
     * 启动所有提供者
     */
    public function boot(): void
    {
        foreach ($this->providers as $provider) {
            if (in_array($provider, $this->booted)) {
                continue;
            }
            
            $instance = new $provider($this->container);
            $instance->boot($this->container);
            
            $this->booted[] = $provider;
        }
    }
    
    /**
     * 注册多个提供者
     */
    public function registerProviders(array $providers): void
    {
        foreach ($providers as $provider) {
            $this->register($provider);
        }
    }
}
```

## 延迟加载

### 延迟服务提供者

```php
<?php
declare(strict_types=1);

interface DeferrableProviderInterface
{
    public function provides(): array;
}

class EmailServiceProvider extends ServiceProvider implements DeferrableProviderInterface
{
    public function provides(): array
    {
        return [EmailServiceInterface::class];
    }
    
    public function register(Container $container): void
    {
        $container->bind(EmailServiceInterface::class, SmtpEmailService::class);
    }
}

class ServiceProviderManager
{
    private array $deferred = [];
    
    public function registerDeferred(string $provider): void
    {
        $instance = new $provider($this->container);
        
        if ($instance instanceof DeferrableProviderInterface) {
            foreach ($instance->provides() as $service) {
                $this->deferred[$service] = $provider;
            }
        }
    }
    
    public function loadDeferred(string $service): void
    {
        if (isset($this->deferred[$service])) {
            $this->register($this->deferred[$service]);
            unset($this->deferred[$service]);
        }
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 应用类
class Application
{
    private Container $container;
    private ServiceProviderManager $providers;
    
    public function __construct()
    {
        $this->container = new Container();
        $this->providers = new ServiceProviderManager($this->container);
    }
    
    public function registerProviders(array $providers): void
    {
        $this->providers->registerProviders($providers);
    }
    
    public function boot(): void
    {
        $this->providers->boot();
    }
    
    public function make(string $abstract): mixed
    {
        return $this->container->make($abstract);
    }
}

// 使用
$app = new Application();

// 注册服务提供者
$app->registerProviders([
    EmailServiceProvider::class,
    DatabaseServiceProvider::class,
    CacheServiceProvider::class,
]);

// 启动应用
$app->boot();

// 使用服务
$emailService = $app->make(EmailServiceInterface::class);
```

## 最佳实践

1. **按功能组织**：每个功能模块一个提供者
2. **延迟加载**：非必需服务使用延迟加载
3. **配置分离**：配置和注册逻辑分离
4. **启动顺序**：注意提供者的启动顺序

## 注意事项

1. 提供者应该只注册服务，不执行业务逻辑
2. 启动逻辑应该轻量，避免耗时操作
3. 注意提供者之间的依赖关系
4. 合理使用延迟加载提升性能

## 练习

1. 实现一个服务提供者，注册邮件服务。

2. 创建提供者管理器，支持提供者注册和启动。

3. 实现延迟加载功能，按需加载服务提供者。

4. 创建一个完整的应用，使用服务提供者组织代码。

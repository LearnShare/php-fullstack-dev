# 10.1.7 PSR-11 容器接口

## 概述

PSR-11（PHP Standards Recommendation 11）定义了依赖注入容器的标准接口，实现了不同框架和库之间的容器互操作。

## 官方文档

- **标准名称**：Container Interface
- **状态**：已接受（Accepted）
- **版本**：1.0.0
- **官方链接**：https://www.php-fig.org/psr/psr-11/

## 核心接口

### ContainerInterface

```php
<?php

namespace Psr\Container;

interface ContainerInterface
{
    /**
     * 根据标识符查找容器中的条目并返回
     */
    public function get(string $id);

    /**
     * 如果容器可以返回给定标识符的条目，返回 true
     */
    public function has(string $id): bool;
}
```

### ContainerExceptionInterface

容器异常接口。

```php
<?php

namespace Psr\Container;

interface ContainerExceptionInterface
{
}
```

### NotFoundExceptionInterface

未找到异常接口。

```php
<?php

namespace Psr\Container;

interface NotFoundExceptionInterface extends ContainerExceptionInterface
{
}
```

## 实际应用

### 在库中使用

```php
<?php
declare(strict_types=1);

namespace MyLibrary;

use Psr\Container\ContainerInterface;

class MyService
{
    public function __construct(
        private ContainerInterface $container
    ) {}
    
    public function doSomething(): void
    {
        $logger = $this->container->get('logger');
        $logger->info('Doing something');
    }
}
```

### 实现示例

```php
<?php
declare(strict_types=1);

use Psr\Container\ContainerInterface;
use Psr\Container\NotFoundExceptionInterface;
use Psr\Container\ContainerExceptionInterface;

class SimpleContainer implements ContainerInterface
{
    private array $services = [];
    
    public function get(string $id)
    {
        if (!$this->has($id)) {
            throw new class extends \Exception implements NotFoundExceptionInterface {};
        }
        
        return $this->services[$id];
    }
    
    public function has(string $id): bool
    {
        return isset($this->services[$id]);
    }
    
    public function set(string $id, $service): void
    {
        $this->services[$id] = $service;
    }
}
```

## 实现库

- **PHP-DI**：`php-di/php-di`
- **Pimple**：`pimple/pimple`
- **Laravel Container**：Laravel 框架的容器

## 总结

PSR-11 提供了统一的容器接口，实现了：

- 框架和库之间的容器互操作
- 统一的依赖注入方式
- 更好的代码可测试性
- 解耦和灵活性

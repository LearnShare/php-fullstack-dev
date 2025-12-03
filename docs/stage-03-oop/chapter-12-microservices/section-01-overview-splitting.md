# 3.12.1 微服务概述与服务拆分

## 概述

微服务架构将单一应用拆分为多个小型、独立的服务。理解微服务的核心概念和拆分策略对于设计可扩展的系统至关重要。

## 什么是微服务

### 微服务定义

- **微服务**：将单一应用拆分为多个小型、独立的服务
- **特点**：每个服务独立部署、独立扩展、独立技术栈
- **原则**：单一职责、高内聚低耦合、自治性

### 微服务 vs 单体架构

| 特性           | 单体架构                     | 微服务架构                       |
| :------------- | :--------------------------- | :------------------------------- |
| **部署**       | 单一部署单元                 | 多个独立服务                     |
| **扩展**       | 整体扩展                     | 按服务扩展                       |
| **技术栈**     | 统一技术栈                   | 可混合技术栈                     |
| **复杂度**     | 相对简单                     | 更复杂（服务治理、监控等）       |
| **开发速度**   | 初期快速                     | 需要更多基础设施                 |
| **故障隔离**   | 单点故障影响整体             | 故障隔离，影响范围小             |
| **数据一致性** | 事务保证                     | 最终一致性                       |

## 服务拆分策略

### 按业务领域拆分

```php
<?php
declare(strict_types=1);

// 电商系统拆分示例

// 用户服务
namespace UserService;

class UserController
{
    public function register(array $data): User
    {
        // 用户注册逻辑
    }
    
    public function login(string $email, string $password): Token
    {
        // 用户登录逻辑
    }
}

// 商品服务
namespace ProductService;

class ProductController
{
    public function listProducts(array $filters): array
    {
        // 商品列表逻辑
    }
    
    public function getProduct(string $id): Product
    {
        // 商品详情逻辑
    }
}

// 订单服务
namespace OrderService;

class OrderController
{
    public function createOrder(array $items): Order
    {
        // 创建订单逻辑
        // 需要调用商品服务验证库存
        // 需要调用用户服务验证用户
    }
}
```

### 按数据模型拆分

```php
<?php
declare(strict_types=1);

// 按数据模型拆分

// 用户服务：管理用户数据
class UserService
{
    private UserRepository $userRepository;
    
    public function getUser(string $id): User
    {
        return $this->userRepository->find($id);
    }
}

// 订单服务：管理订单数据
class OrderService
{
    private OrderRepository $orderRepository;
    
    public function getOrder(string $id): Order
    {
        return $this->orderRepository->find($id);
    }
}
```

### 拆分原则

1. **单一职责**：每个服务只负责一个业务领域
2. **高内聚**：相关功能放在同一服务
3. **低耦合**：服务间依赖最小化
4. **数据独立**：每个服务有独立数据库
5. **可独立部署**：服务可独立发布和回滚

## 完整示例

```php
<?php
declare(strict_types=1);

// 用户服务
namespace UserService;

class User
{
    public function __construct(
        public string $id,
        public string $name,
        public string $email
    ) {
    }
}

class UserService
{
    private UserRepository $repository;

    public function getUser(string $id): User
    {
        return $this->repository->find($id);
    }

    public function createUser(string $name, string $email): User
    {
        $user = new User(uniqid(), $name, $email);
        $this->repository->save($user);
        return $user;
    }
}

// 商品服务
namespace ProductService;

class Product
{
    public function __construct(
        public string $id,
        public string $name,
        public float $price,
        public int $stock
    ) {
    }
}

class ProductService
{
    private ProductRepository $repository;

    public function getProduct(string $id): Product
    {
        return $this->repository->find($id);
    }

    public function checkInventory(string $productId, int $quantity): bool
    {
        $product = $this->getProduct($productId);
        return $product->stock >= $quantity;
    }
}
```

## 注意事项

1. **拆分粒度**：服务不应该太小（增加复杂度）或太大（失去微服务优势）。

2. **数据一致性**：微服务之间使用最终一致性，避免分布式事务。

3. **服务边界**：根据业务领域划分服务边界，而不是技术层次。

4. **独立部署**：每个服务应该可以独立部署和扩展。

5. **团队结构**：微服务架构通常需要团队支持（每个团队负责一个或多个服务）。

## 练习

1. 设计一个电商系统的微服务拆分方案，按业务领域拆分。

2. 实现用户服务和商品服务的基本功能。

3. 分析一个现有单体应用，提出微服务拆分建议。

4. 设计服务间的数据共享策略。

5. 创建一个微服务项目的目录结构。

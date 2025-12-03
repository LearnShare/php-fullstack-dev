# 3.9.4 仓储模式

## 概述

仓储（Repository）模式封装数据访问逻辑，提供领域对象的持久化接口。仓储接口定义在领域层，实现放在基础设施层，实现业务逻辑与技术实现的解耦。

## 仓储模式

### 基本概念

- **仓储**：封装数据访问逻辑，提供领域对象的持久化接口。
- 仓储接口定义在领域层，实现放在基础设施层。

```php
namespace App\Domain\Order;

interface OrderRepository
{
    public function save(Order $order): void;
    public function findById(OrderId $id): ?Order;
    public function findByCustomerId(CustomerId $customerId): array;
}
```

## 仓储实现

### 数据库实现

```php
namespace App\Infrastructure\Persistence;

use App\Domain\Order\Order;
use App\Domain\Order\OrderId;
use App\Domain\Order\OrderRepository;
use App\Domain\Order\CustomerId;
use PDO;

class DatabaseOrderRepository implements OrderRepository
{
    public function __construct(private PDO $db)
    {
    }

    public function save(Order $order): void
    {
        $this->db->beginTransaction();
        try {
            // 保存订单
            $stmt = $this->db->prepare(
                'INSERT INTO orders (id, customer_id, total_amount, total_currency) 
                 VALUES (?, ?, ?, ?)
                 ON DUPLICATE KEY UPDATE 
                 customer_id = ?, total_amount = ?, total_currency = ?'
            );
            $stmt->execute([
                $order->getId()->getValue(),
                $order->getCustomerId()->getValue(),
                $order->getTotal()->amount,
                $order->getTotal()->currency,
                $order->getCustomerId()->getValue(),
                $order->getTotal()->amount,
                $order->getTotal()->currency,
            ]);

            // 删除旧订单项
            $stmt = $this->db->prepare('DELETE FROM order_items WHERE order_id = ?');
            $stmt->execute([$order->getId()->getValue()]);

            // 保存订单项
            foreach ($order->getItems() as $item) {
                $stmt = $this->db->prepare(
                    'INSERT INTO order_items (order_id, product_id, quantity, price_amount, price_currency)
                     VALUES (?, ?, ?, ?, ?)'
                );
                $stmt->execute([
                    $order->getId()->getValue(),
                    $item->getProductId()->getValue(),
                    $item->getQuantity(),
                    $item->getPrice()->amount,
                    $item->getPrice()->currency,
                ]);
            }

            $this->db->commit();
        } catch (\Exception $e) {
            $this->db->rollBack();
            throw $e;
        }
    }

    public function findById(OrderId $id): ?Order
    {
        // 查找订单
        $stmt = $this->db->prepare('SELECT * FROM orders WHERE id = ?');
        $stmt->execute([$id->getValue()]);
        $orderData = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($orderData === false) {
            return null;
        }

        // 查找订单项
        $stmt = $this->db->prepare('SELECT * FROM order_items WHERE order_id = ?');
        $stmt->execute([$id->getValue()]);
        $itemsData = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return $this->mapToOrder($orderData, $itemsData);
    }

    public function findByCustomerId(CustomerId $customerId): array
    {
        $stmt = $this->db->prepare('SELECT * FROM orders WHERE customer_id = ?');
        $stmt->execute([$customerId->getValue()]);
        $ordersData = $stmt->fetchAll(PDO::FETCH_ASSOC);

        $orders = [];
        foreach ($ordersData as $orderData) {
            $orderId = new OrderId($orderData['id']);
            $orders[] = $this->findById($orderId);
        }

        return $orders;
    }

    private function mapToOrder(array $orderData, array $itemsData): Order
    {
        $order = new Order(
            new OrderId($orderData['id']),
            new CustomerId($orderData['customer_id'])
        );

        foreach ($itemsData as $itemData) {
            $order->addItem(
                new ProductId($itemData['product_id']),
                $itemData['quantity'],
                new Money($itemData['price_amount'], $itemData['price_currency'])
            );
        }

        return $order;
    }
}
```

### 内存实现（测试用）

```php
namespace App\Infrastructure\Persistence\Test;

use App\Domain\Order\Order;
use App\Domain\Order\OrderRepository;
use App\Domain\Order\OrderId;
use App\Domain\Order\CustomerId;

class InMemoryOrderRepository implements OrderRepository
{
    private array $orders = [];

    public function save(Order $order): void
    {
        $this->orders[$order->getId()->getValue()] = $order;
    }

    public function findById(OrderId $id): ?Order
    {
        return $this->orders[$id->getValue()] ?? null;
    }

    public function findByCustomerId(CustomerId $customerId): array
    {
        return array_filter(
            $this->orders,
            fn($order) => $order->getCustomerId()->equals($customerId)
        );
    }
}
```

## 仓储使用

### 在应用层使用

```php
namespace App\Application;

use App\Domain\Order\Order;
use App\Domain\Order\OrderRepository;
use App\Domain\Order\OrderId;

class OrderService
{
    public function __construct(
        private OrderRepository $orderRepository
    ) {
    }

    public function getOrder(OrderId $id): Order
    {
        $order = $this->orderRepository->findById($id);
        if ($order === null) {
            throw new OrderNotFoundException($id);
        }
        return $order;
    }

    public function createOrder(CustomerId $customerId): Order
    {
        $order = new Order(OrderId::generate(), $customerId);
        $this->orderRepository->save($order);
        return $order;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 领域层：定义仓储接口
namespace App\Domain\User;

interface UserRepository
{
    public function save(User $user): void;
    public function findById(UserId $id): ?User;
    public function findByEmail(Email $email): ?User;
    public function delete(UserId $id): void;
}

// 基础设施层：实现仓储
namespace App\Infrastructure\Persistence;

use App\Domain\User\{User, UserRepository, UserId, Email};
use PDO;

class DatabaseUserRepository implements UserRepository
{
    public function __construct(private PDO $db) {}

    public function save(User $user): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO users (id, email, name) VALUES (?, ?, ?)
             ON DUPLICATE KEY UPDATE email = ?, name = ?'
        );
        $stmt->execute([
            $user->getId()->getValue(),
            $user->getEmail()->getValue(),
            $user->getName(),
            $user->getEmail()->getValue(),
            $user->getName(),
        ]);
    }

    public function findById(UserId $id): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id->getValue()]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);

        return $data ? $this->mapToUser($data) : null;
    }

    public function findByEmail(Email $email): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE email = ?');
        $stmt->execute([$email->getValue()]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);

        return $data ? $this->mapToUser($data) : null;
    }

    public function delete(UserId $id): void
    {
        $stmt = $this->db->prepare('DELETE FROM users WHERE id = ?');
        $stmt->execute([$id->getValue()]);
    }

    private function mapToUser(array $data): User
    {
        return new User(
            new UserId($data['id']),
            new Email($data['email']),
            $data['name']
        );
    }
}
```

## 注意事项

1. **接口定义**：仓储接口定义在领域层，使用领域语言。

2. **实现位置**：仓储实现放在基础设施层。

3. **聚合保存**：保存聚合时，需要保存聚合内的所有对象。

4. **映射逻辑**：在仓储中处理领域对象与数据库记录的映射。

5. **测试适配器**：创建内存实现的仓储用于测试。

## 练习

1. 实现一个用户仓储接口和数据库实现。

2. 创建一个订单仓储，处理订单和订单项的保存和加载。

3. 实现一个内存仓储，用于单元测试。

4. 设计一个产品仓储，支持按分类、价格范围查询。

5. 创建一个完整的仓储系统，包含接口定义、数据库实现和测试实现。

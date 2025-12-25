# 3.9.3 聚合根（Aggregate Root）

## 概述

聚合（Aggregate）是一组相关对象的集合，作为一个整体进行管理。聚合根（Aggregate Root）是聚合的入口点，外部只能通过聚合根访问聚合内的对象。

## 聚合概念

### 基本定义

- **聚合**：一组相关对象的集合，作为一个整体进行管理。
- **聚合根（Aggregate Root）**：聚合的入口点，外部只能通过聚合根访问聚合内的对象。

```php
namespace App\Domain\Order;

class Order // 聚合根
{
    private array $items = [];

    public function __construct(
        private OrderId $id,
        private CustomerId $customerId,
        private Money $total
    ) {
    }

    public function addItem(ProductId $productId, int $quantity, Money $price): void
    {
        // 业务规则：不能添加已存在的商品
        foreach ($this->items as $item) {
            if ($item->getProductId()->equals($productId)) {
                throw new ProductAlreadyInOrderException();
            }
        }

        $item = new OrderItem($productId, $quantity, $price);
        $this->items[] = $item;
        $this->recalculateTotal();
    }

    public function removeItem(ProductId $productId): void
    {
        $this->items = array_filter(
            $this->items,
            fn($item) => !$item->getProductId()->equals($productId)
        );
        $this->recalculateTotal();
    }

    private function recalculateTotal(): void
    {
        $this->total = new Money(0, 'USD');
        foreach ($this->items as $item) {
            $itemTotal = new Money(
                $item->getPrice()->amount * $item->getQuantity(),
                $item->getPrice()->currency
            );
            $this->total = $this->total->add($itemTotal);
        }
    }

    public function getItems(): array
    {
        return $this->items; // 返回副本，保护内部状态
    }
}

class OrderItem // 聚合内的实体
{
    public function __construct(
        private ProductId $productId,
        private int $quantity,
        private Money $price
    ) {
        if ($quantity <= 0) {
            throw new InvalidArgumentException('Quantity must be positive');
        }
    }

    public function getProductId(): ProductId
    {
        return $this->productId;
    }

    public function getQuantity(): int
    {
        return $this->quantity;
    }

    public function getPrice(): Money
    {
        return $this->price;
    }
}
```

## 聚合设计原则

### 1. 通过聚合根访问

- 外部只能通过聚合根访问聚合内的对象。

```php
class Order
{
    private array $items = [];

    // 正确：通过聚合根添加订单项
    public function addItem(ProductId $productId, int $quantity, Money $price): void
    {
        $item = new OrderItem($productId, $quantity, $price);
        $this->items[] = $item;
    }

    // 错误：不应该直接暴露内部集合
    // public function getItems(): array
    // {
    //     return $this->items; // 允许外部直接修改
    // }

    // 正确：返回副本或只读视图
    public function getItems(): array
    {
        return array_map(
            fn($item) => [
                'product_id' => $item->getProductId()->getValue(),
                'quantity' => $item->getQuantity(),
                'price' => $item->getPrice()->amount
            ],
            $this->items
        );
    }
}
```

### 2. 事务边界

- 一个事务只能修改一个聚合。

```php
class OrderService
{
    public function __construct(
        private OrderRepository $orderRepository,
        private ProductRepository $productRepository
    ) {
    }

    public function addItemToOrder(OrderId $orderId, ProductId $productId, int $quantity): void
    {
        // 加载聚合
        $order = $this->orderRepository->findById($orderId);
        if ($order === null) {
            throw new OrderNotFoundException($orderId);
        }

        // 获取商品信息（另一个聚合）
        $product = $this->productRepository->findById($productId);
        if ($product === null) {
            throw new ProductNotFoundException($productId);
        }

        // 通过聚合根修改
        $order->addItem($productId, $quantity, $product->getPrice());

        // 保存聚合（一个事务）
        $this->orderRepository->save($order);
    }
}
```

### 3. 保持一致性

- 聚合内的对象保持一致状态。

```php
class Order
{
    private function recalculateTotal(): void
    {
        // 确保总额与订单项一致
        $this->total = new Money(0, 'USD');
        foreach ($this->items as $item) {
            $itemTotal = new Money(
                $item->getPrice()->amount * $item->getQuantity(),
                $item->getPrice()->currency
            );
            $this->total = $this->total->add($itemTotal);
        }
    }
}
```

## 聚合边界设计

### 设计原则

- **高内聚**：聚合内的对象应该高度相关。
- **低耦合**：聚合之间应该低耦合。
- **一致性边界**：聚合是事务和一致性边界。

```php
// 订单聚合
class Order
{
    private OrderId $id;
    private CustomerId $customerId;
    private array $items; // 订单项属于订单聚合
    private Money $total;
}

// 商品聚合（独立的聚合）
class Product
{
    private ProductId $id;
    private string $name;
    private Money $price;
}

// 订单聚合引用商品聚合（通过 ID），但不包含商品对象
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Domain\Order;

class Order // 聚合根
{
    private array $items = [];
    private OrderStatus $status;

    public function __construct(
        private OrderId $id,
        private CustomerId $customerId
    ) {
        $this->status = OrderStatus::PENDING;
        $this->total = new Money(0, 'USD');
    }

    public function addItem(ProductId $productId, int $quantity, Money $price): void
    {
        // 业务规则：已发货的订单不能修改
        if ($this->status === OrderStatus::SHIPPED) {
            throw new OrderCannotBeModifiedException('Order is already shipped');
        }

        // 业务规则：不能添加已存在的商品
        foreach ($this->items as $item) {
            if ($item->getProductId()->equals($productId)) {
                throw new ProductAlreadyInOrderException();
            }
        }

        $item = new OrderItem($productId, $quantity, $price);
        $this->items[] = $item;
        $this->recalculateTotal();
    }

    public function removeItem(ProductId $productId): void
    {
        if ($this->status === OrderStatus::SHIPPED) {
            throw new OrderCannotBeModifiedException('Order is already shipped');
        }

        $this->items = array_filter(
            $this->items,
            fn($item) => !$item->getProductId()->equals($productId)
        );
        $this->recalculateTotal();
    }

    public function ship(): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new InvalidOrderStatusException('Only pending orders can be shipped');
        }

        if (empty($this->items)) {
            throw new EmptyOrderException('Cannot ship empty order');
        }

        $this->status = OrderStatus::SHIPPED;
    }

    private function recalculateTotal(): void
    {
        $this->total = new Money(0, 'USD');
        foreach ($this->items as $item) {
            $itemTotal = new Money(
                $item->getPrice()->amount * $item->getQuantity(),
                $item->getPrice()->currency
            );
            $this->total = $this->total->add($itemTotal);
        }
    }

    public function getTotal(): Money
    {
        return $this->total;
    }
}

class OrderItem // 聚合内的实体
{
    public function __construct(
        private ProductId $productId,
        private int $quantity,
        private Money $price
    ) {
        if ($quantity <= 0) {
            throw new InvalidArgumentException('Quantity must be positive');
        }
    }

    public function getProductId(): ProductId
    {
        return $this->productId;
    }

    public function getQuantity(): int
    {
        return $this->quantity;
    }

    public function getPrice(): Money
    {
        return $this->price;
    }
}
```

## 注意事项

1. **聚合根访问**：外部只能通过聚合根访问聚合内的对象。

2. **事务边界**：一个事务只能修改一个聚合。

3. **一致性**：聚合内的对象应该保持一致状态。

4. **边界设计**：合理设计聚合边界，避免过大或过小。

5. **引用方式**：聚合之间通过 ID 引用，不直接包含对象。

## 练习

1. 创建一个订单聚合，包含订单项，演示聚合根的作用。

2. 实现一个购物车聚合，包含商品项，提供添加、删除、清空功能。

3. 设计一个用户聚合，包含用户信息和地址列表。

4. 创建一个博客文章聚合，包含文章内容和评论列表。

5. 实现一个订单聚合，包含业务规则（如不能修改已发货的订单）。

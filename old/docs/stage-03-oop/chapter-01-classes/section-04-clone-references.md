# 3.1.4 对象克隆与引用

## 概述

理解对象克隆和引用的区别对于避免意外的副作用至关重要。PHP 中对象赋值是引用传递，需要使用 `clone` 创建独立副本。

## 对象引用

### 引用传递

- 对象赋值是引用传递，多个变量指向同一对象。
- 修改一个变量会影响所有引用该对象的变量。

```php
$user1 = new User(1, 'Alice');
$user2 = $user1; // $user2 与 $user1 指向同一对象
$user2->name = 'Bob';
echo $user1->name; // Bob（被修改了）
```

### 引用示例

```php
class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {
    }
}

$point1 = new Point(10, 20);
$point2 = $point1;  // 引用传递
$point2->x = 30;

echo $point1->x; // 30（被修改）
echo $point2->x; // 30
```

## 对象克隆

### `clone` 关键字

- **语法**：`$clone = clone $object;`
- 创建对象的浅拷贝（shallow copy），属性值被复制，但对象引用仍指向同一对象。
- 如需深拷贝，需实现 `__clone()` 方法。

```php
class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {
    }
}

$point1 = new Point(10, 20);
$point2 = clone $point1;  // 克隆
$point2->x = 30;

echo $point1->x; // 10（未改变）
echo $point2->x; // 30
```

### `__clone` 方法

- 在 `clone` 操作时自动调用，可用于自定义克隆逻辑。

```php
class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function __clone()
    {
        $this->id = 0; // 克隆时重置 ID
    }
}

$user1 = new User(1, 'Alice');
$user2 = clone $user1;
echo $user2->id; // 0
```

### 深拷贝示例

```php
class Address
{
    public function __construct(
        public string $street,
        public string $city
    ) {
    }
}

class User
{
    public function __construct(
        public int $id,
        public string $name,
        public Address $address
    ) {
    }

    public function __clone()
    {
        // 深拷贝：克隆嵌套对象
        $this->address = clone $this->address;
    }
}

$user1 = new User(1, 'Alice', new Address('123 Main St', 'New York'));
$user2 = clone $user1;
$user2->address->city = 'Los Angeles';

echo $user1->address->city; // New York（未改变）
echo $user2->address->city; // Los Angeles
```

## 克隆时更新属性（PHP 8.5+）

### `clone with` 语法

- **语法**：`clone $object with ['property' => 'value']`
- 在克隆对象的同时更新其指定属性，简化不可变对象的更新。

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email
    ) {
    }
}

$user = new User('Alice', 30, 'alice@example.com');

// PHP 8.5+ clone with 语法
$updatedUser = clone $user with ['age' => 31, 'email' => 'alice.new@example.com'];

// 等价于传统写法（需要重新构造）
$updatedUser = new User($user->name, 31, 'alice.new@example.com');
```

### 实际应用示例

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {
    }
    
    // 使用 clone with 创建新点
    public function move(int $dx, int $dy): self
    {
        return clone $this with [
            'x' => $this->x + $dx,
            'y' => $this->y + $dy,
        ];
    }
}

$point = new Point(10, 20);
$newPoint = $point->move(5, -5); // Point(15, 15)
```

## 对象比较

### `==` 和 `===` 的区别

- `==`：比较对象属性值是否相等。
- `===`：比较是否为同一对象实例。

```php
$user1 = new User(1, 'Alice');
$user2 = new User(1, 'Alice');
$user3 = $user1;

var_dump($user1 == $user2);  // true（属性值相同）
var_dump($user1 === $user2); // false（不同实例）
var_dump($user1 === $user3); // true（同一实例）
```

### 比较示例

```php
class Product
{
    public function __construct(
        public int $id,
        public string $name,
        public float $price
    ) {
    }
}

$product1 = new Product(1, 'Laptop', 999.99);
$product2 = new Product(1, 'Laptop', 999.99);
$product3 = $product1;

// 值比较
var_dump($product1 == $product2);  // true

// 引用比较
var_dump($product1 === $product2); // false
var_dump($product1 === $product3); // true
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ShoppingCart
{
    private array $items = [];

    public function addItem(string $item): void
    {
        $this->items[] = $item;
    }

    public function getItems(): array
    {
        return $this->items;
    }

    public function __clone()
    {
        // 深拷贝：克隆数组（虽然数组是值类型，但这里演示概念）
        $this->items = array_map(fn($item) => $item, $this->items);
    }
}

$cart1 = new ShoppingCart();
$cart1->addItem('Product A');
$cart1->addItem('Product B');

$cart2 = clone $cart1;
$cart2->addItem('Product C');

print_r($cart1->getItems()); // ['Product A', 'Product B']
print_r($cart2->getItems()); // ['Product A', 'Product B', 'Product C']
```

## 注意事项

1. **引用传递**：对象赋值是引用传递，修改会影响所有引用。

2. **浅拷贝**：`clone` 默认是浅拷贝，嵌套对象需要手动深拷贝。

3. **性能考虑**：深拷贝会增加内存和 CPU 开销，只在必要时使用。

4. **不可变对象**：使用 `readonly` 类可以创建不可变对象，避免意外修改。

5. **比较操作**：使用 `===` 比较对象引用，使用 `==` 比较对象值。

## 练习

1. 创建一个 `Point` 类，演示对象引用和克隆的区别。

2. 实现一个 `User` 类，包含 `Address` 对象，实现深拷贝确保克隆时 `Address` 也被克隆。

3. 创建一个 `Config` 类，使用 `readonly` 属性，使用 `clone with` 语法创建更新后的配置对象。

4. 实现一个 `ShoppingCart` 类，包含商品列表，确保克隆时创建独立的商品列表。

5. 创建一个 `Node` 类（链表节点），实现深拷贝确保克隆时整个链表都被克隆。

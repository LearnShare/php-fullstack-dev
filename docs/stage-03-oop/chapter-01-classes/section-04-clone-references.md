# 3.1.4 对象克隆与引用

## 概述

理解对象克隆和引用是掌握面向对象编程的重要部分。在 PHP 中，对象赋值是引用赋值，多个变量可以指向同一个对象。当需要对象的独立副本时，需要使用 `clone` 关键字进行对象克隆。本节详细介绍对象引用传递、`clone` 关键字、`__clone` 方法、浅拷贝和深拷贝、`clone with` 语法（PHP 8.5+），以及对象比较等内容。

对象引用和克隆的区别是 PHP 面向对象编程中的核心概念。理解这些概念有助于避免意外的对象修改，正确实现对象复制，以及设计不可变对象。PHP 8.5+ 引入了 `clone with` 语法，使得在克隆对象时修改属性变得更加简洁。

**主要内容**：
- 对象引用的概念和特点
- 对象赋值是引用赋值
- `clone` 关键字的使用
- `__clone` 方法的使用和自定义克隆逻辑
- 浅拷贝和深拷贝的区别
- `clone with` 语法（PHP 8.5+）
- 对象比较（`==` 和 `===`）
- 对象克隆的最佳实践

## 特性

- **对象引用**：对象赋值是引用赋值，多个变量指向同一个对象
- **对象克隆**：使用 `clone` 关键字创建对象的独立副本
- **浅拷贝**：默认克隆只复制对象本身，不复制嵌套对象
- **深拷贝**：手动实现，复制对象及其所有嵌套对象
- **`clone with`**：PHP 8.5+ 特性，克隆时修改属性

## 语法/定义

### 对象引用赋值

**语法**：`$object2 = $object1;`

**特点**：
- 对象赋值是引用赋值
- 两个变量指向同一个对象
- 修改一个变量会影响另一个变量

### clone 关键字

**语法**：`$clone = clone $object;`

**特点**：
- 创建对象的副本
- 新对象和原对象独立
- 修改新对象不影响原对象

### __clone 方法

**语法**：`public function __clone(): void { ... }`

**特点**：
- 对象克隆时自动调用
- 用于自定义克隆逻辑
- 可以实现深拷贝

### clone with 语法（PHP 8.5+）

**语法**：`$clone = clone $object with (property: value, ...);`

**特点**：
- PHP 8.5+ 引入的新特性
- 克隆时修改属性
- 语法简洁清晰

### 对象比较

**语法**：
- `$obj1 == $obj2`：比较对象内容
- `$obj1 === $obj2`：比较对象引用（是否为同一个对象）

## 基本用法

### 示例 1：对象引用赋值

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}

$user1 = new User("John", 25);
$user2 = $user1;  // 引用赋值，$user2 和 $user1 指向同一个对象

echo "Before modification:\n";
echo "user1->name: " . $user1->name . "\n";
echo "user2->name: " . $user2->name . "\n";

$user2->name = "Jane";  // 修改 $user2 也会影响 $user1

echo "\nAfter modification:\n";
echo "user1->name: " . $user1->name . "\n";  // Jane（被影响了）
echo "user2->name: " . $user2->name . "\n";  // Jane
```

**输出**：

```
Before modification:
user1->name: John
user2->name: John

After modification:
user1->name: Jane
user2->name: Jane
```

**说明**：
- `$user2 = $user1` 是引用赋值，两个变量指向同一个对象
- 修改 `$user2` 的属性会影响 `$user1`，因为它们指向同一个对象

### 示例 2：对象克隆

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}

$user1 = new User("John", 25);
$user2 = clone $user1;  // 克隆对象，创建独立副本

echo "Before modification:\n";
echo "user1->name: " . $user1->name . "\n";
echo "user2->name: " . $user2->name . "\n";

$user2->name = "Jane";  // 修改 $user2 不影响 $user1

echo "\nAfter modification:\n";
echo "user1->name: " . $user1->name . "\n";  // John（未被影响）
echo "user2->name: " . $user2->name . "\n";  // Jane
```

**输出**：

```
Before modification:
user1->name: John
user2->name: John

After modification:
user1->name: John
user2->name: Jane
```

**说明**：
- `clone $user1` 创建对象的独立副本
- 修改 `$user2` 不会影响 `$user1`，因为它们是不同的对象

### 示例 3：对象比较

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}

$user1 = new User("John", 25);
$user2 = $user1;       // 引用赋值
$user3 = clone $user1; // 克隆

echo "Reference comparison:\n";
var_dump($user1 === $user2);  // true（同一个对象）
var_dump($user1 === $user3);  // false（不同对象）

echo "\nValue comparison:\n";
var_dump($user1 == $user2);   // true（内容相同）
var_dump($user1 == $user3);   // true（内容相同）

$user3->age = 30;
var_dump($user1 == $user3);   // false（内容不同）
```

**输出**：

```
Reference comparison:
bool(true)
bool(false)

Value comparison:
bool(true)
bool(true)
bool(false)
```

**说明**：
- `===` 比较对象引用（是否为同一个对象）
- `==` 比较对象内容（属性值是否相同）

### 示例 4：__clone 方法

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $id
    ) {}
    
    public function __clone(): void
    {
        // 克隆时生成新的 ID
        $this->id = uniqid('user_', true);
        echo "Object cloned, new ID: {$this->id}\n";
    }
}

$user1 = new User("John", 25, "user_001");
echo "Original ID: {$user1->id}\n";

$user2 = clone $user1;
echo "Cloned ID: {$user2->id}\n";

echo "\nComparison:\n";
echo "Same ID: " . ($user1->id === $user2->id ? "Yes" : "No") . "\n";
```

**输出**：

```
Original ID: user_001
Object cloned, new ID: user_676a1b2c3d4e5
Cloned ID: user_676a1b2c3d4e5

Comparison:
Same ID: No
```

**说明**：
- `__clone()` 方法在对象克隆时自动调用
- 可以在 `__clone()` 中自定义克隆逻辑
- 常用于生成新的 ID 或重新分配资源

### 示例 5：浅拷贝问题

```php
<?php
declare(strict_types=1);

class Address
{
    public function __construct(
        public string $street,
        public string $city
    ) {}
}

class User
{
    public function __construct(
        public string $name,
        public Address $address
    ) {}
}

$user1 = new User("John", new Address("123 Main St", "New York"));
$user2 = clone $user1;  // 浅拷贝

echo "Before modification:\n";
echo "user1->address->city: " . $user1->address->city . "\n";
echo "user2->address->city: " . $user2->address->city . "\n";

$user2->address->city = "Los Angeles";  // 修改嵌套对象

echo "\nAfter modification:\n";
echo "user1->address->city: " . $user1->address->city . "\n";  // Los Angeles（被影响了！）
echo "user2->address->city: " . $user2->address->city . "\n";  // Los Angeles
```

**输出**：

```
Before modification:
user1->address->city: New York
user2->address->city: New York

After modification:
user1->address->city: Los Angeles
user2->address->city: Los Angeles
```

**说明**：
- 默认克隆是浅拷贝，只复制对象本身
- 嵌套对象（如 `Address`）仍然是引用，修改会影响原对象
- 这是浅拷贝的典型问题

### 示例 6：深拷贝实现

```php
<?php
declare(strict_types=1);

class Address
{
    public function __construct(
        public string $street,
        public string $city
    ) {}
}

class User
{
    public function __construct(
        public string $name,
        public Address $address
    ) {}
    
    public function __clone(): void
    {
        // 深拷贝：克隆嵌套对象
        $this->address = clone $this->address;
    }
}

$user1 = new User("John", new Address("123 Main St", "New York"));
$user2 = clone $user1;  // 深拷贝

echo "Before modification:\n";
echo "user1->address->city: " . $user1->address->city . "\n";
echo "user2->address->city: " . $user2->address->city . "\n";

$user2->address->city = "Los Angeles";  // 修改嵌套对象

echo "\nAfter modification:\n";
echo "user1->address->city: " . $user1->address->city . "\n";  // New York（未被影响）
echo "user2->address->city: " . $user2->address->city . "\n";  // Los Angeles
```

**输出**：

```
Before modification:
user1->address->city: New York
user2->address->city: New York

After modification:
user1->address->city: New York
user2->address->city: Los Angeles
```

**说明**：
- 在 `__clone()` 方法中手动克隆嵌套对象
- 这样实现深拷贝，嵌套对象也是独立的副本
- 修改 `$user2->address` 不会影响 `$user1->address`

### 示例 7：clone with 语法（PHP 8.5+）

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email
    ) {}
}

$user1 = new User("John", 25, "john@example.com");
echo "Original user:\n";
echo "Name: {$user1->name}, Age: {$user1->age}, Email: {$user1->email}\n";

// PHP 8.5+ clone with 语法
$user2 = clone $user1 with (age: 26, email: "jane@example.com");

echo "\nCloned user with modifications:\n";
echo "Name: {$user2->name}, Age: {$user2->age}, Email: {$user2->email}\n";

echo "\nOriginal user unchanged:\n";
echo "Name: {$user1->name}, Age: {$user1->age}, Email: {$user1->email}\n";
```

**输出**（PHP 8.5+）：

```
Original user:
Name: John, Age: 25, Email: john@example.com

Cloned user with modifications:
Name: John, Age: 26, Email: jane@example.com

Original user unchanged:
Name: John, Age: 25, Email: john@example.com
```

**说明**：
- `clone with` 语法（PHP 8.5+）允许在克隆时修改属性
- 语法简洁，不需要先克隆再修改
- 原对象保持不变

## 使用场景

### 场景 1：需要对象的独立副本

当需要修改对象但不影响原对象时，使用克隆。

**示例**：用户配置复制

```php
<?php
declare(strict_types=1);

class UserConfig
{
    public function __construct(
        public array $settings,
        public array $preferences
    ) {}
    
    public function __clone(): void
    {
        // 深拷贝数组
        $this->settings = unserialize(serialize($this->settings));
        $this->preferences = unserialize(serialize($this->preferences));
    }
}

$original = new UserConfig(
    ['theme' => 'dark', 'lang' => 'en'],
    ['notifications' => true]
);

$copy = clone $original;
$copy->settings['theme'] = 'light';  // 修改副本不影响原对象
```

### 场景 2：不可变对象模式

创建修改后的新对象，而不是修改原对象。

**示例**：不可变配置对象

```php
<?php
declare(strict_types=1);

class ImmutableConfig
{
    public function __construct(
        private string $host,
        private int $port
    ) {}
    
    public function getHost(): string { return $this->host; }
    public function getPort(): int { return $this->port; }
    
    // 返回新对象而不是修改原对象
    public function withHost(string $host): self
    {
        $new = clone $this;
        $new->host = $host;
        return $new;
    }
    
    public function withPort(int $port): self
    {
        $new = clone $this;
        $new->port = $port;
        return $new;
    }
}

$config = new ImmutableConfig("localhost", 3306);
$newConfig = $config->withPort(5432);  // 创建新对象，原对象不变
```

### 场景 3：资源重新分配

克隆时重新分配资源（如生成新的 ID、重新连接数据库等）。

**示例**：实体克隆

```php
<?php
declare(strict_types=1);

class Entity
{
    public function __construct(
        public string $id,
        public string $name
    ) {}
    
    public function __clone(): void
    {
        // 克隆时生成新的 ID
        $this->id = uniqid('entity_', true);
    }
}

$entity1 = new Entity("entity_001", "Original");
$entity2 = clone $entity1;

echo "Entity 1 ID: {$entity1->id}\n";
echo "Entity 2 ID: {$entity2->id}\n";  // 不同的 ID
```

## 注意事项

### 对象引用 vs 对象克隆

- **对象引用**：两个变量指向同一个对象，修改一个会影响另一个
- **对象克隆**：创建独立副本，修改一个不会影响另一个
- **选择依据**：如果需要独立副本，使用 `clone`；如果需要共享对象，使用引用

**示例**：

```php
<?php
declare(strict_types=1);

class Counter
{
    public function __construct(public int $count = 0) {}
}

// 引用：共享状态
$counter1 = new Counter(0);
$counter2 = $counter1;
$counter2->count++;
echo $counter1->count . "\n";  // 1（被影响了）

// 克隆：独立状态
$counter3 = new Counter(0);
$counter4 = clone $counter3;
$counter4->count++;
echo $counter3->count . "\n";  // 0（未被影响）
```

### 浅拷贝 vs 深拷贝

- **浅拷贝**：只复制对象本身，嵌套对象仍然是引用
- **深拷贝**：复制对象及其所有嵌套对象
- **默认行为**：PHP 的 `clone` 默认是浅拷贝
- **实现深拷贝**：在 `__clone()` 方法中手动克隆嵌套对象

**示例**：

```php
<?php
declare(strict_types=1);

class NestedExample
{
    public function __construct(public array $data) {}
    
    // 浅拷贝（默认）
    // 不实现 __clone() 或只复制简单属性
    
    // 深拷贝（手动实现）
    public function __clone(): void
    {
        // 深拷贝数组（使用序列化方法）
        $this->data = unserialize(serialize($this->data));
        
        // 或者手动克隆数组中的每个对象
        // foreach ($this->data as $key => $value) {
        //     if (is_object($value)) {
        //         $this->data[$key] = clone $value;
        //     }
        // }
    }
}
```

### clone with 语法要求

- **PHP 版本**：需要 PHP 8.5+
- **属性要求**：属性必须是 `public` 或 `readonly`
- **语法格式**：`clone $object with (property: value, ...)`

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public function __construct(
        public string $name,        // public：可以使用 clone with
        private int $age,            // private：不能使用 clone with
        public readonly string $id  // readonly：可以使用 clone with
    ) {}
}

$obj = new Example("John", 25, "001");

// PHP 8.5+
$clone = clone $obj with (name: "Jane", id: "002");
// $clone = clone $obj with (age: 30);  // 错误：private 属性不能使用
```

### 对象比较的注意事项

- **`===`**：比较对象引用，判断是否为同一个对象
- **`==`**：比较对象内容，判断属性值是否相同
- **比较顺序**：比较顺序可能影响结果

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}

$user1 = new User("John", 25);
$user2 = new User("John", 25);
$user3 = $user1;

var_dump($user1 === $user2);  // false（不同对象）
var_dump($user1 === $user3);  // true（同一对象）
var_dump($user1 == $user2);   // true（内容相同）
```

## 常见问题

### 问题 1：意外修改原对象

**症状**：修改克隆对象时，原对象也被修改

**原因**：使用了引用赋值而不是克隆，或者浅拷贝导致嵌套对象共享

**解决方案**：
- 需要独立副本时使用 `clone`
- 嵌套对象需要深拷贝时，在 `__clone()` 中手动克隆

**示例**：

```php
<?php
declare(strict_types=1);

class BadExample
{
    public function __construct(public array $data) {}
}

$obj1 = new BadExample(['key' => 'value']);
$obj2 = clone $obj1;  // 浅拷贝
$obj2->data['key'] = 'new value';

echo $obj1->data['key'] . "\n";  // new value（被影响了！）

class GoodExample
{
    public function __construct(public array $data) {}
    
    public function __clone(): void
    {
        $this->data = unserialize(serialize($this->data));  // 深拷贝
    }
}

$obj3 = new GoodExample(['key' => 'value']);
$obj4 = clone $obj3;
$obj4->data['key'] = 'new value';

echo $obj3->data['key'] . "\n";  // value（未被影响）
```

### 问题 2：浅拷贝导致的问题

**症状**：克隆对象的嵌套对象修改影响原对象

**原因**：默认克隆是浅拷贝，嵌套对象仍然是引用

**解决方案**：在 `__clone()` 方法中实现深拷贝

**示例**：见"示例 6：深拷贝实现"

### 问题 3：循环引用导致的问题

**症状**：对象之间存在循环引用，克隆可能导致问题

**原因**：对象 A 引用对象 B，对象 B 引用对象 A

**解决方案**：在 `__clone()` 中处理循环引用，或使用弱引用

**示例**：

```php
<?php
declare(strict_types=1);

class Node
{
    public function __construct(
        public string $value,
        public ?Node $next = null
    ) {}
    
    public function __clone(): void
    {
        // 如果存在循环引用，需要特殊处理
        if ($this->next !== null) {
            $this->next = clone $this->next;
        }
    }
}

$node1 = new Node("A");
$node2 = new Node("B", $node1);
$node1->next = $node2;  // 循环引用

// 克隆时可能导致无限递归，需要特殊处理
```

## 最佳实践

### 1. 理解引用和克隆的区别

- **需要共享状态**：使用引用赋值
- **需要独立副本**：使用 `clone`
- **不确定时**：优先使用 `clone`，避免意外的对象修改

### 2. 需要深拷贝时实现 __clone 方法

对于包含嵌套对象的类，在 `__clone()` 中实现深拷贝。

**示例**：

```php
<?php
declare(strict_types=1);

class ComplexObject
{
    public function __construct(
        public string $name,
        public array $data,
        public ?self $child = null
    ) {}
    
    public function __clone(): void
    {
        // 深拷贝数组
        $this->data = unserialize(serialize($this->data));
        
        // 深拷贝嵌套对象
        if ($this->child !== null) {
            $this->child = clone $this->child;
        }
    }
}
```

### 3. 使用 clone with 简化代码（PHP 8.5+）

当需要克隆并修改少量属性时，使用 `clone with` 语法。

**示例**：

```php
<?php
declare(strict_types=1);

// PHP 8.5+：简洁的方式
$newUser = clone $user with (age: 26);

// 传统方式：需要多行代码
$newUser = clone $user;
$newUser->age = 26;
```

### 4. 避免不必要的克隆

克隆对象有性能开销，只在需要时使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 不需要克隆的情况：只读取不修改
function displayUser(User $user): void
{
    echo "Name: {$user->name}\n";  // 不需要克隆
}

// 需要克隆的情况：需要修改但不影响原对象
function updateUserAge(User $user, int $newAge): User
{
    $updated = clone $user;
    $updated->age = $newAge;
    return $updated;
}
```

### 5. 在文档中说明克隆行为

对于可能被克隆的类，在文档中说明克隆行为（浅拷贝或深拷贝）。

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 用户类
 * 
 * 注意：clone 操作是浅拷贝。如果包含嵌套对象，需要在 __clone() 中实现深拷贝。
 */
class User
{
    public function __construct(
        public string $name,
        public Address $address
    ) {}
    
    public function __clone(): void
    {
        $this->address = clone $this->address;  // 深拷贝
    }
}
```

## 对比分析

### 对象引用 vs 对象克隆

| 特性         | 对象引用                     | 对象克隆                     |
|:-------------|:-----------------------------|:-----------------------------|
| **赋值方式**  | `$obj2 = $obj1;`              | `$obj2 = clone $obj1;`        |
| **对象数量**  | 一个对象，多个引用            | 多个独立对象                 |
| **修改影响**  | 修改一个影响所有引用          | 修改一个不影响其他对象       |
| **内存使用**  | 节省内存                     | 使用更多内存                 |
| **适用场景**  | 需要共享状态                 | 需要独立副本                 |

### 浅拷贝 vs 深拷贝

| 特性         | 浅拷贝                       | 深拷贝                       |
|:-------------|:-----------------------------|:-----------------------------|
| **复制范围**  | 只复制对象本身                | 复制对象及其所有嵌套对象     |
| **嵌套对象**  | 仍然是引用                   | 独立副本                     |
| **实现方式**  | 默认行为（直接 `clone`）      | 手动实现（在 `__clone()` 中）|
| **性能**      | 较快                         | 较慢                         |
| **适用场景**  | 简单对象                     | 复杂对象（包含嵌套对象）     |

### 对象比较运算符

| 运算符       | 比较内容                     | 返回 true 的情况              |
|:-------------|:-----------------------------|:------------------------------|
| **`===`**    | 对象引用（是否为同一对象）    | 两个变量指向同一个对象        |
| **`==`**     | 对象内容（属性值是否相同）    | 两个对象的属性值相同          |
| **`!==`**    | 对象引用（是否不是同一对象）  | 两个变量指向不同对象          |
| **`!=`**     | 对象内容（属性值是否不同）    | 两个对象的属性值不同          |

## 练习任务

1. **对象引用练习**：创建一个 `Counter` 类，使用引用赋值创建两个变量，修改其中一个，观察另一个的变化。

2. **对象克隆练习**：创建一个 `Product` 类，使用克隆创建副本，修改副本的属性，验证原对象未被影响。

3. **深拷贝实现**：创建一个包含嵌套对象的类（如 `User` 包含 `Address`），实现深拷贝，确保克隆对象的嵌套对象也是独立的。

4. **__clone 方法练习**：创建一个 `Entity` 类，在 `__clone()` 方法中生成新的 ID，验证克隆对象的 ID 与原对象不同。

5. **clone with 练习**（PHP 8.5+）：使用 `clone with` 语法创建一个类的克隆，并在克隆时修改多个属性。

## 相关章节

- **[3.1.1 类与对象基础](section-01-basics.md)**：回顾类与对象的基础知识
- **[3.1.3 构造函数与析构函数](section-03-constructors-destructors.md)**：了解对象的生命周期管理
- **[3.2.2 Readonly 属性](../chapter-02-modern-oop/section-02-readonly.md)**：了解 readonly 属性在克隆中的行为

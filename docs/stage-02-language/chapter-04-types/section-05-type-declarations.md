# 2.4.5 类型声明与联合类型

## 概述

类型声明是 PHP 7.0+ 引入的重要特性，可以提高代码质量和可维护性。本节详细介绍函数参数和返回值类型声明、可选参数、可空类型、联合类型（PHP 8.0+）、属性类型声明（PHP 7.4+）、只读属性（PHP 8.1+）、严格模式等现代 PHP 特性。

理解和使用类型声明可以显著提高代码质量，减少类型相关的错误，提供更好的 IDE 支持，并有助于性能优化。

## 特性

- **类型安全**：类型声明提供编译时类型检查，减少运行时错误
- **代码提示**：IDE 可以提供更好的代码补全和类型提示
- **文档作用**：类型声明本身就是文档，清晰表达函数契约
- **性能优化**：类型声明有助于 JIT 编译器优化（PHP 8.0+）
- **向后兼容**：类型声明不会影响现有代码，可以逐步采用

## 语法/定义

### 函数参数类型声明

**语法**：`function func(Type $param1, Type $param2, ...)`

**支持类型**：
- 标量类型：`bool`、`int`、`float`、`string`
- 复合类型：`array`、`object`、`callable`
- 特殊类型：`null`（仅用于可空类型）
- 类/接口名：自定义类或接口
- 联合类型：`Type1|Type2`（PHP 8.0+）
- 交集类型：`Type1&Type2`（PHP 8.1+）

**特点**：
- 参数必须是指定类型
- 在严格模式下，不会自动转换类型
- 可以与其他特性组合使用（可空类型、默认值等）

### 函数返回值类型声明

**语法**：`function func(): ReturnType`

**支持类型**：
- 所有参数类型声明支持的类型
- `void`：表示函数不返回值（PHP 7.1+）
- `never`：表示函数不会正常返回（PHP 8.1+）

**特点**：
- 函数必须返回指定类型的值
- `void` 表示函数不返回值（或返回 `null`）
- `never` 表示函数会抛出异常或终止程序

### 可空类型

**语法**：`?Type` 或 `Type|null`

**作用**：允许参数或返回值为指定类型或 `null`

**特点**：
- `?Type` 是 `Type|null` 的简写
- 必须显式处理 `null` 值
- 适合表示可选值

### 联合类型（PHP 8.0+）

**语法**：`Type1|Type2|Type3`

**作用**：允许参数或返回值为多个类型之一

**特点**：
- 支持多个类型
- 类型之间是"或"的关系
- 不能包含 `void` 和 `null`（使用 `?Type` 表示可空）

### 属性类型声明（PHP 7.4+）

**语法**：`public Type $property;`

**要求**：
- PHP 7.4+
- 必须提供默认值或使用构造器属性提升
- 支持所有类型声明支持的类型

### 只读属性（PHP 8.1+）

**语法**：`public readonly Type $property;`

**要求**：
- PHP 8.1+
- 只能在初始化时赋值一次
- 适合表示不可变数据

### 严格模式

**语法**：`declare(strict_types=1);`

**作用**：
- 启用严格类型检查
- 必须放在文件的第一行（在 `<?php` 标签之后）
- 只影响当前文件

## 基本用法

### 示例 1：基本类型声明

```php
<?php
declare(strict_types=1);

// 参数类型声明
function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

// 返回值类型声明
function add(int $a, int $b): int
{
    return $a + $b;
}

// void 返回类型
function logMessage(string $message): void
{
    echo "[LOG] {$message}\n";
    // 不需要 return
}

// 使用
echo greet("World");
echo add(10, 20) . "\n";
logMessage("Application started");
```

**执行**：

```bash
php basic-types.php
```

**输出**：

```
Hello, World!
30
[LOG] Application started
```

### 示例 2：可空类型

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public ?int $age = null
    ) {}
}

// 可空参数
function findUser(?int $id): ?User
{
    if ($id === null) {
        return null;
    }
    // 模拟查找用户
    return new User("Alice", 25);
}

// 使用
$user1 = findUser(1);
if ($user1 !== null) {
    echo "Found: {$user1->name}\n";
}

$user2 = findUser(null);
if ($user2 === null) {
    echo "User not found\n";
}
```

### 示例 3：联合类型（PHP 8.0+）

```php
<?php
declare(strict_types=1);

// 联合类型参数
function process(string|int $value): string|int
{
    if (is_string($value)) {
        return strtoupper($value);
    }
    return $value * 2;
}

// 使用
echo process("hello") . "\n";  // HELLO
echo process(5) . "\n";         // 10

// 联合类型返回值
function getValue(bool $flag): string|array
{
    if ($flag) {
        return "string";
    }
    return [1, 2, 3];
}

$result1 = getValue(true);
$result2 = getValue(false);
var_dump($result1);  // string(6) "string"
var_dump($result2);  // array(3) { ... }
```

### 示例 4：属性类型声明（PHP 7.4+）

```php
<?php
declare(strict_types=1);

class User
{
    // 属性类型声明
    public string $name;
    public int $age;
    public ?string $email = null;  // 可空类型，提供默认值
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
}

// 构造器属性提升（PHP 8.0+）
class Product
{
    public function __construct(
        public int $id,
        public string $name,
        public float $price
    ) {}
}

$user = new User("Alice", 25);
$product = new Product(1, "iPhone", 999.99);
```

### 示例 5：只读属性（PHP 8.1+）

```php
<?php
declare(strict_types=1);

class User
{
    public readonly string $name;
    public readonly int $id;
    
    public function __construct(string $name, int $id)
    {
        $this->name = $name;  // 只能在构造器中赋值
        $this->id = $id;
    }
}

$user = new User("Alice", 1);
echo "User: {$user->name}, ID: {$user->id}\n";

// 错误：不能修改只读属性
// $user->name = "Bob";  // Fatal error
```

## 完整代码示例

### 示例 1：类型安全的 API

```php
<?php
declare(strict_types=1);

class ApiClient
{
    public function getUser(int $userId): ?User
    {
        // 模拟 API 调用
        if ($userId > 0) {
            return new User("Alice", $userId);
        }
        return null;
    }
    
    public function createUser(string $name, int $age): User
    {
        // 验证参数
        if (empty($name)) {
            throw new InvalidArgumentException("Name cannot be empty");
        }
        if ($age < 0) {
            throw new InvalidArgumentException("Age cannot be negative");
        }
        
        return new User($name, $age);
    }
}

class User
{
    public function __construct(
        public string $name,
        public int $id
    ) {}
}

$client = new ApiClient();
$user = $client->getUser(1);
if ($user !== null) {
    echo "User: {$user->name}\n";
}
```

### 示例 2：联合类型应用

```php
<?php
declare(strict_types=1);

// 处理多种输入类型
function formatValue(string|int|float $value): string
{
    return match (true) {
        is_string($value) => "String: {$value}",
        is_int($value) => "Integer: {$value}",
        is_float($value) => sprintf("Float: %.2f", $value),
    };
}

echo formatValue("Hello") . "\n";
echo formatValue(42) . "\n";
echo formatValue(99.99) . "\n";

// 返回多种类型
function parseInput(string $input): string|int|float|bool
{
    if ($input === "true" || $input === "false") {
        return $input === "true";
    }
    
    if (is_numeric($input)) {
        if (strpos($input, '.') !== false) {
            return (float) $input;
        }
        return (int) $input;
    }
    
    return $input;
}
```

### 示例 3：严格模式对比

```php
<?php
// 文件1：非严格模式
// file1.php
function add(int $a, int $b): int
{
    return $a + $b;
}

echo add("10", "20") . "\n";  // 30（自动转换）

// 文件2：严格模式
// file2.php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

// echo add("10", "20");  // TypeError（严格模式不允许转换）
echo add(10, 20) . "\n";  // 30（正确）
```

## 使用场景

### 函数参数类型声明

- **API 接口**：明确 API 参数类型，提供类型安全
- **库函数**：为库函数提供类型声明，方便使用
- **内部函数**：提高代码质量和可维护性

### 函数返回值类型声明

- **明确契约**：明确函数返回什么类型
- **错误处理**：区分正常返回和错误情况
- **代码提示**：IDE 可以提供更好的代码补全

### 可空类型

- **可选参数**：表示可选参数
- **可能失败的操作**：表示可能返回 null 的操作
- **数据库查询**：表示可能找不到记录

### 联合类型

- **灵活输入**：接受多种类型的输入
- **多态返回**：根据条件返回不同类型
- **API 兼容**：保持 API 的灵活性

### 属性类型声明

- **数据模型**：为数据模型添加类型声明
- **DTO**：数据传输对象
- **配置类**：配置类属性

### 只读属性

- **不可变对象**：创建不可变对象
- **值对象**：值对象模式
- **配置常量**：配置常量

## 注意事项

### 严格模式

- **必须启用**：使用类型声明时应该启用严格模式
- **文件级别**：严格模式只影响当前文件
- **第一行**：`declare(strict_types=1);` 必须放在文件第一行
- **不会自动转换**：严格模式下不会自动转换类型

### 类型转换

- **非严格模式**：会尝试自动转换类型
- **严格模式**：不会自动转换，类型不匹配会抛出 `TypeError`
- **显式转换**：需要时进行显式类型转换

### 向后兼容

- **可选特性**：类型声明是可选的，不会破坏现有代码
- **逐步采用**：可以逐步为代码添加类型声明
- **混合使用**：可以混合使用有类型声明和无类型声明的代码

### 性能考虑

- **编译时检查**：类型检查在编译时进行，运行时开销很小
- **JIT 优化**：类型声明有助于 JIT 编译器优化（PHP 8.0+）
- **建议使用**：建议为所有函数添加类型声明

## 常见问题

### 问题 1：类型不匹配错误

**症状**：

```
TypeError: Argument #1 must be of type int, string given
```

**原因**：传递的类型与声明不匹配

**错误示例**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

add("10", "20");  // TypeError
```

**解决方法**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

// 方法1：传递正确类型
add(10, 20);

// 方法2：显式转换
add((int) "10", (int) "20");

// 方法3：使用联合类型
function addFlexible(string|int $a, string|int $b): int
{
    return (int) $a + (int) $b;
}
```

### 问题 2：严格模式未启用

**症状**：类型不匹配但没有报错

**原因**：未使用 `declare(strict_types=1);`

**解决方法**：

```php
<?php
declare(strict_types=1);  // 必须放在文件第一行

function add(int $a, int $b): int
{
    return $a + $b;
}
```

### 问题 3：可空类型处理

**症状**：可空类型参数未正确处理 null

**原因**：未检查 null 值

**错误示例**：

```php
<?php
declare(strict_types=1);

function process(?string $value): string
{
    return strtoupper($value);  // 如果 $value 是 null 会报错
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

function process(?string $value): string
{
    if ($value === null) {
        return "";
    }
    return strtoupper($value);
}

// 或使用空合并运算符
function process(?string $value): string
{
    return strtoupper($value ?? "");
}
```

## 最佳实践

### 类型声明

- **始终使用**：为所有函数添加类型声明
- **启用严格模式**：使用 `declare(strict_types=1);`
- **明确类型**：使用最具体的类型，避免使用 `mixed`
- **文档补充**：类型声明是文档，但复杂逻辑仍需注释

### 可空类型

- **明确表示可选**：使用可空类型明确表示可选值
- **处理 null**：始终检查和处理 null 值
- **提供默认值**：可能时提供默认值

### 联合类型

- **适度使用**：不要过度使用联合类型
- **文档说明**：为联合类型添加文档说明
- **类型检查**：在函数内部进行类型检查

### 属性类型

- **为所有属性添加类型**：为类属性添加类型声明
- **使用构造器属性提升**：PHP 8.0+ 使用构造器属性提升简化代码
- **只读属性**：适合不可变数据时使用只读属性

## 对比分析

### 严格模式 vs 非严格模式

| 特性 | 严格模式 | 非严格模式 |
|:-----|:---------|:-----------|
| 类型转换 | 不允许 | 允许 |
| 类型安全 | 高 | 低 |
| 错误检测 | 编译时 | 运行时 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **新项目**：使用严格模式
- **现有项目**：逐步迁移到严格模式
- **库开发**：使用严格模式

### 可空类型 vs 默认值

| 特性 | 可空类型 | 默认值 |
|:-----|:---------|:-------|
| 表示方式 | `?Type` | `Type $param = null` |
| 类型安全 | 是 | 是 |
| 明确性 | 高 | 中 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **可选参数**：使用可空类型或默认值
- **可能失败**：使用可空类型
- **有默认值**：使用默认值

## 相关章节

- **2.4.1 标量类型**：了解基本数据类型
- **2.4.2 复合类型**：了解复合数据类型
- **2.5 类型转换与比较**：了解类型转换规则
- **阶段三：面向对象编程基础**：详细了解类和对象

## 练习任务

1. **基本类型声明练习**：
   - 为函数添加参数和返回值类型声明
   - 启用严格模式
   - 测试类型不匹配的错误
   - 理解类型声明的作用

2. **可空类型练习**：
   - 使用可空类型表示可选参数
   - 处理可空类型的 null 值
   - 实现可能返回 null 的函数
   - 理解可空类型的使用场景

3. **联合类型练习**：
   - 使用联合类型接受多种输入
   - 在函数内部进行类型检查
   - 实现返回多种类型的函数
   - 理解联合类型的灵活性

4. **属性类型练习**：
   - 为类属性添加类型声明
   - 使用构造器属性提升
   - 使用只读属性创建不可变对象
   - 测试属性类型声明

5. **综合练习**：
   - 创建一个完整的类型安全的 API
   - 使用所有类型声明特性
   - 启用严格模式
   - 进行代码审查，确保类型安全

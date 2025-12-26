# 2.14.4 PHP 8.5 新特性与示例

## 概述

PHP 8.5 引入了管道操作符、`clone with`、URI parser 配置支持、OPcache 必选、废弃非规范化类型转换等新特性。

理解 PHP 8.5 的新特性对于编写更现代、更函数式的 PHP 代码非常重要。掌握这些新特性可以帮助提高代码质量、开发效率和代码可读性。

## 特性

- **管道操作符**：函数式编程风格，简化链式调用
- **clone with**：克隆对象时修改属性，简化对象克隆
- **URI parser 配置支持**：改进的 URI 解析配置
- **OPcache 必选**：OPcache 成为必选扩展，提升性能
- **废弃非规范化类型转换**：更严格的类型转换规则

## 语法/定义

### 管道操作符（|>）

**语法**：`$value |> function(%) |> another(%)`

**要求**：PHP 8.5+

**特点**：
- `%` 表示前一步的结果
- 支持链式调用
- 函数式编程风格

### clone with 语法

**语法**：`clone $object with (property: value, ...)`

**要求**：PHP 8.5+

**特点**：
- 克隆对象时修改属性
- 简化对象克隆
- 不可变对象模式

### OPcache 必选

**要求**：PHP 8.5+

**特点**：
- OPcache 成为必选扩展
- 默认启用
- 提升性能

## 基本用法

### 示例 1：管道操作符

```php
<?php
declare(strict_types=1);

// 基本使用
$result = "  hello world  "
    |> trim(%)
    |> strtoupper(%)
    |> substr(%, 0, 5);

echo $result . "\n";  // HELLO

// 链式处理
$numbers = [1, 2, 3, 4, 5];
$result = $numbers
    |> array_map(fn($n) => $n * 2, %)
    |> array_filter(fn($n) => $n > 5, %)
    |> array_sum(%);

echo $result . "\n";  // 18
```

### 示例 2：clone with

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

// 基本使用
$user = new User('John', 25, 'john@example.com');
$newUser = clone $user with (age: 26, email: 'john.new@example.com');

echo $newUser->name . "\n";   // John
echo $newUser->age . "\n";    // 26
echo $newUser->email . "\n";  // john.new@example.com
```

### 示例 3：OPcache 配置

```php
<?php
declare(strict_types=1);

// PHP 8.5+ OPcache 必选
if (function_exists('opcache_get_status')) {
    $status = opcache_get_status();
    if ($status !== false) {
        echo "OPcache is enabled\n";
        echo "Cached scripts: " . $status['opcache_statistics']['num_cached_scripts'] . "\n";
    }
}
```

## 完整代码示例

### 示例 1：管道操作符实现

```php
<?php
declare(strict_types=1);

// 数据处理管道
function processData(string $input): string
{
    return $input
        |> trim(%)
        |> strtolower(%)
        |> ucwords(%)
        |> str_replace(' ', '-', %);
}

// 使用
$result = processData("  HELLO WORLD  ");
echo $result . "\n";  // Hello-World

// 数组处理管道
function processArray(array $data): array
{
    return $data
        |> array_map(fn($item) => $item * 2, %)
        |> array_filter(fn($item) => $item > 10, %)
        |> array_values(%);
}

// 使用
$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
$result = processArray($numbers);
print_r($result);  // Array ( [0] => 12 [1] => 14 [2] => 16 [3] => 18 [4] => 20 )
```

### 示例 2：clone with 使用

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public float $x,
        public float $y
    ) {}
    
    public function move(float $dx, float $dy): Point
    {
        return clone $this with (
            x: $this->x + $dx,
            y: $this->y + $dy
        );
    }
}

// 使用
$point = new Point(0, 0);
$newPoint = $point->move(3, 4);
echo "({$newPoint->x}, {$newPoint->y})\n";  // (3, 4)
```

### 示例 3：函数式编程风格

```php
<?php
declare(strict_types=1);

// 使用管道操作符实现函数式编程
class DataProcessor
{
    public static function process(string $input): string
    {
        return $input
            |> self::sanitize(%)
            |> self::normalize(%)
            |> self::format(%);
    }
    
    private static function sanitize(string $input): string
    {
        return trim($input);
    }
    
    private static function normalize(string $input): string
    {
        return strtolower($input);
    }
    
    private static function format(string $input): string
    {
        return ucwords($input);
    }
}

// 使用
$result = DataProcessor::process("  hello world  ");
echo $result . "\n";  // Hello World
```

## 使用场景

### 管道操作符

- **函数式编程**：实现函数式编程风格
- **数据处理**：处理数据流
- **链式调用**：简化链式调用

### clone with

- **不可变对象**：实现不可变对象模式
- **对象克隆**：简化对象克隆
- **值对象**：实现值对象

### OPcache

- **性能优化**：提升 PHP 性能
- **生产环境**：生产环境必选
- **开发环境**：开发环境也推荐启用

## 注意事项

### 版本要求

- **PHP 8.5+**：所有新特性都需要 PHP 8.5 或更高版本
- **兼容性**：注意向后兼容性
- **迁移**：评估迁移成本

### 管道操作符

- **语法适应**：需要适应新语法
- **可读性**：可能影响代码可读性
- **工具支持**：确保工具支持

### OPcache 配置

- **必选扩展**：OPcache 成为必选
- **默认启用**：默认启用
- **配置优化**：根据项目优化配置

## 常见问题

### 问题 1：管道操作符语法

**症状**：不理解管道操作符语法

**原因**：新语法需要适应

**解决方法**：

```php
<?php
declare(strict_types=1);

// 管道操作符：% 表示前一步的结果
$result = "hello"
    |> strtoupper(%)      // "HELLO"
    |> substr(%, 0, 3);  // "HEL"

// 等价于
$result = substr(strtoupper("hello"), 0, 3);
```

### 问题 2：clone with 限制

**症状**：clone with 使用错误

**原因**：不理解语法或限制

**解决方法**：

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

$user = new User('John', 25);

// 正确：使用 clone with
$newUser = clone $user with (age: 26);

// 错误：不能修改 readonly 属性
// readonly class User { ... }
// $newUser = clone $user with (age: 26);  // 错误
```

## 最佳实践

### 新特性使用

- **了解场景**：了解新特性的适用场景
- **合理使用**：合理使用，不要过度使用
- **逐步采用**：逐步采用新特性

### 代码质量

- **函数式编程**：使用管道操作符实现函数式编程
- **不可变对象**：使用 clone with 实现不可变对象
- **性能优化**：利用 OPcache 提升性能

## 相关章节

- **2.10 函数与作用域**：了解函数基础
- **阶段三：面向对象编程基础**：了解对象和克隆
- **1.5 PHP 配置与扩展**：了解 OPcache 配置

## 练习任务

1. **管道操作符练习**：
   - 使用管道操作符处理数据
   - 实现函数式编程风格
   - 测试链式调用
   - 理解语法规则

2. **clone with 练习**：
   - 使用 clone with 克隆对象
   - 实现不可变对象
   - 测试对象克隆
   - 理解使用场景

3. **实际应用练习**：
   - 在项目中使用新特性
   - 重构现有代码
   - 测试兼容性
   - 评估效果

4. **综合练习**：
   - 创建一个使用 PHP 8.5 新特性的项目
   - 实现各种新特性
   - 进行代码审查
   - 测试功能改进

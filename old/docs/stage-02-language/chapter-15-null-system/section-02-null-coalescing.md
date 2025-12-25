# 2.15.2 空合并运算符

## 概述

空合并运算符（Null Coalescing Operator）`??` 是 PHP 7.0+ 引入的特性，用于提供默认值。它比传统的 `isset()` + 三元运算符更简洁和安全。

## 空合并运算符（??）

### 基本语法

```php
$value = $input ?? $default;
```

### 基本用法

```php
<?php
declare(strict_types=1);

// 如果 $input 不存在或为 null，使用默认值
$name = $input['name'] ?? 'Guest';
$age = $user->age ?? 0;
$email = $_GET['email'] ?? null;
```

### 与 isset() + 三元运算符对比

```php
<?php
declare(strict_types=1);

// 旧方式
$name = isset($input['name']) ? $input['name'] : 'Guest';

// 新方式（推荐）
$name = $input['name'] ?? 'Guest';
```

### 链式使用

```php
<?php
declare(strict_types=1);

// 返回第一个非 null 的值
$value = $a ?? $b ?? $c ?? 'default';

// 实际应用
$config = [
    'database' => [
        'host' => 'localhost'
    ]
];

$host = $config['database']['host'] 
    ?? $config['db_host'] 
    ?? 'localhost';
```

## 空合并赋值运算符（??=）

### 基本语法（PHP 7.4+）

```php
$variable ??= $default;
```

### 基本用法

```php
<?php
declare(strict_types=1);

$name ??= 'Guest';  // 如果 $name 为 null 或未定义，赋值为 'Guest'

$name = 'Alice';
$name ??= 'Guest';  // $name 已有值，不会重新赋值
```

### 实际应用

```php
<?php
declare(strict_types=1);

class Config
{
    private array $settings = [];
    
    public function get(string $key, mixed $default = null): mixed
    {
        return $this->settings[$key] ?? $default;
    }
    
    public function set(string $key, mixed $value): void
    {
        $this->settings[$key] ??= $value;  // 只在未设置时赋值
    }
}
```

## 与三元运算符的区别

### ?? vs ?:

```php
<?php
declare(strict_types=1);

$value = "0";

// ?? 只检查 null 或未定义
$result1 = $value ?? 'default';  // "0"

// ?: 检查 falsy 值
$result2 = $value ?: 'default';  // "default"（因为 "0" 是 falsy）
```

### 优先级

```php
<?php
declare(strict_types=1);

// ?? 优先级低于 ?:
$result = $a ?? $b ? $c : $d;  // 等价于 ($a ?? $b) ? $c : $d

// 建议加括号明确意图
$result = ($a ?? $b) ? $c : $d;
```

## 完整示例

```php
<?php
declare(strict_types=1);

class NullCoalescing
{
    public static function demonstrate(): void
    {
        echo "=== 基本用法 ===\n";
        $input = ['name' => 'Alice'];
        $name = $input['name'] ?? 'Guest';
        $email = $input['email'] ?? 'N/A';
        echo "Name: {$name}, Email: {$email}\n";
        
        echo "\n=== 链式使用 ===\n";
        $value = null ?? null ?? 'default';
        echo $value . "\n";  // default
        
        echo "\n=== ??= 赋值 ===\n";
        $config = [];
        $config['timezone'] ??= 'UTC';
        $config['timezone'] ??= 'Asia/Shanghai';  // 不会覆盖
        echo $config['timezone'] . "\n";  // UTC
        
        echo "\n=== 与 ?: 的区别 ===\n";
        $value = "0";
        echo "??: " . ($value ?? 'default') . "\n";  // 0
        echo "?: " . ($value ?: 'default') . "\n";   // default
    }
}

NullCoalescing::demonstrate();
```

## 注意事项

1. **只检查 null**：`??` 只检查 `null` 或未定义，不检查其他 falsy 值。

2. **优先级**：注意 `??` 的优先级低于 `?:`，组合使用时需加括号。

3. **性能**：`??` 的性能与 `isset()` + 三元运算符相同。

4. **可读性**：`??` 比 `isset()` + 三元运算符更简洁易读。

5. **链式使用**：可以链式使用，返回第一个非 null 的值。

## 练习

1. 创建一个函数，使用 `??` 安全地获取嵌套数组的值。

2. 编写一个函数，使用 `??=` 实现配置的默认值设置。

3. 实现一个函数，比较 `??` 和 `?:` 在处理不同值时的差异。

4. 创建一个函数，使用链式 `??` 实现多级默认值。

5. 编写一个函数，演示 `??` 在处理未定义变量时的优势。

# 2.12.2 空合并运算符

## 概述

空合并运算符（`??`）是 PHP 7.0 引入的特性，用于提供默认值。本节详细介绍空合并运算符的语法、用法、链式使用、空合并赋值运算符（`??=`，PHP 7.4+）、与三元运算符的区别及完整示例。

理解空合并运算符的使用对于编写简洁、安全的代码非常重要。掌握链式使用和空合并赋值运算符可以帮助提高代码可读性和开发效率。

## 特性

- **简洁性**：简化默认值赋值代码
- **安全性**：避免未定义变量错误
- **链式使用**：支持链式使用，检查多个值
- **空合并赋值**：PHP 7.4+ 支持空合并赋值运算符

## 语法/定义

### 空合并运算符（??）

**语法**：`$a ?? $b`

**功能**：
- 如果 `$a` 存在且不为 `null`，返回 `$a`
- 否则返回 `$b`

**要求**：PHP 7.0+

**特点**：
- 只检查 `null`，不检查其他 falsy 值
- 不会触发 `E_NOTICE` 或 `E_WARNING`
- 可以链式使用

### 空合并赋值运算符（??=）

**语法**：`$a ??= $b`

**功能**：
- 如果 `$a` 为 `null` 或未定义，将 `$b` 赋值给 `$a`
- 等价于 `$a = $a ?? $b`

**要求**：PHP 7.4+

**特点**：
- 简化空合并赋值代码
- 只检查 `null`，不检查其他 falsy 值

## 基本用法

### 示例 1：基本使用

```php
<?php
declare(strict_types=1);

// 基本用法
$name = $userName ?? "Guest";
echo $name . "\n";  // 如果 $userName 未定义或为 null，输出 "Guest"

// 数组键检查
$value = $config['key'] ?? "default";
echo $value . "\n";  // 如果键不存在或值为 null，输出 "default"

// 对象属性检查
$email = $user->email ?? "no-email@example.com";
echo $email . "\n";
```

### 示例 2：链式使用

```php
<?php
declare(strict_types=1);

// 链式使用：检查多个值
$value = $a ?? $b ?? $c ?? "default";

// 实际应用：配置读取
$dbHost = $config['database']['host'] 
    ?? $env['DB_HOST'] 
    ?? 'localhost';

// 实际应用：用户信息
$displayName = $user->nickname 
    ?? $user->username 
    ?? $user->email 
    ?? 'Anonymous';
```

### 示例 3：空合并赋值运算符

```php
<?php
declare(strict_types=1);

// 基本用法
$name ??= "Guest";  // 如果 $name 为 null，赋值为 "Guest"

// 等价于
if (!isset($name) || $name === null) {
    $name = "Guest";
}

// 实际应用
$config['timeout'] ??= 30;
$config['retries'] ??= 3;
```

### 示例 4：与三元运算符对比

```php
<?php
declare(strict_types=1);

// 空合并运算符
$name = $userName ?? "Guest";

// 三元运算符（等价但更复杂）
$name = isset($userName) ? $userName : "Guest";

// 注意：三元运算符需要先检查 isset()
// 空合并运算符自动处理未定义变量
```

## 完整代码示例

### 示例 1：配置读取工具

```php
<?php
declare(strict_types=1);

class ConfigReader
{
    public static function get(array $config, string $key, mixed $default = null): mixed
    {
        // 使用空合并运算符
        return $config[$key] ?? $default;
    }
    
    public static function getNested(array $config, array $keys, mixed $default = null): mixed
    {
        $value = $config;
        foreach ($keys as $key) {
            if (!is_array($value) || !isset($value[$key])) {
                return $default;
            }
            $value = $value[$key];
        }
        return $value ?? $default;
    }
}

// 使用
$config = [
    'database' => [
        'host' => 'localhost',
        'port' => 3306
    ]
];

$host = ConfigReader::get($config, 'host', 'localhost');
$port = ConfigReader::getNested($config, ['database', 'port'], 3306);
```

### 示例 2：用户信息处理

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        private ?string $name = null,
        private ?string $email = null,
        private ?string $nickname = null
    ) {}
    
    public function getDisplayName(): string
    {
        // 链式使用空合并运算符
        return $this->nickname 
            ?? $this->name 
            ?? $this->email 
            ?? 'Anonymous';
    }
    
    public function getEmail(): string
    {
        return $this->email ?? 'no-email@example.com';
    }
}

// 使用
$user = new User(null, 'user@example.com', null);
echo $user->getDisplayName() . "\n";  // user@example.com
```

### 示例 3：环境配置

```php
<?php
declare(strict_types=1);

class EnvironmentConfig
{
    public static function load(): array
    {
        $config = [];
        
        // 使用空合并赋值运算符
        $config['app_name'] ??= getenv('APP_NAME') ?: 'My App';
        $config['app_env'] ??= getenv('APP_ENV') ?: 'development';
        $config['debug'] ??= filter_var(getenv('APP_DEBUG') ?: false, FILTER_VALIDATE_BOOLEAN);
        
        // 数据库配置
        $config['db_host'] ??= getenv('DB_HOST') ?: 'localhost';
        $config['db_port'] ??= (int) (getenv('DB_PORT') ?: 3306);
        $config['db_name'] ??= getenv('DB_NAME') ?: 'mydb';
        
        return $config;
    }
}

// 使用
$config = EnvironmentConfig::load();
print_r($config);
```

### 示例 4：表单数据处理

```php
<?php
declare(strict_types=1);

function processFormData(array $data): array
{
    $result = [];
    
    // 使用空合并运算符提供默认值
    $result['name'] = $data['name'] ?? 'Unknown';
    $result['email'] = $data['email'] ?? '';
    $result['age'] = (int) ($data['age'] ?? 0);
    $result['active'] = filter_var($data['active'] ?? false, FILTER_VALIDATE_BOOLEAN);
    
    return $result;
}

// 使用
$formData = ['name' => 'John'];
$processed = processFormData($formData);
print_r($processed);
// Array
// (
//     [name] => John
//     [email] => 
//     [age] => 0
//     [active] => 
// )
```

## 使用场景

### 默认值提供

- **变量默认值**：为变量提供默认值
- **配置读取**：读取配置值，提供默认值
- **可选参数**：处理可选参数

### 链式检查

- **多级配置**：检查多级配置值
- **用户信息**：检查多个用户信息字段
- **环境变量**：检查多个环境变量

### 空合并赋值

- **配置初始化**：初始化配置值
- **变量默认值**：为变量设置默认值
- **条件赋值**：根据条件赋值

## 注意事项

### 只检查 null

- **不检查其他 falsy 值**：只检查 `null`，不检查 `false`、`0`、`""` 等
- **与 empty() 区别**：`empty()` 检查所有 falsy 值，`??` 只检查 `null`
- **理解差异**：理解 `??` 和 `empty()` 的差异

### 优先级

- **优先级较低**：`??` 的优先级较低，注意使用括号
- **结合性**：右结合，从右到左求值
- **使用括号**：复杂表达式使用括号明确优先级

### 链式使用

- **从右到左**：链式使用从右到左求值
- **短路求值**：找到第一个非 null 值后停止
- **性能考虑**：链式使用有性能开销，但通常可忽略

## 常见问题

### 问题 1：只检查 null

**症状**：不理解 `??` 只检查 `null` 的特性

**原因**：与 `empty()` 混淆

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = 0;
$result = $value ?? "default";
echo $result . "\n";  // 输出 0，不是 "default"

// 误以为会输出 "default"
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = 0;

// 空合并运算符：只检查 null
$result = $value ?? "default";
echo $result . "\n";  // 0（因为 0 不是 null）

// 如果需要检查所有 falsy 值，使用三元运算符
$result = ($value == false) ? "default" : $value;
echo $result . "\n";  // default

// 或使用 empty()
$result = empty($value) ? "default" : $value;
echo $result . "\n";  // default
```

### 问题 2：优先级问题

**症状**：表达式结果不符合预期

**原因**：不理解运算符优先级

**错误示例**：

```php
<?php
declare(strict_types=1);

$result = $a ?? $b . " suffix";
// 可能不符合预期，因为 . 的优先级高于 ??
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 使用括号明确优先级
$result = ($a ?? $b) . " suffix";

// 或分开写
$value = $a ?? $b;
$result = $value . " suffix";
```

### 问题 3：链式使用误解

**症状**：链式使用结果不符合预期

**原因**：不理解链式使用的求值顺序

**解决方法**：

```php
<?php
declare(strict_types=1);

// 链式使用：从右到左求值
$result = $a ?? $b ?? $c ?? "default";

// 等价于
$result = $a ?? ($b ?? ($c ?? "default"));

// 实际应用
$displayName = $user->nickname 
    ?? $user->name 
    ?? $user->email 
    ?? 'Anonymous';
// 按顺序检查，找到第一个非 null 值后停止
```

## 最佳实践

### 默认值提供

- **使用空合并运算符**：优先使用 `??` 提供默认值
- **理解检查规则**：理解只检查 `null` 的特性
- **避免过度使用**：不要过度使用，保持代码可读性

### 链式使用

- **合理使用**：合理使用链式检查
- **明确意图**：使用有意义的变量名
- **性能考虑**：注意链式使用的性能开销

### 代码可读性

- **使用括号**：复杂表达式使用括号
- **分开写**：必要时分开写，提高可读性
- **注释说明**：为复杂逻辑添加注释

## 对比分析

### ?? vs 三元运算符

| 特性 | ?? | 三元运算符 |
|:-----|:---|:-----------|
| 语法 | 简洁 | 复杂 |
| 检查 | 只检查 null | 需要先检查 isset() |
| 安全性 | 高（不触发 Notice） | 中（需要检查） |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **默认值提供**：使用 `??`
- **复杂条件**：使用三元运算符

### ?? vs empty()

| 特性 | ?? | empty() |
|:-----|:---|:--------|
| 检查值 | 只检查 null | 检查所有 falsy 值 |
| 返回值 | 原值或默认值 | bool |
| 使用场景 | 提供默认值 | 检查是否为空 |
| 推荐度 | 推荐（默认值） | 推荐（空值检查） |

**选择建议**：
- **提供默认值**：使用 `??`
- **检查是否为空**：使用 `empty()` 或显式检查

## 相关章节

- **2.12.1 isset、empty 与 is_null**：了解变量检查函数
- **2.6.4 三元运算符与空合并运算符**：了解三元运算符
- **2.3.1 变量基础**：了解变量的基本概念

## 练习任务

1. **基本使用练习**：
   - 练习使用 `??` 提供默认值
   - 理解只检查 `null` 的特性
   - 测试各种变量状态
   - 观察运算符行为

2. **链式使用练习**：
   - 练习链式使用 `??`
   - 理解求值顺序
   - 实现多级配置读取
   - 测试各种链式场景

3. **空合并赋值练习**：
   - 练习使用 `??=` 运算符
   - 实现配置初始化
   - 测试各种赋值场景
   - 对比与普通赋值的区别

4. **实际应用练习**：
   - 实现配置读取工具
   - 实现用户信息处理
   - 实现表单数据处理
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用空合并运算符的程序
   - 实现各种默认值处理功能
   - 处理各种边界情况
   - 进行代码审查，确保正确性

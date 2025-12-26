# 2.14.2 PHP 8.3 新特性与示例

## 概述

PHP 8.3 继续改进语言功能。本节详细介绍 `json_validate()`、类常量类型、`#[\Override]`、`mb_str_pad()`、覆盖属性类型等语法增强。

理解 PHP 8.3 的新特性对于编写更高效、更安全的 PHP 代码非常重要。掌握这些新特性可以帮助提高代码质量、开发效率和类型安全性。

## 特性

- **json_validate()**：快速验证 JSON 字符串，无需解析
- **类常量类型**：类常量可以声明类型，提高类型安全
- **#[\Override]**：标记覆盖方法，防止意外错误
- **mb_str_pad()**：多字节字符串填充函数
- **覆盖属性类型**：子类可以覆盖父类属性的类型（更严格）

## 语法/定义

### json_validate() - JSON 验证函数

**语法**：`json_validate(string $json, int $depth = 512, int $flags = 0): bool`

**参数**：
- `$json`：要验证的 JSON 字符串
- `$depth`：最大嵌套深度，默认 512
- `$flags`：JSON 解码标志，默认 0

**返回值**：如果 JSON 有效返回 `true`，否则返回 `false`

**要求**：PHP 8.3+

**特点**：
- 只验证不解析，性能更好
- 不分配内存解析 JSON
- 适合仅需验证的场景

### 类常量类型声明

**语法**：`public const Type CONSTANT_NAME = value;`

**要求**：PHP 8.3+

**特点**：
- 类常量可以声明类型
- 提高类型安全性
- 支持标量类型和类名

### #[\Override] 属性

**语法**：`#[\Override]`

**要求**：PHP 8.3+

**功能**：标记覆盖父类方法的方法

**特点**：
- 如果方法没有覆盖父类方法，会报错
- 防止意外错误
- 提高代码可维护性

### mb_str_pad() - 多字节字符串填充

**语法**：`mb_str_pad(string $string, int $length, string $pad_string = " ", int $pad_type = STR_PAD_RIGHT, ?string $encoding = null): string`

**参数**：
- `$string`：要填充的字符串
- `$length`：目标长度
- `$pad_string`：填充字符串，默认空格
- `$pad_type`：填充类型（`STR_PAD_RIGHT`、`STR_PAD_LEFT`、`STR_PAD_BOTH`）
- `$encoding`：字符编码，默认内部编码

**返回值**：填充后的字符串

**要求**：PHP 8.3+

## 基本用法

### 示例 1：json_validate()

```php
<?php
declare(strict_types=1);

// 验证 JSON 字符串
$json = '{"name": "John", "age": 25}';
if (json_validate($json)) {
    echo "Valid JSON\n";
} else {
    echo "Invalid JSON\n";
}

// 验证无效 JSON
$invalid = '{"name": "John", "age":}';
if (!json_validate($invalid)) {
    echo "Invalid JSON detected\n";
}

// 性能对比：json_validate() vs json_decode()
// json_validate() 只验证不解析，性能更好
```

### 示例 2：类常量类型

```php
<?php
declare(strict_types=1);

class Config
{
    public const string APP_NAME = "My Application";
    public const int MAX_USERS = 100;
    public const float VERSION = 1.0;
    public const bool DEBUG = false;
    public const array SETTINGS = ['key' => 'value'];
}

// 使用
echo Config::APP_NAME . "\n";  // My Application
echo Config::MAX_USERS . "\n";  // 100
```

### 示例 3：#[\Override] 属性

```php
<?php
declare(strict_types=1);

class ParentClass
{
    public function method(): void
    {
        echo "Parent method\n";
    }
}

class ChildClass extends ParentClass
{
    #[\Override]
    public function method(): void
    {
        echo "Child method\n";
    }
}

// 如果方法名拼写错误，会报错
class WrongChild extends ParentClass
{
    #[\Override]
    public function metod(): void  // 错误：方法名拼写错误
    {
        // Fatal error: ChildClass::metod() has #[\Override] attribute, but no matching parent method exists
    }
}
```

### 示例 4：mb_str_pad()

```php
<?php
declare(strict_types=1);

// 基本使用
$str = "Hello";
$padded = mb_str_pad($str, 10, " ", STR_PAD_RIGHT);
echo "'{$padded}'\n";  // 'Hello     '

// 左填充
$padded = mb_str_pad($str, 10, " ", STR_PAD_LEFT);
echo "'{$padded}'\n";  // '     Hello'

// 两端填充
$padded = mb_str_pad($str, 10, " ", STR_PAD_BOTH);
echo "'{$padded}'\n";  // '  Hello   '

// 多字节字符串
$str = "你好";
$padded = mb_str_pad($str, 10, " ", STR_PAD_RIGHT, 'UTF-8');
echo "'{$padded}'\n";  // '你好        '
```

## 完整代码示例

### 示例 1：JSON 验证工具

```php
<?php
declare(strict_types=1);

class JsonValidator
{
    public static function validate(string $json, bool $strict = true): bool
    {
        if ($strict) {
            // PHP 8.3+ 使用 json_validate()
            return json_validate($json);
        }
        
        // 兼容旧版本
        json_decode($json);
        return json_last_error() === JSON_ERROR_NONE;
    }
    
    public static function validateAndDecode(string $json): array
    {
        // 先验证
        if (!self::validate($json)) {
            throw new InvalidArgumentException("Invalid JSON");
        }
        
        // 再解析
        $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        return $data;
    }
}

// 使用
$json = '{"name": "John", "age": 25}';
if (JsonValidator::validate($json)) {
    $data = JsonValidator::validateAndDecode($json);
    print_r($data);
}
```

### 示例 2：类型安全的配置类

```php
<?php
declare(strict_types=1);

class AppConfig
{
    public const string APP_NAME = "My Application";
    public const string APP_VERSION = "1.0.0";
    public const int MAX_USERS = 100;
    public const int MAX_REQUESTS = 1000;
    public const float TIMEOUT = 30.0;
    public const bool DEBUG = false;
    public const array DEFAULT_SETTINGS = [
        'cache' => true,
        'log' => true
    ];
    
    public static function get(string $key): mixed
    {
        return match($key) {
            'name' => self::APP_NAME,
            'version' => self::APP_VERSION,
            'max_users' => self::MAX_USERS,
            'max_requests' => self::MAX_REQUESTS,
            'timeout' => self::TIMEOUT,
            'debug' => self::DEBUG,
            'default_settings' => self::DEFAULT_SETTINGS,
            default => throw new InvalidArgumentException("Unknown config key: {$key}")
        };
    }
}

// 使用
echo AppConfig::APP_NAME . "\n";  // My Application
echo AppConfig::get('max_users') . "\n";  // 100
```

### 示例 3：使用 #[\Override]

```php
<?php
declare(strict_types=1);

abstract class BaseRepository
{
    abstract public function find(int $id): ?object;
    
    public function findAll(): array
    {
        return [];
    }
}

class UserRepository extends BaseRepository
{
    #[\Override]
    public function find(int $id): ?object
    {
        // 实现查找逻辑
        return null;
    }
    
    #[\Override]
    public function findAll(): array
    {
        // 覆盖父类方法
        return [];
    }
}

// 如果方法名错误，会报错
class WrongRepository extends BaseRepository
{
    #[\Override]
    public function findById(int $id): ?object  // 错误：方法名不匹配
    {
        // Fatal error: WrongRepository::findById() has #[\Override] attribute, but no matching parent method exists
    }
}
```

### 示例 4：字符串格式化工具

```php
<?php
declare(strict_types=1);

class StringFormatter
{
    public static function padLeft(string $str, int $length, string $pad = " "): string
    {
        return mb_str_pad($str, $length, $pad, STR_PAD_LEFT);
    }
    
    public static function padRight(string $str, int $length, string $pad = " "): string
    {
        return mb_str_pad($str, $length, $pad, STR_PAD_RIGHT);
    }
    
    public static function padBoth(string $str, int $length, string $pad = " "): string
    {
        return mb_str_pad($str, $length, $pad, STR_PAD_BOTH);
    }
    
    public static function formatTable(array $data): string
    {
        $result = [];
        $maxLength = 0;
        
        foreach ($data as $item) {
            $maxLength = max($maxLength, mb_strlen($item));
        }
        
        foreach ($data as $item) {
            $result[] = mb_str_pad($item, $maxLength, " ", STR_PAD_RIGHT);
        }
        
        return implode("\n", $result);
    }
}

// 使用
$data = ["Name", "Age", "Email"];
echo StringFormatter::formatTable($data);
```

## 使用场景

### json_validate()

- **JSON 验证**：仅需验证 JSON 格式时使用
- **性能优化**：不需要解析 JSON 时使用
- **API 验证**：验证 API 请求中的 JSON

### 类常量类型

- **类型安全**：提高类常量的类型安全性
- **配置管理**：管理类型安全的配置常量
- **代码提示**：IDE 可以提供更好的代码提示

### #[\Override]

- **方法覆盖**：标记覆盖父类方法
- **防止错误**：防止方法名拼写错误
- **代码维护**：提高代码可维护性

### mb_str_pad()

- **多字节字符串**：处理多字节字符串填充
- **格式化输出**：格式化表格、对齐文本
- **国际化**：处理国际化字符串

## 注意事项

### 版本要求

- **PHP 8.3+**：所有新特性都需要 PHP 8.3 或更高版本
- **兼容性**：注意向后兼容性
- **迁移**：评估迁移成本

### json_validate() 性能

- **仅验证**：只验证不解析，性能更好
- **不分配内存**：不分配内存解析 JSON
- **适用场景**：仅需验证时使用

### #[\Override] 使用

- **必须覆盖**：标记的方法必须覆盖父类方法
- **防止错误**：防止方法名拼写错误
- **可选使用**：不是强制要求，但推荐使用

## 常见问题

### 问题 1：json_validate() vs json_decode()

**症状**：不理解两者的区别

**原因**：功能相似但用途不同

**解决方法**：

```php
<?php
declare(strict_types=1);

$json = '{"name": "John"}';

// json_validate()：只验证，不解析
if (json_validate($json)) {
    echo "Valid JSON\n";
}

// json_decode()：验证并解析
$data = json_decode($json, true);
if ($data !== null) {
    echo "Valid JSON and parsed\n";
}

// 性能：json_validate() 更快（只验证）
// 使用：仅需验证时使用 json_validate()
```

### 问题 2：类常量类型限制

**症状**：某些类型不支持

**原因**：类常量类型有限制

**解决方法**：

```php
<?php
declare(strict_types=1);

class Config
{
    // 支持的类型
    public const string NAME = "App";
    public const int COUNT = 100;
    public const float VERSION = 1.0;
    public const bool DEBUG = false;
    public const array SETTINGS = [];
    
    // 不支持的类型（使用注释）
    /** @var callable */
    public const CALLBACK = null;
}
```

## 最佳实践

### 新特性使用

- **了解场景**：了解新特性的适用场景
- **性能考虑**：考虑性能影响
- **逐步采用**：逐步采用新特性

### 代码质量

- **类型安全**：使用类常量类型提高类型安全
- **错误预防**：使用 `#[\Override]` 防止错误
- **性能优化**：使用 `json_validate()` 优化性能

## 相关章节

- **2.4.6 JSON**：了解 JSON 处理
- **2.7.2 字符串函数**：了解字符串函数
- **阶段三：面向对象编程基础**：了解类和方法

## 练习任务

1. **json_validate() 练习**：
   - 使用 `json_validate()` 验证 JSON
   - 对比 `json_decode()` 的性能
   - 实现 JSON 验证工具
   - 测试各种 JSON 格式

2. **类常量类型练习**：
   - 创建带类型声明的类常量
   - 测试类型检查
   - 实现配置类
   - 测试 IDE 支持

3. **#[\Override] 练习**：
   - 使用 `#[\Override` 标记覆盖方法
   - 测试错误检测
   - 重构现有代码
   - 提高代码质量

4. **实际应用练习**：
   - 在项目中使用新特性
   - 优化 JSON 验证性能
   - 提高类型安全性
   - 测试兼容性

5. **综合练习**：
   - 创建一个使用 PHP 8.3 新特性的项目
   - 实现各种新特性
   - 进行代码审查
   - 测试性能改进

# 2.11.1 模块基础

## 概述

模块化是组织代码的重要方式。本节详细介绍模块的概念、优势、如何编写独立的文件/模块（函数、类、配置、常量）、文件组织和目录结构、模块设计原则。

理解模块化的概念和原则对于编写可维护、可复用的代码非常重要。掌握模块设计原则可以帮助组织清晰的代码结构。

## 特性

- **独立性**：模块是独立的代码单元
- **可复用性**：模块可以在不同项目中复用
- **可维护性**：模块化代码易于维护
- **可测试性**：模块化代码易于测试

## 语法/定义

### 模块概念

**定义**：模块是独立的代码单元，可以被其他代码导入和使用

**特点**：
- 包含相关的功能代码
- 有明确的职责
- 可以被其他模块导入
- 可以导出函数、类、常量等

### 模块类型

#### 函数模块

**特点**：包含相关函数的文件

**示例**：
```php
<?php
declare(strict_types=1);

// math.php - 数学函数模块
function add(int $a, int $b): int
{
    return $a + $b;
}

function subtract(int $a, int $b): int
{
    return $a - $b;
}
```

#### 类模块

**特点**：包含类的文件

**示例**：
```php
<?php
declare(strict_types=1);

// User.php - 用户类模块
class User
{
    public function __construct(
        private string $name,
        private int $age
    ) {}
    
    public function getName(): string
    {
        return $this->name;
    }
}
```

#### 配置模块

**特点**：包含配置常量的文件

**示例**：
```php
<?php
declare(strict_types=1);

// config.php - 配置模块
const DB_HOST = "localhost";
const DB_NAME = "mydb";
const DB_USER = "user";
const DB_PASS = "password";
```

#### 工具模块

**特点**：包含工具函数的文件

**示例**：
```php
<?php
declare(strict_types=1);

// utils.php - 工具模块
function formatDate(string $date): string
{
    return date('Y-m-d', strtotime($date));
}

function sanitizeInput(string $input): string
{
    return htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
}
```

## 基本用法

### 示例 1：函数模块

```php
<?php
declare(strict_types=1);

// math.php
function add(int $a, int $b): int
{
    return $a + $b;
}

function multiply(int $a, int $b): int
{
    return $a * $b;
}

function divide(int $a, int $b): float
{
    if ($b == 0) {
        throw new DivisionByZeroError("Division by zero");
    }
    return $a / $b;
}
```

### 示例 2：配置模块

```php
<?php
declare(strict_types=1);

// config.php
const APP_NAME = "My Application";
const APP_VERSION = "1.0.0";
const DEBUG_MODE = false;

// 数据库配置
const DB_CONFIG = [
    'host' => 'localhost',
    'port' => 3306,
    'name' => 'mydb',
    'user' => 'user',
    'pass' => 'password'
];
```

### 示例 3：类模块

```php
<?php
declare(strict_types=1);

// User.php
class User
{
    public function __construct(
        private string $name,
        private int $age,
        private string $email
    ) {}
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
}
```

## 完整代码示例

### 示例 1：模块化项目结构

```
project/
├── config/
│   ├── database.php
│   ├── app.php
│   └── cache.php
├── functions/
│   ├── math.php
│   ├── string.php
│   └── array.php
├── classes/
│   ├── User.php
│   ├── Database.php
│   └── Logger.php
├── utils/
│   ├── helpers.php
│   └── validators.php
└── index.php
```

### 示例 2：配置模块

```php
<?php
declare(strict_types=1);

// config/database.php
return [
    'host' => getenv('DB_HOST') ?: 'localhost',
    'port' => (int) (getenv('DB_PORT') ?: 3306),
    'name' => getenv('DB_NAME') ?: 'mydb',
    'user' => getenv('DB_USER') ?: 'user',
    'pass' => getenv('DB_PASS') ?: 'password',
    'charset' => 'utf8mb4'
];

// config/app.php
return [
    'name' => 'My Application',
    'version' => '1.0.0',
    'env' => getenv('APP_ENV') ?: 'development',
    'debug' => filter_var(getenv('APP_DEBUG') ?: false, FILTER_VALIDATE_BOOLEAN)
];
```

### 示例 3：函数模块

```php
<?php
declare(strict_types=1);

// functions/string.php
function truncate(string $str, int $length, string $suffix = '...'): string
{
    if (mb_strlen($str) <= $length) {
        return $str;
    }
    return mb_substr($str, 0, $length - mb_strlen($suffix)) . $suffix;
}

function slugify(string $str): string
{
    $str = mb_strtolower($str);
    $str = preg_replace('/[^a-z0-9]+/', '-', $str);
    return trim($str, '-');
}

// functions/math.php
function percentage(float $part, float $total): float
{
    if ($total == 0) {
        return 0.0;
    }
    return ($part / $total) * 100;
}

function roundTo(float $value, int $decimals = 2): float
{
    return round($value, $decimals);
}
```

## 使用场景

### 代码组织

- **功能分组**：将相关功能组织在一起
- **职责分离**：每个模块职责单一
- **结构清晰**：清晰的目录结构

### 代码复用

- **跨项目复用**：在不同项目中复用模块
- **团队共享**：团队共享通用模块
- **库开发**：开发可复用的库

### 团队协作

- **并行开发**：团队成员可以并行开发不同模块
- **代码审查**：模块化代码易于审查
- **版本控制**：模块化代码易于版本控制

## 注意事项

### 单一职责

- **一个模块一个职责**：每个模块应该只有一个明确的职责
- **避免混合**：避免在模块中混合不相关的功能
- **清晰边界**：模块之间应该有清晰的边界

### 命名规范

- **有意义的文件名**：使用有意义的文件名
- **一致的命名**：保持命名一致性
- **描述性名称**：文件名应该描述模块的功能

### 依赖管理

- **最小依赖**：模块应该最小化依赖
- **明确依赖**：明确声明模块的依赖
- **避免循环**：避免循环依赖

## 常见问题

### 问题 1：模块职责不清

**症状**：模块包含过多不相关的功能

**原因**：没有遵循单一职责原则

**错误示例**：

```php
<?php
declare(strict_types=1);

// utils.php - 包含多种不相关的功能
function formatDate(string $date): string { }
function calculateTax(float $amount): float { }
function sendEmail(string $to, string $subject): void { }
function validateInput(string $input): bool { }
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// date.php - 日期相关功能
function formatDate(string $date): string { }

// tax.php - 税务计算功能
function calculateTax(float $amount): float { }

// email.php - 邮件发送功能
function sendEmail(string $to, string $subject): void { }

// validation.php - 验证功能
function validateInput(string $input): bool { }
```

### 问题 2：文件组织混乱

**症状**：文件组织混乱，难以找到代码

**原因**：没有清晰的目录结构

**解决方法**：

```
project/
├── config/        # 配置文件
├── functions/     # 函数模块
├── classes/       # 类模块
├── utils/         # 工具模块
└── index.php      # 入口文件
```

## 最佳实践

### 模块设计

- **单一职责**：每个模块职责单一
- **高内聚低耦合**：模块内部高内聚，模块间低耦合
- **接口清晰**：模块接口清晰明确

### 文件组织

- **按功能组织**：按功能组织文件
- **清晰的目录结构**：使用清晰的目录结构
- **命名规范**：遵循命名规范

### 代码质量

- **文档完善**：为模块添加文档
- **测试覆盖**：为模块编写测试
- **版本控制**：使用版本控制管理模块

## 对比分析

### 模块化 vs 非模块化

| 特性 | 模块化 | 非模块化 |
|:-----|:-------|:---------|
| 可维护性 | 高 | 低 |
| 可复用性 | 高 | 低 |
| 可测试性 | 高 | 低 |
| 代码组织 | 清晰 | 混乱 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **所有项目**：使用模块化组织代码
- **大型项目**：模块化是必需的

## 相关章节

- **2.11.2 include 与 require**：了解如何导入模块
- **2.11.3 使用导入的内容**：了解如何使用导入的内容
- **阶段三：面向对象编程基础**：深入学习模块化

## 练习任务

1. **模块设计练习**：
   - 设计模块化的项目结构
   - 创建不同类型的模块
   - 理解模块设计原则
   - 测试模块的独立性

2. **文件组织练习**：
   - 组织项目文件结构
   - 按功能分组文件
   - 创建清晰的目录结构
   - 测试文件组织效果

3. **实际应用练习**：
   - 创建配置模块
   - 创建函数模块
   - 创建类模块
   - 测试模块的使用

4. **综合练习**：
   - 创建一个模块化的小项目
   - 实现各种类型的模块
   - 组织清晰的目录结构
   - 进行代码审查，确保模块化质量

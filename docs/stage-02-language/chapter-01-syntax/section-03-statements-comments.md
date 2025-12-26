# 2.1.3 语句与注释

## 概述

语句和注释是 PHP 代码的基本组成部分。本节详细介绍 PHP 语句的规则、分号的使用、三种注释类型（单行、多行、PHPDoc）、PSR-12 代码风格要求，帮助养成规范的代码风格。

理解语句规则和注释用法是编写规范代码的基础。正确的语句格式和注释习惯不仅能提高代码可读性，还能帮助团队协作和代码维护。

## 特性

- **语句规则**：PHP 语句以分号结尾，控制结构不需要分号
- **注释类型**：支持单行、多行、PHPDoc 三种注释类型
- **代码风格**：遵循 PSR-12 标准，确保代码风格统一
- **文档注释**：PHPDoc 注释提供完整的 API 文档支持

## 语法/定义

### 语句规则

**基本规则**：
- PHP 语句以分号（`;`）结尾
- 最后一个语句的分号可以省略（不推荐，不符合 PSR-12）
- 控制结构（`if`、`for`、`while` 等）不需要分号
- 函数/方法定义不需要分号

**语句示例**：

```php
<?php
declare(strict_types=1);

// 赋值语句：需要分号
$name = "World";

// 函数调用语句：需要分号
echo "Hello, {$name}!\n";

// 控制结构：不需要分号
if ($name === "World") {
    echo "Hello, World!\n";
}
```

### 注释类型

#### 单行注释

**语法**：`// 注释内容`

**特点**：
- 从 `//` 开始到行尾都是注释
- 可以放在代码行的末尾
- 可以单独成行

**用途**：
- 简短说明代码功能
- 临时禁用代码
- 解释复杂逻辑

#### 多行注释

**语法**：`/* 注释内容 */`

**特点**：
- 可以跨越多行
- 可以嵌套（PHP 不支持嵌套多行注释）
- 可以放在代码的任何位置

**用途**：
- 详细说明代码块
- 临时禁用大段代码
- 版权信息、许可证声明

#### PHPDoc 注释

**语法**：`/** 注释内容 */`

**特点**：
- 以 `/**` 开始（注意是两个星号）
- 支持特殊标签（`@param`、`@return`、`@throws` 等）
- 可以被 IDE 和文档生成工具识别

**用途**：
- 函数、类、属性的文档注释
- API 文档生成
- IDE 代码提示

## 基本用法

### 语句示例

**示例 1：基本语句**

```php
<?php
declare(strict_types=1);

// 变量赋值语句
$name = "World";

// 函数调用语句
echo "Hello, {$name}!\n";

// 表达式语句
$result = 1 + 2;
```

**示例 2：控制结构（不需要分号）**

```php
<?php
declare(strict_types=1);

// if 语句：不需要分号
if ($condition) {
    echo "Condition is true\n";
}

// for 循环：不需要分号
for ($i = 0; $i < 10; $i++) {
    echo $i . "\n";
}

// while 循环：不需要分号
while ($i < 10) {
    echo $i . "\n";
    $i++;
}
```

**示例 3：函数定义（不需要分号）**

```php
<?php
declare(strict_types=1);

// 函数定义：不需要分号
function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

// 函数调用：需要分号
echo greet("World");
```

### 注释示例

**示例 1：单行注释**

```php
<?php
declare(strict_types=1);

// 这是变量定义
$name = "World";

// 这是输出语句
echo "Hello, {$name}!\n";

// 临时禁用代码
// echo "This line is disabled";
```

**示例 2：多行注释**

```php
<?php
declare(strict_types=1);

/*
 * 这是多行注释
 * 可以跨越多行
 * 用于详细说明代码功能
 */
$name = "World";

/*
 * 临时禁用大段代码
 * echo "Line 1";
 * echo "Line 2";
 * echo "Line 3";
 */
```

**示例 3：PHPDoc 注释**

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 *
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 * @throws TypeError 如果参数类型不匹配
 */
function add(int $a, int $b): int
{
    return $a + $b;
}

/**
 * 用户类
 *
 * @package App\Models
 */
class User
{
    /**
     * 用户名称
     *
     * @var string
     */
    private string $name;

    /**
     * 获取用户名称
     *
     * @return string 用户名称
     */
    public function getName(): string
    {
        return $this->name;
    }
}
```

## 完整代码示例

### 示例 1：语句和注释的综合使用

**文件**：`example.php`

```php
<?php
declare(strict_types=1);

// 变量定义
$name = "World";
$age = 25;

// 输出信息
echo "Name: {$name}, Age: {$age}\n";

/**
 * 计算年龄的平方
 *
 * @param int $age 年龄
 * @return int 年龄的平方
 */
function squareAge(int $age): int
{
    return $age * $age;
}

// 调用函数
$squared = squareAge($age);
echo "Squared age: {$squared}\n";
```

**执行**：

```bash
php example.php
```

**输出**：

```
Name: World, Age: 25
Squared age: 625
```

### 示例 2：PSR-12 代码风格

**文件**：`psr12-example.php`

```php
<?php
declare(strict_types=1);

namespace App\Example;

/**
 * 示例类
 *
 * @package App\Example
 */
class Example
{
    /**
     * 示例方法
     *
     * @param string $name 名称
     * @return string 问候语
     */
    public function greet(string $name): string
    {
        // 检查名称是否为空
        if ($name === '') {
            return 'Hello, Anonymous!';
        }

        return "Hello, {$name}!";
    }
}
```

**说明**：
- 遵循 PSR-12 代码风格
- 使用 PHPDoc 注释
- 代码结构清晰
- 详细说明见 [2.13 代码规范](../chapter-13-standards/readme.md)

## 使用场景

### 单行注释

**适用场景**：
- 简短说明代码功能
- 解释变量含义
- 临时禁用单行代码
- 标记 TODO、FIXME 等

**示例**：

```php
<?php
declare(strict_types=1);

// TODO: 添加错误处理
$result = calculate();

// FIXME: 修复类型转换问题
$value = (int) $input;

// 临时禁用：等待 API 更新
// $response = apiCall();
```

### 多行注释

**适用场景**：
- 详细说明复杂算法
- 解释代码块的功能
- 临时禁用大段代码
- 版权信息、许可证声明

**示例**：

```php
<?php
declare(strict_types=1);

/*
 * 这个函数实现了快速排序算法
 * 时间复杂度：O(n log n)
 * 空间复杂度：O(log n)
 */
function quickSort(array $arr): array
{
    // 实现代码
}
```

### PHPDoc 注释

**适用场景**：
- 公共 API 文档
- 类、方法、属性的文档
- IDE 代码提示
- 文档生成工具

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 *
 * 提供用户相关的业务逻辑处理
 *
 * @package App\Services
 */
class UserService
{
    /**
     * 根据 ID 获取用户
     *
     * @param int $id 用户 ID
     * @return User|null 用户对象，如果不存在返回 null
     * @throws InvalidArgumentException 如果 ID 无效
     */
    public function getUserById(int $id): ?User
    {
        // 实现代码
    }
}
```

## 注意事项

### 分号规则

- **语句必须加分号**：所有语句（赋值、函数调用、表达式）都必须以分号结尾
- **控制结构不需要分号**：`if`、`for`、`while`、`function` 等控制结构不需要分号
- **最后一个语句**：虽然可以省略，但不推荐，不符合 PSR-12 标准

### 注释规范

- **使用有意义的注释**：注释应解释"为什么"而不是"是什么"
- **避免无意义的注释**：不要注释显而易见的代码
- **保持注释更新**：代码修改时，同步更新注释
- **遵循 PSR-12 标准**：注释格式应符合 PSR-12 要求

### PHPDoc 规范

- **为公共 API 编写 PHPDoc**：所有公共方法、类都应提供 PHPDoc
- **使用标准标签**：使用 `@param`、`@return`、`@throws` 等标准标签
- **保持完整**：PHPDoc 应包含所有必要信息
- **详细说明**：见 [2.13.2 PHPDoc 规范](../chapter-13-standards/section-02-phpdoc.md)

### 代码风格

- **遵循 PSR-12 标准**：确保代码风格统一
- **使用 EditorConfig**：统一编辑器配置
- **使用代码格式化工具**：如 PHP-CS-Fixer
- **详细说明**：见 [2.13 代码规范](../chapter-13-standards/readme.md)

## 常见问题

### 问题 1：缺少分号

**症状**：

```
Parse error: syntax error, unexpected 'echo' (T_ECHO) in file.php on line 2
```

**原因**：语句未加分号

**错误示例**：

```php
<?php
declare(strict_types=1);

$name = "World"  // 错误：缺少分号
echo "Hello, {$name}!\n";
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$name = "World";  // 正确：添加分号
echo "Hello, {$name}!\n";
```

**详细说明**：见 [2.1.5 常见错误与解决方案](section-05-common-errors.md)

### 问题 2：注释格式不规范

**症状**：代码格式化工具报错，或不符合团队规范

**原因**：未遵循 PSR-12 标准

**解决方法**：

1. **使用代码格式化工具**：

```bash
# 使用 PHP-CS-Fixer
php-cs-fixer fix file.php
```

2. **配置 EditorConfig**：见 [1.4.2 IDE 与扩展配置](../../stage-01-foundation/chapter-04-toolchain/section-02-ide-extensions.md)

3. **遵循 PSR-12 标准**：见 [2.13 代码规范](../chapter-13-standards/readme.md)

### 问题 3：PHPDoc 注释不完整

**症状**：IDE 无法提供代码提示，或文档生成工具报错

**原因**：PHPDoc 注释缺少必要信息

**错误示例**：

```php
<?php
declare(strict_types=1);

/**
 * 计算和
 */
function add(int $a, int $b): int
{
    return $a + $b;
}
```

**正确示例**：

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 *
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 */
function add(int $a, int $b): int
{
    return $a + $b;
}
```

**详细说明**：见 [2.13.2 PHPDoc 规范](../chapter-13-standards/section-02-phpdoc.md)

### 问题 4：注释与代码不一致

**症状**：注释描述的功能与实际代码不符

**原因**：代码修改后未更新注释

**解决方法**：
- 代码修改时同步更新注释
- 使用代码审查确保注释准确性
- 定期检查注释与代码的一致性

## 最佳实践

### 语句规范

- **所有语句都加分号**：即使是最后一个语句，也建议加分号
- **控制结构不加分号**：`if`、`for`、`while`、`function` 等不需要分号
- **保持一致性**：整个项目使用统一的语句风格

### 注释规范

- **解释"为什么"而非"是什么"**：注释应解释代码的目的和原因
- **保持简洁**：注释应简洁明了，避免冗长
- **及时更新**：代码修改时同步更新注释
- **使用有意义的注释**：避免无意义的注释

### PHPDoc 规范

- **为公共 API 编写完整的 PHPDoc**：所有公共方法、类都应提供完整的 PHPDoc
- **使用标准标签**：使用 `@param`、`@return`、`@throws` 等标准标签
- **保持一致性**：整个项目使用统一的 PHPDoc 格式
- **详细说明**：见 [2.13.2 PHPDoc 规范](../chapter-13-standards/section-02-phpdoc.md)

### 代码风格

- **遵循 PSR-12 标准**：确保代码风格统一
- **使用 EditorConfig**：统一编辑器配置
- **使用代码格式化工具**：如 PHP-CS-Fixer、PHP_CodeSniffer
- **建立代码审查流程**：确保代码风格一致

## 对比分析

### 注释类型对比

| 特性 | 单行注释 | 多行注释 | PHPDoc 注释 |
|:-----|:---------|:---------|:------------|
| 语法 | `//` | `/* */` | `/** */` |
| 行数 | 单行 | 多行 | 多行 |
| 用途 | 简短说明 | 详细说明 | API 文档 |
| IDE 支持 | 基础 | 基础 | 完整 |
| 文档生成 | 不支持 | 不支持 | 支持 |
| 推荐场景 | 代码内说明 | 代码块说明 | 公共 API |

**选择建议**：
- **简短说明**：使用单行注释
- **详细说明**：使用多行注释
- **公共 API**：使用 PHPDoc 注释

### 代码风格对比

| 特性 | PSR-12 | 其他风格 | 推荐度 |
|:-----|:-------|:---------|:-------|
| 标准性 | 行业标准 | 项目特定 | 推荐 |
| 工具支持 | 完整 | 有限 | 推荐 |
| 团队协作 | 统一 | 可能不一致 | 推荐 |
| 学习成本 | 低 | 可能高 | 推荐 |

**选择建议**：
- **新项目**：使用 PSR-12 标准
- **现有项目**：逐步迁移到 PSR-12
- **团队协作**：统一使用 PSR-12

## 相关章节

- **2.1.1 第一个 PHP 程序：Hello World**：了解 PHP 基本语法
- **2.1.2 文件结构与编码**：了解文件结构规范
- **2.1.4 文件执行方式**：了解不同执行环境
- **2.1.5 常见错误与解决方案**：了解常见语法错误
- **2.13 代码规范**：深入学习 PSR 标准和代码规范
- **2.13.2 PHPDoc 规范**：详细了解 PHPDoc 注释规范
- **1.4.2 IDE 与扩展配置**：了解如何配置代码格式化工具

## 练习任务

1. **语句练习**：
   - 创建多个 PHP 语句（赋值、函数调用、表达式）
   - 验证所有语句都正确使用分号
   - 测试控制结构（`if`、`for`、`while`）不需要分号

2. **注释练习**：
   - 使用单行注释说明代码功能
   - 使用多行注释详细说明复杂逻辑
   - 为函数编写 PHPDoc 注释

3. **代码风格练习**：
   - 编写符合 PSR-12 标准的代码
   - 使用代码格式化工具检查代码风格
   - 配置 EditorConfig 统一代码风格

4. **PHPDoc 练习**：
   - 为一个类编写完整的 PHPDoc 注释
   - 为类的方法编写 PHPDoc 注释
   - 使用文档生成工具生成 API 文档

5. **综合练习**：
   - 创建一个完整的 PHP 文件，包含所有注释类型
   - 确保代码符合 PSR-12 标准
   - 验证所有注释都准确描述了代码功能

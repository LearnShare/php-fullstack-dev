# 代码格式规范

## 代码示例要求

### 基本要求

**必须遵守**：
- 代码示例必须完整、可运行
- 提供语法格式、参数说明
- 每个知识点都配有完整的代码示例
- 默认使用 ASCII 字符
- PHP 代码必须符合 PSR-12 编码规范

### 代码完整性

**要求**：
- 代码示例必须包含必要的上下文
- 不能是伪代码或片段（除非明确标注为片段）
- 必须包含必要的导入语句（如 `declare(strict_types=1);`）

**示例**：

```php
<?php
declare(strict_types=1);

function calculateSum(int $a, int $b): int
{
    return $a + $b;
}

echo calculateSum(5, 3) . "\n";  // 8
```

## PHP 代码规范

### PSR-12 编码规范

**必须遵守**：
- 代码必须符合 PSR-12 编码规范
- 使用 4 个空格缩进（不使用 Tab）
- 行尾不能有尾随空格
- 文件必须以空行结尾

### 类型声明

**必须遵守**：
- 所有函数必须包含类型声明（参数类型、返回类型）
- 使用 PHP 8.2+ 语法特性
- 优先使用现代特性（构造器属性提升、枚举、只读属性等）

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        private string $name,
        private int $age
    ) {
    }

    public function getName(): string
    {
        return $this->name;
    }
}
```

### PHPDoc 注释

**规则**：
- 仅在必要时添加 PHPDoc 注释
- 复杂函数或公共 API 应提供 PHPDoc
- 遵循 PHPDoc 标准格式

**示例**：

```php
/**
 * 计算两个数的和
 *
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 */
function calculateSum(int $a, int $b): int
{
    return $a + $b;
}
```

## 代码注释

### 注释原则

**规则**：
- 仅在必要时添加简洁注释，解释复杂逻辑
- 避免显而易见的注释
- 代码块内可添加路径、行号、上下文注释（见 Markdown 格式规范）

### 注释格式

**代码块标注**（推荐）：
- 在代码块内使用注释标注文件路径、行号、上下文
- 使用 `// 文件:` 标注文件路径
- 使用 `// 行号:` 标注行号范围
- 使用 `// 功能:` 标注功能说明

**示例**：

```php
// 文件: src/Utils/Helper.php
// 行号: 10-15
// 功能: 格式化日期

function formatDate(DateTime $date): string
{
    return $date->format('Y-m-d');
}
```

## 代码示例组织

### 示例结构

**推荐结构**：
1. 文件头部（`<?php`、`declare(strict_types=1);`）
2. 必要的导入或定义
3. 主要代码逻辑
4. 使用示例
5. 输出说明（注释形式）

**示例**：

```php
<?php
declare(strict_types=1);

// 函数定义
function greet(string $name): string
{
    return "Hello, {$name}!";
}

// 使用示例
echo greet("Alice") . "\n";  // 输出: Hello, Alice!
```

### 完整示例类

**规则**：
- 复杂示例应使用完整的类结构
- 包含必要的属性和方法
- 提供使用示例

**示例**：

```php
<?php
declare(strict_types=1);

class Calculator
{
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }

    public function multiply(int $a, int $b): int
    {
        return $a * $b;
    }
}

// 使用示例
$calc = new Calculator();
echo $calc->add(5, 3) . "\n";        // 8
echo $calc->multiply(4, 7) . "\n";  // 28
```

---

**最后更新**：2025-12-22

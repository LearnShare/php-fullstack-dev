# 2.1.2 文件结构与编码

## 概述

PHP 文件的结构和编码是编写规范代码的基础。本节详细介绍 PHP 起始标签、字符编码（UTF-8）、严格类型声明、命名空间基础等文件结构规范，帮助建立良好的代码基础。

正确的文件结构和编码设置不仅能确保代码在所有环境中正常运行，还能避免常见的编码问题和类型错误，为后续的模块化开发和代码组织打下坚实基础。

## 特性

- **标准标签**：`<?php` 是推荐使用的起始标签，符合 PSR-1 标准
- **UTF-8 编码**：支持所有字符，是 Web 标准，确保国际化支持
- **严格类型**：`declare(strict_types=1);` 启用严格类型检查，提高代码质量
- **无 BOM**：避免输出问题，确保 HTTP 头设置正常
- **命名空间**：支持代码组织和模块化（详细内容见阶段三）

## 语法/定义

### PHP 起始标签

PHP 代码必须包含在起始标签中。PHP 支持多种起始标签类型，但根据 PSR-1 标准，应使用标准标签。

| 标签类型 | 语法 | 说明 | 推荐度 | PSR-1 兼容 |
|:---------|:-----|:-----|:-------|:-----------|
| 标准标签 | `<?php` | 推荐使用，所有环境支持 | 推荐 | 是 |
| 短标签 | `<?` | 需要 `short_open_tag=On`，不推荐 | 不推荐 | 否 |
| 短输出标签 | `<?=` | 等同于 `<?php echo`，PHP 5.4+ 始终可用 | 可用 | 是 |
| ASP 风格 | `<%` | 已废弃，不应使用 | 废弃 | 否 |

**标准标签示例**：

```php
<?php
echo "Hello, World!";
```

**短输出标签示例**（仅用于模板）：

```php
<?= "Hello, World!" ?>
```

**注意**：短输出标签 `<?=` 在 PHP 5.4+ 中始终可用，即使 `short_open_tag` 关闭。但根据 PSR-1 标准，纯 PHP 文件应使用标准标签。

### 字符编码

#### UTF-8 无 BOM 编码

**为什么使用 UTF-8**：
- 支持所有 Unicode 字符，包括中文、日文、韩文等
- 是 Web 标准，所有现代浏览器都支持
- 向后兼容 ASCII，ASCII 字符在 UTF-8 中编码不变
- 变长编码，节省存储空间

**为什么无 BOM**：
- BOM（Byte Order Mark）是 UTF-8 编码的可选标记
- BOM 会导致文件开头有不可见字符（`EF BB BF`）
- 在 Web 开发中，BOM 会导致 `Cannot modify header information - headers already sent` 错误
- 纯 PHP 文件不应包含 BOM

**检查文件编码**：

**VS Code**：
1. 查看右下角编码指示器
2. 点击编码指示器 → "通过编码重新打开" → 选择 "UTF-8"
3. 保存时确保选择 "UTF-8" 或 "UTF-8 without BOM"

**命令行检查（Linux/macOS）**：

```bash
# 检查文件编码
file -I hello.php
# 输出：hello.php: text/plain; charset=utf-8

# 检查是否有 BOM
head -c 3 hello.php | od -An -tx1
# 如果输出包含 ef bb bf，说明有 BOM
```

**Windows PowerShell**：

```powershell
# 检查文件编码
Get-Content hello.php -Encoding UTF8 | Out-File -Encoding UTF8 hello.php
```

### 严格类型声明

**语法**：`declare(strict_types=1);`

**位置**：必须在文件的第一行，紧跟在 `<?php` 之后

**作用**：
- 启用严格类型检查
- 函数参数和返回值必须严格匹配声明的类型
- 避免隐式类型转换导致的错误

**示例**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

// 正确：传递整数
echo add(1, 2);  // 输出: 3

// 错误：传递字符串（在严格模式下会报错）
// echo add("1", "2");  // TypeError
```

**详细说明**：严格类型检查的详细内容见 [2.5 类型转换与比较](../chapter-05-type-conversion/readme.md)

### 命名空间基础

**命名空间的作用**：
- 避免类名、函数名、常量名冲突
- 组织代码结构，实现模块化
- 支持 PSR-4 自动加载标准

**命名空间的定义语法**：

```php
<?php
declare(strict_types=1);

namespace App\Example;

// 代码内容
```

**说明**：
- `namespace` 关键字用于声明命名空间
- 命名空间使用反斜杠 `\` 分隔层级
- 命名空间声明必须在文件的第一行（在 `declare` 之后）
- 详细内容见阶段三：面向对象编程基础

### 文件结构模板

**标准文件结构**：

```php
<?php
declare(strict_types=1);

namespace App\Example;

// use 声明（如果需要）
use App\Other\SomeClass;

// 代码内容
```

**文件结构说明**：
1. `<?php`：PHP 起始标签
2. `declare(strict_types=1);`：严格类型声明
3. `namespace App\Example;`：命名空间声明（可选）
4. `use` 声明：导入其他命名空间的类（可选）
5. 代码内容：实际的 PHP 代码

**注意**：根据 PSR-1 标准，纯 PHP 文件不应使用结束标签 `?>`。

## 基本用法

### 示例 1：基础文件结构

**文件**：`example.php`

```php
<?php
declare(strict_types=1);

echo "Hello, World!\n";
```

**说明**：
- 最简单的文件结构
- 包含起始标签和严格类型声明
- 不使用命名空间（适合简单脚本）

### 示例 2：带命名空间的文件结构

**文件**：`src/App/Example.php`

```php
<?php
declare(strict_types=1);

namespace App;

class Example
{
    public function greet(string $name): string
    {
        return "Hello, {$name}!\n";
    }
}
```

**说明**：
- 包含命名空间声明
- 符合 PSR-4 自动加载标准
- 类名与文件名匹配

### 示例 3：使用 use 声明的文件结构

**文件**：`src/App/Service.php`

```php
<?php
declare(strict_types=1);

namespace App;

use App\Models\User;
use App\Repositories\UserRepository;

class Service
{
    public function __construct(
        private UserRepository $repository
    ) {
    }

    public function getUser(int $id): ?User
    {
        return $this->repository->find($id);
    }
}
```

**说明**：
- 使用 `use` 声明导入其他命名空间的类
- `use` 声明应在命名空间声明之后
- 符合 PSR-12 编码规范

## 使用场景

### 简单脚本

**适用场景**：命令行脚本、一次性任务、简单工具

**文件结构**：

```php
<?php
declare(strict_types=1);

// 脚本代码
```

### 类文件

**适用场景**：面向对象编程、模块化开发

**文件结构**：

```php
<?php
declare(strict_types=1);

namespace App\Module;

class ClassName
{
    // 类内容
}
```

### 配置文件

**适用场景**：应用配置、常量定义

**文件结构**：

```php
<?php
declare(strict_types=1);

return [
    'key' => 'value',
];
```

## 注意事项

### PHP 起始标签

- **永远使用 `<?php`**：确保代码在所有环境中都能正常运行
- **不要使用短标签**：短标签需要服务器配置，可能在某些环境中不可用
- **纯 PHP 文件不使用结束标签**：根据 PSR-1 标准，纯 PHP 文件不应使用结束标签 `?>`

### 字符编码

- **使用 UTF-8 无 BOM 编码**：避免编码问题和 BOM 导致的输出问题
- **统一编码**：确保所有文件使用相同的编码
- **检查编码**：在编辑器中检查文件编码，确保正确

### 严格类型声明

- **在文件开头启用**：`declare(strict_types=1);` 必须在文件第一行
- **作用范围**：严格类型声明只影响当前文件
- **建议启用**：启用严格类型检查可以提高代码质量

### 文件结构规范

- **遵循 PSR 标准**：遵循 PSR-1 和 PSR-12 编码规范
- **统一结构**：建立标准的文件结构模板
- **命名空间**：使用命名空间组织代码（详细内容见阶段三）

## 常见问题

### 问题 1：BOM 导致的问题

**症状**：

```
Warning: Cannot modify header information - headers already sent by (output started at /path/to/file.php:1)
```

**原因**：文件开头有 BOM（Byte Order Mark）

**解决方法**：

1. **检查文件编码**：
   - 在编辑器中检查文件编码
   - 确保使用 UTF-8 无 BOM 编码

2. **移除 BOM**：
   - VS Code：保存时选择 "UTF-8 without BOM"
   - 其他编辑器：重新保存文件，选择 UTF-8 无 BOM 编码

3. **验证修复**：

```bash
# Linux/macOS：检查 BOM
head -c 3 file.php | od -An -tx1
# 如果输出不包含 ef bb bf，说明已移除 BOM
```

### 问题 2：编码不一致

**症状**：中文字符显示乱码

**原因**：文件编码和输出编码不一致

**解决方法**：

1. **统一编码**：
   - 确保文件使用 UTF-8 编码
   - 确保输出也使用 UTF-8 编码

2. **设置 HTTP 头**（Web 模式）：

```php
<?php
declare(strict_types=1);

header('Content-Type: text/html; charset=UTF-8');
echo "中文内容";
```

3. **检查编辑器设置**：
   - 确保编辑器默认使用 UTF-8 编码
   - 保存文件时选择 UTF-8 编码

### 问题 3：短标签不可用

**症状**：使用 `<?` 标签时代码不执行

**原因**：服务器未启用 `short_open_tag` 配置

**解决方法**：

1. **使用标准标签**：
   - 将 `<?` 改为 `<?php`
   - 这是推荐的做法，符合 PSR-1 标准

2. **检查配置**（不推荐修改）：

```ini
; php.ini
short_open_tag = On
```

**注意**：即使启用了 `short_open_tag`，也不建议使用短标签，因为不符合 PSR-1 标准。

### 问题 4：严格类型错误

**症状**：

```
TypeError: Argument 1 passed to function() must be of type int, string given
```

**原因**：在严格类型模式下，类型不匹配

**解决方法**：

1. **检查类型声明**：
   - 确保函数参数类型声明正确
   - 确保传递的参数类型匹配

2. **类型转换**（如果需要）：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

// 正确：先转换类型
$result = add((int) "1", (int) "2");
```

3. **详细说明**：见 [2.5 类型转换与比较](../chapter-05-type-conversion/readme.md)

## 最佳实践

### 文件结构

- **使用标准标签**：永远使用 `<?php` 标准标签
- **启用严格类型**：在文件开头添加 `declare(strict_types=1);`
- **使用命名空间**：使用命名空间组织代码（详细内容见阶段三）
- **遵循 PSR 标准**：遵循 PSR-1 和 PSR-12 编码规范

### 编码管理

- **统一使用 UTF-8 无 BOM**：确保所有文件使用相同的编码
- **检查编码**：在编辑器中检查文件编码
- **配置编辑器**：设置编辑器默认使用 UTF-8 编码

### 代码组织

- **建立文件模板**：创建标准的文件结构模板
- **统一命名规范**：遵循 PSR-4 自动加载标准
- **模块化开发**：使用命名空间组织代码结构

## 对比分析

### 不同起始标签对比

| 特性 | `<?php` | `<?` | `<?=` | `<%` |
|:-----|:--------|:-----|:------|:-----|
| PSR-1 兼容 | 是 | 否 | 是 | 否 |
| 环境支持 | 所有环境 | 需配置 | PHP 5.4+ | 已废弃 |
| 推荐度 | 推荐 | 不推荐 | 可用（模板） | 废弃 |
| 使用场景 | 所有场景 | 不推荐 | 模板输出 | 不应使用 |

**选择建议**：
- **纯 PHP 文件**：使用 `<?php` 标准标签
- **模板文件**：可以使用 `<?=` 短输出标签（但建议使用标准标签）
- **其他标签**：不应使用

### 编码格式对比

| 编码格式 | 优点 | 缺点 | 推荐度 |
|:---------|:-----|:-----|:-------|
| UTF-8 无 BOM | 支持所有字符，Web 标准，无输出问题 | - | 推荐 |
| UTF-8 有 BOM | 支持所有字符 | 可能导致输出问题 | 不推荐 |
| GBK/GB2312 | 中文支持 | 不支持其他语言，非标准 | 不推荐 |
| ASCII | 兼容性好 | 不支持非 ASCII 字符 | 仅限英文 |

**选择建议**：
- **所有文件**：使用 UTF-8 无 BOM 编码
- **国际化项目**：必须使用 UTF-8 编码

## 相关章节

- **2.1.1 第一个 PHP 程序：Hello World**：了解 PHP 起始标签的基本使用
- **2.1.3 语句与注释**：了解语句规则和注释用法
- **2.5 类型转换与比较**：详细了解严格类型检查的作用
- **2.13 代码规范**：深入学习 PSR 标准和代码规范
- **阶段三：面向对象编程基础**：详细了解命名空间的使用
- **1.4.2 IDE 与扩展配置**：了解如何配置编辑器使用 UTF-8 编码

## 练习任务

1. **创建标准文件结构**：
   - 创建一个 PHP 文件，包含起始标签、严格类型声明
   - 验证文件编码为 UTF-8 无 BOM
   - 执行文件，确保正常运行

2. **测试编码问题**：
   - 创建一个包含中文的文件，使用 UTF-8 编码
   - 尝试使用其他编码保存文件，观察输出差异
   - 修复编码问题，确保正确显示

3. **测试严格类型**：
   - 创建一个函数，包含参数类型和返回类型
   - 启用严格类型，测试类型检查
   - 尝试传递错误类型的参数，观察错误信息

4. **使用命名空间**：
   - 创建一个带命名空间的类文件
   - 验证命名空间声明的位置和格式
   - 测试类的使用（详细内容见阶段三）

5. **建立文件模板**：
   - 在 IDE 中创建文件模板
   - 模板应包含起始标签、严格类型声明、命名空间声明
   - 使用模板创建新文件，验证模板是否正确

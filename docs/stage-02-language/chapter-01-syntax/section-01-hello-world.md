# 2.1.1 第一个 PHP 程序：Hello World

## 概述

创建并执行第一个 PHP 程序是学习 PHP 的第一步。本节介绍如何创建 PHP 文件、使用 PHP 起始标签、执行 PHP 程序（CLI 和 Web 两种方式），帮助零基础学员完成第一个 PHP 程序。

本节是 PHP 学习的起点，通过创建和运行第一个程序，学员将了解 PHP 的基本语法结构、文件格式和执行方式，为后续深入学习打下基础。

## 特性

- **简单易学**：Hello World 程序是最简单的 PHP 程序，适合初学者入门
- **多种执行方式**：支持 CLI（命令行）和 Web 两种执行方式
- **即时反馈**：程序执行后立即看到输出结果，便于理解程序运行流程
- **基础语法**：涵盖 PHP 起始标签、输出语句等基础语法

## 实践步骤

### 步骤 1：创建 PHP 文件

**文件命名规范**：
- 文件名可以使用任意名称，但建议使用有意义的名称
- 文件扩展名必须是 `.php`
- 文件名区分大小写（在 Linux/macOS 系统中）

**创建文件示例**：

```bash
# 在项目目录中创建文件
touch hello.php

# 或使用编辑器创建文件
# VS Code: 新建文件 → 保存为 hello.php
# PhpStorm: File → New → PHP File → 输入 hello.php
```

### 步骤 2：编写代码

**基本代码结构**：

```php
<?php
echo "Hello, World!";
```

**代码说明**：
- `<?php`：PHP 起始标签，表示 PHP 代码的开始（详细说明见 [2.1.2 文件结构与编码](section-02-file-structure.md)）
- `echo`：PHP 的输出语句，用于向输出流输出内容（详细说明见 [2.2 输出与调试基础](../chapter-02-output/readme.md)）
- `"Hello, World!"`：字符串字面量，要输出的内容
- `;`：语句结束符，PHP 语句必须以分号结尾（详细说明见 [2.1.3 语句与注释](section-03-statements-comments.md)）

### 步骤 3：执行程序

**CLI 方式（命令行）**：

```bash
php hello.php
```

**输出**：
```
Hello, World!
```

**Web 方式**：

1. 将文件放在 Web 服务器目录中（如 `public/` 或 `htdocs/`）
2. 启动 Web 服务器（如果使用内置服务器，见 [1.2.2 运行模式与验证](../../stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)）
3. 通过浏览器访问：`http://localhost/hello.php`
4. 浏览器显示：`Hello, World!`

## 完整代码示例

### 示例 1：基础 Hello World

**文件**：`hello.php`

```php
<?php
echo "Hello, World!\n";
```

**执行**：

```bash
php hello.php
```

**输出**：

```
Hello, World!
```

**说明**：
- `\n` 是换行符，在 CLI 模式下会在输出后换行
- 在 Web 模式下，`\n` 会被浏览器忽略，需要使用 `<br>` 标签实现换行

### 示例 2：带严格类型的 Hello World

**文件**：`hello-strict.php`

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

echo greet("World");
```

**执行**：

```bash
php hello-strict.php
```

**输出**：

```
Hello, World!
```

**说明**：
- `declare(strict_types=1);`：启用严格类型检查（详细说明见 [2.1.2 文件结构与编码](section-02-file-structure.md)）
- `function greet(string $name): string`：定义函数，包含参数类型和返回类型（详细说明见 [2.10 函数与作用域](../chapter-10-functions/readme.md)）
- `{$name}`：字符串插值，在双引号字符串中嵌入变量（详细说明见 [2.7 字符串操作](../chapter-07-strings/readme.md)）

### 示例 3：多行输出

**文件**：`hello-multi.php`

```php
<?php
declare(strict_types=1);

echo "Hello, World!\n";
echo "Welcome to PHP!\n";
echo "Let's start learning!\n";
```

**执行**：

```bash
php hello-multi.php
```

**输出**：

```
Hello, World!
Welcome to PHP!
Let's start learning!
```

### 示例 4：使用变量

**文件**：`hello-variable.php`

```php
<?php
declare(strict_types=1);

$name = "World";
echo "Hello, {$name}!\n";
```

**执行**：

```bash
php hello-variable.php
```

**输出**：

```
Hello, World!
```

**说明**：
- `$name = "World";`：定义变量并赋值（详细说明见 [2.3 变量与常量](../chapter-03-variables/readme.md)）
- `{$name}`：在字符串中嵌入变量值

## 输出结果说明

### CLI 方式输出

**特点**：
- 输出直接显示在终端/命令行窗口中
- 支持换行符 `\n`、制表符 `\t` 等转义字符
- 输出格式与代码中的格式一致

**示例输出**：

```
Hello, World!
```

### Web 方式输出

**特点**：
- 输出显示在浏览器中
- 换行符 `\n` 会被浏览器忽略，需要使用 HTML 标签实现换行
- 输出会被浏览器解析为 HTML

**示例输出**：

浏览器显示：`Hello, World!`

**注意**：如果需要在 Web 模式下显示换行，可以使用：

```php
<?php
echo "Hello, World!<br>";
echo "Welcome to PHP!<br>";
```

## 注意事项

### 文件编码

- **必须使用 UTF-8 编码**：PHP 文件应使用 UTF-8 编码（无 BOM）
- **检查编码**：在编辑器中检查文件编码，确保是 UTF-8
- **编码问题**：如果文件编码不正确，可能导致中文字符显示乱码（详细说明见 [2.1.2 文件结构与编码](section-02-file-structure.md)）

### PHP 起始标签

- **必须使用 `<?php`**：这是 PSR-1 标准要求，确保代码在所有环境中都能正常运行
- **纯 PHP 文件不使用结束标签**：根据 PSR-1 标准，纯 PHP 文件不应使用结束标签 `?>`（详细说明见 [2.1.2 文件结构与编码](section-02-file-structure.md)）

### 执行环境差异

- **CLI 和 Web 方式的执行环境不同**：
  - CLI 模式：使用系统环境变量，输出到终端
  - Web 模式：使用 Web 服务器配置，输出到浏览器
- **详细说明**：见 [2.1.4 文件执行方式](section-04-execution.md)

### 严格类型声明

- **建议启用**：`declare(strict_types=1);` 可以提高代码质量，避免类型相关的错误
- **位置**：必须在文件的第一行，在 `<?php` 之后
- **详细说明**：见 [2.1.2 文件结构与编码](section-02-file-structure.md)

## 常见问题

### 问题 1：找不到 php 命令

**症状**：

```bash
$ php hello.php
bash: php: command not found
```

**原因**：PHP 未添加到系统 PATH 环境变量中

**解决方法**：

1. **检查 PHP 是否已安装**：

```bash
# Windows
where php

# Linux/macOS
which php
```

2. **添加 PHP 到 PATH**：

   - **Windows**：在系统环境变量中添加 PHP 安装目录
   - **Linux/macOS**：在 `~/.bashrc` 或 `~/.zshrc` 中添加 PHP 路径

3. **详细安装说明**：见 [1.2.1 PHP 安装与版本管理](../../stage-01-foundation/chapter-02-runtime/section-01-installation.md)

### 问题 2：浏览器显示源代码

**症状**：浏览器显示 PHP 源代码，而不是执行结果

**原因**：Web 服务器未配置 PHP 或未正确配置

**解决方法**：

1. **检查 Web 服务器配置**：
   - 确保 Web 服务器已安装 PHP
   - 确保 PHP 模块已启用

2. **使用 PHP 内置服务器**：

```bash
php -S localhost:8000
```

然后访问 `http://localhost:8000/hello.php`

3. **详细说明**：见 [1.2.2 运行模式与验证](../../stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)

### 问题 3：编码问题

**症状**：中文字符显示乱码

**原因**：文件编码不正确

**解决方法**：

1. **检查文件编码**：
   - VS Code：查看右下角编码指示器
   - 确保文件使用 UTF-8 编码（无 BOM）

2. **转换编码**：
   - 在编辑器中重新保存文件，选择 UTF-8 编码

3. **详细说明**：见 [2.1.2 文件结构与编码](section-02-file-structure.md)

### 问题 4：语法错误

**症状**：

```
Parse error: syntax error, unexpected 'echo' (T_ECHO) in hello.php on line 2
```

**原因**：代码语法错误，如缺少分号、括号不匹配等

**解决方法**：

1. **检查语法**：

```bash
php -l hello.php
```

2. **常见语法错误**：见 [2.1.5 常见错误与解决方案](section-05-common-errors.md)

## 最佳实践

### 代码规范

- **使用标准标签**：永远使用 `<?php` 标准标签
- **启用严格类型**：在文件开头添加 `declare(strict_types=1);`
- **遵循 PSR 标准**：遵循 PSR-1 和 PSR-12 编码规范（详细说明见 [2.13 代码规范](../chapter-13-standards/readme.md)）

### 文件组织

- **使用有意义的文件名**：文件名应反映文件内容
- **统一文件结构**：建立标准的文件结构模板（详细说明见 [2.1.6 实践建议与示例](section-06-best-practices.md)）

### 开发流程

- **先测试再完善**：先创建简单的程序，确保能运行，再逐步完善
- **使用版本控制**：使用 Git 管理代码（详细说明见 [1.4.3 Git 与任务管理](../../stage-01-foundation/chapter-04-toolchain/section-03-git-tasks.md)）

## 对比分析

### CLI vs Web 执行方式

| 特性 | CLI 模式 | Web 模式 |
|:-----|:--------|:--------|
| 执行环境 | 命令行 | Web 服务器 |
| 输出目标 | 终端 | 浏览器 |
| 换行符 | `\n` 有效 | `\n` 无效，需使用 `<br>` |
| 适用场景 | 脚本、工具 | Web 应用 |
| 环境变量 | 系统环境变量 | Web 服务器配置 |

**选择建议**：
- **学习阶段**：建议使用 CLI 模式，输出更直观
- **Web 开发**：使用 Web 模式，模拟实际运行环境
- **详细说明**：见 [2.1.4 文件执行方式](section-04-execution.md)

## 相关章节

- **2.1.2 文件结构与编码**：了解 PHP 起始标签、编码、严格类型等详细内容
- **2.1.3 语句与注释**：掌握语句规则和注释用法
- **2.1.4 文件执行方式**：详细了解 CLI 和 Web 执行方式的区别
- **2.2 输出与调试基础**：深入学习输出函数和调试技巧
- **2.3 变量与常量**：学习变量和常量的使用
- **1.2.2 运行模式与验证**：了解 PHP 运行模式的详细内容

## 练习任务

1. **创建第一个程序**：
   - 创建 `hello.php` 文件
   - 编写输出 "Hello, World!" 的代码
   - 在 CLI 模式下执行程序
   - 验证输出结果

2. **尝试不同的输出方式**：
   - 使用 `echo` 输出多行内容
   - 尝试使用变量输出内容
   - 在 CLI 和 Web 两种模式下执行，观察输出差异

3. **添加函数定义**：
   - 创建一个 `greet()` 函数，接受一个名称参数
   - 函数返回问候语
   - 使用函数输出 "Hello, World!"

4. **添加类型声明**：
   - 在文件开头添加 `declare(strict_types=1);`
   - 为函数添加参数类型和返回类型
   - 验证类型检查是否生效

5. **错误处理练习**：
   - 故意创建一些语法错误（如缺少分号）
   - 使用 `php -l` 检查语法错误
   - 修复错误并重新执行程序

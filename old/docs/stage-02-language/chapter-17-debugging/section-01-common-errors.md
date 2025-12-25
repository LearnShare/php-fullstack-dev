# 2.17.1 常见错误类型与修复流程

## 概述

本节聚焦开发过程中最常见的错误类型：语法错误、类型错误、未定义变量/索引、调用未定义函数与内存耗尽。通过“错误场景 → 报错信息 → 修复手段”的方式，帮助你迅速定位问题根因。

## 语法错误（Parse Error）

### 典型报错

- 缺少分号、括号、引号
- 错误的关键字拼写或嵌套结构

```php
<?php
// 错误：缺少分号
$name = "Alice"
echo $name;

// 错误：未闭合括号
if ($condition {
    // ...
}

// 错误：未闭合引号
$text = "Hello World;
```

### 修复流程

```bash
# 使用 PHP 内置语法检查
php -l file.php

# 输出：PHP Parse error: syntax error, unexpected 'echo' in file.php on line 2
```

1. 先在 IDE/CI 中开启语法检查，优先定位第一条错误。
2. 注意看“unexpected/expecting”提示，通常说明丢失分隔符或嵌套不匹配。
3. 多文件 include 时，从最外层脚本向下排查。

## 类型错误（TypeError）

### 触发场景

- `declare(strict_types=1)` 下传递了错误类型
- 函数返回值与声明类型不一致

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

add("5", 3); // TypeError: Argument #1 must be of type int, string given
```

### 防御手段

```php
<?php
declare(strict_types=1);

function safeAdd(mixed $a, mixed $b): int
{
    if (!is_int($a) || !is_int($b)) {
        throw new TypeError('Arguments must be integers');
    }
    return $a + $b;
}

add((int) "5", 3);
```

- 所有入口（Controller/Command）做参数校验与类型转换。
- 对第三方输入使用 DTO/表单对象集中校验。

## 未定义变量 / 索引

### 未定义变量

```php
<?php
echo $undefinedVariable; // Warning: Undefined variable $undefinedVariable
```

**修复要点**：

```php
$variable = null;
echo $variable ?? 'default'; // 空合并

if (isset($variable)) {
    echo $variable;
}
```

### 未定义索引

```php
<?php
$data = ['name' => 'Alice'];
echo $data['email']; // Warning: Undefined array key "email"
```

**修复要点**：

```php
if (array_key_exists('email', $data)) {
    echo $data['email'];
}

echo $data['email'] ?? 'N/A';
```

## 调用未定义函数

```php
<?php
unknownFunction(); // Fatal error: Call to undefined function
```

### 定位步骤

1. 确认函数是否在当前命名空间中，必要时加 `\` 前缀。
2. 检查 `require/include` 是否遗漏，或 Composer autoload 是否更新。
3. 对扩展函数，确认扩展已安装并启用。

```php
if (!function_exists('pcntl_fork')) {
    throw new RuntimeException('pcntl 扩展未启用');
}
```

## 内存耗尽（Allowed memory size exhausted）

### 常见原因

- 无限循环或错误的递归
- 一次性载入超大数组/文件
- 结果集未分页

```php
<?php
$data = [];
while (true) {
    $data[] = str_repeat('x', 1_000_000);
}
```

### 优化策略

```php
ini_set('memory_limit', '512M'); // 临时扩容

function generateData(): Generator
{
    for ($i = 0; $i < 1_000_000; $i++) {
        yield str_repeat('x', 1000);
    }
}

foreach (generateData() as $chunk) {
    process($chunk);
}
```

- 使用流式处理（生成器、`yield`、分页查询）。
- 对 CLI 脚本增加 `memory_get_usage()` 监控，超阈值时预警。

## 小结

- **先读错误信息**：抓住“文件 + 行号 + unexpected/undefined”关键字。
- **复现与最小化**：将问题代码抽离为最小可复现脚本，降低干扰。
- **记录与归档**：把排查过程写入团队 wiki，形成问题 → 原因 → 解决的知识库。

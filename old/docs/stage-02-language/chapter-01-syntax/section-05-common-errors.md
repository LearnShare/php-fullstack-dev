# 2.1.5 常见错误与解决方案

## 1. 语法错误：缺少分号

**错误信息**：

```
Parse error: syntax error, unexpected 'echo' (T_ECHO) in hello.php on line 3
```

**错误代码**：

```php
<?php
$name = "张三"
echo $name;  // 缺少分号
```

**解决方案**：

```php
<?php
$name = "张三";  // 添加分号
echo $name;
```

## 2. 编码问题：BOM 导致输出错误

**错误信息**：

```
Warning: Cannot modify header information - headers already sent
```

**原因**：文件开头有 BOM（Byte Order Mark）

**解决方案**：
1. 使用支持 UTF-8 无 BOM 的编辑器
2. 保存时选择 "UTF-8 without BOM"
3. 使用工具移除 BOM：

```bash
# Linux/macOS
sed -i '1s/^\xEF\xBB\xBF//' file.php

# 或使用 dos2unix
dos2unix file.php
```

## 3. 严格类型错误

**错误代码**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

echo add("10", "20");  // TypeError
```

**错误信息**：

```
TypeError: add(): Argument #1 ($a) must be of type int, string given
```

**解决方案**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

echo add(10, 20);  // 传入整数类型
// 或
echo add((int)"10", (int)"20");  // 显式转换
```

## 4. 文件路径错误

**错误代码**：

```php
<?php
include 'config.php';  // 文件不存在
```

**错误信息**：

```
Warning: include(config.php): failed to open stream: No such file or directory
```

**解决方案**：

```php
<?php
// 使用绝对路径
include __DIR__ . '/config.php';

// 或检查文件是否存在
if (file_exists('config.php')) {
    include 'config.php';
}
```

## 5. 命名空间错误

**错误代码**：

```php
<?php
namespace App;

class User {}

// 错误：找不到类
$user = new User();  // 实际是 App\User
```

**解决方案**：

```php
<?php
namespace App;

class User {}

// 使用完全限定名
$user = new \App\User();

// 或使用 use
use App\User;
$user = new User();
```

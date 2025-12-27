# 4.2.11 文件权限详解

## 概述

文件权限是操作系统安全性的关键部分，控制哪些用户或进程可以访问、修改或执行文件。理解文件权限的概念、表示方法、修改方法以及在不同平台上的差异，对于构建安全的 PHP 应用至关重要。

文件权限系统在 Unix/Linux 和 Windows 上有所不同。在类 Unix 系统上，权限通过读、写、执行权限来控制，分为所有者、所属组和其他用户三类。在 Windows 上，权限系统更为复杂，但 PHP 的 `chmod()` 函数主要处理只读属性。

**主要内容**：
- 文件权限的概念和分类
- 权限表示方法（符号表示法、八进制表示法）
- `chmod()` 函数（修改文件权限）
- `chown()` 函数（修改文件所有者）
- `chgrp()` 函数（修改文件所属组）
- `umask()` 函数（设置默认权限掩码）
- 获取文件权限信息
- 特殊权限位（SUID、SGID、粘滞位）
- 跨平台权限处理
- 实际应用场景和最佳实践

## 特性

- **安全控制**：通过权限控制文件访问，提高系统安全性
- **灵活配置**：支持多种权限配置方式
- **用户分类**：区分所有者、所属组和其他用户
- **特殊权限**：支持 SUID、SGID、粘滞位等特殊权限
- **跨平台支持**：在 Unix/Linux 和 Windows 上都有支持（功能有限）

## 语法/定义

### chmod() 函数

**语法**：`chmod(string $filename, int $permissions): bool`

**参数**：
- `$filename`：文件或目录路径
- `$permissions`：权限值（八进制数）

**返回值**：成功返回 `true`，失败返回 `false`。

### chown() 函数

**语法**：`chown(string $filename, string|int $user): bool`

**参数**：
- `$filename`：文件或目录路径
- `$user`：用户名（字符串）或用户 ID（整数）

**返回值**：成功返回 `true`，失败返回 `false`。

### chgrp() 函数

**语法**：`chgrp(string $filename, string|int $group): bool`

**参数**：
- `$filename`：文件或目录路径
- `$group`：组名（字符串）或组 ID（整数）

**返回值**：成功返回 `true`，失败返回 `false`。

### umask() 函数

**语法**：`umask(?int $mask = null): int`

**参数**：
- `$mask`：权限掩码（八进制数），如果为 `null`，则只返回当前掩码

**返回值**：返回之前的权限掩码。

## 基本用法

### 示例 1：使用 chmod() 修改文件权限

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 设置权限为 644（所有者读写，组和其他只读）
if (chmod($file, 0644)) {
    echo "Permissions changed successfully\n";
} else {
    echo "Failed to change permissions\n";
}

// 设置权限为 755（所有者读写执行，组和其他读执行）
chmod($file, 0755);

// 设置权限为 600（仅所有者读写）
chmod($file, 0600);

// 设置目录权限为 755
$dir = __DIR__ . '/directory';
chmod($dir, 0755);
```

**说明**：
- `chmod()` 使用八进制数设置权限
- 权限值需要以 `0` 开头（如 `0644`）
- 只有文件所有者或超级用户（root）可以修改文件权限

### 示例 2：理解权限表示法

```php
<?php
declare(strict_types=1);

// 权限表示法说明
// 八进制权限值：
// 644 = rw-r--r--（所有者读写，组和其他只读）
// 755 = rwxr-xr-x（所有者读写执行，组和其他读执行）
// 600 = rw-------（仅所有者读写）
// 777 = rwxrwxrwx（所有用户都有全部权限，不安全）

$file = __DIR__ . '/file.txt';

// 644：所有者读写，组和其他只读（推荐的普通文件权限）
chmod($file, 0644);

// 755：所有者读写执行，组和其他读执行（推荐的目录权限）
chmod($file, 0755);

// 600：仅所有者读写（推荐的敏感文件权限）
chmod($file, 0600);
```

**说明**：
- 权限分为三组：所有者（Owner）、所属组（Group）、其他用户（Others）
- 每组有三种权限：读（Read, 4）、写（Write, 2）、执行（Execute, 1）
- 权限值 = 读(4) + 写(2) + 执行(1)

### 示例 3：使用 umask() 设置默认权限

```php
<?php
declare(strict_types=1);

// 获取当前 umask
$oldUmask = umask();
echo "Current umask: " . sprintf('%04o', $oldUmask) . "\n";

// 设置 umask 为 022（屏蔽组和其他用户的写权限）
umask(0022);

// 创建新文件，权限将是 0644（0666 - 0022）
file_put_contents(__DIR__ . '/newfile.txt', 'content');
// 实际权限：rw-r--r-- (644)

// 创建新目录，权限将是 0755（0777 - 0022）
mkdir(__DIR__ . '/newdir');
// 实际权限：rwxr-xr-x (755)

// 恢复原来的 umask
umask($oldUmask);
```

**说明**：
- `umask` 定义了创建新文件或目录时应该屏蔽的权限位
- 新文件的权限 = 默认权限（0666）- umask
- 新目录的权限 = 默认权限（0777）- umask

### 示例 4：获取文件权限信息

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 使用 fileperms() 获取权限
$perms = fileperms($file);
if ($perms !== false) {
    // 获取权限（八进制）
    $permissions = $perms & 0777;
    echo "Permissions (octal): " . sprintf('%04o', $permissions) . "\n";
}

// 使用 stat() 获取详细信息
$stat = stat($file);
if ($stat !== false) {
    // 获取权限（八进制）
    $permissions = $stat['mode'] & 0777;
    echo "Permissions: " . sprintf('%04o', $permissions) . "\n";
    
    // 检查是否为目录
    $isDir = ($stat['mode'] & 040000) !== 0;
    echo "Is directory: " . ($isDir ? 'Yes' : 'No') . "\n";
    
    // 检查特殊权限位
    $hasSUID = ($stat['mode'] & 04000) !== 0;
    $hasSGID = ($stat['mode'] & 02000) !== 0;
    $hasSticky = ($stat['mode'] & 01000) !== 0;
    
    echo "SUID: " . ($hasSUID ? 'Yes' : 'No') . "\n";
    echo "SGID: " . ($hasSGID ? 'Yes' : 'No') . "\n";
    echo "Sticky: " . ($hasSticky ? 'Yes' : 'No') . "\n";
}
```

**说明**：
- `fileperms()` 返回文件的权限值
- `stat()` 返回文件的详细信息，包括权限
- 权限值需要与 `0777` 进行按位与操作，获取基本权限

### 示例 5：修改文件所有者和所属组

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 修改文件所有者（需要 root 权限）
if (chown($file, 'www-data')) {
    echo "Owner changed successfully\n";
}

// 使用用户 ID
chown($file, 33);  // 假设 33 是 www-data 的用户 ID

// 修改文件所属组
if (chgrp($file, 'www-data')) {
    echo "Group changed successfully\n";
}

// 使用组 ID
chgrp($file, 33);  // 假设 33 是 www-data 的组 ID
```

**说明**：
- `chown()` 修改文件所有者，需要 root 权限
- `chgrp()` 修改文件所属组，文件所有者或 root 可以修改
- 在 Windows 上，这些函数可能不可用或功能有限

### 示例 6：权限检查函数

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 检查是否可读
if (is_readable($file)) {
    echo "File is readable\n";
}

// 检查是否可写
if (is_writable($file)) {
    echo "File is writable\n";
}

// 检查是否可执行
if (is_executable($file)) {
    echo "File is executable\n";
}

// 检查目录权限
$dir = __DIR__ . '/directory';
if (is_dir($dir) && is_writable($dir)) {
    echo "Directory is writable\n";
}
```

**说明**：
- `is_readable()` 检查文件是否可读
- `is_writable()` 检查文件是否可写
- `is_executable()` 检查文件是否可执行

## 使用场景

### 场景 1：文件上传后的权限设置

设置上传文件的权限，确保安全。

**示例**：

```php
<?php
declare(strict_types=1);

function handleFileUpload(array $file): bool
{
    $tmpFile = $file['tmp_name'];
    $destination = __DIR__ . '/uploads/' . basename($file['name']);
    
    if (!move_uploaded_file($tmpFile, $destination)) {
        return false;
    }
    
    // 设置权限为 644（所有者读写，其他只读）
    chmod($destination, 0644);
    
    return true;
}

if (isset($_FILES['file'])) {
    handleFileUpload($_FILES['file']);
}
```

### 场景 2：敏感文件的权限保护

设置敏感文件（如配置文件）的权限，仅所有者可读写。

**示例**：

```php
<?php
declare(strict_types=1);

function protectConfigFile(string $configFile): void
{
    if (file_exists($configFile)) {
        // 设置权限为 600（仅所有者可读写）
        chmod($configFile, 0600);
    }
}

protectConfigFile(__DIR__ . '/config.php');
protectConfigFile(__DIR__ . '/.env');
```

## 注意事项

### 权限设置的权限要求

只有文件所有者或超级用户（root）可以修改文件权限。

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 尝试修改权限
if (!chmod($file, 0644)) {
    // 检查是否是权限问题
    if (!is_writable(dirname($file))) {
        throw new RuntimeException('No permission to change file permissions');
    }
}
```

### 跨平台兼容性

`chmod()` 在 Windows 上功能有限，主要用于设置只读属性。

**示例**：

```php
<?php
declare(strict_types=1);

function setFilePermissions(string $file, int $permissions): bool
{
    // Windows 上 chmod() 功能有限
    if (PHP_OS_FAMILY === 'Windows') {
        // Windows 主要支持只读属性
        if (($permissions & 0200) === 0) {
            // 设置为只读
            return chmod($file, 0444);
        }
    }
    
    return chmod($file, $permissions);
}
```

### 八进制表示法

权限值需要使用八进制表示，以 `0` 开头。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：使用八进制（以 0 开头）
chmod(__DIR__ . '/file.txt', 0644);
chmod(__DIR__ . '/file.txt', 0755);

// ❌ 错误：使用十进制
// chmod(__DIR__ . '/file.txt', 644);  // 这是十进制，不是八进制
```

## 常见问题

### 问题 1：如何理解权限数字？

**回答**：权限数字是八进制数，分为三位，分别表示所有者、所属组和其他用户的权限。每位是 0-7 的数字，由读(4)、写(2)、执行(1)组合而成。

**示例**：

```php
<?php
declare(strict_types=1);

// 权限数字解析
// 644 = 6(rw-) 4(r--) 4(r--)
// 755 = 7(rwx) 5(r-x) 5(r-x)
// 600 = 6(rw-) 0(---) 0(---)

// 6 = 4(读) + 2(写) + 0(执行) = rw-
// 5 = 4(读) + 0(写) + 1(执行) = r-x
// 4 = 4(读) + 0(写) + 0(执行) = r--
```

### 问题 2：chmod() 和 umask() 的区别是什么？

**回答**：
- `chmod()`：直接设置文件或目录的权限
- `umask()`：设置默认权限掩码，影响后续创建的文件和目录的默认权限

**示例**：

```php
<?php
declare(strict_types=1);

// chmod()：直接设置权限
chmod(__DIR__ . '/file.txt', 0644);

// umask()：设置默认权限掩码
umask(0022);  // 后续创建的文件权限为 0644（0666 - 0022）
file_put_contents(__DIR__ . '/newfile.txt', 'content');  // 权限自动为 0644
```

### 问题 3：Windows 下如何处理权限？

**回答**：在 Windows 上，`chmod()` 功能有限，主要用于设置只读属性。如果需要更复杂的权限控制，需要使用 Windows 特定的 API 或工具。

**示例**：

```php
<?php
declare(strict_types=1);

function setPermissions(string $file, int $permissions): bool
{
    if (PHP_OS_FAMILY === 'Windows') {
        // Windows 上主要支持只读属性
        if (($permissions & 0200) === 0) {
            // 设置为只读
            return chmod($file, 0444);
        } else {
            // 设置为可写
            return chmod($file, 0666);
        }
    }
    
    return chmod($file, $permissions);
}
```

### 问题 4：如何递归修改目录权限？

**回答**：`chmod()` 不会递归修改，需要手动递归处理。

**示例**：

```php
<?php
declare(strict_types=1);

function chmodRecursive(string $path, int $filePerm, int $dirPerm): void
{
    if (is_dir($path)) {
        chmod($path, $dirPerm);
        $files = scandir($path);
        if ($files !== false) {
            foreach ($files as $file) {
                if ($file !== '.' && $file !== '..') {
                    $filePath = $path . DIRECTORY_SEPARATOR . $file;
                    chmodRecursive($filePath, $filePerm, $dirPerm);
                }
            }
        }
    } else {
        chmod($path, $filePerm);
    }
}

// 使用
chmodRecursive(__DIR__ . '/directory', 0644, 0755);
```

## 最佳实践

### 1. 遵循最小权限原则

只授予必要的权限，不要过度授权。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：普通文件使用 644
chmod(__DIR__ . '/file.txt', 0644);

// ✅ 推荐：目录使用 755
chmod(__DIR__ . '/directory', 0755);

// ✅ 推荐：敏感文件使用 600
chmod(__DIR__ . '/config.php', 0600);

// ❌ 不推荐：使用 777（所有用户都有全部权限，不安全）
// chmod(__DIR__ . '/file.txt', 0777);
```

### 2. 使用合适的默认权限

通过 `umask()` 设置合适的默认权限。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置 umask 为 022（推荐）
umask(0022);  // 文件权限 644，目录权限 755

// 创建文件
file_put_contents(__DIR__ . '/file.txt', 'content');  // 权限自动为 644
mkdir(__DIR__ . '/directory');  // 权限自动为 755
```

### 3. 注意跨平台兼容性

在跨平台应用中，注意 Windows 和 Unix/Linux 的权限差异。

**示例**：见"跨平台兼容性"部分的示例

### 4. 敏感文件设置严格权限

对于配置文件、密钥文件等敏感文件，设置严格的权限（600）。

**示例**：

```php
<?php
declare(strict_types=1);

$sensitiveFiles = [
    __DIR__ . '/config.php',
    __DIR__ . '/.env',
    __DIR__ . '/private.key',
];

foreach ($sensitiveFiles as $file) {
    if (file_exists($file)) {
        chmod($file, 0600);  // 仅所有者可读写
    }
}
```

### 5. 定期审查文件权限

定期检查文件权限，确保符合安全要求。

**示例**：

```php
<?php
declare(strict_types=1);

function checkFilePermissions(string $file, int $expectedPerms): bool
{
    $perms = fileperms($file);
    if ($perms === false) {
        return false;
    }
    
    $actualPerms = $perms & 0777;
    return $actualPerms === $expectedPerms;
}

// 检查配置文件权限
if (!checkFilePermissions(__DIR__ . '/config.php', 0600)) {
    echo "Warning: config.php has insecure permissions\n";
}
```

## 对比分析

### 常用权限值

| 权限值 | 符号表示    | 说明                      | 适用场景               |
|:-------|:------------|:--------------------------|:-----------------------|
| **644** | `rw-r--r--` | 所有者读写，其他只读      | 普通文件（推荐）       |
| **755** | `rwxr-xr-x` | 所有者读写执行，其他读执行| 目录、可执行文件（推荐）|
| **600** | `rw-------` | 仅所有者读写              | 敏感文件（推荐）       |
| **777** | `rwxrwxrwx` | 所有用户都有全部权限      | 不推荐（不安全）       |

### chmod() vs umask()

| 特性         | chmod()                      | umask()                     |
|:-------------|:-----------------------------|:----------------------------|
| **作用对象** | 指定文件或目录               | 后续创建的所有文件和目录    |
| **使用时机** | 文件创建后                   | 文件创建前                  |
| **灵活性**   | ✅ 可以单独设置每个文件      | ⚠️ 影响所有后续创建的文件   |
| **适用场景** | 修改现有文件权限             | 设置默认权限                |

## 练习任务

1. **权限管理工具类**：创建一个权限管理工具类，封装权限设置、检查和验证功能。

2. **敏感文件保护工具**：实现一个工具，自动设置敏感文件的权限（600）。

3. **权限审查工具**：编写一个工具，检查目录中所有文件的权限，发现不安全的权限设置。

4. **递归权限修改工具**：创建一个工具，递归修改目录及其所有文件的权限。

5. **跨平台权限处理工具**：实现一个工具，在不同平台上正确处理文件权限。

## 相关章节

- **[4.2.3 文件操作](section-03-file-operations.md)**：了解文件操作的相关内容
- **[4.2.8 文件信息](section-08-file-info.md)**：了解如何获取文件信息

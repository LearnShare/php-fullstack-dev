# 5.8.11 文件权限详解

## 概述

文件权限是操作系统安全性的关键部分，控制哪些用户或进程可以访问、修改或执行文件。理解文件权限的概念、表示方法、修改方法以及在不同平台上的差异，对于构建安全的 PHP 应用至关重要。

## 文件权限的概念

**文件权限**定义了文件或目录的访问控制规则，包括：
- **读取权限（Read）**：允许读取文件内容或列出目录内容
- **写入权限（Write）**：允许修改文件内容或在目录中创建/删除文件
- **执行权限（Execute）**：允许执行文件或进入目录

## Unix/Linux 文件权限

### 权限表示方法

在类 Unix 系统（Linux、macOS）中，文件权限通常以两种方式表示：

**1. 符号表示法（Symbolic Notation）**

使用字符表示权限：
- `r`：读取权限（Read）
- `w`：写入权限（Write）
- `x`：执行权限（Execute）
- `-`：无权限

权限分为三组，每组对应不同的用户类别：
- **所有者（Owner）**：文件的所有者
- **所属组（Group）**：文件所属的用户组
- **其他用户（Others）**：其他所有用户

**示例**：

```
-rw-r--r--
```

这个权限字符串表示：
- `-`：文件类型（`-` 表示普通文件，`d` 表示目录）
- `rw-`：所有者权限（读写，无执行）
- `r--`：所属组权限（只读）
- `r--`：其他用户权限（只读）

**2. 八进制表示法（Octal Notation）**

使用三位八进制数表示权限，每位对应一组权限：

| 权限 | 二进制 | 八进制 |
| :--- | :--- | :--- |
| `---` | 000 | 0 |
| `--x` | 001 | 1 |
| `-w-` | 010 | 2 |
| `-wx` | 011 | 3 |
| `r--` | 100 | 4 |
| `r-x` | 101 | 5 |
| `rw-` | 110 | 6 |
| `rwx` | 111 | 7 |

**示例**：
- `644`：所有者读写（6），所属组只读（4），其他用户只读（4）
- `755`：所有者读写执行（7），所属组读执行（5），其他用户读执行（5）
- `600`：所有者读写（6），所属组无权限（0），其他用户无权限（0）

### 特殊权限位

除了基本的读、写、执行权限外，还有三个特殊权限位：

**1. SetUID（SUID）**

- **作用**：执行文件时，进程以文件所有者的权限运行
- **表示**：在所有者执行位上，`x` 变为 `s`
- **示例**：`-rwsr-xr-x`（4755）
- **安全风险**：如果可执行文件设置了 SUID，且所有者是 root，则任何用户执行该文件都会以 root 权限运行

**2. SetGID（SGID）**

- **作用**：执行文件时，进程以文件所属组的权限运行；对于目录，新创建的文件会继承目录的组
- **表示**：在所属组执行位上，`x` 变为 `s`
- **示例**：`-rwxr-sr-x`（2755）
- **安全风险**：类似 SUID，需要谨慎使用

**3. 粘滞位（Sticky Bit）**

- **作用**：对于目录，只有文件的所有者才能删除或修改该文件，即使其他用户对该目录有写权限
- **表示**：在其他用户执行位上，`x` 变为 `t`
- **示例**：`drwxrwxrwt`（1777），常用于 `/tmp` 目录
- **安全风险**：相对较低，主要用于共享目录

## chmod() - 修改文件权限

**语法**：`chmod(string $filename, int $permissions): bool`

**参数**：
- `$filename`：文件或目录路径
- `$permissions`：权限值（八进制数）

**返回值**：成功返回 `true`，失败返回 `false`。

**注意事项**：
- 只有文件所有者或超级用户（root）可以修改文件权限
- 在 Windows 上，`chmod()` 的功能有限，主要用于设置只读属性

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 设置权限为 644（所有者读写，其他只读）
if (chmod($file, 0644)) {
    echo "Permissions changed successfully\n";
} else {
    echo "Failed to change permissions\n";
}

// 设置权限为 755（所有者读写执行，其他读执行）
chmod($file, 0755);

// 设置权限为 600（仅所有者读写）
chmod($file, 0600);

// 设置目录权限为 755
$dir = __DIR__ . '/directory';
chmod($dir, 0755);
```

**使用符号表示法**：

```php
<?php
declare(strict_types=1);

// 注意：PHP 的 chmod() 不支持符号表示法
// 需要使用八进制数或使用 exec() 调用系统命令

// 使用 exec() 调用 chmod 命令（仅 Unix/Linux）
exec("chmod u+x " . escapeshellarg(__DIR__ . '/script.sh'));
exec("chmod g+w " . escapeshellarg(__DIR__ . '/file.txt'));
exec("chmod o-r " . escapeshellarg(__DIR__ . '/file.txt'));
```

## chown() - 修改文件所有者

**语法**：`chown(string $filename, string|int $user): bool`

**参数**：
- `$filename`：文件或目录路径
- `$user`：用户名（字符串）或用户 ID（整数）

**返回值**：成功返回 `true`，失败返回 `false`。

**注意事项**：
- 只有超级用户（root）可以修改文件所有者
- 在 Windows 上，此函数可能不可用或功能有限

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 使用用户名
if (chown($file, 'www-data')) {
    echo "Owner changed successfully\n";
}

// 使用用户 ID
chown($file, 33);  // 假设 33 是 www-data 的用户 ID
```

## chgrp() - 修改文件所属组

**语法**：`chgrp(string $filename, string|int $group): bool`

**参数**：
- `$filename`：文件或目录路径
- `$group`：组名（字符串）或组 ID（整数）

**返回值**：成功返回 `true`，失败返回 `false`。

**注意事项**：
- 只有文件所有者或超级用户（root）可以修改文件所属组
- 在 Windows 上，此函数可能不可用或功能有限

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 使用组名
if (chgrp($file, 'www-data')) {
    echo "Group changed successfully\n";
}

// 使用组 ID
chgrp($file, 33);  // 假设 33 是 www-data 的组 ID
```

## umask() - 设置默认权限掩码

**语法**：`umask(?int $mask = null): int`

**参数**：
- `$mask`：权限掩码（八进制数），如果为 `null`，则只返回当前掩码

**返回值**：返回之前的权限掩码。

**工作原理**：
- `umask` 定义了创建新文件或目录时应该**屏蔽**的权限位
- 新文件的权限 = 默认权限 - umask
- 默认情况下，新文件的权限通常是 `0666`，新目录的权限通常是 `0777`

**示例**：

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

**常用 umask 值**：

| umask | 文件权限 | 目录权限 | 说明 |
| :--- | :--- | :--- | :--- |
| `0000` | `666` | `777` | 无限制（不安全） |
| `0022` | `644` | `755` | 屏蔽组和其他用户的写权限（常用） |
| `0002` | `664` | `775` | 屏蔽其他用户的写权限 |
| `0077` | `600` | `700` | 仅所有者可访问（最安全） |

## 获取文件权限信息

### 使用 stat() 获取权限

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$stat = stat($file);

if ($stat !== false) {
    // 获取权限（八进制）
    $permissions = $stat['mode'] & 0777;
    echo "Permissions (octal): " . sprintf('%04o', $permissions) . "\n";
    
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

### 使用 fileperms() 获取权限

**语法**：`fileperms(string $filename): int|false`

**返回值**：成功返回权限值，失败返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$perms = fileperms($file);

if ($perms !== false) {
    // 获取权限（八进制）
    $permissions = $perms & 0777;
    echo "Permissions: " . sprintf('%04o', $permissions) . "\n";
    
    // 转换为符号表示
    $symbolic = substr(sprintf('%o', $perms), -4);
    echo "Symbolic: " . $symbolic . "\n";
}
```

## Windows 文件权限

在 Windows 系统上，文件权限系统与 Unix/Linux 不同：

- **访问控制列表（ACL）**：Windows 使用 ACL 来控制文件访问
- **PHP 函数限制**：`chmod()`、`chown()`、`chgrp()` 在 Windows 上功能有限
- **只读属性**：可以使用 `chmod($file, 0444)` 设置只读属性

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

// 在 Windows 上设置只读属性
if (PHP_OS_FAMILY === 'Windows') {
    chmod($file, 0444);  // 设置只读
    chmod($file, 0666);  // 取消只读
}
```

## 文件权限的最佳实践

1. **最小权限原则**：只授予必要的权限，不要过度授权
2. **文件权限**：普通文件通常设置为 `644`（所有者读写，其他只读）
3. **目录权限**：目录通常设置为 `755`（所有者读写执行，其他读执行）
4. **敏感文件**：配置文件、密钥文件等设置为 `600`（仅所有者可读写）
5. **可执行文件**：脚本文件设置为 `755`（所有者读写执行，其他读执行）
6. **共享目录**：使用粘滞位（`1777`）确保只有文件所有者可以删除自己的文件
7. **避免 SUID/SGID**：除非必要，否则不要设置 SUID 或 SGID 位
8. **定期审查**：定期检查文件权限，确保符合安全要求

## 完整示例

```php
<?php
declare(strict_types=1);

class FilePermissionManager
{
    /**
     * 安全地设置文件权限
     */
    public static function setSecurePermissions(string $file, int $permissions): bool
    {
        if (!file_exists($file)) {
            return false;
        }
        
        // 验证权限值是否合理
        if ($permissions < 0 || $permissions > 0777) {
            return false;
        }
        
        return chmod($file, $permissions);
    }
    
    /**
     * 获取文件权限（符号表示）
     */
    public static function getSymbolicPermissions(string $file): ?string
    {
        $perms = fileperms($file);
        if ($perms === false) {
            return null;
        }
        
        $perms = $perms & 0777;
        return sprintf('%04o', $perms);
    }
    
    /**
     * 检查文件是否为安全权限（仅所有者可读写）
     */
    public static function isSecurePermission(string $file): bool
    {
        $perms = fileperms($file);
        if ($perms === false) {
            return false;
        }
        
        $perms = $perms & 0777;
        // 检查是否为 600 或 700
        return $perms === 0600 || $perms === 0700;
    }
}

// 使用
$file = __DIR__ . '/config.php';

// 设置安全权限（仅所有者可读写）
FilePermissionManager::setSecurePermissions($file, 0600);

// 获取权限
$perms = FilePermissionManager::getSymbolicPermissions($file);
echo "Permissions: {$perms}\n";

// 检查是否为安全权限
if (FilePermissionManager::isSecurePermission($file)) {
    echo "File has secure permissions\n";
}
```

## 注意事项

1. **平台差异**：Windows 和 Unix/Linux 的权限系统不同，需要注意跨平台兼容性
2. **权限检查**：修改权限前检查文件是否存在和当前权限
3. **错误处理**：始终检查 `chmod()`、`chown()`、`chgrp()` 的返回值
4. **安全风险**：设置过宽的权限可能导致安全漏洞
5. **SUID/SGID 风险**：谨慎使用 SUID 和 SGID，可能导致权限提升漏洞

## 练习

1. 编写一个函数，安全地设置文件权限（包含验证和错误处理）
2. 实现一个函数，检查文件权限是否符合安全要求
3. 创建一个函数，获取文件的详细权限信息（包括特殊权限位）
4. 编写一个函数，批量修改目录下所有文件的权限

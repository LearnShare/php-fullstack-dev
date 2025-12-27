# 3.5.3 自动加载基础

## 概述

自动加载（Autoloading）是按需加载类文件的机制，当代码需要使用一个类时，自动加载器会根据类名自动加载对应的类文件，无需手动使用 `require` 或 `include`。自动加载是现代 PHP 项目的基础，极大简化了代码管理。

理解自动加载对于组织大型项目至关重要。传统方式需要手动引入每个类文件，这在大型项目中变得非常繁琐和容易出错。自动加载机制通过 `spl_autoload_register()` 函数注册自动加载器，当类未定义时自动触发加载过程。

自动加载与命名空间紧密配合，形成了现代 PHP 项目的代码组织基础。理解自动加载的工作原理有助于理解 Composer 的自动加载机制和 PSR-4 标准。

**主要内容**：
- 传统手动引入方式的问题
- `spl_autoload_register()` 函数的使用
- 简单自动加载器的实现
- 多个自动加载器的注册和优先级
- 命名空间与自动加载的结合
- 自动加载的最佳实践

## 特性

- **按需加载**：只在需要时加载类文件，提高性能
- **简化代码**：无需手动 `require` 每个类文件
- **灵活配置**：可以注册多个自动加载器
- **命名空间支持**：与命名空间完美配合
- **性能优化**：避免加载不需要的类

## 语法/定义

### spl_autoload_register()

**语法**：`spl_autoload_register(callable $autoloader, bool $throw = true, bool $prepend = false): bool`

**参数**：
- `$autoloader`：自动加载函数，接收类名作为参数
- `$throw`：是否在加载失败时抛出异常（默认 `true`）
- `$prepend`：是否将加载器添加到队列前面（默认 `false`，添加到后面）

**返回值**：`bool`，注册成功返回 `true`

**功能**：注册自动加载器函数，当类未定义时按顺序调用已注册的自动加载器

### __autoload()（已废弃）

**说明**：PHP 5.1.2+ 引入了 `__autoload()` 函数，但在 PHP 7.2.0 中已废弃，应该使用 `spl_autoload_register()`

## 基本用法

### 示例 1：传统方式的问题

```php
<?php
// 传统方式：需要手动引入每个文件
require_once 'App/Models/User.php';
require_once 'App/Models/Product.php';
require_once 'App/Models/Order.php';
require_once 'App/Services/UserService.php';
require_once 'App/Services/ProductService.php';
require_once 'App/Controllers/UserController.php';
require_once 'App/Controllers/ProductController.php';
// ... 更多文件

// 问题：
// 1. 需要知道所有文件的位置
// 2. 维护困难，容易遗漏
// 3. 代码冗余
// 4. 如果类文件很多，列表会非常长
```

**说明**：
- 传统方式需要手动引入每个类文件
- 在大型项目中变得非常繁琐
- 容易遗漏文件导致错误

### 示例 2：基础自动加载器

```php
<?php
declare(strict_types=1);

// 注册自动加载器
spl_autoload_register(function (string $className): void {
    // 将命名空间分隔符转换为目录分隔符
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    
    // 检查文件是否存在
    if (file_exists($file)) {
        require $file;
    }
});

// 现在可以直接使用类，无需手动 require
namespace App\Models;

class User
{
    public function __construct(
        private int $id,
        private string $name
    ) {}
}

// 另一个文件：index.php
// 由于自动加载器已注册，可以直接使用类
use App\Models\User;

$user = new User(1, 'John');  // 自动加载 App\Models\User 类
```

**说明**：
- 使用 `spl_autoload_register()` 注册自动加载器
- 自动加载器接收类名作为参数
- 根据类名构造文件路径并加载
- 使用类时会自动触发加载

### 示例 3：支持命名空间前缀的自动加载器

```php
<?php
declare(strict_types=1);

class SimpleAutoloader
{
    private array $prefixes = [];
    
    /**
     * 添加命名空间前缀到目录的映射
     */
    public function addNamespace(string $prefix, string $baseDir): void
    {
        // 标准化前缀（确保以 \ 结尾）
        $prefix = trim($prefix, '\\') . '\\';
        
        // 标准化基础目录（确保以 / 结尾）
        $baseDir = rtrim($baseDir, DIRECTORY_SEPARATOR) . '/';
        
        // 存储映射
        $this->prefixes[$prefix] = $baseDir;
    }
    
    /**
     * 注册自动加载器
     */
    public function register(): void
    {
        spl_autoload_register([$this, 'loadClass']);
    }
    
    /**
     * 加载类
     */
    public function loadClass(string $class): bool
    {
        // 从完整类名开始，逐步缩短前缀
        $prefix = $class;
        
        while (false !== $pos = strrpos($prefix, '\\')) {
            // 提取前缀（包含 \）
            $prefix = substr($class, 0, $pos + 1);
            
            // 提取相对类名
            $relativeClass = substr($class, $pos + 1);
            
            // 尝试加载映射的文件
            $mappedFile = $this->loadMappedFile($prefix, $relativeClass);
            if ($mappedFile) {
                return true;
            }
            
            // 缩短前缀，继续尝试
            $prefix = rtrim($prefix, '\\');
        }
        
        return false;
    }
    
    /**
     * 加载映射的文件
     */
    private function loadMappedFile(string $prefix, string $relativeClass): bool
    {
        // 检查前缀是否存在
        if (!isset($this->prefixes[$prefix])) {
            return false;
        }
        
        // 构造文件路径
        $file = $this->prefixes[$prefix]
              . str_replace('\\', '/', $relativeClass)
              . '.php';
        
        // 加载文件
        if (file_exists($file)) {
            require $file;
            return true;
        }
        
        return false;
    }
}

// 使用自动加载器
$loader = new SimpleAutoloader();
$loader->addNamespace('App', __DIR__ . '/src');
$loader->register();

// 现在可以直接使用类
use App\Models\User;
$user = new User(1, 'John');
```

**说明**：
- 支持命名空间前缀到目录的映射
- 可以注册多个命名空间前缀
- 按命名空间前缀匹配并加载类文件

### 示例 4：多个自动加载器

```php
<?php
declare(strict_types=1);

// 第一个自动加载器：加载应用代码
spl_autoload_register(function (string $className): void {
    if (str_starts_with($className, 'App\\')) {
        $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
        if (file_exists($file)) {
            require $file;
        }
    }
});

// 第二个自动加载器：加载第三方库（优先级更高）
spl_autoload_register(function (string $className): void {
    if (str_starts_with($className, 'Vendor\\')) {
        $file = __DIR__ . '/vendor/' . str_replace('\\', '/', $className) . '.php';
        if (file_exists($file)) {
            require $file;
        }
    }
}, false, true);  // prepend = true，添加到队列前面

// 自动加载器的调用顺序：
// 1. Vendor\* 自动加载器（prepend = true，优先级高）
// 2. App\* 自动加载器
```

**说明**：
- 可以注册多个自动加载器
- 使用 `prepend` 参数控制优先级
- `prepend = true` 添加到队列前面，优先级更高
- 自动加载器按顺序调用，直到找到类文件

### 示例 5：完整的自动加载器类

```php
<?php
declare(strict_types=1);

class Autoloader
{
    private array $namespaces = [];
    
    /**
     * 添加命名空间映射
     */
    public function addNamespace(string $prefix, string $baseDir): void
    {
        $this->namespaces[$prefix] = rtrim($baseDir, '/') . '/';
    }
    
    /**
     * 注册自动加载器
     */
    public function register(): void
    {
        spl_autoload_register([$this, 'loadClass']);
    }
    
    /**
     * 加载类
     */
    private function loadClass(string $class): void
    {
        // 遍历所有命名空间映射
        foreach ($this->namespaces as $prefix => $baseDir) {
            // 检查类名是否以该前缀开头
            if (str_starts_with($class, $prefix)) {
                // 移除前缀，获取相对类名
                $relativeClass = substr($class, strlen($prefix));
                
                // 构造文件路径
                $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
                
                // 加载文件
                if (file_exists($file)) {
                    require $file;
                    return;
                }
            }
        }
    }
}

// 使用
$loader = new Autoloader();
$loader->addNamespace('App\\', __DIR__ . '/src');
$loader->addNamespace('Vendor\\', __DIR__ . '/vendor');
$loader->register();

// 现在可以直接使用类
use App\Models\User;
use Vendor\Library\Helper;

$user = new User(1, 'John');
$helper = new Helper();
```

**说明**：
- 完整的自动加载器类实现
- 支持多个命名空间映射
- 简洁清晰的代码结构

### 示例 6：自动加载器与目录结构

```php
<?php
declare(strict_types=1);

// 目录结构：
// src/
//   App/
//     Models/
//       User.php          -> namespace App\Models;
//     Controllers/
//       UserController.php -> namespace App\Controllers;

spl_autoload_register(function (string $className): void {
    // 将命名空间转换为文件路径
    // App\Models\User -> src/App/Models/User.php
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    
    if (file_exists($file)) {
        require $file;
    }
});

// 使用
use App\Models\User;
use App\Controllers\UserController;

$user = new User(1, 'John');
$controller = new UserController();
```

**说明**：
- 自动加载器将命名空间转换为文件路径
- 命名空间结构与目录结构对应
- 这是 PSR-4 标准的基础

## 使用场景

### 场景 1：大型项目组织

自动加载用于组织大型项目的代码结构。

**示例**：

```php
<?php
declare(strict_types=1);

$loader = new Autoloader();
$loader->addNamespace('App\\', __DIR__ . '/src');
$loader->addNamespace('App\\Tests\\', __DIR__ . '/tests');
$loader->register();

// 项目中的所有类都可以自动加载
```

### 场景 2：框架开发

框架使用自动加载机制加载框架类和用户代码。

**示例**：

```php
<?php
declare(strict_types=1);

// 框架自动加载器
spl_autoload_register(function (string $className): void {
    // 框架类
    if (str_starts_with($className, 'Framework\\')) {
        $file = __DIR__ . '/framework/' . str_replace('\\', '/', $className) . '.php';
        if (file_exists($file)) {
            require $file;
        }
    }
    // 应用类
    elseif (str_starts_with($className, 'App\\')) {
        $file = __DIR__ . '/app/' . str_replace('\\', '/', $className) . '.php';
        if (file_exists($file)) {
            require $file;
        }
    }
});
```

## 注意事项

### 性能考虑

- **文件检查开销**：`file_exists()` 有文件系统开销
- **避免复杂逻辑**：自动加载器应该简单高效
- **缓存机制**：考虑使用类映射（classmap）提升性能
- **OPcache**：使用 OPcache 缓存已加载的文件

### 错误处理

- **文件不存在**：应该优雅处理文件不存在的情况
- **加载失败**：不要在加载失败时抛出异常（除非必要）
- **返回 false**：自动加载器应该返回 `false` 表示未找到，让其他加载器继续尝试

**示例**：

```php
<?php
declare(strict_types=1);

spl_autoload_register(function (string $className): bool {
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    
    if (file_exists($file)) {
        require $file;
        return true;  // 成功加载
    }
    
    return false;  // 未找到，让其他加载器尝试
});
```

### 路径处理

- **路径分隔符**：使用 `DIRECTORY_SEPARATOR` 或 `/` 处理路径
- **路径规范化**：确保路径格式正确
- **相对路径与绝对路径**：使用绝对路径更可靠

**示例**：

```php
<?php
declare(strict_types=1);

spl_autoload_register(function (string $className): void {
    // 使用绝对路径
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    
    // 或使用 DIRECTORY_SEPARATOR（跨平台）
    $parts = explode('\\', $className);
    $file = __DIR__ . DIRECTORY_SEPARATOR . 'src' . DIRECTORY_SEPARATOR 
          . implode(DIRECTORY_SEPARATOR, $parts) . '.php';
    
    if (file_exists($file)) {
        require $file;
    }
});
```

### 自动加载器顺序

- **调用顺序**：自动加载器按注册顺序调用
- **prepend 参数**：使用 `prepend = true` 可以添加到队列前面
- **优先级**：优先使用 `prepend = true` 的加载器

## 常见问题

### 问题 1：类文件找不到

**错误信息**：`Class 'X' not found`

**原因**：
- 自动加载器逻辑错误
- 文件路径不正确
- 命名空间不匹配

**解决方案**：
- 检查自动加载器逻辑
- 检查文件路径和命名空间
- 使用调试输出检查路径

**示例**：

```php
<?php
declare(strict_types=1);

spl_autoload_register(function (string $className): void {
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    
    // 调试：输出尝试加载的路径
    echo "尝试加载: {$file}\n";
    
    if (file_exists($file)) {
        require $file;
    } else {
        echo "文件不存在: {$file}\n";
    }
});
```

### 问题 2：自动加载器未触发

**症状**：类未定义，但没有触发自动加载器

**原因**：
- 自动加载器未注册
- 类已定义（之前已加载）
- 使用了 `require` 手动加载

**解决方案**：
- 确保自动加载器已注册
- 检查类是否已定义
- 移除手动 `require` 语句

### 问题 3：性能问题

**症状**：应用运行较慢

**原因**：
- 自动加载器逻辑复杂
- 文件系统操作过多
- 没有使用缓存

**解决方案**：
- 简化自动加载器逻辑
- 使用类映射（classmap）缓存
- 使用 OPcache

## 最佳实践

### 1. 使用 spl_autoload_register

使用 `spl_autoload_register()` 而不是已废弃的 `__autoload()`。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的做法：使用 spl_autoload_register
spl_autoload_register(function (string $className): void {
    // 加载逻辑
});

// 不好的做法：使用已废弃的 __autoload
// function __autoload(string $className): void
// {
//     // 加载逻辑
// }
```

### 2. 简单高效的自动加载器

自动加载器应该简单高效，避免复杂逻辑。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的做法：简单高效
spl_autoload_register(function (string $className): bool {
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    if (file_exists($file)) {
        require $file;
        return true;
    }
    return false;
});

// 不好的做法：过于复杂
// spl_autoload_register(function (string $className): void {
//     // 复杂的逻辑、数据库查询、网络请求等
// });
```

### 3. 使用类映射提升性能

对于频繁使用的类，使用类映射（classmap）提升性能。

**示例**：

```php
<?php
declare(strict_types=1);

class Autoloader
{
    private array $classmap = [];
    private array $prefixes = [];
    
    public function addClassmap(array $classmap): void
    {
        $this->classmap = array_merge($this->classmap, $classmap);
    }
    
    public function addNamespace(string $prefix, string $baseDir): void
    {
        $this->prefixes[$prefix] = $baseDir;
    }
    
    public function register(): void
    {
        spl_autoload_register([$this, 'loadClass']);
    }
    
    private function loadClass(string $class): bool
    {
        // 首先检查类映射
        if (isset($this->classmap[$class])) {
            require $this->classmap[$class];
            return true;
        }
        
        // 然后检查命名空间映射
        foreach ($this->prefixes as $prefix => $baseDir) {
            if (str_starts_with($class, $prefix)) {
                $relativeClass = substr($class, strlen($prefix));
                $file = $baseDir . '/' . str_replace('\\', '/', $relativeClass) . '.php';
                
                if (file_exists($file)) {
                    require $file;
                    return true;
                }
            }
        }
        
        return false;
    }
}
```

### 4. 错误处理

优雅处理文件不存在的情况，不要抛出不必要的异常。

**示例**：

```php
<?php
declare(strict_types=1);

spl_autoload_register(function (string $className): bool {
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    
    if (file_exists($file)) {
        require $file;
        return true;
    }
    
    // 优雅处理：返回 false，让其他加载器尝试
    return false;
}, false);  // throw = false，不在加载失败时抛出异常
```

### 5. 考虑使用 Composer

对于实际项目，考虑使用 Composer 的自动加载机制（PSR-4 标准）。

**示例**：

```php
<?php
// 使用 Composer 自动加载
require __DIR__ . '/vendor/autoload.php';

// 直接使用类
use App\Models\User;
$user = new User();
```

## 对比分析

### 自动加载 vs 手动 require

| 特性         | 自动加载                           | 手动 require                       |
|:-------------|:--------------------------------|:--------------------------------|
| **代码量**    | 较少（注册一次）                    | 较多（每个文件都需要）              |
| **维护性**    | ✅ 更容易维护                       | ❌ 难以维护                        |
| **性能**      | ✅ 按需加载                         | ⚠️ 可能加载不需要的文件             |
| **灵活性**    | ✅ 更灵活                           | ❌ 不够灵活                        |
| **适用场景**  | 大型项目、框架、库                  | 简单脚本                          |

### spl_autoload_register vs __autoload

| 特性         | spl_autoload_register             | __autoload                        |
|:-------------|:--------------------------------|:--------------------------------|
| **多个加载器** | ✅ 支持                            | ❌ 只能定义一个                    |
| **优先级**    | ✅ 可以控制                        | ❌ 无法控制                       |
| **状态**      | ✅ 推荐使用                        | ❌ 已废弃（PHP 7.2.0+）           |
| **灵活性**    | ✅ 更灵活                           | ❌ 不够灵活                        |

## 练习任务

1. **实现简单自动加载器**：实现一个简单的自动加载器，支持命名空间到目录的映射。

2. **多个命名空间前缀**：创建一个自动加载器，支持多个命名空间前缀映射到不同的基础目录。

3. **自动加载器优先级**：注册多个自动加载器，使用 `prepend` 参数控制优先级。

4. **类映射自动加载**：实现一个自动加载器，支持类映射（classmap）提升性能。

5. **完整自动加载器类**：创建一个完整的自动加载器类，支持类映射和命名空间映射。

## 相关章节

- **[3.5.1 命名空间基础](section-01-basics.md)**：了解命名空间的基础知识
- **[3.5.2 use 语句](section-02-use-statements.md)**：了解 use 语句的使用
- **[3.5.4 Composer 自动加载与 PSR-4](section-04-composer-psr4.md)**：了解 Composer 的自动加载机制和 PSR-4 标准

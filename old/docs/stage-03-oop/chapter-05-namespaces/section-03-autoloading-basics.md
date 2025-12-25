# 3.5.3 自动加载基础

## 概述

自动加载机制允许在需要时自动加载类文件，无需手动 `require` 或 `include`。理解自动加载的工作原理对于组织大型项目至关重要。

## 传统方式的问题

### 手动引入文件

```php
// 需要手动引入每个文件
require_once 'App/Models/User.php';
require_once 'App/Models/Product.php';
require_once 'App/Services/UserService.php';
// ... 更多文件
```

### 问题

- 需要知道所有文件的位置
- 维护困难，容易遗漏
- 代码冗余

## spl_autoload_register

### 基本自动加载

- PHP 提供自动加载机制，当类未定义时触发。

```php
spl_autoload_register(function ($className) {
    // 将命名空间分隔符转换为目录分隔符
    $file = __DIR__ . '/' . str_replace('\\', '/', $className) . '.php';
    
    if (file_exists($file)) {
        require_once $file;
    }
});

// 现在可以直接使用类，无需手动 require
use App\Models\User;
$user = new User(1, 'Alice');
```

### 多个自动加载器

```php
// 注册多个自动加载器
spl_autoload_register(function ($className) {
    $file = __DIR__ . '/src/' . str_replace('\\', '/', $className) . '.php';
    if (file_exists($file)) {
        require_once $file;
    }
});

spl_autoload_register(function ($className) {
    $file = __DIR__ . '/vendor/' . str_replace('\\', '/', $className) . '.php';
    if (file_exists($file)) {
        require_once $file;
    }
});
```

## 简单自动加载器示例

### 实现自动加载器类

```php
class Autoloader
{
    private array $prefixes = [];

    public function addNamespace(string $prefix, string $baseDir): void
    {
        $prefix = trim($prefix, '\\') . '\\';
        $baseDir = rtrim($baseDir, DIRECTORY_SEPARATOR) . '/';
        $this->prefixes[$prefix] = $baseDir;
    }

    public function register(): void
    {
        spl_autoload_register([$this, 'loadClass']);
    }

    public function loadClass(string $class): bool
    {
        $prefix = $class;
        
        while (false !== $pos = strrpos($prefix, '\\')) {
            $prefix = substr($class, 0, $pos + 1);
            $relativeClass = substr($class, $pos + 1);
            
            $mappedFile = $this->loadMappedFile($prefix, $relativeClass);
            if ($mappedFile) {
                return $mappedFile;
            }
            
            $prefix = rtrim($prefix, '\\');
        }
        
        return false;
    }

    private function loadMappedFile(string $prefix, string $relativeClass): bool
    {
        if (!isset($this->prefixes[$prefix])) {
            return false;
        }
        
        $file = $this->prefixes[$prefix]
              . str_replace('\\', '/', $relativeClass)
              . '.php';
        
        if (file_exists($file)) {
            require $file;
            return true;
        }
        
        return false;
    }
}

// 使用
$loader = new Autoloader();
$loader->addNamespace('App', __DIR__ . '/src');
$loader->register();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SimpleAutoloader
{
    private array $namespaces = [];

    public function addNamespace(string $prefix, string $baseDir): void
    {
        $this->namespaces[$prefix] = $baseDir;
    }

    public function register(): void
    {
        spl_autoload_register([$this, 'loadClass']);
    }

    private function loadClass(string $class): void
    {
        foreach ($this->namespaces as $prefix => $baseDir) {
            if (str_starts_with($class, $prefix)) {
                $relativeClass = substr($class, strlen($prefix));
                $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
                
                if (file_exists($file)) {
                    require $file;
                    return;
                }
            }
        }
    }
}

// 使用
$loader = new SimpleAutoloader();
$loader->addNamespace('App\\', __DIR__ . '/src');
$loader->register();

// 现在可以直接使用类
use App\Models\User;
$user = new User(1, 'Alice');
```

## 注意事项

1. **性能考虑**：自动加载器会在类未定义时触发，避免在热路径中频繁触发。

2. **文件存在检查**：使用 `file_exists()` 检查文件是否存在，避免不必要的错误。

3. **路径处理**：正确处理路径分隔符，确保跨平台兼容。

4. **命名空间匹配**：正确匹配命名空间前缀，避免加载错误的文件。

5. **错误处理**：妥善处理文件不存在的情况，避免抛出不必要的错误。

## 练习

1. 实现一个简单的自动加载器，支持命名空间到目录的映射。

2. 创建一个自动加载器，支持多个命名空间前缀映射到不同的基础目录。

3. 实现一个自动加载器，处理命名空间解析优先级。

4. 创建一个自动加载器，支持类映射（classmap）提升性能。

5. 实现一个自动加载器，支持文件自动加载（files autoload）。

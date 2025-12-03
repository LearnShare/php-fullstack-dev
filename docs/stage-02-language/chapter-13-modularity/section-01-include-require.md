# 2.13.1 include 与 require

## 概述

`include` 和 `require` 用于在 PHP 脚本中引入其他文件。理解它们的区别和正确使用方式对于代码组织和模块化至关重要。

## include / require 对比

### 基本区别

| 关键字      | 失败时行为         |
| :---------- | :----------------- |
| `include`   | 发出警告 (`E_WARNING`)，脚本继续执行 |
| `require`   | 抛出致命错误 (`E_ERROR`)，脚本终止 |
| `include_once` | 仅在第一次包含文件时执行，避免重复定义 |
| `require_once` | 同上，但致命错误行为不变 |

### 基本用法

```php
<?php
declare(strict_types=1);

// include - 文件不存在时发出警告，继续执行
include 'config.php';

// require - 文件不存在时抛出致命错误，停止执行
require 'database.php';

// include_once - 避免重复包含
include_once 'functions.php';

// require_once - 避免重复包含，关键依赖
require_once __DIR__ . '/vendor/autoload.php';
```

## include 语句

### 基本语法

```php
include 'filename.php';
include $filename;
```

### 使用场景

- 模板文件（可选）
- 配置文件（非关键）
- 可选的辅助文件

### 示例

```php
<?php
declare(strict_types=1);

// 包含模板文件
include __DIR__ . '/templates/header.php';

// 处理逻辑
$content = 'Main content';

// 包含页脚
include __DIR__ . '/templates/footer.php';
```

## require 语句

### 基本语法

```php
require 'filename.php';
require $filename;
```

### 使用场景

- 关键依赖文件
- 类定义文件
- 必须存在的配置文件

### 示例

```php
<?php
declare(strict_types=1);

// 必须的配置文件
require __DIR__ . '/config/database.php';

// 必须的类文件
require __DIR__ . '/src/User.php';

// 使用
$user = new User();
```

## include_once / require_once

### 避免重复定义

```php
<?php
declare(strict_types=1);

// 使用 _once 避免重复定义
require_once __DIR__ . '/functions.php';
require_once __DIR__ . '/functions.php';  // 不会重复执行

// 如果使用 require，会报错：函数已定义
```

### 实际应用

```php
<?php
declare(strict_types=1);

// Composer autoload
require_once __DIR__ . '/vendor/autoload.php';

// 项目配置
require_once __DIR__ . '/config/app.php';

// 辅助函数
require_once __DIR__ . '/helpers/functions.php';
```

## 路径处理

### 使用 __DIR__

```php
<?php
declare(strict_types=1);

// 推荐：使用 __DIR__ 获取当前文件目录
$config = require __DIR__ . '/config/app.php';

// 不推荐：依赖当前工作目录
// $config = require 'config/app.php';
```

### 相对路径

```php
<?php
declare(strict_types=1);

// 当前目录
require __DIR__ . '/file.php';

// 上一级目录
require dirname(__DIR__) . '/file.php';

// 上两级目录
require dirname(__DIR__, 2) . '/file.php';
```

### 绝对路径

```php
<?php
declare(strict_types=1);

// 使用绝对路径
require '/var/www/app/config.php';
```

## 返回值

### include/require 的返回值

`include` 和 `require` 可以返回值：

```php
<?php
// config.php
return [
    'app_name' => 'My Application',
    'version' => '1.0.0'
];
```

```php
<?php
// main.php
$config = require __DIR__ . '/config.php';
echo $config['app_name'] . "\n";
```

### 条件返回

```php
<?php
// config.php
if (php_sapi_name() === 'cli') {
    return ['env' => 'development'];
} else {
    return ['env' => 'production'];
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FileInclusion
{
    public static function demonstrate(): void
    {
        echo "=== include ===\n";
        // 包含可选文件
        if (file_exists(__DIR__ . '/optional.php')) {
            include __DIR__ . '/optional.php';
        }
        
        echo "\n=== require ===\n";
        // 包含必需文件
        $config = require __DIR__ . '/config.php';
        print_r($config);
        
        echo "\n=== require_once ===\n";
        // 避免重复包含
        require_once __DIR__ . '/functions.php';
        require_once __DIR__ . '/functions.php';  // 不会重复执行
    }
}

FileInclusion::demonstrate();
```

## 注意事项

1. **选择合适的关键字**：关键依赖用 `require`，可选文件用 `include`。

2. **使用 _once**：对于可能被多次包含的文件，使用 `_once` 版本。

3. **路径处理**：始终使用 `__DIR__` 构建路径，避免依赖当前工作目录。

4. **返回值**：`include`/`require` 可以返回数组或配置，充分利用这个特性。

5. **性能考虑**：`_once` 版本需要检查文件是否已包含，性能略低。

## 练习

1. 创建一个配置加载函数，使用 `require` 加载配置文件并返回配置数组。

2. 编写一个函数，安全地包含模板文件，处理文件不存在的情况。

3. 实现一个模块加载器，使用 `require_once` 避免重复加载。

4. 创建一个函数，使用 `__DIR__` 构建相对路径。

5. 编写一个函数，检查文件是否存在后再包含。

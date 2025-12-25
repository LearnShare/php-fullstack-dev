# 3.5.2 use 语句

## 概述

`use` 语句用于导入命名空间中的类、函数和常量。本节介绍基础导入、别名（Alias）、分组导入、函数和常量导入等内容。

**章节类型**：语法性章节

**主要内容**：
- `use` 语句的基础语法
- 导入类
- 别名（Alias）的使用
- 分组导入（PHP 7.0+）
- 导入函数（PHP 5.6+）
- 导入常量（PHP 5.6+）
- `use` 语句的最佳实践

## 特性

- **简化代码**：避免使用完全限定名
- **别名支持**：可以给类起别名
- **分组导入**：可以分组导入多个类

## 语法/定义

### 基础导入

**语法**：`use Namespace\ClassName;`
**作用**：导入类到当前命名空间

### 别名

**语法**：`use Namespace\ClassName as Alias;`
**作用**：给类起别名

### 分组导入

**语法**：`use Namespace\{Class1, Class2, Class3};`
**要求**：PHP 7.0+
**作用**：一次导入多个类

### 函数导入

**语法**：`use function Namespace\functionName;`
**要求**：PHP 5.6+

### 常量导入

**语法**：`use const Namespace\CONSTANT_NAME;`
**要求**：PHP 5.6+

## 基本用法

### 基础导入示例

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use App\Model\User;
use App\Service\UserService;

class UserController
{
    public function index(): void
    {
        $user = new User();
    }
}
```

### 别名示例

```php
<?php
declare(strict_types=1);

use App\Model\User as UserModel;
use App\Service\User as UserService;

class UserController
{
    public function index(): void
    {
        $user = new UserModel();
    }
}
```

### 分组导入示例

```php
<?php
declare(strict_types=1);

use App\Model\{User, Post, Comment};
```

### 函数导入示例

```php
<?php
declare(strict_types=1);

use function App\Helper\formatDate;

$date = formatDate(time());
```

## 使用场景

- **简化代码**：避免使用完全限定名
- **解决冲突**：使用别名解决类名冲突
- **代码组织**：组织导入语句

## 注意事项

- **导入位置**：`use` 语句通常在命名空间声明之后
- **别名使用**：合理使用别名解决冲突
- **分组导入**：提高代码可读性

## 常见问题

### 问题 1：类名冲突
- **原因**：导入的类名冲突
- **解决**：使用别名

### 问题 2：导入顺序
- **建议**：按字母顺序组织导入
- **工具**：使用工具自动排序

## 最佳实践

- 使用 `use` 语句简化代码
- 合理使用别名解决冲突
- 使用分组导入提高可读性
- 按字母顺序组织导入

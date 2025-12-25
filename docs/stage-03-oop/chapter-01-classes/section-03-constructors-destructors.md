# 3.1.3 构造函数与析构函数

## 概述

构造函数和析构函数管理对象的生命周期。本节介绍 `__construct` 方法、构造器属性提升、析构函数 `__destruct`、资源管理等内容。

**章节类型**：语法性章节

**主要内容**：
- 构造函数的定义和使用
- 构造器属性提升（PHP 8.0+）
- 析构函数的定义和使用
- 资源管理（文件、数据库连接等）
- 对象初始化最佳实践
- 析构函数的使用场景

## 特性

- **构造函数**：对象创建时自动调用，用于初始化
- **析构函数**：对象销毁时自动调用，用于清理
- **构造器属性提升**：简化属性定义和初始化

## 语法/定义

### 构造函数

**语法**：`public function __construct(...) { ... }`
**调用时机**：对象创建时自动调用
**作用**：初始化对象属性

### 构造器属性提升

**语法**：`public function __construct(public Type $property) { ... }`
**要求**：PHP 8.0+
**作用**：同时定义和初始化属性

### 析构函数

**语法**：`public function __destruct() { ... }`
**调用时机**：对象销毁时自动调用
**作用**：清理资源

## 基本用法

### 构造函数示例

```php
<?php
declare(strict_types=1);

class User
{
    private string $name;
    private int $age;
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
}
```

### 构造器属性提升示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}
```

### 析构函数示例

```php
<?php
declare(strict_types=1);

class FileHandler
{
    private $file;
    
    public function __construct(string $filename)
    {
        $this->file = fopen($filename, 'r');
    }
    
    public function __destruct()
    {
        if ($this->file) {
            fclose($this->file);
        }
    }
}
```

## 使用场景

- **构造函数**：初始化对象属性、验证参数、建立连接
- **析构函数**：关闭文件、断开连接、清理资源

## 注意事项

- **资源管理**：及时释放资源
- **异常处理**：构造函数中抛出异常会影响对象创建
- **性能考虑**：析构函数不应执行耗时操作

## 常见问题

### 问题 1：资源泄漏
- **原因**：未在析构函数中释放资源
- **解决**：在析构函数中释放所有资源

### 问题 2：构造函数异常
- **原因**：构造函数中抛出异常
- **解决**：处理异常或使用工厂方法

## 最佳实践

- 使用构造器属性提升简化代码
- 在析构函数中释放资源
- 避免在析构函数中执行耗时操作
- 使用 try-finally 确保资源释放

# 2.20.4 PHP 8.5 新特性与示例

## 概述

PHP 8.5（计划2025年发布）引入管道操作符、`clone with` 语法、内置 URI 扩展、OPcache 必选等特性，进一步提升代码表达力与运行时性能。

> **注意**：PHP 8.5 目前仍在开发中，本文档内容基于 RFC 提案和开发计划。实际发布时特性可能有所变化。

## 管道操作符（|>）

```php
$result = "Hello World"
    |> htmlentities(...)
    |> str_split(...)
    |> fn($x) => array_map(strtoupper(...), $x)
    |> fn($x) => array_filter($x, fn($v) => $v !== 'O');
```

- 简化函数调用链，提升可读性，减少嵌套。

## 克隆时更新属性（Clone with）

```php
class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email,
    ) {}
}

$user = new User('Alice', 30, 'alice@example.com');
$newUser = clone $user with ['age' => 31, 'email' => 'alice.new@example.com'];
```

- 简化不可变对象的更新操作，避免手动克隆后逐个赋值。

## 内置 URI 扩展

```php
use Uri\Uri;

$uri = new Uri('https://example.com/path?query=value#fragment');
echo $uri->getScheme();   // https
echo $uri->getHost();      // example.com
$uri->setHost('newdomain.com');
```

- 提供统一、强大的 URI 处理 API，替代字符串拼接。

## PHP_BUILD_DATE 常量

```php
echo PHP_BUILD_DATE; // 2025-01-15 10:30:45
```

- 直接获取 PHP 二进制构建时间，便于版本追踪。

## 废弃非规范化类型转换

```php
// 废弃（会发出弃用警告）
$int = (integer) $value;
$float = (double) $value;
$bool = (boolean) $value;

// 推荐
$int = (int) $value;
$float = (float) $value;
$bool = (bool) $value;
```

- 统一类型转换语法，提升代码一致性。

## OPcache 成为必选扩展

- OPcache 现在总是可用，无需检查扩展是否存在，简化部署配置。

## 升级建议

1. 使用管道操作符重构复杂的函数调用链。
2. 用 `clone with` 简化不可变对象的更新逻辑。
3. 迁移到内置 URI 扩展，替代字符串拼接。
4. 更新所有非规范化类型转换为标准写法。
5. 利用 OPcache 必选特性，简化配置检查。

## 练习

- 使用管道操作符重构一个复杂的数据处理流程。
- 使用 `clone with` 创建不可变对象的更新方法。
- 使用内置 URI 扩展解析和修改 URL。

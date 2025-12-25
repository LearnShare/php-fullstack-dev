# 2.18.3 PHP 8.4 新特性与示例

## 概述

PHP 8.4（2024.11）引入属性钩子、不对称可见性、数组工具函数与 DOM API 改进，进一步简化代码并提升类型安全。

## 属性钩子（Property Hooks）

```php
class Locale
{
    public string $languageCode;

    public string $countryCode
    {
        set (string $countryCode) {
            $this->countryCode = strtoupper($countryCode);
        }
    }

    public string $combinedCode
    {
        get => sprintf("%s_%s", $this->languageCode, $this->countryCode);
        set (string $value) {
            [$this->languageCode, $this->countryCode] = explode('_', $value, 2);
        }
    }
}
```

- 替代传统 getter/setter，减少样板代码，提升可读性。

## 不对称可见性（Asymmetric Visibility）

```php
class User
{
    public string $name { get; private set; }
    public int $age { get; protected set; }

    public function __construct(string $name, int $age)
    {
        $this->name = $name; // 内部可写
        $this->age = $age;
    }
}

$user = new User('Alice', 30);
echo $user->name; // 可读
// $user->name = 'Bob'; // 错误：外部不可写
```

- 实现“只读对外、内部可写”的访问控制模式。

## array_first() 和 array_last()

```php
$numbers = [1, 2, 3, 4, 5];
$first = array_first($numbers); // 1
$last = array_last($numbers);  // 5
// 不影响数组内部指针
```

- 替代 `reset()`/`end()`，语义更清晰，不改变指针。

## #[\NoDiscard]

```php
#[\NoDiscard]
function importantFunction(): int
{
    return 42;
}

importantFunction(); // Warning: Return value must be used
```

- 强制调用方处理返回值，避免忽略关键结果。

## 构造函数属性提升支持 final

```php
class Base
{
    public function __construct(public final string $id) {}
}
```

- 在构造器属性提升中声明 `final`，防止子类覆盖。

## DOM API 改进

- 提供更现代、一致的 DOM 操作接口，简化 HTML/XML 处理。

## 升级建议

1. 将现有 getter/setter 逻辑迁移到属性钩子，减少代码量。
2. 使用不对称可见性简化只读属性的实现。
3. 用 `array_first/array_last` 替换所有 `reset/end` 调用。
4. 为关键函数添加 `#[\NoDiscard]`，提升代码健壮性。

## 练习

- 使用属性钩子创建 `Temperature` 类，自动转换摄氏度/华氏度。
- 使用不对称可见性实现 `BankAccount` 类，余额只读。
- 重构现有代码，用 `array_first/array_last` 替换相关函数。

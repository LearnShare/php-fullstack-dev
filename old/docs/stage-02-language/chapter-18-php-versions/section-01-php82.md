# 2.18.1 PHP 8.2 新特性与示例

## 概述

PHP 8.2（2022.12）聚焦“类型安全 + 不可变性 + 安全调试”。以下列出最具实战价值的特性。

## Readonly 类与析构器访问

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
    ) {}
}

$user = new User(1, 'Alice', 'alice@example.com');
// $user->name = 'Bob'; // Fatal: Cannot modify readonly property
```

- `readonly class` 内的属性默认只读，适合 DTO / 值对象。
- 析构器可安全访问只读属性，便于释放资源。

## 独立类型（Standalone Types）

- `null`、`false`、`true` 可独立声明，减少 `?Type` 或 `|null` 组合。

```php
function findUser(int $id): User|false
{
    $user = getUserFromDb($id);
    return $user !== null ? $user : false;
}
```

## 枚举在常量表达式中使用

```php
enum Status: string
{
    case PENDING = 'pending';
    case ACTIVE = 'active';
}

class Config
{
    public const DEFAULT_STATUS = Status::PENDING;
}
```

- 更易在配置类、属性默认值中使用枚举。

## 敏感参数值重写

```php
function login(
    string $username,
    #[SensitiveParameter] string $password,
): void {
    authenticate($username, $password);
}
```

- 栈追踪中自动隐藏标记的参数，防止敏感信息泄露。

## 新 Random 扩展

```php
use Random\Randomizer;
use Random\Engine\Secure;

$randomizer = new Randomizer(new Secure());
$token = bin2hex($randomizer->getBytes(32));
$number = $randomizer->getInt(1, 100);
```

- 新引擎提供可插拔策略，性能、安全性均提升。

## 升级建议

1. 评估哪些实体可以改写为 `readonly class`，提升不可变性。
2. 若大量使用“函数返回 null/false”，利用独立类型提升可读性。
3. 针对敏感接口（登录、支付）尽快加上 `#[SensitiveParameter]`。
4. 如果项目依赖自实现随机数，考虑迁移到 `Random` 扩展，减少安全隐患。

## 练习

- 将现有 DTO 改写为 `readonly class`，比较调试体验差异。
- 为枚举配置编写单元测试，验证常量绑定是否正确。
- 使用 `Randomizer` 实现一个密码生成器或验证码服务。

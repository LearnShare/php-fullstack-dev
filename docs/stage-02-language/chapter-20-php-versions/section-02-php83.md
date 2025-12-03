# 2.20.2 PHP 8.3 新特性与示例

## 概述

PHP 8.3（2023.11）持续加强类型系统与工具链，可概括为：更严格的类型声明、更丰富的字符串/数组 API，以及覆盖检测能力。

## json_validate()

```php
$json = '{"name":"Alice"}';
if (json_validate($json)) {
    $data = json_decode($json, true);
}
```

- 仅验证不解码，适合网关、ETL 等高频校验场景。

## 类常量显式类型

```php
class Config
{
    public const int MAX_USERS = 100;
    public const array ALLOWED_IPS = ['127.0.0.1'];
}
```

- 静态分析工具可据此检测错误赋值，增强约束力。

## 只读属性动态访问 & 覆盖属性类型

```php
readonly class Point
{
    public function __construct(public int $x, public int $y) {}

    public function withX(int $newX): self
    {
        return new self($newX, $this->y);
    }
}

class Child extends ParentClass
{
    public string $value; // 将 string|int 缩小为 string
}
```

- 克隆时可创建修改后的新对象；子类可声明更精确的类型。

## mb_str_pad()

- 正确处理多字节字符的字符串填充函数。

```php
mb_str_pad('你好', 10, ' ', STR_PAD_RIGHT, 'UTF-8');
```

## #[\Override]

```php
class Child extends Parent
{
    #[\Override]
    public function handle(): void
    {
        parent::handle();
    }
}
```

- 如果父类不存在该方法或签名不一致，将在编译期报错。

## 动态类常量获取

```php
$className = 'App\\Models\\User';
$ref = $className::class; // App\Models\User
```

- 有助于动态模块生成、反射等场景。

## 升级建议

1. 在 API 网关、消息队列消费者中优先使用 `json_validate()`，降低异常解析成本。
2. 逐步为配置类、枚举添加常量类型声明，配合静态分析提升安全性。
3. 在重构大型类继承体系前，使用 `#[\Override]` 保证重写的正确性。
4. 采用 `mb_str_pad()` 清理所有对多字节文本的错误处理代码。

## 练习

- 将 JSON 处理函数改造为“先验证后解码”。
- 为核心配置类添加常量类型，运行 PHPStan/ Psalm 验证效果。
- 练习使用 `#[\Override]` 捕捉方法签名漂移。

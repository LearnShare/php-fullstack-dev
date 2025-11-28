# 2.20 PHP 8.2-8.5 版本新特性

## 目标

- 了解 PHP 8.2、8.3、8.4、8.5 各版本的新特性。
- 掌握新特性的使用场景和最佳实践。
- 理解版本升级的注意事项和迁移指南。

## PHP 8.2 新特性（2022年12月发布）

### Readonly 类

- **语法**：`readonly class ClassName { ... }`
- 类的所有属性自动为 `readonly`。

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
    }
}

$user = new User(1, 'Alice', 'alice@example.com');
// $user->name = 'Bob'; // 错误：Cannot modify readonly property
```

### 析构器中的只读属性

- PHP 8.2 允许在析构器中访问只读属性。

```php
<?php
declare(strict_types=1);

readonly class Resource
{
    public function __construct(
        private $handle
    ) {
    }
    
    public function __destruct()
    {
        // PHP 8.2+ 允许在析构器中访问只读属性
        fclose($this->handle);
    }
}
```

### 独立类型（Standalone Types）

- `null`、`false`、`true` 可以作为独立类型使用。

```php
<?php
declare(strict_types=1);

// PHP 8.2+
function processValue(null|string $value): void
{
    // ...
}

// false 作为独立类型
function findUser(int $id): User|false
{
    $user = getUserFromDb($id);
    return $user !== null ? $user : false;
}

// true 作为独立类型（较少使用）
function isValid(): true
{
    // 必须返回 true
    return true;
}
```

### 常量表达式中的枚举

- 枚举可以在常量表达式中使用。

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case PENDING = 'pending';
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

class Config
{
    // PHP 8.2+ 允许在常量表达式中使用枚举
    public const DEFAULT_STATUS = Status::PENDING;
}
```

### 敏感参数值重写

- 在堆栈跟踪中隐藏敏感参数值。

```php
<?php
declare(strict_types=1);

use SensitiveParameter;

function login(
    string $username,
    #[\SensitiveParameter] string $password
): void
{
    // 密码在堆栈跟踪中会被隐藏
    authenticate($username, $password);
}

// 调用时如果发生异常，堆栈跟踪中密码会显示为 [SensitiveParameter]
```

### 新的随机扩展

- 新的 `Random` 扩展，提供更好的随机数生成器。

```php
<?php
declare(strict_types=1);

use Random\Randomizer;
use Random\Engine\Secure;

// PHP 8.2+ 新的随机扩展
$randomizer = new Randomizer(new Secure());

// 生成随机整数
$number = $randomizer->getInt(1, 100);

// 生成随机字符串
$string = $randomizer->getBytes(32);

// 打乱数组
$shuffled = $randomizer->shuffleArray([1, 2, 3, 4, 5]);
```

## PHP 8.3 新特性（2023年11月发布）

### json_validate() 函数

- **语法**：`json_validate(string $json, int $depth = 512, int $flags = 0): bool`
- 验证 JSON 字符串是否有效，无需解码。

```php
<?php
declare(strict_types=1);

// PHP 8.3+ 新增函数
$json = '{"name": "Alice", "age": 30}';

if (json_validate($json)) {
    // JSON 有效，可以安全解码
    $data = json_decode($json, true);
} else {
    // JSON 无效
    echo "Invalid JSON";
}

// 性能优势：只验证不解码，比 json_decode 更快
if (json_validate($largeJsonString)) {
    // 只在验证通过后才解码
    $data = json_decode($largeJsonString, true);
}
```

### 类常量显式类型

- **语法**：`public const Type $CONSTANT = value;`
- 类常量可以声明类型。

```php
<?php
declare(strict_types=1);

class Config
{
    // PHP 8.3+ 类常量显式类型
    public const int MAX_USERS = 100;
    public const string DEFAULT_LANGUAGE = 'en';
    public const array ALLOWED_IPS = ['127.0.0.1', '::1'];
    
    // 使用常量
    public function checkLimit(int $count): bool
    {
        return $count <= self::MAX_USERS;
    }
}
```

### 只读属性的动态访问

- PHP 8.3 允许在克隆时重新初始化只读属性。

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {
    }
    
    // PHP 8.3+ 允许在克隆时重新初始化只读属性
    public function withX(int $newX): self
    {
        return new self($newX, $this->y);
    }
}

$point = new Point(10, 20);
$newPoint = $point->withX(30); // 创建新对象，不是修改原对象
```

### 覆盖属性类型

- 子类可以覆盖父类的属性类型，使其更具体。

```php
<?php
declare(strict_types=1);

class ParentClass
{
    public string|int $value;
}

class ChildClass extends ParentClass
{
    // PHP 8.3+ 允许覆盖属性类型，使其更具体
    public string $value; // 从 string|int 缩小到 string
}
```

### 新的 mb_str_pad() 函数

- **语法**：`mb_str_pad(string $string, int $length, string $pad_string = " ", int $pad_type = STR_PAD_RIGHT, ?string $encoding = null): string`
- 多字节字符串填充函数。

```php
<?php
declare(strict_types=1);

// PHP 8.3+ 新增函数
$text = '你好';
$padded = mb_str_pad($text, 10, ' ', STR_PAD_RIGHT, 'UTF-8');
echo $padded; // "你好        "

// 与 str_pad() 的区别：正确处理多字节字符
$chinese = '世界';
$padded = mb_str_pad($chinese, 10, '·', STR_PAD_BOTH, 'UTF-8');
```

### 新的 #[\Override] 属性

- 明确标记覆盖父类方法，如果父类方法不存在会报错。

```php
<?php
declare(strict_types=1);

class ParentClass
{
    public function method(): void
    {
        echo "Parent";
    }
}

class ChildClass extends ParentClass
{
    // PHP 8.3+ 使用 #[\Override] 明确标记覆盖
    #[\Override]
    public function method(): void
    {
        echo "Child";
    }
    
    // 如果父类没有 method2()，会报错
    // #[\Override]
    // public function method2(): void {} // 错误：ParentClass 没有 method2()
}
```

### 动态类常量获取

- 使用 `::class` 获取类名时，可以处理动态类名。

```php
<?php
declare(strict_types=1);

// PHP 8.3+ 改进
$className = 'App\\Models\\User';
$class = $className::class; // "App\Models\User"

// 在命名空间中使用
namespace App\Http\Controllers;

use App\Models\User;

$class = User::class; // "App\Models\User"
```

## PHP 8.4 新特性（2024年11月发布）

### 属性钩子（Property Hooks）

- **语法**：在属性定义中使用 `get` 和 `set` 钩子
- 允许在属性的读取和写入操作中定义自定义逻辑，减少样板代码。

```php
<?php
declare(strict_types=1);

class Locale
{
    public string $languageCode;

    // set 钩子：写入时自动转换为大写
    public string $countryCode
    {
        set (string $countryCode) {
            $this->countryCode = strtoupper($countryCode);
        }
    }

    // get 和 set 钩子：自动组合和拆分
    public string $combinedCode
    {
        get => sprintf("%s_%s", $this->languageCode, $this->countryCode);
        set (string $value) {
            [$this->languageCode, $this->countryCode] = explode('_', $value, 2);
        }
    }

    public function __construct(string $languageCode, string $countryCode)
    {
        $this->languageCode = $languageCode;
        $this->countryCode = $countryCode;
    }
}

$locale = new Locale('en', 'us');
$locale->countryCode = 'gb';
echo $locale->combinedCode; // 输出：en_GB
```

### 不对称可见性（Asymmetric Visibility）

- 允许属性的读取和写入具有不同的可见性修饰符。

```php
<?php
declare(strict_types=1);

class User
{
    // 公有读取，私有写入
    public string $name
    {
        get;
        private set;
    }
    
    // 公有读取，受保护写入
    public int $age
    {
        get;
        protected set;
    }
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name; // 可以设置（在类内部）
        $this->age = $age;
    }
}

$user = new User('Alice', 30);
echo $user->name; // 可以读取
// $user->name = 'Bob'; // 错误：无法从外部设置
```

### array_first() 和 array_last() 函数

- **语法**：`array_first(array $array): mixed`、`array_last(array $array): mixed`
- 获取数组的第一个和最后一个元素，不影响内部指针。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

$first = array_first($numbers); // 1
$last = array_last($numbers);    // 5

// 不影响数组内部指针
current($numbers); // 仍然是 1
```

### #[\NoDiscard] 属性

- 标记函数或方法的返回值不应被忽略。

```php
<?php
declare(strict_types=1);

#[\NoDiscard]
function importantFunction(): int
{
    return 42;
}

// 使用返回值（正确）
$result = importantFunction();

// 忽略返回值（会触发警告）
importantFunction(); // Warning: Return value of importantFunction() must be used
```

### 构造函数属性提升支持 final

- 在构造函数属性提升语法中，允许将属性声明为 `final`。

```php
<?php
declare(strict_types=1);

class Base
{
    public function __construct(
        public final string $id  // PHP 8.4+ 支持
    ) {
    }
}
```

### 更新的 DOM API

- 对 DOM 扩展进行了更新，提供了更现代和一致的 API。

```php
<?php
declare(strict_types=1);

// PHP 8.4+ 改进的 DOM API
$dom = new DOMDocument();
$dom->loadHTML('<html><body><p>Hello</p></body></html>');

// 更简洁的 API
$paragraph = $dom->getElementsByTagName('p')[0];
echo $paragraph->textContent; // Hello
```

## PHP 8.5 新特性（2025年发布）

### 管道操作符（|>）

- **语法**：`表达式 |> 函数(...)`
- 将表达式的结果作为参数传递给下一个函数，简化函数调用链。

```php
<?php
declare(strict_types=1);

// PHP 8.5+ 管道操作符
$result = "Hello World"
    |> htmlentities(...)
    |> str_split(...)
    |> fn($x) => array_map(strtoupper(...), $x)
    |> fn($x) => array_filter($x, fn($v) => $v !== 'O');

var_dump($result); 
// 输出：['H', 'E', 'L', 'L', 'W', 'R', 'D']

// 等价于传统写法
$result = array_filter(
    array_map(
        strtoupper(...),
        str_split(htmlentities("Hello World"))
    ),
    fn($v) => $v !== 'O'
);
```

### 克隆时更新属性（Clone with）

- **语法**：`clone $object with ['property' => 'value']`
- 在克隆对象的同时更新其指定属性。

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email
    ) {
    }
}

$user = new User('Alice', 30, 'alice@example.com');

// PHP 8.5+ clone with 语法
$newUser = clone $user with ['age' => 31, 'email' => 'alice.new@example.com'];

// 等价于传统写法
$newUser = clone $user;
$newUser->age = 31;
$newUser->email = 'alice.new@example.com';
```

### 内置 URI 扩展

- 基于 uriparser 和 Lexbor 的内置 URI 扩展，提供统一且强大的 API。

```php
<?php
declare(strict_types=1);

// PHP 8.5+ 内置 URI 扩展
use Uri\Uri;

$uri = new Uri('https://example.com/path?query=value#fragment');

echo $uri->getScheme();   // https
echo $uri->getHost();     // example.com
echo $uri->getPath();     // /path
echo $uri->getQuery();    // query=value
echo $uri->getFragment(); // fragment

// 修改 URI
$uri->setHost('newdomain.com');
echo (string) $uri; // https://newdomain.com/path?query=value#fragment
```

### PHP_BUILD_DATE 常量

- 新增 `PHP_BUILD_DATE` 常量，直接暴露 PHP 二进制文件构建的日期和时间。

```php
<?php
echo PHP_BUILD_DATE; // 例如：2025-01-15 10:30:45
```

### 废弃非规范化标量类型转换

- 废弃了 `(integer)`、`(double)`、`(boolean)` 和 `(binary)` 等非规范化的类型转换写法。

```php
<?php
// PHP 8.5+ 废弃的写法（会发出弃用警告）
$int = (integer) $value;    // 使用 (int) 替代
$float = (double) $value;  // 使用 (float) 替代
$bool = (boolean) $value;  // 使用 (bool) 替代
$str = (binary) $value;    // 使用 (string) 替代

// 推荐写法
$int = (int) $value;
$float = (float) $value;
$bool = (bool) $value;
$str = (string) $value;
```

### OPcache 成为必选扩展

- 在 PHP 8.5 中，OPcache 现在是一个必选扩展，会自动内置到每个 PHP 二进制文件中。

```php
<?php
// PHP 8.5+ OPcache 总是可用
if (function_exists('opcache_get_status')) {
    $status = opcache_get_status();
    // 使用 OPcache 状态信息
}
```

## 版本升级指南

### 从 PHP 8.1 升级到 8.2

1. **检查废弃功能**
   ```bash
   php -l your-file.php
   ```

2. **更新只读类**
   - 将需要不可变的对象标记为 `readonly class`

3. **使用新的随机扩展**
   - 考虑迁移到新的 `Random` 扩展

### 从 PHP 8.2 升级到 8.3

1. **使用 json_validate()**
   - 在解码前验证 JSON，提升性能

2. **添加类常量类型**
   - 为类常量添加类型声明

3. **使用 #[\Override] 属性**
   - 明确标记覆盖的方法

### 从 PHP 8.3 升级到 8.4

1. **使用属性钩子**
   - 将 getter/setter 逻辑迁移到属性钩子

2. **使用不对称可见性**
   - 利用不对称可见性简化属性访问控制

3. **使用 array_first() 和 array_last()**
   - 替换 `reset()` 和 `end()` 的使用

4. **添加 #[\NoDiscard] 属性**
   - 标记不应忽略返回值的函数

### 从 PHP 8.4 升级到 8.5

1. **使用管道操作符**
   - 简化函数调用链，提升可读性

2. **使用 clone with 语法**
   - 简化对象克隆和属性更新

3. **迁移到内置 URI 扩展**
   - 使用新的 URI API 替代旧的解析方法

4. **更新类型转换**
   - 将 `(integer)`、`(double)` 等替换为 `(int)`、`(float)`

5. **利用 OPcache 必选特性**
   - OPcache 现在总是可用，无需检查扩展是否存在

### 兼容性检查

```php
<?php
// 检查 PHP 版本
if (version_compare(PHP_VERSION, '8.3.0', '>=')) {
    // 使用 PHP 8.3+ 特性
    if (json_validate($json)) {
        // ...
    }
} else {
    // 降级处理
    $data = json_decode($json, true);
    if (json_last_error() === JSON_ERROR_NONE) {
        // ...
    }
}
```

## 最佳实践

### 1. 使用 json_validate() 提升性能

```php
<?php
declare(strict_types=1);

// 推荐：先验证再解码
if (json_validate($json)) {
    $data = json_decode($json, true);
}

// 不推荐：直接解码（如果 JSON 无效会浪费资源）
$data = json_decode($json, true);
if (json_last_error() !== JSON_ERROR_NONE) {
    // 处理错误
}
```

### 2. 使用只读类创建不可变对象

```php
<?php
declare(strict_types=1);

readonly class Money
{
    public function __construct(
        public float $amount,
        public string $currency
    ) {
    }
    
    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Currency mismatch');
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
}
```

### 3. 使用 #[\Override] 提高代码可维护性

```php
<?php
declare(strict_types=1);

class BaseRepository
{
    public function find(int $id): ?Entity
    {
        // 基础实现
    }
}

class UserRepository extends BaseRepository
{
    #[\Override]
    public function find(int $id): ?User
    {
        // 如果父类方法签名改变，这里会报错
        return parent::find($id);
    }
}
```

## 练习

1. 使用 `json_validate()` 优化一个 JSON 处理函数，比较性能差异。

2. 创建一个只读类 `Address`，包含地址信息，并提供 `__toString()` 方法。

3. 使用新的 `Random` 扩展实现一个密码生成器。

4. 为现有类添加类常量类型声明。

5. 使用 `#[\Override]` 属性标记所有覆盖的方法。

6. 编写一个版本兼容性检查工具，根据 PHP 版本选择不同的实现。

7. **PHP 8.4 练习**：
   - 使用属性钩子创建一个 `Temperature` 类，自动在摄氏度和华氏度之间转换。
   - 使用不对称可见性创建一个 `BankAccount` 类，余额只能读取，不能直接写入。
   - 使用 `array_first()` 和 `array_last()` 重写现有代码。

8. **PHP 8.5 练习**：
   - 使用管道操作符重构一个复杂的函数调用链。
   - 使用 `clone with` 语法创建一个不可变对象的更新方法。
   - 使用内置 URI 扩展解析和修改 URL。

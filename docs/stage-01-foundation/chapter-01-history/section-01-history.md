# 1.0.1 PHP 起源与发展历程

## 概述

PHP（PHP: Hypertext Preprocessor）是一种广泛使用的开源服务器端脚本语言，特别适合 Web 开发。本节介绍 PHP 的起源、发展历程和重要里程碑。

## PHP 的诞生（1994-1995）

### Rasmus Lerdorf 的初衷

1994 年，Rasmus Lerdorf 创建了 PHP，最初是为了维护个人主页而编写的一组 Perl 脚本。PHP 最初代表 "Personal Home Page"（个人主页）。

### PHP/FI（1995）

1995 年，Rasmus 发布了 PHP/FI（PHP/FI 2.0），这是 PHP 的第一个公开版本。PHP/FI 包含：

- 基本的表单处理功能
- 数据库访问能力
- 简单的脚本执行

```php
<!-- 早期 PHP/FI 示例 -->
<!--#exec cgi="/cgi-bin/guestbook.cgi" -->
```

## PHP 3.0（1998）- 真正的 PHP

### 重要里程碑

1998 年，Andi Gutmans 和 Zeev Suraski 重写了 PHP 解析器，发布了 PHP 3.0。这是 PHP 历史上第一个真正意义上的版本：

- **名称变更**：PHP 的含义从 "Personal Home Page" 改为 "PHP: Hypertext Preprocessor"（递归缩写）
- **语法改进**：引入了更现代的语法结构
- **扩展性**：支持扩展模块
- **跨平台**：支持多种操作系统

### 主要特性

- 支持面向对象编程（基础）
- 更好的错误处理
- 支持多种数据库
- 更强大的字符串处理

## PHP 4.0（2000）- Zend Engine

### Zend Engine 的引入

2000 年，Andi 和 Zeev 创建了 Zend 公司，并发布了 PHP 4.0，引入了 Zend Engine：

- **性能提升**：Zend Engine 显著提升了 PHP 的执行性能
- **会话管理**：内置 Session 支持
- **输出缓冲**：支持输出缓冲机制
- **更多扩展**：丰富的扩展库

### 影响

PHP 4.0 使 PHP 成为主流的 Web 开发语言，被广泛用于构建动态网站。

## PHP 5.0（2004）- 面向对象革命

### 重大改进

2004 年发布的 PHP 5.0 引入了完整的面向对象编程支持：

- **完整的 OOP**：类、对象、继承、接口、抽象类
- **异常处理**：try-catch 异常机制
- **改进的 XML 支持**：SimpleXML、DOM、XMLReader
- **PDO**：PHP Data Objects，统一的数据库访问接口
- **Zend Engine 2**：全新的引擎架构

### 代码示例

```php
<?php
// PHP 5.0 引入的完整 OOP 支持
class User
{
    private $name;
    
    public function __construct($name)
    {
        $this->name = $name;
    }
    
    public function getName()
    {
        return $this->name;
    }
}

// 异常处理
try {
    $user = new User("张三");
    echo $user->getName();
} catch (Exception $e) {
    echo "错误: " . $e->getMessage();
}
```

## PHP 5.x 系列（2005-2012）

### PHP 5.1（2005）

- PDO 成为核心扩展
- 性能改进

### PHP 5.2（2006）

- JSON 支持（`json_encode()`、`json_decode()`）
- 改进的错误处理
- 新的过滤扩展

### PHP 5.3（2009）

- **命名空间**：支持命名空间
- **闭包**：匿名函数和闭包
- **后期静态绑定**：`static::` 关键字
- **Nowdoc**：类似 Heredoc，但不解析变量
- **`__DIR__` 魔术常量**：获取当前目录

```php
<?php
// PHP 5.3 命名空间
namespace App\Controllers;

class UserController
{
    // PHP 5.3 闭包
    public function processUsers($users)
    {
        return array_map(function($user) {
            return $user->getName();
        }, $users);
    }
}
```

### PHP 5.4（2012）

- **短数组语法**：`[]` 替代 `array()`
- **Traits**：代码复用机制
- **内置 Web 服务器**：`php -S`
- **数组解引用**：函数返回数组可直接访问
- **改进的错误报告**

```php
<?php
// PHP 5.4 短数组语法
$arr = [1, 2, 3];  // 替代 array(1, 2, 3)

// PHP 5.4 Traits
trait Loggable
{
    public function log($message)
    {
        echo $message;
    }
}

class User
{
    use Loggable;
}
```

### PHP 5.5（2013）

- **生成器**：`yield` 关键字
- **`finally` 关键字**：异常处理的 finally 块
- **`::class` 魔术常量**：获取类名
- **`array_column()` 函数**：提取数组列
- **密码哈希 API**：`password_hash()`、`password_verify()`

```php
<?php
// PHP 5.5 生成器
function numbers()
{
    for ($i = 1; $i <= 10; $i++) {
        yield $i;
    }
}

foreach (numbers() as $num) {
    echo $num . "\n";
}
```

### PHP 5.6（2014）

- **可变参数**：`...` 操作符
- **参数解包**：`...$args`
- **幂运算**：`**` 操作符
- **`use const` 和 `use function`**：导入常量和函数
- **`__debugInfo()` 魔术方法**

```php
<?php
// PHP 5.6 可变参数
function sum(...$numbers)
{
    return array_sum($numbers);
}

echo sum(1, 2, 3, 4);  // 10

// PHP 5.6 幂运算
$result = 2 ** 3;  // 8
```

## PHP 7.0（2015）- 性能革命

### 重大突破

2015 年发布的 PHP 7.0 带来了性能的巨大提升：

- **性能提升**：比 PHP 5.6 快 2-3 倍
- **标量类型声明**：`int`、`float`、`string`、`bool`
- **返回类型声明**：函数返回值类型
- **空合并运算符**：`??`
- **太空船运算符**：`<=>`
- **匿名类**：`new class {}`
- **Unicode 码点转义**：`\u{xxxx}`
- **Zend Engine 3**：全新的引擎

### 代码示例

```php
<?php
declare(strict_types=1);

// PHP 7.0 标量类型声明和返回类型
function calculate(int $a, int $b): int
{
    return $a + $b;
}

// PHP 7.0 空合并运算符
$name = $_GET['name'] ?? 'Guest';

// PHP 7.0 太空船运算符
usort($users, fn($a, $b) => $a->age <=> $b->age);
```

### 性能对比

| 版本 | 相对性能 | 说明 |
| :--- | :------- | :--- |
| PHP 5.6 | 1x | 基准 |
| PHP 7.0 | 2-3x | 性能翻倍 |
| PHP 7.1 | 2.5-3.5x | 进一步优化 |
| PHP 7.2 | 2.5-3.5x | 持续改进 |
| PHP 7.3 | 2.5-3.5x | 小幅提升 |
| PHP 7.4 | 2.5-3.5x | 预加载特性 |

## PHP 7.x 系列（2016-2020）

### PHP 7.1（2016）

- **可空类型**：`?Type`
- **`void` 返回类型**
- **类常量可见性**：`public`、`protected`、`private`
- **多 catch 语句**：`catch (A \| B $e)`
- **列表解构**：`[$a, $b] = [1, 2]`

### PHP 7.2（2017）

- **参数类型扩展**：`object` 类型
- **`count()` 优化**：对可数对象更高效
- **Libsodium 扩展**：现代加密库
- **弃用 `each()` 函数**

### PHP 7.3（2018）

- **灵活的 Heredoc/Nowdoc**：结束标记可以缩进
- **`array_key_first()` 和 `array_key_last()`**
- **`is_countable()` 函数**
- **改进的错误处理**

### PHP 7.4（2019）

- **类型属性**：类属性类型声明
- **箭头函数**：`fn() =>`
- **预加载**：OPcache 预加载
- **弱引用**：`WeakReference` 类
- **协变返回和逆变参数**
- **`str_split()` 支持多字节**

```php
<?php
// PHP 7.4 类型属性
class User
{
    public int $id;
    public string $name;
}

// PHP 7.4 箭头函数
$users = array_map(fn($user) => $user->name, $users);
```

## PHP 8.0（2020）- 现代 PHP

### 重大特性

2020 年发布的 PHP 8.0 标志着 PHP 进入现代编程语言行列：

- **JIT 编译**：Just-In-Time 编译，进一步提升性能
- **命名参数**：函数调用时指定参数名
- **联合类型**：`int \| string`
- **`match` 表达式**：更强大的 switch
- **构造器属性提升**：简化类定义
- **`str_contains()` 等新函数**
- **改进的错误处理**：更友好的错误信息
- **Attributes（注解）**：替代 PHPDoc 注释

### 代码示例

```php
<?php
declare(strict_types=1);

// PHP 8.0 构造器属性提升
class User
{
    public function __construct(
        public int $id,
        public string $name,
    ) {}
}

// PHP 8.0 联合类型
function process(int | string $value): void
{
    // ...
}

// PHP 8.0 match 表达式
$result = match ($status) {
    1 => 'active',
    2 => 'inactive',
    default => 'unknown',
};

// PHP 8.0 命名参数
createUser(name: '张三', age: 25);
```

### 性能提升

PHP 8.0 在 PHP 7.4 的基础上进一步提升了性能，特别是在 JIT 编译的帮助下，CPU 密集型任务性能提升显著。

## PHP 8.1（2021）

### 主要特性

- **枚举（Enums）**：原生枚举支持
- **只读属性**：`readonly` 关键字
- **`first-class callable`**：`$fn = strlen(...)`
- **交集类型**：`A & B`
- **`never` 返回类型**：表示永不返回
- **数组解构支持字符串键**

```php
<?php
// PHP 8.1 枚举
enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

// PHP 8.1 只读属性
class User
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
    ) {}
}
```

## PHP 8.2+（2022-2025）

### PHP 8.2（2022年12月8日）

PHP 8.2 于 2022 年 12 月 8 日发布，主要特性包括：

- **Readonly 类**：整个类可以声明为只读，所有属性自动为只读
- **独立类型**：`null`、`true`、`false` 可以作为独立类型使用
- **枚举常量表达式**：枚举中可以使用常量表达式
- **敏感参数**：使用 `#[\SensitiveParameter]` 属性标记敏感参数，避免在错误堆栈中泄露
- **全新 Random 扩展**：提供更强大和灵活的随机数生成

详细内容请参考：[2.19.1 PHP 8.2 新特性与示例](../../stage-02-language/chapter-19-php-versions/section-01-php82.md)

### PHP 8.3（2023年11月23日）

PHP 8.3 于 2023 年 11 月 23 日发布，主要特性包括：

- **`json_validate()` 函数**：快速验证 JSON 字符串是否有效，无需解码
- **类常量类型**：类常量可以声明类型
- **`#[\Override]` 属性**：标记重写父类方法，提供编译时检查
- **`mb_str_pad()` 函数**：多字节字符串填充
- **覆盖属性类型**：子类可以覆盖父类属性的类型（更严格）

详细内容请参考：[2.19.2 PHP 8.3 新特性与示例](../../stage-02-language/chapter-19-php-versions/section-02-php83.md)

### PHP 8.4（2024年11月21日）

PHP 8.4 于 2024 年 11 月 21 日发布，主要特性包括：

- **属性钩子**：属性的 getter 和 setter 钩子
- **不对称可见性**：属性的读写可见性可以不同
- **`array_first()` 和 `array_last()` 函数**：获取数组的第一个和最后一个元素
- **`#[\NoDiscard]` 属性**：标记不应丢弃的返回值
- **DOM API 改进**：更好的 DOM 操作 API

详细内容请参考：[2.19.3 PHP 8.4 新特性与示例](../../stage-02-language/chapter-19-php-versions/section-03-php84.md)

### PHP 8.5（2025年11月20日）

PHP 8.5 于 2025 年 11 月 20 日正式发布，主要特性包括：

- **管道操作符**：`|>` 操作符，用于函数式编程
- **`clone with` 表达式**：克隆对象并修改属性
- **内置 URI 扩展**：提供了标准化的 URL 解析和操作能力
- **`#[\NoDiscard]` 属性**：标记不应丢弃的返回值
- **OPcache 必选**：OPcache 成为核心扩展，不再可选

详细内容请参考：[2.19.4 PHP 8.5 新特性与示例](../../stage-02-language/chapter-19-php-versions/section-04-php85.md)

## 版本演进总结

### 性能演进

| 版本 | 相对性能 | 主要改进 |
| :--- | :------- | :------- |
| PHP 5.6 | 1x | 基准 |
| PHP 7.0 | 2-3x | Zend Engine 3 |
| PHP 7.1 | 2.5-3.5x | 进一步优化 |
| PHP 7.2 | 2.5-3.5x | 持续改进 |
| PHP 7.3 | 2.5-3.5x | 小幅提升 |
| PHP 7.4 | 2.5-3.5x | 预加载特性 |
| PHP 8.0 | 3-5x | JIT 编译 |
| PHP 8.1 | 3-5x | 枚举优化 |
| PHP 8.2+ | 3-5x | 持续优化 |

### 类型系统演进

- **PHP 5.0**：基础类型提示（类、接口）
- **PHP 7.0**：标量类型声明（`int`、`string`、`float`、`bool`）
- **PHP 7.1**：可空类型（`?Type`）
- **PHP 7.2**：`object` 类型
- **PHP 8.0**：联合类型（`int | string`）
- **PHP 8.1**：交集类型（`A & B`）
- **PHP 8.2**：独立类型（`null`、`true`、`false`）

### OOP 演进

- **PHP 5.0**：完整的 OOP 支持
- **PHP 5.3**：命名空间、后期静态绑定
- **PHP 5.4**：Traits
- **PHP 7.0**：匿名类
- **PHP 7.4**：类型属性
- **PHP 8.0**：构造器属性提升、Attributes
- **PHP 8.1**：枚举、只读属性
- **PHP 8.2**：Readonly 类
- **PHP 8.4**：属性钩子

### 版本选择建议

#### 开发环境

- **推荐**：PHP 8.4 或 8.5（最新稳定版）
- **原因**：获得最新特性，开发体验更好

#### 生产环境

- **新项目**：PHP 8.4 或 8.5
- **现有项目**：根据依赖兼容性选择
- **旧项目迁移**：逐步升级，先到 8.1，再到 8.2+

#### 版本支持周期

| 版本 | 发布时间 | 安全支持截止 | 说明 |
| :--- | :------- | :----------- | :--- |
| PHP 7.4 | 2019-11 | 2022-11 | 已停止支持 |
| PHP 8.0 | 2020-11 | 2023-11 | 已停止支持 |
| PHP 8.1 | 2021-11 | 2024-11 | 已停止支持 |
| PHP 8.2 | 2022-12 | 2025-12 | 安全支持中 |
| PHP 8.3 | 2023-11 | 2026-11 | 安全支持中 |
| PHP 8.4 | 2024-11 | 2027-11 | 安全支持中 |
| PHP 8.5 | 2025-11 | 2028-11 | 安全支持中 |

**建议**：使用仍在安全支持期内的版本。

## PHP 社区发展

### 重要组织

- **PHP Group**：PHP 核心开发团队
- **Zend Technologies**：Zend Engine 的维护者
- **PHP-FIG**：PHP Framework Interop Group，制定 PSR 标准
- **Composer**：PHP 依赖管理工具

### 生态系统

- **框架**：Laravel、Symfony、CodeIgniter、Yii 等
- **CMS**：WordPress、Drupal、Joomla 等
- **包管理**：Composer、PEAR
- **测试框架**：PHPUnit、Codeception 等

## 时间线总结

| 年份 | 版本 | 发布日期 | 重要特性 |
| :--- | :--- | :------- | :------- |
| 1995 | PHP/FI | 1995年6月8日 | 第一个公开版本 |
| 1998 | PHP 3.0 | 1998年6月6日 | 真正的 PHP，Zend Engine 前身 |
| 2000 | PHP 4.0 | 2000年5月22日 | Zend Engine，性能提升 |
| 2004 | PHP 5.0 | 2004年7月13日 | 完整 OOP 支持，Zend Engine 2 |
| 2009 | PHP 5.3 | 2009年6月30日 | 命名空间、闭包 |
| 2012 | PHP 5.4 | 2012年3月1日 | 短数组语法、Traits |
| 2015 | PHP 7.0 | 2015年12月3日 | 性能革命，类型声明 |
| 2020 | PHP 8.0 | 2020年11月26日 | JIT 编译，现代特性 |
| 2021 | PHP 8.1 | 2021年11月25日 | 枚举、只读属性 |
| 2022 | PHP 8.2 | 2022年12月8日 | Readonly 类、独立类型 |
| 2023 | PHP 8.3 | 2023年11月23日 | `json_validate()`、类常量类型 |
| 2024 | PHP 8.4 | 2024年11月21日 | 属性钩子、不对称可见性 |
| 2025 | PHP 8.5 | 2025年11月20日 | 管道操作符、`clone with`、内置 URI 扩展 |

## 学习建议

1. **了解历史**：理解 PHP 的发展历程有助于理解当前的设计决策
2. **版本选择**：根据项目需求选择合适的 PHP 版本
3. **持续学习**：关注 PHP 新版本的发布和特性更新

## 相关章节

- **[1.0.2 PHP 生态系统](section-02-ecosystem.md)**：PHP 流行的技术、框架、工具等相关生态
- **[2.19 PHP 版本新特性](../../stage-02-language/chapter-19-php-versions/readme.md)**：详细了解 PHP 8.2-8.5 的新特性

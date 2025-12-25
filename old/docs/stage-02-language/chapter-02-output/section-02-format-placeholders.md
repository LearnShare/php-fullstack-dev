# 2.2.2 格式化占位符详解

## 目标

- 深入理解 `printf` 和 `sprintf` 的格式化占位符语法。
- 掌握字符串、整数、浮点数的各种格式化选项。
- 学会使用参数位置指定、填充、对齐等高级特性。
- 能够在实际项目中应用格式化占位符解决实际问题。

## 占位符语法结构

`printf` 和 `sprintf` 使用格式化字符串，其中占位符遵循 C 语言 `printf` 标准。占位符的完整语法结构如下：

```
%[参数位置$][填充字符][对齐标志][宽度][.精度][类型]
```

### 语法组件说明

| 组件 | 说明 | 示例 |
| :--- | :--- | :--- |
| `%` | 占位符起始符 | 必需 |
| `参数位置$` | 指定使用第几个参数（从 1 开始） | `%1$s` 使用第一个参数 |
| `填充字符` | 用于填充空位的字符（默认空格） | `%05d` 用 0 填充 |
| `对齐标志` | `-` 左对齐，`+` 显示正负号 | `%-10s` 左对齐 |
| `宽度` | 最小输出宽度（数字） | `%10s` 至少 10 个字符 |
| `.精度` | 小数点后位数或字符串最大长度 | `%.2f` 保留 2 位小数 |
| `类型` | 数据类型标识符 | `s` 字符串，`d` 整数 |

## 基础类型占位符

### 字符串类型

| 占位符 | 语法 | 说明 | 示例 |
| :----- | :--- | :--- | :--- |
| `%s` | `%s` | 字符串 | `printf("%s", "Hello");` → `Hello` |
| `%10s` | `%宽度s` | 右对齐，最小宽度 10 | `printf("%10s", "Hi");` → `        Hi` |
| `%-10s` | `%-宽度s` | 左对齐，最小宽度 10 | `printf("%-10s", "Hi");` → `Hi        ` |
| `%.5s` | `%.精度s` | 最多显示 5 个字符 | `printf("%.5s", "Hello World");` → `Hello` |
| `%10.5s` | `%宽度.精度s` | 宽度 10，最多 5 个字符 | `printf("%10.5s", "Hello World");` → `     Hello` |

示例代码：

```php
<?php
declare(strict_types=1);

$name = "张三";
$longText = "这是一段很长的文本内容";

// 基础字符串
printf("姓名：%s\n", $name);  // 姓名：张三

// 指定宽度（右对齐）
printf("姓名：%10s\n", $name);  // 姓名：        张三

// 左对齐
printf("姓名：%-10s\n", $name);  // 姓名：张三        

// 限制长度
printf("摘要：%.10s\n", $longText);  // 摘要：这是一段很长的文本

// 组合使用
printf("姓名：%-10.5s\n", $name);  // 姓名：张三      
```

### 整数类型

| 占位符 | 语法 | 说明 | 示例 |
| :----- | :--- | :--- | :--- |
| `%d` | `%d` | 有符号十进制整数 | `printf("%d", 42);` → `42` |
| `%u` | `%u` | 无符号十进制整数 | `printf("%u", 42);` → `42` |
| `%o` | `%o` | 八进制整数 | `printf("%o", 42);` → `52` |
| `%x` / `%X` | `%x` | 十六进制（小写/大写） | `printf("%x", 255);` → `ff` |
| `%05d` | `%填充宽度d` | 用 0 填充到 5 位 | `printf("%05d", 42);` → `00042` |
| `%+d` | `%+d` | 显示正负号 | `printf("%+d", 42);` → `+42` |
| `%-10d` | `%-宽度d` | 左对齐，宽度 10 | `printf("%-10d", 42);` → `42        ` |

示例代码：

```php
<?php
declare(strict_types=1);

$id = 7;
$userId = 12345;
$negative = -42;

// 基础整数
printf("ID: %d\n", $id);  // ID: 7

// 左侧补零（常用于订单号、编号）
printf("订单号：%05d\n", $id);  // 订单号：00007
printf("用户ID：%08d\n", $userId);  // 用户ID：00012345

// 显示正负号
printf("余额：%+d\n", $negative);  // 余额：-42
printf("余额：%+d\n", 100);  // 余额：+100

// 指定宽度
printf("ID: %10d\n", $id);  // ID:          7
printf("ID: %-10d\n", $id);  // ID: 7         

// 不同进制
printf("十进制：%d\n", 255);  // 十进制：255
printf("八进制：%o\n", 255);  // 八进制：377
printf("十六进制：%x\n", 255);  // 十六进制：ff
printf("十六进制：%X\n", 255);  // 十六进制：FF
```

### 浮点数类型

| 占位符 | 语法 | 说明 | 示例 |
| :----- | :--- | :--- | :--- |
| `%f` | `%f` | 浮点数（默认 6 位小数） | `printf("%f", 3.14);` → `3.140000` |
| `%.2f` | `%.精度f` | 保留 2 位小数 | `printf("%.2f", 3.14159);` → `3.14` |
| `%10.2f` | `%宽度.精度f` | 宽度 10，保留 2 位小数 | `printf("%10.2f", 3.14);` → `      3.14` |
| `%e` / `%E` | `%e` | 科学计数法（小写/大写） | `printf("%e", 1000);` → `1.000000e+3` |
| `%g` / `%G` | `%g` | 自动选择最短格式 | `printf("%g", 3.14);` → `3.14` |

示例代码：

```php
<?php
declare(strict_types=1);

$price = 199.9;
$pi = 3.14159265359;
$large = 1234567.89;

// 基础浮点数
printf("价格：%f\n", $price);  // 价格：199.900000

// 保留小数位数（常用于金额）
printf("价格：%.2f\n", $price);  // 价格：199.90
printf("圆周率：%.4f\n", $pi);  // 圆周率：3.1416

// 指定宽度和精度
printf("价格：%10.2f\n", $price);  // 价格：    199.90
printf("价格：%-10.2f\n", $price);  // 价格：199.90    

// 科学计数法
printf("大数：%e\n", $large);  // 大数：1.234568e+6
printf("大数：%E\n", $large);  // 大数：1.234568E+6

// 自动格式（去除不必要的零）
printf("价格：%g\n", $price);  // 价格：199.9
printf("整数：%g\n", 100.0);  // 整数：100
```

## 参数位置指定

使用 `参数位置$` 可以重复使用同一个参数或改变参数顺序：

| 语法 | 说明 | 示例 |
| :--- | :--- | :--- |
| `%1$s` | 使用第 1 个参数 | `printf("%1$s %1$s", "Hi");` → `Hi Hi` |
| `%2$s` | 使用第 2 个参数 | `printf("%2$s %1$s", "A", "B");` → `B A` |
| `%1$d %2$s` | 混合使用 | `printf("%1$d %2$s", 42, "items");` → `42 items` |

示例代码：

```php
<?php
declare(strict_types=1);

$firstName = "张";
$lastName = "三";
$age = 25;

// 重复使用参数
printf("姓名：%1$s%2$s，年龄：%3$d 岁\n", $firstName, $lastName, $age);
// 姓名：张三，年龄：25 岁

// 改变参数顺序
printf("英文格式：%2$s, %1$s\n", $firstName, $lastName);
// 英文格式：三, 张

// 重复使用同一参数
printf("重复：%1$s %1$s %1$s\n", "Hello");
// 重复：Hello Hello Hello

// 复杂组合
$price = 99.99;
$currency = "CNY";
printf("价格：%2$s %.2f（%1$s）\n", $currency, $price);
// 价格：99.99 CNY（CNY）
```

## 实际应用场景

### 场景 1：格式化订单信息

```php
<?php
declare(strict_types=1);

function formatOrderInfo(int $orderId, float $amount, string $status): string
{
    return sprintf(
        "订单号：%05d | 金额：%.2f CNY | 状态：%-10s",
        $orderId,
        $amount,
        $status
    );
}

echo formatOrderInfo(7, 199.9, "已支付");
// 订单号：00007 | 金额：199.90 CNY | 状态：已支付      
```

### 场景 2：生成日志格式

```php
<?php
declare(strict_types=1);

function formatLog(string $level, string $message, array $context = []): string
{
    $timestamp = date('Y-m-d H:i:s');
    $contextStr = json_encode($context, JSON_UNESCAPED_UNICODE);
    
    return sprintf(
        "[%s] %-8s | %s | %s\n",
        $timestamp,
        $level,
        $message,
        $contextStr
    );
}

echo formatLog("INFO", "用户登录", ["user_id" => 123]);
// [2025-01-15 14:30:00] INFO     | 用户登录 | {"user_id":123}
```

### 场景 3：格式化表格输出

```php
<?php
declare(strict_types=1);

$products = [
    ["iPhone 15", 5999.00, 10],
    ["MacBook Pro", 12999.00, 5],
    ["AirPods", 999.00, 20],
];

echo sprintf("%-20s | %10s | %5s\n", "商品名称", "价格", "库存");
echo str_repeat("-", 40) . "\n";

foreach ($products as [$name, $price, $stock]) {
    printf(
        "%-20s | %10.2f | %5d\n",
        $name,
        $price,
        $stock
    );
}

// 输出：
// 商品名称             |       价格 |  库存
// ----------------------------------------
// iPhone 15            |    5999.00 |    10
// MacBook Pro          |   12999.00 |     5
// AirPods              |     999.00 |    20
```

### 场景 4：生成文件名

```php
<?php
declare(strict_types=1);

function generateFileName(int $userId, int $fileIndex): string
{
    return sprintf(
        "user_%05d_file_%03d.jpg",
        $userId,
        $fileIndex
    );
}

echo generateFileName(42, 7);  // user_00042_file_007.jpg
```

## 常见错误与注意事项

1. **参数数量不匹配**：占位符数量必须与参数数量一致（除非使用位置指定）

```php
// 错误示例
printf("%s %s", "Hello");  // Warning: Too few arguments

// 正确示例
printf("%s %s", "Hello", "World");  // Hello World
printf("%1$s %1$s", "Hello");  // Hello Hello（重复使用）
```

2. **类型不匹配**：占位符类型应与参数类型匹配

```php
// 可能产生意外结果
printf("%d", "abc");  // 0（字符串转整数失败）
printf("%s", 123);  // 123（数字转字符串）

// 建议显式转换
printf("%d", (int)"123");  // 123
printf("%s", (string)123);  // 123
```

3. **精度溢出**：浮点数精度可能丢失

```php
$value = 0.1 + 0.2;
printf("%.20f\n", $value);  // 0.30000000000000004441（浮点数精度问题）
```

## 完整示例：格式化用户信息

```php
<?php
declare(strict_types=1);

class UserFormatter
{
    public static function formatProfile(
        int $id,
        string $name,
        float $balance,
        string $email
    ): string {
        return sprintf(
            "用户ID：%05d\n" .
            "姓名：%-20s\n" .
            "余额：%+.2f CNY\n" .
            "邮箱：%-30s\n",
            $id,
            $name,
            $balance,
            $email
        );
    }
    
    public static function formatTableRow(
        int $id,
        string $name,
        float $balance
    ): string {
        return sprintf(
            "| %05d | %-20s | %10.2f |\n",
            $id,
            $name,
            $balance
        );
    }
}

// 使用示例
echo UserFormatter::formatProfile(42, "张三", 199.9, "zhang@example.com");
// 用户ID：00042
// 姓名：张三                  
// 余额：+199.90 CNY
// 邮箱：zhang@example.com            

echo UserFormatter::formatTableRow(1, "李四", 999.99);
// | 00001 | 李四                  |     999.99 |
```

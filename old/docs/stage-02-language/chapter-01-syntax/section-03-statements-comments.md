# 2.1.3 语句与注释

## 语句和分号

### 基本规则

- **每条语句必须以分号 `;` 结束**
- **控制结构（`if`、`for`、`while` 等）后跟代码块 `{}`，不需要分号**

### 正确示例

```php
<?php
declare(strict_types=1);

// 变量赋值（需要分号）
$name = "张三";
$age = 25;

// 函数调用（需要分号）
echo "Hello\n";
print("World\n");

// 控制结构（不需要分号）
if ($age >= 18) {
    echo "成年人\n";
}

// for 循环（不需要分号）
for ($i = 0; $i < 10; $i++) {
    echo $i, "\n";
}
```

### 常见错误

```php
<?php
// 错误：缺少分号
$name = "张三"  // Parse error

// 错误：控制结构后加分号
if ($age >= 18); {  // 语法正确但逻辑错误
    echo "成年人\n";
}

// 正确
if ($age >= 18) {
    echo "成年人\n";
}
```

## 注释类型

### 单行注释

**语法**：`//` 或 `#`

```php
<?php
// 这是单行注释
$name = "张三";  // 行尾注释

# 这也是单行注释（不常用）
$age = 25;  # 行尾注释
```

**使用场景**：
- 解释代码意图
- 临时禁用一行代码
- 行尾说明

### 多行注释

**语法**：`/* ... */`

```php
<?php
/*
 * 这是多行注释
 * 可以写多行内容
 * 用于详细说明
 */

/* 也可以写成单行 */
```

**使用场景**：
- 临时注释掉一段代码
- 详细的功能说明
- 版权信息

**注意事项**：
- 不能嵌套（`/* /* 嵌套 */ */` 会报错）
- 注释中的代码不会执行

### PHPDoc 注释

**语法**：`/** ... */`

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 *
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 * @throws InvalidArgumentException 当参数为负数时抛出
 */
function add(int $a, int $b): int
{
    if ($a < 0 || $b < 0) {
        throw new InvalidArgumentException('参数不能为负数');
    }
    return $a + $b;
}
```

**常用标签**：

| 标签 | 说明 | 示例 |
| :--- | :--- | :--- |
| `@param` | 参数说明 | `@param string $name 用户名` |
| `@return` | 返回值说明 | `@return int 用户ID` |
| `@throws` | 可能抛出的异常 | `@throws InvalidArgumentException` |
| `@var` | 变量类型 | `@var string $name` |
| `@deprecated` | 已废弃 | `@deprecated 使用新方法代替` |
| `@since` | 版本信息 | `@since 1.0.0` |

**使用场景**：
- 为类、函数、属性提供文档
- IDE 自动补全和类型提示
- 静态分析工具（PHPStan、Psalm）使用
- 自动生成 API 文档

## 代码风格（Whitespace）

### PSR-12 标准

- **缩进**：使用 4 个空格（不使用 Tab）
- **花括号**：独占一行
- **行尾**：删除行尾空格
- **文件末尾**：以空行结束

### 示例对比

**不符合 PSR-12**：

```php
<?php
class User{
    public function getName(){
        return $this->name;
    }
}
```

**符合 PSR-12**：

```php
<?php
declare(strict_types=1);

class User
{
    public function getName(): string
    {
        return $this->name;
    }
}
```

### EditorConfig 配置

创建 `.editorconfig` 文件：

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.php]
indent_style = space
indent_size = 4
```

这样 IDE 会自动应用这些规则。

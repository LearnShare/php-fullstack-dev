# 2.1.2 文件结构与编码

## PHP 起始标签

PHP 代码必须包含在起始标签中：

| 标签类型 | 语法 | 说明 | 推荐度 |
| :------- | :--- | :--- | :------ |
| 标准标签 | `<?php` | 推荐使用，所有环境支持 | 推荐 |
| 短标签 | `<?` | 需要 `short_open_tag=On`，不推荐 | 不推荐 |
| 短输出标签 | `<?=` | 等同于 `<?php echo`，PHP 5.4+ 始终可用 | 可用 |
| ASP 风格 | `<%` | 已废弃，不应使用 | 废弃 |

**最佳实践**：永远使用 `<?php`，确保代码在所有环境中都能正常运行。

```php
<?php
// 正确：标准标签
echo "Hello";

<? 
// 不推荐：短标签（可能在某些服务器上不可用）
echo "Hello";

<?=
// 可用：短输出标签（PHP 5.4+）
"Hello"
?>
```

## 字符编码

### UTF-8 无 BOM 编码

- **为什么使用 UTF-8**：支持所有字符，是 Web 标准
- **为什么无 BOM**：BOM（Byte Order Mark）会导致输出问题，特别是在设置 HTTP 头之前

### 检查文件编码

**VS Code**：
1. 查看右下角编码指示器
2. 点击编码指示器 → "通过编码重新打开" → 选择 "UTF-8"
3. 保存时确保选择 "UTF-8" 或 "UTF-8 without BOM"

**命令行检查**：

```bash
# Linux/macOS
file -I hello.php
# 输出：hello.php: text/plain; charset=utf-8

# 检查是否有 BOM
head -c 3 hello.php | od -An -tx1
# 如果输出包含 ef bb bf，说明有 BOM
```

**Windows PowerShell**：

```powershell
# 检查文件编码
Get-Content hello.php -Encoding UTF8 | Out-File -Encoding UTF8 hello.php
```

### 常见编码问题

1. **BOM 导致的问题**：

```php
<?php
// 如果文件开头有 BOM，以下代码会失败
header('Content-Type: text/html');
// 错误：Cannot modify header information - headers already sent
```

**解决方案**：使用支持 UTF-8 无 BOM 的编辑器，保存时选择正确的编码。

2. **编码不一致**：

```php
<?php
// 文件是 GBK 编码，但浏览器期望 UTF-8
echo "中文";  // 显示乱码
```

**解决方案**：统一使用 UTF-8 编码，并在 HTML 中声明：

```php
<?php
header('Content-Type: text/html; charset=UTF-8');
echo "中文";  // 正确显示
```

## 严格类型声明

### 语法

```php
declare(strict_types=1);
```

### 位置要求

- 必须放在文件的第一行（`<?php` 之后）
- 每个文件只能声明一次
- 只影响当前文件，不影响其他文件

### 作用

启用严格类型后，函数参数和返回值必须完全匹配类型，否则会抛出 `TypeError`。

**示例对比**：

```php
<?php
// 未启用严格类型
function add(int $a, int $b): int
{
    return $a + $b;
}

echo add("10", "20");  // 输出：30（自动转换）

<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

echo add("10", "20");  // TypeError: Argument must be of type int
```

### 最佳实践

- 所有新文件都应该启用严格类型
- 在文件模板中包含 `declare(strict_types=1);`
- 使用 IDE 代码片段自动插入

## 命名空间

命名空间用于组织代码，避免类名冲突。详细内容在阶段三讲解，此处先了解基本语法：

```php
<?php
declare(strict_types=1);

namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

**命名规则**：
- 只能包含字母、数字、下划线
- 使用反斜杠 `\` 分隔层级
- 通常使用 PSR-4 标准：`Vendor\Package\SubPackage`

## 关键语法速查

| 语法       | 形态                           | 参数说明                               | 示例                                 |
| :--------- | :----------------------------- | :------------------------------------- | :----------------------------------- |
| 严格类型   | `declare(strict_types=1);`     | 只能出现在文件顶部；`1` 为开启         | `declare(strict_types=1);`           |
| 命名空间   | `namespace Vendor\Package;`    | 仅允许字母/数字/下划线，使用 `\` 分隔层级 | `namespace App\Http\Controllers;`    |
| 引入命名空间 | `use Vendor\Package\ClassName;` | 可使用 `as` 起别名                      | `use App\Services\Mailer as AppMailer;` |
| CLI 语法检查 | `php -l file.php`             | `-l` 表示 lint，仅检测语法不执行代码   | `php -l app/Hello.php`               |

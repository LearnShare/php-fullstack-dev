# 2.1 PHP 基本语法结构

## 目标

- 理解 PHP 文件的起始标签、编码要求与执行方式，新手也能独立创建首个 PHP 程序。
- 熟悉语句、分号、缩进、注释的最佳实践，养成良好代码风格。
- 为后续模块化、自动加载奠定一致的代码结构。

## 文件结构与编码

- **起始标签**：永远使用标准 `<?php`，避免 `<?`（短标签）或 `<?=`（仅在输出时使用）。
- **字符编码**：统一 UTF-8 无 BOM。可以通过 VS Code 右下角编码指示器或 `file -I` 命令检查。
- **严格类型**：在文件第二行声明 `declare(strict_types=1);`，可防止隐式类型转换带来的隐患。
- **命名空间**：从阶段三开始会大量使用，此处提前展示：`namespace App\Examples;`。

### 关键语法速查

| 语法       | 形态                           | 参数说明                               | 示例                                 |
| :--------- | :----------------------------- | :------------------------------------- | :----------------------------------- |
| 严格类型   | `declare(strict_types=1);`     | 只能出现在文件顶部；`1` 为开启         | `declare(strict_types=1);`           |
| 命名空间   | `namespace Vendor\Package;`    | 仅允许字母/数字/下划线，使用 `\` 分隔层级 | `namespace App\Http\Controllers;`    |
| 引入命名空间 | `use Vendor\Package\ClassName;` | 可使用 `as` 起别名                      | `use App\Services\Mailer as AppMailer;` |
| CLI 语法检查 | `php -l file.php`             | `-l` 表示 lint，仅检测语法不执行代码   | `php -l app/Hello.php`               |

## 语句与注释

- **分号**：每条语句必须以 `;` 结束，控制结构（`if`、`for`）后跟代码块 `{}`。
- **注释类型**：
  - `//` 单行注释：描述简短说明。
  - `/* ... */` 多行注释：临时注释掉一段代码时使用。
  - `/** ... */` PHPDoc：为类、函数、属性提供结构化注释，供 IDE/静态分析使用。
- **Whitespace**：PSR-12 要求 4 空格缩进，花括号独占一行。使用 EditorConfig 自动保证一致性。

## 实践建议

1. 在 IDE 中启用“显示不可见字符”“保存时格式化”“删除行尾空格”。
2. 使用模板：
   ```php
   <?php
   declare(strict_types=1);

   namespace Playground;

   // TODO: 描述该文件用途
   ```
3. 在 CI 中执行 `php -l` 或 `composer run lint`，及时发现语法错误。
4. 将常用片段（命名空间、strict types）设置为 IDE 代码片段（Snippet）。

## 示例

```php
<?php
declare(strict_types=1);

namespace App\Http;

/**
 * 基本示例
 */
final class HelloWorld
{
    public function greet(string $name): string
    {
        return "Hello, {$name}";
    }
}
```

保持一致的语法结构有助于团队协作、代码评审与静态分析，减少低级错误。

### 注释示例

```php
<?php
declare(strict_types=1);

namespace Playground;

/**
 * 计算折扣价格
 *
 * @param float $price 原价
 * @param float $rate  折扣（0~1）
 */
function applyDiscount(float $price, float $rate): float
{
    // 验证输入
    if ($rate < 0 || $rate > 1) {
        throw new InvalidArgumentException('Rate must be between 0 and 1');
    }

    return $price * (1 - $rate);
}
```

## 练习

1. 创建 `app/Hello.php`，包含严格类型、命名空间、类与方法，并输出问候语。
2. 修改编码为 GBK 后再改回 UTF-8，观察 `git status` 中的差异，体会编码的重要性。
3. 使用 `php -l` 校验多个文件，练习定位语法错误信息。

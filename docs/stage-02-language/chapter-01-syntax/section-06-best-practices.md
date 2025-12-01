# 2.1.6 实践建议与示例

## 实践建议

### IDE 配置

1. **启用显示不可见字符**：帮助发现空格、Tab、BOM 等问题
2. **保存时自动格式化**：确保代码风格一致
3. **删除行尾空格**：符合 PSR-12 标准
4. **显示行号**：方便定位错误

### 文件模板

创建新 PHP 文件时使用以下模板：

```php
<?php
declare(strict_types=1);

namespace App\Examples;

/**
 * 文件用途说明
 *
 * @author Your Name
 * @since 1.0.0
 */
```

### 代码片段（Snippet）

在 VS Code 中创建代码片段：

1. 打开：`File → Preferences → Configure User Snippets → php.json`
2. 添加：

```json
{
    "PHP File Template": {
        "prefix": "phpfile",
        "body": [
            "<?php",
            "declare(strict_types=1);",
            "",
            "namespace ${1:App\\Examples};",
            "",
            "/**",
            " * ${2:File description}",
            " */"
        ],
        "description": "PHP file template with strict types"
    }
}
```

### 语法检查

#### 命令行检查

```bash
# 检查单个文件
php -l file.php

# 检查目录下所有 PHP 文件
find . -name "*.php" -exec php -l {} \;

# 或使用 xargs
find . -name "*.php" | xargs -I {} php -l {}
```

#### Composer 脚本

在 `composer.json` 中添加：

```json
{
    "scripts": {
        "lint": "find . -name '*.php' -exec php -l {} \\;"
    }
}
```

执行：

```bash
composer run lint
```

#### CI/CD 集成

在 GitHub Actions 中：

```yaml
- name: Check PHP syntax
  run: |
    find . -name "*.php" -exec php -l {} \;
```

### 编码检查清单

创建新文件时检查：

- [ ] 文件编码为 UTF-8 无 BOM
- [ ] 使用 `<?php` 标准标签
- [ ] 包含 `declare(strict_types=1);`
- [ ] 符合 PSR-12 代码风格
- [ ] 通过 `php -l` 语法检查
- [ ] 包含必要的 PHPDoc 注释

## 完整示例

### 示例 1：基础 Hello World

```php
<?php
echo "Hello, World!\n";
```

**执行**：

```bash
php hello.php
```

**输出**：

```
Hello, World!
```

### 示例 2：带严格类型的函数

```php
<?php
declare(strict_types=1);

/**
 * 问候函数
 *
 * @param string $name 姓名
 * @return string 问候语
 */
function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

// 使用函数
echo greet("World");
echo greet("PHP");
```

**执行**：

```bash
php greet.php
```

**输出**：

```
Hello, World!
Hello, PHP!
```

### 示例 3：类和方法

```php
<?php
declare(strict_types=1);

namespace App\Http;

/**
 * Hello World 类示例
 */
final class HelloWorld
{
    /**
     * 问候方法
     *
     * @param string $name 姓名
     * @return string 问候语
     */
    public function greet(string $name): string
    {
        return "Hello, {$name}!";
    }
}

// 使用类
$hello = new HelloWorld();
echo $hello->greet("World"), "\n";
```

**执行**：

```bash
php HelloWorld.php
```

**输出**：

```
Hello, World!
```

### 示例 4：带注释的完整示例

```php
<?php
declare(strict_types=1);

namespace App\Examples;

/**
 * 价格计算工具类
 *
 * @author Your Name
 * @since 1.0.0
 */
class PriceCalculator
{
    /**
     * 计算折扣价格
     *
     * @param float $price 原价
     * @param float $rate  折扣率（0~1）
     * @return float 折扣后价格
     * @throws \InvalidArgumentException 当折扣率不在 0~1 之间时抛出
     */
    public function applyDiscount(float $price, float $rate): float
    {
        // 验证输入参数
        if ($rate < 0 || $rate > 1) {
            throw new \InvalidArgumentException('折扣率必须在 0 到 1 之间');
        }

        // 计算折扣后价格
        return $price * (1 - $rate);
    }

    /**
     * 计算含税价格
     *
     * @param float $price 原价
     * @param float $taxRate 税率（如 0.1 表示 10%）
     * @return float 含税价格
     */
    public function addTax(float $price, float $taxRate): float
    {
        return $price * (1 + $taxRate);
    }
}

// 使用示例
$calculator = new PriceCalculator();

// 计算折扣
$originalPrice = 100.0;
$discountRate = 0.2;  // 20% 折扣
$discountedPrice = $calculator->applyDiscount($originalPrice, $discountRate);
echo "原价：{$originalPrice}，折扣后：{$discountedPrice}\n";

// 计算含税价格
$taxRate = 0.1;  // 10% 税
$finalPrice = $calculator->addTax($discountedPrice, $taxRate);
echo "最终价格：{$finalPrice}\n";
```

**执行**：

```bash
php PriceCalculator.php
```

**输出**：

```
原价：100，折扣后：80
最终价格：88
```

## 自检清单

完成本章学习后，检查以下项目：

- [ ] 能够创建并执行一个简单的 PHP 文件
- [ ] 理解 `<?php` 标签的作用和位置
- [ ] 知道如何检查文件编码（UTF-8 无 BOM）
- [ ] 理解 `declare(strict_types=1);` 的作用
- [ ] 掌握语句分号的使用规则
- [ ] 能够使用三种注释类型（单行、多行、PHPDoc）
- [ ] 了解 PSR-12 代码风格要求
- [ ] 能够使用 `php -l` 检查语法错误
- [ ] 理解 CLI 和 Web 执行环境的差异
- [ ] 能够识别并解决常见的语法错误

## 练习

### 练习 1：创建第一个 PHP 程序

1. 创建文件 `app/Hello.php`
2. 包含以下内容：
   - 严格类型声明
   - 命名空间 `App\Examples`
   - 一个 `Hello` 类
   - 一个 `greet()` 方法，接受姓名参数并返回问候语
3. 在文件末尾实例化类并调用方法
4. 使用 `php -l` 检查语法
5. 执行文件并查看输出

**参考代码**：

```php
<?php
declare(strict_types=1);

namespace App\Examples;

class Hello
{
    public function greet(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$hello = new Hello();
echo $hello->greet("World"), "\n";
```

### 练习 2：编码实验

1. 创建一个包含中文的 PHP 文件
2. 将文件编码改为 GBK，保存
3. 执行文件，观察输出
4. 将文件编码改回 UTF-8，保存
5. 使用 `git status` 和 `git diff` 观察差异
6. 理解编码对代码的影响

### 练习 3：语法检查实践

1. 创建多个包含语法错误的 PHP 文件：
   - 缺少分号
   - 括号不匹配
   - 引号不匹配
   - 缺少花括号
2. 使用 `php -l` 检查每个文件
3. 根据错误信息修复问题
4. 再次检查，确保没有错误

### 练习 4：注释练习

1. 创建一个计算器类 `Calculator`
2. 为每个方法添加 PHPDoc 注释，包含：
   - 方法说明
   - `@param` 参数说明
   - `@return` 返回值说明
   - `@throws` 异常说明（如果有）
3. 在方法内部使用单行注释说明关键逻辑
4. 使用多行注释说明类的整体用途

### 练习 5：代码风格

1. 创建 `.editorconfig` 文件
2. 配置 PHP 文件的缩进、编码等规则
3. 创建一个不符合 PSR-12 的 PHP 文件
4. 使用 IDE 自动格式化功能
5. 对比格式化前后的差异

保持一致的语法结构有助于团队协作、代码评审与静态分析，减少低级错误。完成这些练习后，你将具备创建规范 PHP 文件的基础能力。

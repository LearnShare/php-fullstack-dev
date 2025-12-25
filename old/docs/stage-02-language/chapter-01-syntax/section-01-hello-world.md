# 2.1.1 第一个 PHP 程序：Hello World

## 创建文件

1. 创建文件 `hello.php`（文件名可以是任意，但建议使用 `.php` 扩展名）

2. 写入以下内容：

```php
<?php
echo "Hello, World!\n";
```

3. 保存文件（确保编码为 UTF-8）

## 执行方式

### CLI 方式（命令行）

```bash
php hello.php
```

输出：
```
Hello, World!
```

### Web 方式

1. 将文件放在 Web 服务器目录（如 `public/` 或 `htdocs/`）
2. 通过浏览器访问：`http://localhost/hello.php`
3. 浏览器显示：`Hello, World!`

## 完整示例：带严格类型的 Hello World

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

echo greet("World");
```

执行：
```bash
php hello.php
```

输出：
```
Hello, World!
```

## 下一步

完成第一个 PHP 程序后，继续学习：
- [文件结构与编码](section-02-file-structure.md)：了解 PHP 标签、编码、严格类型等基础概念
- [语句与注释](section-03-statements-comments.md)：掌握语句规则和注释用法

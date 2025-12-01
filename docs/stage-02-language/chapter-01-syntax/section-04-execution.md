# 2.1.4 文件执行方式

## CLI 模式（命令行）

### 直接执行

```bash
php hello.php
```

### 语法检查

```bash
php -l hello.php
```

**输出示例**：

```bash
# 语法正确
$ php -l hello.php
No syntax errors detected in hello.php

# 语法错误
$ php -l hello.php
Parse error: syntax error, unexpected '}' in hello.php on line 5
Errors parsing hello.php
```

### 交互式模式

```bash
php -a
```

进入交互式模式，可以直接输入 PHP 代码：

```php
php > echo "Hello\n";
Hello
php > $name = "World";
php > echo $name, "\n";
World
php > exit
```

## Web 模式

### 内置服务器（开发用）

```bash
php -S localhost:8000 -t public
```

访问：`http://localhost:8000/hello.php`

### 生产环境

使用 Nginx + PHP-FPM 或 Apache + mod_php（详见阶段一）

## 执行环境差异

| 特性 | CLI | Web |
| :--- | :--- | :--- |
| 输出目标 | 标准输出（终端） | HTTP 响应体 |
| 换行符 | `\n` | `<br>` 或保留格式 |
| 错误显示 | 直接输出到终端 | 需要配置 `display_errors` |
| 超时限制 | 无（或很长） | 通常 30 秒 |
| 内存限制 | 通常较大 | 通常 128M-256M |

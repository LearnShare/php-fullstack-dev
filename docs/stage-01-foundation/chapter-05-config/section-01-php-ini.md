# 1.5.1 php.ini 配置

## 概述

`php.ini` 是 PHP 的主配置文件，控制 PHP 的行为和功能。理解如何配置 `php.ini` 对于开发和生产环境都至关重要。

## 定位 php.ini

### 查找配置文件

**命令行方式**：

```bash
php --ini
```

**输出示例**：

```
Configuration File (php.ini) Path: /etc/php/8.3/cli
Loaded Configuration File:         /etc/php/8.3/cli/php.ini
```

**代码方式**：

```php
<?php
phpinfo();
```

### 不同运行模式的配置

- **CLI 模式**：使用 `php --ini` 查看的配置文件
- **PHP-FPM 模式**：通常有独立的配置文件（如 `/etc/php/8.3/fpm/php.ini`）
- **Apache 模块**：使用 Apache 的配置文件

## 常用配置项

### 错误报告

```ini
; 开发环境
display_errors = On
display_startup_errors = On
error_reporting = E_ALL
log_errors = On
error_log = /var/log/php_errors.log

; 生产环境
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php_errors.log
```

### 内存和性能

```ini
; 内存限制
memory_limit = 256M

; 最大执行时间（秒）
max_execution_time = 30

; 最大输入时间
max_input_time = 60

; 最大输入变量数
max_input_vars = 1000
```

### 文件上传

```ini
; 允许文件上传
file_uploads = On

; 上传文件最大大小
upload_max_filesize = 10M

; 最大 POST 数据大小
post_max_size = 10M

; 最大上传文件数
max_file_uploads = 20
```

### 时区

```ini
date.timezone = Asia/Shanghai
```

### 扩展

```ini
; 启用扩展
extension=pdo_mysql
extension=mysqli
extension=curl
extension=mbstring
```

## 运行时配置

### ini_set()

**语法**：`ini_set(string $option, string|int|float|bool|null $value): string|false`

**参数**：
- `$option`：配置项名称
- `$value`：配置值

**返回值**：成功返回旧值，失败返回 `false`

**示例**：

```php
<?php
// 设置内存限制
ini_set('memory_limit', '512M');

// 设置时区
ini_set('date.timezone', 'Asia/Shanghai');

// 设置错误报告
ini_set('display_errors', '1');
ini_set('error_reporting', E_ALL);
```

### ini_get()

**语法**：`ini_get(string $option): string|false`

**参数**：
- `$option`：配置项名称

**返回值**：配置值，失败返回 `false`

**示例**：

```php
<?php
// 获取内存限制
echo ini_get('memory_limit'); // 输出: 256M

// 获取时区
echo ini_get('date.timezone'); // 输出: Asia/Shanghai

// 获取错误报告级别
echo ini_get('error_reporting'); // 输出: 32767
```

## 完整示例

### 示例 1：开发环境配置

```ini
; php.ini (开发环境)

; 错误报告
display_errors = On
display_startup_errors = On
error_reporting = E_ALL
log_errors = On

; 性能
memory_limit = 512M
max_execution_time = 60

; 时区
date.timezone = Asia/Shanghai

; 扩展
extension=pdo_mysql
extension=mysqli
extension=curl
```

### 示例 2：生产环境配置

```ini
; php.ini (生产环境)

; 错误报告
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php_errors.log

; 性能
memory_limit = 256M
max_execution_time = 300

; 时区
date.timezone = Asia/Shanghai
```

### 示例 3：运行时配置

```php
<?php
// 开发环境配置
if (getenv('APP_ENV') === 'development') {
    ini_set('display_errors', '1');
    ini_set('error_reporting', E_ALL);
    ini_set('memory_limit', '512M');
} else {
    ini_set('display_errors', '0');
    ini_set('error_reporting', E_ALL & ~E_DEPRECATED);
    ini_set('memory_limit', '256M');
}

// 获取配置
echo "Memory limit: " . ini_get('memory_limit') . "\n";
echo "Timezone: " . ini_get('date.timezone') . "\n";
```

## 注意事项

- **配置生效**：修改 php.ini 后需要重启 PHP 服务
- **运行时修改**：部分配置可以在运行时修改，但不是所有配置都支持
- **环境区分**：开发和生产环境应使用不同的配置
- **配置验证**：使用 `php -i` 或 `phpinfo()` 验证配置

## 常见问题

### 问题 1：配置不生效

**原因**：配置文件路径错误、未重启服务

**解决方法**：

```bash
# 检查配置文件路径
php --ini

# 重启 PHP 服务
sudo systemctl restart php-fpm
```

### 问题 2：内存不足

**原因**：`memory_limit` 设置过小

**解决方法**：

```ini
memory_limit = 512M
```

### 问题 3：文件上传失败

**原因**：`upload_max_filesize` 或 `post_max_size` 设置过小

**解决方法**：

```ini
upload_max_filesize = 10M
post_max_size = 10M
```

## 最佳实践

- 开发和生产环境使用不同的配置
- 使用 `ini_set()` 在代码中动态配置
- 定期检查配置，确保符合需求
- 记录配置变更，便于问题排查

## 相关章节

- **阶段四：系统编程**：深入学习错误处理和配置管理

## 练习任务

1. 使用 `php --ini` 查找 php.ini 配置文件位置。
2. 修改错误报告配置，启用错误显示。
3. 使用 `ini_set()` 在代码中动态设置内存限制。
4. 使用 `ini_get()` 获取当前配置值。
5. 创建开发和生产环境的配置文件，并验证配置差异。

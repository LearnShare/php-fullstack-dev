# 1.5.1 php.ini 配置

## 概述

`php.ini` 是 PHP 的主配置文件，控制 PHP 的行为和功能。理解如何配置 `php.ini` 对于开发和生产环境都至关重要。本节介绍如何定位、配置和验证 php.ini 文件。

**章节类型**：配置性章节

**主要内容**：
- 定位 php.ini 文件
- 不同运行模式的配置文件
- 常用配置项（错误报告、内存和性能、文件上传、时区、扩展）
- 开发/生产环境配置差异
- 运行时配置（ini_set、ini_get）
- 配置验证方法

## 特性

- **集中配置**：所有 PHP 配置集中在一个文件中
- **环境区分**：可以为不同环境配置不同的值
- **运行时修改**：部分配置可以在运行时修改
- **扩展配置**：支持为扩展单独配置

## 配置步骤/语法

### 定位 php.ini

**查找配置文件**：
- `php --ini`：查看加载的配置文件
- `php -i`：查看所有配置信息
- `phpinfo()`：在代码中查看配置

**不同运行模式的配置**：
- CLI 模式：使用 `php --ini` 查看的配置文件
- PHP-FPM 模式：通常有独立的配置文件
- Apache 模块：使用 Apache 的配置文件

### 常用配置项

**错误报告**：
- `display_errors`：是否在页面显示错误（开发 On，生产 Off）
- `display_startup_errors`：是否显示启动错误
- `error_reporting`：错误报告级别
- `log_errors`：是否记录错误日志
- `error_log`：错误日志路径

**内存和性能**：
- `memory_limit`：脚本内存限制（推荐 256M 或 512M）
- `max_execution_time`：最大执行时间（开发 30，生产 300）
- `max_input_time`：最大输入时间
- `post_max_size`：POST 数据最大大小
- `upload_max_filesize`：上传文件最大大小

**文件上传**：
- `file_uploads`：是否允许文件上传
- `upload_max_filesize`：上传文件最大大小
- `max_file_uploads`：最大上传文件数

**时区配置**：
- `date.timezone`：默认时区

**扩展配置**：
- 扩展的启用和配置
- 扩展特定的配置项

### 开发/生产环境配置

**开发环境**：
- `display_errors = On`
- `error_reporting = E_ALL`
- `display_startup_errors = On`

**生产环境**：
- `display_errors = Off`
- `error_reporting = E_ALL & ~E_DEPRECATED`
- `display_startup_errors = Off`
- `log_errors = On`

### 运行时配置

**ini_set()**：
- 语法和用法
- 可修改的配置项
- 作用域限制

**ini_get()**：
- 语法和用法
- 获取配置值

**示例**：
- 运行时修改内存限制
- 运行时修改错误报告级别

### 配置验证

**验证方法**：
- `php -i`：查看所有配置
- `php -i | grep config_name`：查看特定配置
- `phpinfo()`：在代码中查看配置

## 注意事项

- **备份配置**：修改前备份原始配置文件
- **环境区分**：开发和生产环境使用不同的配置
- **性能影响**：某些配置可能影响性能
- **安全考虑**：生产环境关闭错误显示
- **扩展依赖**：某些配置依赖扩展是否启用

## 常见问题

### 问题 1：找不到 php.ini 文件
- **原因**：配置文件位置不明确
- **解决**：使用 `php --ini` 查找配置文件位置

### 问题 2：配置修改不生效
- **原因**：修改了错误的配置文件、需要重启服务
- **解决**：确认配置文件位置、重启 PHP-FPM 或 Web 服务器

### 问题 3：内存限制不足
- **原因**：memory_limit 设置过小
- **解决**：增加 memory_limit 值，注意系统内存限制

### 问题 4：文件上传失败
- **原因**：upload_max_filesize 或 post_max_size 设置过小
- **解决**：增加相关配置值，注意 post_max_size 应大于 upload_max_filesize

## 输出结果说明

**php --ini 输出示例**：
```
Configuration File (php.ini) Path: /usr/local/etc/php/8.2
Loaded Configuration File:         /usr/local/etc/php/8.2/php.ini
Scan for additional .ini files in: /usr/local/etc/php/8.2/conf.d
```

## 最佳实践

- 为不同环境维护不同的配置文件
- 使用版本控制管理配置文件
- 定期审查配置，移除不必要的配置
- 记录配置变更，便于问题排查
- 使用 ini_set() 进行临时配置修改
- 生产环境关闭错误显示，启用错误日志

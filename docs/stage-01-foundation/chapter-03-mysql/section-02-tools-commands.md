# 1.3.2 客户端工具与命令

## 概述

选择合适的客户端工具和掌握常用命令是高效管理 MySQL 数据库的关键。本节介绍命令行工具和图形化工具的使用，以及常用的 MySQL 命令和 SQL 命令。

**章节类型**：工具性章节

**主要内容**：
- 客户端工具选择（命令行、图形化）
- 命令行工具使用（mysql、mysqldump、mysql_config_editor、mysqladmin）
- 常用 SQL 命令
- 图形化工具介绍（phpMyAdmin、MySQL Workbench、DBeaver、TablePlus）
- 常见问题排查

## 特性

- **多种工具选择**：命令行和图形化工具各有优势
- **功能丰富**：提供完整的数据库管理功能
- **跨平台支持**：支持 Windows、macOS、Linux

## 基本用法/命令

### 命令行工具

**mysql**：
- 连接数据库：`mysql -u username -p`
- 执行 SQL 文件：`mysql -u username -p database < file.sql`
- 导出数据：`mysql -u username -p -e "SELECT * FROM table" database`

**mysqldump**：
- 备份数据库：`mysqldump -u username -p database > backup.sql`
- 备份所有数据库：`mysqldump -u username -p --all-databases > all_backup.sql`
- 只备份结构：`mysqldump -u username -p --no-data database > structure.sql`

**mysql_config_editor**：
- 保存登录凭据：`mysql_config_editor set --login-path=local --host=localhost --user=root --password`
- 使用保存的凭据：`mysql --login-path=local`

**mysqladmin**：
- 检查服务器状态：`mysqladmin -u root -p status`
- 创建数据库：`mysqladmin -u root -p create database_name`
- 删除数据库：`mysqladmin -u root -p drop database_name`

### 常用 SQL 命令

**数据库操作**：
- `CREATE DATABASE`：创建数据库
- `DROP DATABASE`：删除数据库
- `SHOW DATABASES`：显示所有数据库
- `USE database_name`：选择数据库

**表操作**：
- `CREATE TABLE`：创建表
- `DROP TABLE`：删除表
- `ALTER TABLE`：修改表结构
- `SHOW TABLES`：显示所有表
- `DESCRIBE table_name`：显示表结构

**数据操作**：
- `SELECT`：查询数据
- `INSERT`：插入数据
- `UPDATE`：更新数据
- `DELETE`：删除数据

**用户和权限**：
- `CREATE USER`：创建用户
- `GRANT`：授权
- `REVOKE`：撤销权限
- `SHOW GRANTS`：显示用户权限

### 图形化工具

**phpMyAdmin**：
- Web 界面，易于使用
- 适合初学者
- 安装和配置方法

**MySQL Workbench**：
- 官方工具，功能强大
- 支持数据库设计和管理
- 跨平台支持

**DBeaver**：
- 开源工具，支持多种数据库
- 功能丰富，界面友好

**TablePlus**：
- 现代化界面
- 支持多种数据库
- 付费工具，有免费版本

## 使用场景

- **命令行工具**：适合脚本自动化、服务器管理、批量操作
- **图形化工具**：适合日常开发、数据查看、表结构设计
- **mysqldump**：适合数据库备份和迁移
- **mysqladmin**：适合服务器管理和监控

## 注意事项

- **密码安全**：不要在命令行中直接输入密码，使用 `-p` 参数交互输入
- **权限管理**：确保用户有足够的权限执行操作
- **备份策略**：定期备份，测试恢复流程
- **工具选择**：根据场景选择合适的工具

## 常见问题

### 问题 1：无法连接到数据库
- **原因**：用户名密码错误、权限不足、服务未启动
- **解决**：检查凭据、权限、服务状态

### 问题 2：mysqldump 备份失败
- **原因**：权限不足、磁盘空间不足
- **解决**：检查权限、磁盘空间

### 问题 3：图形化工具连接超时
- **原因**：网络问题、防火墙阻止、MySQL 配置
- **解决**：检查网络、防火墙、MySQL 远程连接配置

## 最佳实践

- 掌握命令行工具，便于脚本自动化
- 使用图形化工具提高开发效率
- 使用 mysql_config_editor 保存常用连接
- 定期备份数据库，测试恢复流程
- 记录常用命令，建立命令库

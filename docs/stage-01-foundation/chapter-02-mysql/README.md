# 1.2 MySQL 环境搭建与工具

## 目标

- 让零基础学员在一台电脑上成功运行 MySQL 8.0+，并明白每一步在做什么。
- 选择合适的 GUI/CLI 工具，建立安全连接、创建数据库与用户。
- 了解数据目录、字符集、备份策略，具备排查常见故障的能力。

## 版本与发行版

- **MySQL Community 8.0**：默认选项，提供 InnoDB、GTID 复制。
- **Percona Server / MariaDB**：仅在明确需求（XtraBackup、兼容 MySQL 5.7）时使用。
- 建议统一 `utf8mb4` 字符集、`InnoDB` 存储引擎。

## 安装方式

### Docker 方式（推荐）

```bash
docker run -d \
  --name mysql8 \
  -e MYSQL_ROOT_PASSWORD=secret \
  -p 3306:3306 \
  -v ./mysql-data:/var/lib/mysql \
  mysql:8.0
```
- 使用 `docker exec -it mysql8 mysql -uroot -p` 进入 CLI。
- 将数据卷放在快速磁盘上，避免 Docker Desktop I/O 性能瓶颈。
- 若要修改默认字符集，可在环境变量中增加 `--character-set-server=utf8mb4`。

### 本地安装

- Windows：Herd 内置 MySQL，或使用 MySQL Installer 选择“Developer Default”，安装后在服务面板中启动 `MySQL80`。
- macOS：`brew install mysql@8.0` 并运行 `brew services start mysql`；第一次启动会提示默认 root 密码，建议立即修改。
- Linux：`apt install mysql-server`，通过 `sudo mysql_secure_installation` 完成密码与安全设置；若需自定义数据目录，执行 `mysqld --initialize --datadir=/data/mysql`.

## 客户端工具

| 工具                | 适用场景           | 特点                                   |
| :------------------ | :----------------- | :------------------------------------- |
| MySQL CLI (`mysql`) | 快速执行 SQL、编写脚本 | 配合 `history`、`tee` 保留执行记录。   |
| TablePlus / DataGrip | 可视化管理、ER 图 | 支持 SSH Tunnel、连接模板、数据比对。 |
| DBeaver             | 跨数据库教学场景   | 内置 ER 反向工程、数据导入导出向导。   |
| MySQL Workbench     | 新手向、图形化建模 | 提供查询面板、性能监控图表。           |

> 新手可先使用 GUI 观察数据库结构，再对照 CLI 输出记忆命令。

## 第一个数据库

1. 使用 CLI 连接：`mysql -uroot -p`。
2. 创建数据库并设置字符集：
   ```sql
   CREATE DATABASE demo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```
3. 创建应用账号：
   ```sql
   CREATE USER 'demo_app'@'%' IDENTIFIED BY 'P@ssw0rd!';
   GRANT ALL PRIVILEGES ON demo.* TO 'demo_app'@'%';
   FLUSH PRIVILEGES;
   ```
4. 记录以上命令，保存到 `docs/sql/init-demo.sql`，便于日后复用。

## 连接安全

- 永远禁用 `root` 远程访问，改用只读/读写账户。
- 开启 `require_secure_transport=ON` 强制 TLS。
- 通过 `mysql_config_editor` 保存凭据，避免明文密码。

## 常用命令语法

### `mysql`

- **语法**：`mysql -h host -P port -u user -p`
- **关键参数**：
  - `-e`：直接执行 SQL 语句，例如 `-e "SHOW DATABASES;"`。
  - `--default-character-set=utf8mb4`：显式设置客户端编码。
- **示例**：
  ```bash
  mysql -h127.0.0.1 -uroot -p -e "SHOW DATABASES;"
  ```

### `mysqldump`

- **语法**：`mysqldump [options] db [tables]`
- **关键参数**：
  - `--single-transaction`：避免锁表，适合 InnoDB。
  - `--routines`：包含存储过程与函数。
  - `--set-gtid-purged=OFF`：开启 GTID 时必备。
- **示例**：
  ```bash
  mysqldump --single-transaction demo > backup.sql
  ```

### `mysql_config_editor`

- **语法**：`mysql_config_editor set --login-path=name --user=user --host=host --password`
- **关键参数**：
  - `print`：查看已保存的登录信息。
  - `remove`：删除指定 login-path。
- **示例**：
  ```bash
  mysql_config_editor set \
    --login-path=local \
    --user=demo_app \
    --host=127.0.0.1 \
    --password
  ```

### `mysqladmin`

- **语法**：`mysqladmin -u user -p command`
- **常用命令**：
  - `status`：查看当前连接、运行时间、吞吐量。
  - `variables`：导出运行时变量。
  - `shutdown`：优雅关闭实例（需权限）。
- **示例**：
  ```bash
  mysqladmin -uroot -p status
  ```

> 初学者可将以上命令整理到 `docs/sql/command-reference.md`，组建“输入→结果截图→问题记录”的学习档案。

## 数据目录与备份

- 数据目录位置：
  - Docker：`./mysql-data`（由 `-v` 参数决定）。
  - Windows Installer：`C:\ProgramData\MySQL\MySQL Server 8.0\Data\`。
  - Homebrew：`/opt/homebrew/var/mysql/`。
- 逻辑备份：`mysqldump --single-transaction --routines demo > backup.sql`。
- 物理备份：使用 `Percona XtraBackup` 或 `mydumper`，适合大数据量与在线备份。
- 养成“双重备份”习惯：本地每日、云端每周，编写 `backup.sh` 自动化脚本。

## 常见问题排查

- **端口被占用**：`netstat -ano | findstr 3306`（Windows）或 `lsof -iTCP:3306`（macOS/Linux）定位进程，或将 MySQL 端口改为 3307。
- **字符集不一致**：执行
  ```sql
  SHOW VARIABLES LIKE 'character_set%';
  SHOW VARIABLES LIKE 'collation%';
  ```
  若 `client` 与 `server` 不一致，先调整 `my.cnf`，再重启服务。
- **权限错误**：`SHOW GRANTS FOR 'app'@'%';`，确认是否授予 `SELECT`/`INSERT` 等权限。
- **连接拒绝**：检查 `bind-address` 是否为 `0.0.0.0`，Docker 场景需映射 `-p 3306:3306`。
- **慢查询**：开启 `slow_query_log=ON`、`long_query_time=1`，使用 `pt-query-digest` 分析。

## 自检清单

- [ ] 能使用 Docker 和本地安装两种方式运行 MySQL。
- [ ] 能够创建数据库、用户、授权并记录在脚本中。
- [ ] 知道数据目录位置，能够执行逻辑备份与恢复。
- [ ] 面对常见报错（端口、权限、字符集）能查日志并修复。

完成本章后，你可以快速搭建稳定安全的 MySQL 8 环境，并通过脚本或 GUI 工具执行常规管理任务。

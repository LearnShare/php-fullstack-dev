# 6.6.2 备份与恢复 (Backup and Restore)

## 概述

**数据是任何应用最宝贵的资产**。无论你的代码写得多完美，服务器性能多强劲，如果数据丢失，一切都将失去意义。因此，建立一套可靠的**备份与恢复 (Backup and Restore)** 机制，是数据库管理中**最重要**的工作，没有之一。

本节将介绍为什么备份是必须的，备份的两种主要类型，并重点实践如何使用 `mysqldump` 工具来对 MySQL 数据库进行逻辑备份和恢复。

## 为什么备份是必须的？

数据丢失的风险无处不在，备份是抵御这些风险的最后一道，也是最可靠的一道防线。常见的数据丢失原因包括：
-   **硬件故障**：硬盘损坏、服务器宕机等。
-   **软件缺陷**：应用程序中的 Bug 可能会意外地删除或损坏数据。
-   **人为错误**：开发或运维人员可能会错误地执行 `DELETE` 或 `DROP TABLE` 命令。
-   **恶意攻击**：黑客攻击、勒索软件等可能会加密或删除你的数据。

没有备份，就意味着你的业务随时可能因为一次意外而彻底终结。

### 核心备份指标

在制定备份策略时，通常会考虑两个核心指标：
-   **RPO (Recovery Point Objective / 恢复点目标)**：你能容忍丢失**多长时间**的数据？例如，RPO 为 1 小时，意味着最坏情况下会丢失最近 1 小时的数据。这个指标决定了你的**备份频率**。
-   **RTO (Recovery Time Objective / 恢复时间目标)**：在发生灾难后，你需要**多长时间**内恢复服务？例如，RTO 为 30 分钟，意味着数据库必须在半小时内恢复正常。这个指标决定了你的**恢复方案和硬件配置**。

## 备份的类型

对于 MySQL 来说，备份主要分为两大类：逻辑备份和物理备份。

### 1. 逻辑备份 (Logical Backup)

-   **是什么**：将数据库中的数据导出为一系列 SQL 语句（如 `CREATE TABLE ...`, `INSERT INTO ...`）并保存在一个文本文件中。
-   **常用工具**：`mysqldump` 是 MySQL 官方提供的、最常用的逻辑备份工具。
-   **优点**：
    -   **高度灵活**：备份文件是纯文本的 SQL，可以在不同 MySQL 版本、不同操作系统甚至不同类型的数据库（需手动修改）之间迁移。
    -   **可读性好**：可以打开 `.sql` 文件直接查看内容。
    -   **粒度精细**：可以只备份单个数据库、单张表，甚至满足特定 `WHERE` 条件的数据。
-   **缺点**：
    -   **恢复速度慢**：恢复时需要重新执行所有 SQL 语句，对于大数据量的数据库来说，这个过程可能非常漫长。
    -   **备份速度相对较慢**。

### 2. 物理备份 (Physical Backup)

-   **是什么**：直接复制数据库的原始数据文件、日志文件等。
-   **常用工具**：`Percona XtraBackup` 是最流行、功能最强大的开源物理备份工具。
-   **优点**：
    -   **备份和恢复速度极快**：因为只是文件的复制，所以对于 TB 级别的大型数据库，物理备份是唯一现实的选择。
    -   **不阻塞数据库**：可以在不影响线上业务的情况下进行“热备份”。
-   **缺点**：
    -   **灵活性差**：通常只能恢复到相同或非常相似的 MySQL 版本和系统环境中。
    -   **更复杂**：配置和管理比逻辑备份复杂。

**结论**：对于中小型项目，`mysqldump` 提供的逻辑备份因其简单和灵活而成为首选。物理备份主要用于对性能和恢复时间有极高要求的大型企业级应用。

---

## 使用 `mysqldump` 进行备份与恢复

### a) 备份数据库

`mysqldump` 是一个命令行工具，其基本语法格式为：
`mysqldump -u [用户名] -p[密码] [数据库名] > [备份文件名.sql]`

**注意**：`-p` 和密码之间**没有空格**。如果为了安全，希望在执行时再输入密码，可以只写 `-p`。

**常用备份命令**：

-   **备份单个数据库**:
    ```bash
    mysqldump -u root -p my_blog > my_blog_backup.sql
    ```

-   **备份所有数据库**:
    ```bash
    mysqldump -u root -p --all-databases > all_databases.sql
    ```
    
-   **备份特定几张表**:
    ```bash
    mysqldump -u root -p my_blog users posts > users_and_posts.sql
    ```

**InnoDB 数据库的重要选项**：
-   `--single-transaction`：**强烈推荐对 InnoDB 表使用此选项**。它会在备份开始前启动一个事务，从而获取一个数据库在当前时间点的一致性快照。最重要的是，它在备份期间**不会锁定表**，允许应用正常读写，实现了“热备份”。
-   `--routines`：包含存储过程和函数。
-   `--triggers`：包含触发器。

**推荐的 InnoDB 备份命令**：
```bash
mysqldump -u root -p --single-transaction --routines --triggers my_blog > my_blog_backup_$(date +%F).sql
```
这里我们用 `$(date +%F)` 在文件名中加入了当前日期，是一个很好的实践。

### b) 恢复数据库

从 `mysqldump` 的备份文件中恢复数据，本质上就是让 `mysql` 客户端执行这个 SQL 脚本。

**恢复命令**：
`mysql -u [用户名] -p[密码] [数据库名] < [备份文件名.sql]`

**步骤**：
1.  首先，你需要创建一个空的数据库（如果它还不存在的话）。
    ```sql
    mysql -u root -p -e "CREATE DATABASE my_blog_new CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
    ```
2.  然后，将数据导入这个新数据库。
    ```bash
    mysql -u root -p my_blog_new < my_blog_backup_2026-01-10.sql
    ```

### c) 自动化备份

在生产环境中，备份必须是自动化的。在 Linux 服务器上，通常使用 **`cron` 作业**来定时执行备份脚本。

一个简单的 `backup.sh` 脚本可能如下：
```bash
#!/bin/bash
DB_USER="root"
DB_PASS="your_password"
DB_NAME="my_blog"
BACKUP_DIR="/var/backups/mysql"
DATE=$(date +%F-%H-%M)
FILENAME="${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz"

# 使用 mysqldump 并通过 gzip 压缩
mysqldump -u ${DB_USER} -p${DB_PASS} --single-transaction ${DB_NAME} | gzip > ${FILENAME}

# 删除 7 天前的旧备份
find ${BACKUP_DIR} -type f -name "*.sql.gz" -mtime +7 -delete
```
然后，你可以设置一个 `cron` 作业，让它在每天凌晨 2 点执行此脚本。

---

## 练习任务

1.  **手动备份**：
    使用 `mysqldump` 命令，为你本地的 `my_blog` 数据库创建一个完整的逻辑备份，并确保使用了 `--single-transaction` 选项。

2.  **模拟灾难恢复**：
    1.  在本地创建一个名为 `my_blog_restored` 的新数据库。
    2.  使用 `mysql` 命令，将你在上一步创建的备份文件恢复到这个新数据库中。
    3.  连接到 `my_blog_restored` 数据库，检查其中的数据是否与原数据库一致。

3.  **编写备份脚本**：
    参照本节的示例，编写一个你自己的 `backup.sh` 脚本。除了备份数据库，尝试在脚本末尾加入一行命令，将备份文件上传到一个云存储位置（即使只是一个伪命令，如 `echo "Uploading to cloud..."`）。
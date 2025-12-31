# 6.6.2 数据库备份与恢复

## 概述

数据库备份与恢复是数据库运维的重要工作，用于保护数据安全、应对数据丢失、支持数据迁移等。理解备份与恢复的方法和策略对于保证数据安全和系统可靠性至关重要。

本节详细介绍数据库备份与恢复的方法，包括备份类型（全量备份、增量备份）、备份工具（mysqldump、物理备份）、恢复方法、备份策略、自动化备份等，帮助零基础学员掌握数据库备份与恢复技能。

**主要内容**：
- 备份与恢复概述
- 备份类型（全量备份、增量备份、差异备份）
- 备份工具和方法（mysqldump、物理备份）
- 恢复方法和流程
- 备份策略和最佳实践
- 自动化备份实现
- 完整示例和最佳实践

---

## 特性

- **数据保护**：保护数据安全，防止数据丢失
- **灾难恢复**：支持灾难恢复
- **数据迁移**：支持数据迁移
- **版本管理**：支持数据版本管理
- **自动化**：支持自动化备份

---

## 备份与恢复概述

### 为什么需要备份

数据库备份的重要性：

- **数据安全**：防止数据丢失
- **灾难恢复**：应对系统故障
- **数据迁移**：支持数据迁移
- **版本管理**：支持数据版本管理
- **合规要求**：满足合规要求

### 备份的目标

备份的主要目标：

- **数据完整性**：保证备份数据的完整性
- **可恢复性**：确保备份可以恢复
- **时效性**：备份要及时
- **可验证性**：备份可以验证

---

## 备份类型

### 全量备份

全量备份是备份整个数据库。

**特点**：

- 备份完整
- 恢复简单
- 备份时间长
- 占用空间大

**示例**：

```bash
# 使用 mysqldump 全量备份
mysqldump -u root -p --all-databases > backup_all.sql

# 备份单个数据库
mysqldump -u root -p database_name > backup_database.sql

# 备份单个表
mysqldump -u root -p database_name table_name > backup_table.sql
```

### 增量备份

增量备份是备份自上次备份以来的变更。

**特点**：

- 备份快速
- 占用空间小
- 恢复复杂
- 需要二进制日志

**示例**：

```bash
# 启用二进制日志
# 在 my.cnf 中设置
log-bin=mysql-bin

# 增量备份（备份二进制日志）
mysqlbinlog mysql-bin.000001 > incremental_backup.sql
```

### 差异备份

差异备份是备份自上次全量备份以来的变更。

**特点**：

- 介于全量和增量之间
- 恢复相对简单
- 占用空间中等

---

## 备份工具

### mysqldump

mysqldump 是 MySQL 的官方备份工具。

**基本用法**：

```bash
# 备份整个数据库
mysqldump -u root -p database_name > backup.sql

# 备份多个数据库
mysqldump -u root -p --databases db1 db2 > backup.sql

# 备份所有数据库
mysqldump -u root -p --all-databases > backup_all.sql

# 只备份结构
mysqldump -u root -p --no-data database_name > structure.sql

# 只备份数据
mysqldump -u root -p --no-create-info database_name > data.sql
```

**高级选项**：

```bash
# 使用事务（保证一致性）
mysqldump -u root -p --single-transaction database_name > backup.sql

# 锁定所有表
mysqldump -u root -p --lock-all-tables database_name > backup.sql

# 压缩备份
mysqldump -u root -p database_name | gzip > backup.sql.gz

# 远程备份
mysqldump -h remote_host -u root -p database_name > backup.sql
```

### 物理备份

物理备份是直接复制数据库文件。

**方法**：

- 停止 MySQL 服务，复制数据文件
- 使用 MySQL Enterprise Backup
- 使用文件系统快照

**示例**：

```bash
# 停止 MySQL
systemctl stop mysql

# 复制数据目录
cp -r /var/lib/mysql /backup/mysql_backup

# 启动 MySQL
systemctl start mysql
```

---

## 恢复方法

### 使用 mysqldump 恢复

使用 mysqldump 备份文件恢复。

**示例**：

```bash
# 恢复整个数据库
mysql -u root -p database_name < backup.sql

# 恢复所有数据库
mysql -u root -p < backup_all.sql

# 恢复单个表
mysql -u root -p database_name < backup_table.sql
```

### 使用二进制日志恢复

使用二进制日志进行时间点恢复。

**示例**：

```bash
# 恢复全量备份
mysql -u root -p < full_backup.sql

# 应用增量备份（二进制日志）
mysqlbinlog mysql-bin.000001 | mysql -u root -p

# 恢复到指定时间点
mysqlbinlog --stop-datetime="2024-01-01 12:00:00" mysql-bin.000001 | mysql -u root -p
```

---

## 备份策略

### 备份频率

根据数据重要性确定备份频率。

**建议**：

- **重要数据**：每天全量备份 + 每小时增量备份
- **一般数据**：每天全量备份
- **不重要数据**：每周全量备份

### 备份保留

确定备份保留策略。

**建议**：

- **最近备份**：保留 7 天
- **周备份**：保留 4 周
- **月备份**：保留 12 个月

### 备份验证

定期验证备份的有效性。

**方法**：

- 恢复测试
- 校验和验证
- 定期演练

---

## 完整示例

### 自动化备份脚本

```php
<?php
declare(strict_types=1);

class DatabaseBackup
{
    private string $backupDir;
    private string $host;
    private string $username;
    private string $password;
    private string $database;
    
    public function __construct(
        string $backupDir,
        string $host,
        string $username,
        string $password,
        string $database
    ) {
        $this->backupDir = $backupDir;
        $this->host = $host;
        $this->username = $username;
        $this->password = $password;
        $this->database = $database;
    }
    
    public function backup(): string
    {
        $filename = $this->database . '_' . date('Y-m-d_H-i-s') . '.sql';
        $filepath = $this->backupDir . '/' . $filename;
        
        $command = sprintf(
            'mysqldump -h %s -u %s -p%s %s > %s',
            escapeshellarg($this->host),
            escapeshellarg($this->username),
            escapeshellarg($this->password),
            escapeshellarg($this->database),
            escapeshellarg($filepath)
        );
        
        exec($command, $output, $returnCode);
        
        if ($returnCode !== 0) {
            throw new RuntimeException('备份失败');
        }
        
        // 压缩备份
        $this->compress($filepath);
        
        return $filepath . '.gz';
    }
    
    private function compress(string $filepath): void
    {
        $command = sprintf('gzip %s', escapeshellarg($filepath));
        exec($command);
    }
    
    public function restore(string $backupFile): void
    {
        if (pathinfo($backupFile, PATHINFO_EXTENSION) === 'gz') {
            // 解压
            $uncompressed = str_replace('.gz', '', $backupFile);
            $command = sprintf('gunzip -c %s > %s', escapeshellarg($backupFile), escapeshellarg($uncompressed));
            exec($command);
            $backupFile = $uncompressed;
        }
        
        $command = sprintf(
            'mysql -h %s -u %s -p%s %s < %s',
            escapeshellarg($this->host),
            escapeshellarg($this->username),
            escapeshellarg($this->password),
            escapeshellarg($this->database),
            escapeshellarg($backupFile)
        );
        
        exec($command, $output, $returnCode);
        
        if ($returnCode !== 0) {
            throw new RuntimeException('恢复失败');
        }
    }
    
    public function cleanup(int $keepDays = 7): void
    {
        $files = glob($this->backupDir . '/*.sql.gz');
        $cutoffTime = time() - ($keepDays * 24 * 3600);
        
        foreach ($files as $file) {
            if (filemtime($file) < $cutoffTime) {
                unlink($file);
            }
        }
    }
}

// 使用
$backup = new DatabaseBackup(
    '/backup/mysql',
    'localhost',
    'root',
    'password',
    'test'
);

// 执行备份
$backupFile = $backup->backup();
echo "备份完成: {$backupFile}\n";

// 清理旧备份
$backup->cleanup(7);
```

---

## 使用场景

### 定期备份

定期执行备份，保护数据安全。

### 灾难恢复

在系统故障时恢复数据。

### 数据迁移

使用备份进行数据迁移。

---

## 注意事项

### 备份验证

定期验证备份的有效性。

### 备份安全

保护备份文件的安全。

### 恢复测试

定期进行恢复测试。

---

## 常见问题

### 如何备份数据库？

使用 mysqldump 或物理备份。

### 如何恢复数据库？

使用备份文件恢复。

### 如何实现自动化备份？

使用脚本和定时任务实现自动化备份。

---

## 最佳实践

### 定期备份

根据数据重要性定期备份。

### 验证备份

定期验证备份的有效性。

### 测试恢复

定期进行恢复测试。

---

## 练习任务

1. **备份操作**
   - 实现全量备份
   - 实现增量备份
   - 测试备份功能
   - 验证备份文件

2. **恢复操作**
   - 实现数据恢复
   - 实现时间点恢复
   - 测试恢复功能
   - 验证数据完整性

3. **自动化备份**
   - 实现自动化备份脚本
   - 实现备份清理
   - 测试自动化功能
   - 编写备份文档

4. **备份策略**
   - 设计备份策略
   - 实现备份策略
   - 测试备份策略
   - 优化备份流程

5. **综合应用**
   - 创建一个完整的备份系统
   - 实现所有功能
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.6.1 数据库连接池](section-01-connection-pool.md)
- [6.6.3 消息队列基础](section-03-message-queue.md)
- [6.2 PDO 入门与高安全模式](../chapter-02-pdo/readme.md)

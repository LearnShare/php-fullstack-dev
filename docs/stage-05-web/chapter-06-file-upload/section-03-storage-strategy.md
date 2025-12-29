# 5.6.3 文件存储策略

## 概述

文件存储策略影响文件管理的效率、安全性和可扩展性。选择合适的存储位置、设计合理的文件命名策略、组织清晰的目录结构、实现有效的访问控制，对于构建可靠的文件管理系统至关重要。本节详细介绍文件存储的策略选择，包括存储位置选择（本地存储、云存储）、文件命名策略、目录结构设计、文件访问控制、云存储集成等内容，帮助零基础学员设计合理的文件存储方案。

文件存储策略需要考虑多个因素：存储容量、访问性能、备份恢复、成本控制、安全性等。不同的应用场景需要不同的存储策略。

**主要内容**：
- 存储策略概述（影响因素、选择原则）
- 存储位置选择（本地文件系统、云存储服务、对象存储）
- 文件命名策略（随机文件名、基于内容的命名、时间戳命名、哈希命名）
- 目录结构设计（按日期组织、按类型组织、按用户组织、哈希目录）
- 文件访问控制（权限设置、访问验证、直接访问限制、下载控制）
- 云存储集成（AWS S3、阿里云 OSS、腾讯云 COS）
- 实际应用示例和最佳实践

## 特性

- **灵活选择**：支持本地存储和云存储
- **可扩展性**：支持大规模文件存储
- **安全性**：提供访问控制和权限管理
- **性能优化**：通过目录结构优化访问性能
- **成本控制**：根据需求选择合适的存储方案

## 存储策略概述

### 影响因素

1. **存储容量**：需要存储多少文件
2. **访问性能**：文件访问的频率和速度要求
3. **备份恢复**：备份和恢复的需求
4. **成本控制**：存储成本预算
5. **安全性**：数据安全要求
6. **可扩展性**：未来扩展需求

### 选择原则

- **小规模应用**：本地文件系统
- **中大规模应用**：云存储服务
- **高可用性要求**：分布式存储
- **成本敏感**：根据实际需求选择

## 存储位置选择

### 本地文件系统

**优点**：
- 成本低
- 控制完全
- 访问速度快（本地网络）

**缺点**：
- 容量有限
- 备份复杂
- 扩展困难
- 单点故障风险

**适用场景**：
- 小规模应用
- 内部系统
- 临时文件存储

**示例**：
```php
<?php
declare(strict_types=1);

$uploadDir = __DIR__ . '/uploads/';

// 确保目录存在
if (!is_dir($uploadDir)) {
    mkdir($uploadDir, 0755, true);
}

$destination = $uploadDir . $filename;
```

### 云存储服务

**优点**：
- 容量无限
- 自动备份
- 易于扩展
- 高可用性

**缺点**：
- 成本较高
- 依赖网络
- 需要集成

**适用场景**：
- 大规模应用
- 高可用性要求
- 需要 CDN 加速

**主流云存储**：
- **AWS S3**：Amazon Simple Storage Service
- **阿里云 OSS**：Object Storage Service
- **腾讯云 COS**：Cloud Object Storage
- **七牛云**：对象存储服务

## 文件命名策略

### 策略一：随机文件名

**优点**：
- 唯一性好
- 安全性高
- 实现简单

**缺点**：
- 无意义
- 难以管理

**示例**：
```php
<?php
declare(strict_types=1);

function generateRandomFileName(string $extension): string
{
    $randomBytes = random_bytes(16);
    $randomHex = bin2hex($randomBytes);
    return $randomHex . '.' . $extension;
}

// 使用
$filename = generateRandomFileName('jpg');
// 输出: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6.jpg
```

### 策略二：基于内容的命名

**优点**：
- 内容唯一性
- 去重容易
- 有意义

**缺点**：
- 计算开销
- 哈希冲突风险

**示例**：
```php
<?php
declare(strict_types=1);

function generateHashFileName(string $filepath, string $extension): string
{
    $hash = hash_file('sha256', $filepath);
    return substr($hash, 0, 32) . '.' . $extension;
}

// 使用
$filename = generateHashFileName($file['tmp_name'], 'jpg');
// 输出: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6.jpg
```

### 策略三：时间戳命名

**优点**：
- 时间顺序
- 易于管理
- 实现简单

**缺点**：
- 可能冲突
- 安全性较低

**示例**：
```php
<?php
declare(strict_types=1);

function generateTimestampFileName(string $extension): string
{
    $timestamp = time();
    $microtime = (int) (microtime(true) * 1000000);
    return $timestamp . '_' . $microtime . '.' . $extension;
}

// 使用
$filename = generateTimestampFileName('jpg');
// 输出: 1703123456_123456.jpg
```

### 策略四：组合命名

**示例**：
```php
<?php
declare(strict_types=1);

function generateCombinedFileName(string $filepath, string $extension): string
{
    // 时间戳 + 随机数 + 哈希
    $timestamp = time();
    $random = bin2hex(random_bytes(4));
    $hash = substr(hash_file('md5', $filepath), 0, 8);
    
    return $timestamp . '_' . $random . '_' . $hash . '.' . $extension;
}

// 使用
$filename = generateCombinedFileName($file['tmp_name'], 'jpg');
// 输出: 1703123456_a1b2c3d4_e5f6g7h8.jpg
```

## 目录结构设计

### 策略一：按日期组织

**优点**：
- 时间顺序清晰
- 易于归档
- 实现简单

**示例**：
```php
<?php
declare(strict_types=1);

function getDateBasedPath(string $baseDir): string
{
    $year = date('Y');
    $month = date('m');
    $day = date('d');
    
    $path = $baseDir . '/' . $year . '/' . $month . '/' . $day . '/';
    
    if (!is_dir($path)) {
        mkdir($path, 0755, true);
    }
    
    return $path;
}

// 使用
$baseDir = __DIR__ . '/uploads';
$path = getDateBasedPath($baseDir);
// 输出: /uploads/2024/01/15/
```

### 策略二：按类型组织

**优点**：
- 类型清晰
- 易于管理
- 访问优化

**示例**：
```php
<?php
declare(strict_types=1);

function getTypeBasedPath(string $baseDir, string $extension): string
{
    $typeMap = [
        'jpg' => 'images',
        'jpeg' => 'images',
        'png' => 'images',
        'gif' => 'images',
        'pdf' => 'documents',
        'doc' => 'documents',
        'docx' => 'documents',
    ];
    
    $type = $typeMap[strtolower($extension)] ?? 'others';
    $path = $baseDir . '/' . $type . '/';
    
    if (!is_dir($path)) {
        mkdir($path, 0755, true);
    }
    
    return $path;
}
```

### 策略三：按用户组织

**优点**：
- 用户隔离
- 权限管理
- 易于清理

**示例**：
```php
<?php
declare(strict_types=1);

function getUserBasedPath(string $baseDir, int $userId): string
{
    // 使用用户 ID 的哈希值分散目录
    $hash = md5((string) $userId);
    $subDir = substr($hash, 0, 2) . '/' . substr($hash, 2, 2);
    
    $path = $baseDir . '/users/' . $subDir . '/' . $userId . '/';
    
    if (!is_dir($path)) {
        mkdir($path, 0755, true);
    }
    
    return $path;
}
```

### 策略四：哈希目录

**优点**：
- 文件分散
- 性能优化
- 避免单目录文件过多

**示例**：
```php
<?php
declare(strict_types=1);

function getHashBasedPath(string $baseDir, string $filename): string
{
    $hash = md5($filename);
    $level1 = substr($hash, 0, 2);
    $level2 = substr($hash, 2, 2);
    
    $path = $baseDir . '/' . $level1 . '/' . $level2 . '/';
    
    if (!is_dir($path)) {
        mkdir($path, 0755, true);
    }
    
    return $path;
}

// 使用
$filename = 'example.jpg';
$path = getHashBasedPath(__DIR__ . '/uploads', $filename);
// 输出: /uploads/a1/b2/
```

## 文件访问控制

### 权限设置

**示例**：
```php
<?php
declare(strict_types=1);

function setFilePermissions(string $filepath): void
{
    // 设置文件权限（所有者可读写，其他用户只读）
    chmod($filepath, 0644);
}

function setDirectoryPermissions(string $dirpath): void
{
    // 设置目录权限（所有者可读写执行，其他用户只读执行）
    chmod($dirpath, 0755);
}
```

### 访问验证

**示例**：
```php
<?php
declare(strict_types=1);

class FileAccessController
{
    public static function canAccess(string $filepath, int $userId): bool
    {
        // 检查文件是否存在
        if (!file_exists($filepath)) {
            return false;
        }
        
        // 检查文件所有者
        $fileOwner = fileowner($filepath);
        // 这里需要根据实际需求实现用户 ID 到系统用户 ID 的映射
        
        // 检查权限
        // ...
        
        return true;
    }
    
    public static function serveFile(string $filepath): void
    {
        if (!file_exists($filepath)) {
            http_response_code(404);
            exit;
        }
        
        // 设置响应头
        header('Content-Type: ' . mime_content_type($filepath));
        header('Content-Length: ' . filesize($filepath));
        header('Content-Disposition: inline; filename="' . basename($filepath) . '"');
        
        // 输出文件
        readfile($filepath);
        exit;
    }
}
```

### 直接访问限制

**示例**（.htaccess）：
```apache
# 禁止直接访问上传目录
<Directory "/path/to/uploads">
    Options -Indexes
    AllowOverride None
    Require all denied
</Directory>
```

**示例**（Nginx）：
```nginx
location /uploads {
    deny all;
    return 403;
}
```

### 下载控制

**示例**：
```php
<?php
declare(strict_types=1);

function downloadFile(string $filepath, string $filename): void
{
    if (!file_exists($filepath)) {
        http_response_code(404);
        exit;
    }
    
    // 设置响应头
    header('Content-Type: application/octet-stream');
    header('Content-Disposition: attachment; filename="' . $filename . '"');
    header('Content-Length: ' . filesize($filepath));
    
    // 输出文件
    readfile($filepath);
    exit;
}
```

## 云存储集成

### AWS S3 集成

**示例**（使用 AWS SDK）：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Aws\S3\S3Client;
use Aws\Exception\AwsException;

class S3Storage
{
    private S3Client $s3Client;
    private string $bucket;

    public function __construct(string $bucket, string $region = 'us-east-1')
    {
        $this->s3Client = new S3Client([
            'version' => 'latest',
            'region' => $region,
        ]);
        $this->bucket = $bucket;
    }

    public function upload(string $filepath, string $key): bool
    {
        try {
            $result = $this->s3Client->putObject([
                'Bucket' => $this->bucket,
                'Key' => $key,
                'SourceFile' => $filepath,
            ]);
            return true;
        } catch (AwsException $e) {
            error_log($e->getMessage());
            return false;
        }
    }

    public function getUrl(string $key, int $expires = 3600): string
    {
        $cmd = $this->s3Client->getCommand('GetObject', [
            'Bucket' => $this->bucket,
            'Key' => $key,
        ]);
        
        $request = $this->s3Client->createPresignedRequest($cmd, "+{$expires} seconds");
        return (string) $request->getUri();
    }
}

// 使用
$storage = new S3Storage('my-bucket');
$storage->upload($file['tmp_name'], 'uploads/' . $filename);
```

### 统一存储接口

**示例**：
```php
<?php
declare(strict_types=1);

interface StorageInterface
{
    public function upload(string $filepath, string $key): bool;
    public function getUrl(string $key, int $expires = 3600): string;
    public function delete(string $key): bool;
}

class LocalStorage implements StorageInterface
{
    public function __construct(private string $baseDir) {}

    public function upload(string $filepath, string $key): bool
    {
        $destination = $this->baseDir . '/' . $key;
        $dir = dirname($destination);
        
        if (!is_dir($dir)) {
            mkdir($dir, 0755, true);
        }
        
        return copy($filepath, $destination);
    }

    public function getUrl(string $key, int $expires = 3600): string
    {
        return '/uploads/' . $key;
    }

    public function delete(string $key): bool
    {
        $filepath = $this->baseDir . '/' . $key;
        return unlink($filepath);
    }
}

// 使用
$storage = new LocalStorage(__DIR__ . '/uploads');
$storage->upload($file['tmp_name'], 'images/' . $filename);
```

## 使用场景

### 文件管理系统

- 企业文档管理
- 内容管理系统
- 文件共享平台

### 内容管理系统

- 图片存储
- 媒体文件存储
- 附件存储

### 云存储应用

- 大规模文件存储
- CDN 加速
- 高可用性存储

### 大规模文件存储

- 分布式存储
- 对象存储
- 文件归档

## 注意事项

### 目录权限

- **上传目录**：设置正确的权限（通常 755）
- **文件权限**：设置正确的权限（通常 644）
- **Web 服务器权限**：确保 Web 服务器有写入权限

### 存储空间

- **监控使用量**：定期监控存储空间使用
- **清理策略**：实现文件清理策略
- **容量规划**：根据需求规划存储容量

### 访问性能

- **目录分散**：避免单目录文件过多
- **CDN 加速**：使用 CDN 加速文件访问
- **缓存策略**：实现文件访问缓存

### 备份策略

- **定期备份**：定期备份重要文件
- **增量备份**：使用增量备份减少备份时间
- **异地备份**：实现异地备份提高可靠性

## 常见问题

### 如何组织文件存储？

根据应用需求选择：
- **按日期**：时间顺序清晰
- **按类型**：类型管理方便
- **按用户**：用户隔离
- **哈希目录**：性能优化

### 如何生成安全的文件名？

1. **随机文件名**：使用 `random_bytes()`
2. **哈希文件名**：使用文件哈希
3. **组合命名**：时间戳 + 随机数 + 哈希

### 本地存储和云存储如何选择？

- **小规模应用**：本地存储
- **大规模应用**：云存储
- **高可用性要求**：云存储
- **成本敏感**：根据实际需求选择

### 如何控制文件访问？

1. **权限设置**：设置正确的文件权限
2. **访问验证**：验证用户权限
3. **直接访问限制**：限制直接访问上传目录
4. **下载控制**：控制文件下载

## 最佳实践

### 使用哈希目录分散文件

- 避免单目录文件过多
- 提高文件系统性能
- 使用哈希值分散文件

### 生成唯一文件名

- 使用随机数或哈希
- 避免文件名冲突
- 提高安全性

### 设置合适的权限

- 上传目录：755
- 文件：644
- 确保 Web 服务器有写入权限

### 考虑使用云存储

- 大规模应用使用云存储
- 高可用性要求使用云存储
- 需要 CDN 加速使用云存储

## 相关章节

- **[5.6.1 文件上传基础](section-01-upload-basics.md)**：了解文件上传的基础内容
- **[5.6.2 文件验证与安全](section-02-validation-security.md)**：了解文件验证的详细内容
- **[4.2 文件系统操作](../../stage-04-system/chapter-02-filesystem/readme.md)**：了解文件系统操作的详细内容

## 练习任务

1. **实现文件命名策略**
   - 随机文件名生成
   - 哈希文件名生成
   - 组合命名策略

2. **实现目录结构设计**
   - 按日期组织
   - 按类型组织
   - 哈希目录

3. **实现文件访问控制**
   - 权限设置
   - 访问验证
   - 下载控制

4. **实现云存储集成**
   - AWS S3 集成
   - 统一存储接口
   - 存储切换

5. **实现完整的文件存储系统**
   - 存储策略选择
   - 文件管理
   - 访问控制

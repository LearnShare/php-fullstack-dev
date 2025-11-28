# 6.3 Secrets 管理

## 目标

- 理解 Secrets 管理的重要性。
- 掌握 `.env` 文件的安全使用。
- 了解 Vault、AWS Secret Manager 等密钥管理服务。
- 熟悉密钥轮换（Key Rotation）的实现。

## .env 文件安全

### 基础使用

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Dotenv\Dotenv;

// 加载 .env 文件
$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();

// 读取环境变量
$dbHost = $_ENV['DB_HOST'] ?? 'localhost';
$dbPassword = $_ENV['DB_PASSWORD'] ?? '';
```

### 安全实践

```php
<?php
declare(strict_types=1);

// 1. .env 文件不应提交到版本控制
// .gitignore
// .env
// .env.local
// .env.*.local

// 2. 使用 .env.example 作为模板
// .env.example
// DB_HOST=localhost
// DB_PASSWORD=

// 3. 验证必需的环境变量
$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();
$dotenv->required(['DB_HOST', 'DB_PASSWORD', 'APP_KEY']);

// 4. 类型转换
$debug = filter_var($_ENV['APP_DEBUG'] ?? 'false', FILTER_VALIDATE_BOOLEAN);
$port = (int) ($_ENV['DB_PORT'] ?? 3306);
```

### 环境变量管理

```php
<?php
declare(strict_types=1);

class Env
{
    public static function get(string $key, mixed $default = null): mixed
    {
        $value = $_ENV[$key] ?? $_SERVER[$key] ?? getenv($key);
        return $value !== false ? $value : $default;
    }
    
    public static function getInt(string $key, int $default = 0): int
    {
        return (int) self::get($key, $default);
    }
    
    public static function getBool(string $key, bool $default = false): bool
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        return filter_var($value, FILTER_VALIDATE_BOOLEAN);
    }
    
    public static function require(string $key): string
    {
        $value = self::get($key);
        if ($value === null) {
            throw new RuntimeException("Required environment variable {$key} is not set");
        }
        return $value;
    }
}

// 使用
$dbHost = Env::require('DB_HOST');
$dbPort = Env::getInt('DB_PORT', 3306);
$debug = Env::getBool('APP_DEBUG', false);
```

## 密钥管理服务

### Vault 集成

```php
<?php
declare(strict_types=1);

class VaultClient
{
    private string $vaultUrl;
    private string $token;
    
    public function __construct(string $vaultUrl, string $token)
    {
        $this->vaultUrl = rtrim($vaultUrl, '/');
        $this->token = $token;
    }
    
    public function getSecret(string $path): array
    {
        $url = "{$this->vaultUrl}/v1/{$path}";
        
        $ch = curl_init($url);
        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => [
                "X-Vault-Token: {$this->token}",
            ],
        ]);
        
        $response = curl_exec($ch);
        $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);
        
        if ($status !== 200) {
            throw new RuntimeException("Failed to get secret from Vault: {$path}");
        }
        
        $data = json_decode($response, true);
        return $data['data']['data'] ?? [];
    }
}

// 使用
$vault = new VaultClient('https://vault.example.com', $vaultToken);
$dbCredentials = $vault->getSecret('secret/data/database');
```

### AWS Secrets Manager

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Aws\SecretsManager\SecretsManagerClient;
use Aws\Exception\AwsException;

class AwsSecretsManager
{
    private SecretsManagerClient $client;
    
    public function __construct(array $config)
    {
        $this->client = new SecretsManagerClient($config);
    }
    
    public function getSecret(string $secretName): array
    {
        try {
            $result = $this->client->getSecretValue([
                'SecretId' => $secretName,
            ]);
            
            $secret = $result['SecretString'] ?? base64_decode($result['SecretBinary']);
            return json_decode($secret, true);
            
        } catch (AwsException $e) {
            throw new RuntimeException("Failed to get secret: {$e->getMessage()}");
        }
    }
}

// 使用
$secretsManager = new AwsSecretsManager([
    'region' => 'us-east-1',
    'version' => 'latest',
]);

$dbCredentials = $secretsManager->getSecret('prod/database/credentials');
```

## 密钥轮换

### 密钥轮换策略

```php
<?php
declare(strict_types=1);

class KeyRotationManager
{
    private PDO $db;
    private EncryptionService $encryption;
    
    public function __construct(PDO $db, EncryptionService $encryption)
    {
        $this->db = $db;
        $this->encryption = $encryption;
    }
    
    public function rotateKey(string $oldKey, string $newKey): void
    {
        $this->db->beginTransaction();
        
        try {
            // 1. 使用旧密钥解密所有数据
            $stmt = $this->db->query("SELECT id, encrypted_data FROM sensitive_data");
            $records = $stmt->fetchAll();
            
            // 2. 使用新密钥重新加密
            $tempEncryption = new EncryptionService($oldKey);
            $newEncryption = new EncryptionService($newKey);
            
            foreach ($records as $record) {
                $decrypted = $tempEncryption->decrypt($record['encrypted_data']);
                $reencrypted = $newEncryption->encrypt($decrypted);
                
                $updateStmt = $this->db->prepare("UPDATE sensitive_data SET encrypted_data = ? WHERE id = ?");
                $updateStmt->execute([$reencrypted, $record['id']]);
            }
            
            // 3. 更新密钥版本
            $this->db->exec("UPDATE key_versions SET current_key = 'new_key', rotated_at = NOW()");
            
            $this->db->commit();
        } catch (Exception $e) {
            $this->db->rollBack();
            throw $e;
        }
    }
}
```

## 最佳实践

### 1. 不要硬编码密钥

```php
// 不推荐
$apiKey = 'sk_live_1234567890';

// 推荐
$apiKey = $_ENV['STRIPE_API_KEY'];
```

### 2. 使用不同的密钥环境

```php
// 开发环境
$key = $_ENV['DEV_ENCRYPTION_KEY'];

// 生产环境
$key = $_ENV['PROD_ENCRYPTION_KEY'];
```

### 3. 定期轮换密钥

- 设置密钥过期时间。
- 自动触发轮换流程。
- 保留旧密钥用于解密历史数据。

## 练习

1. 创建一个环境变量管理类，支持类型转换和必需变量验证。

2. 实现一个密钥管理服务接口，支持从 Vault 或 AWS Secrets Manager 获取密钥。

3. 设计一个密钥轮换系统，安全地更新加密密钥。

4. 编写一个配置管理类，支持从多个来源（.env、数据库、密钥管理服务）读取配置。

5. 创建一个密钥版本管理系统，支持多版本密钥并存。

6. 实现一个配置验证工具，检查所有必需的配置项是否已设置。

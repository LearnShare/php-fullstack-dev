# 6.3.1 Secrets 管理

## 概述

Secrets 管理是生产环境安全的关键环节。本节介绍 Secrets 管理的重要性、`.env` 文件的安全使用、密钥管理服务（Vault、AWS Secret Manager）的集成，以及密钥轮换的实现。

## Secrets 管理的重要性

### 为什么需要 Secrets 管理

1. **安全性**：敏感信息不应硬编码在代码中
2. **合规性**：满足安全合规要求（如 PCI DSS、GDPR）
3. **可维护性**：集中管理，便于更新和轮换
4. **审计性**：记录密钥访问和使用情况

### 常见 Secrets 类型

- 数据库密码
- API 密钥（如 Stripe、AWS）
- 加密密钥
- JWT 密钥
- OAuth 客户端密钥

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

#### 1. 不要提交 .env 文件到版本控制

```gitignore
# .gitignore
.env
.env.local
.env.*.local
```

#### 2. 使用 .env.example 作为模板

```bash
# .env.example
DB_HOST=localhost
DB_PASSWORD=
APP_KEY=
STRIPE_API_KEY=
```

#### 3. 验证必需的环境变量

```php
<?php
declare(strict_types=1);

$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();
$dotenv->required(['DB_HOST', 'DB_PASSWORD', 'APP_KEY']);
```

#### 4. 类型转换

```php
<?php
declare(strict_types=1);

$debug = filter_var($_ENV['APP_DEBUG'] ?? 'false', FILTER_VALIDATE_BOOLEAN);
$port = (int) ($_ENV['DB_PORT'] ?? 3306);
```

## 密钥管理服务

### Vault 集成

HashiCorp Vault 是一个用于安全存储和访问密钥的工具。

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
    
    public function setSecret(string $path, array $data): void
    {
        $url = "{$this->vaultUrl}/v1/{$path}";
        
        $ch = curl_init($url);
        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => json_encode(['data' => $data]),
            CURLOPT_HTTPHEADER => [
                "X-Vault-Token: {$this->token}",
                "Content-Type: application/json",
            ],
        ]);
        
        $response = curl_exec($ch);
        $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);
        
        if ($status !== 200 && $status !== 204) {
            throw new RuntimeException("Failed to set secret in Vault: {$path}");
        }
    }
}

// 使用
$vault = new VaultClient('https://vault.example.com', $vaultToken);
$dbCredentials = $vault->getSecret('secret/data/database');
```

### AWS Secrets Manager

AWS Secrets Manager 是 AWS 提供的密钥管理服务。

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
    
    public function createSecret(string $secretName, array $secretValue): void
    {
        try {
            $this->client->createSecret([
                'Name' => $secretName,
                'SecretString' => json_encode($secretValue),
            ]);
        } catch (AwsException $e) {
            throw new RuntimeException("Failed to create secret: {$e->getMessage()}");
        }
    }
    
    public function updateSecret(string $secretName, array $secretValue): void
    {
        try {
            $this->client->updateSecret([
                'SecretId' => $secretName,
                'SecretString' => json_encode($secretValue),
            ]);
        } catch (AwsException $e) {
            throw new RuntimeException("Failed to update secret: {$e->getMessage()}");
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

### 统一的 Secrets 管理接口

```php
<?php
declare(strict_types=1);

interface SecretsManagerInterface
{
    public function getSecret(string $key): mixed;
    public function setSecret(string $key, mixed $value): void;
}

class EnvSecretsManager implements SecretsManagerInterface
{
    public function getSecret(string $key): mixed
    {
        return $_ENV[$key] ?? null;
    }
    
    public function setSecret(string $key, mixed $value): void
    {
        $_ENV[$key] = $value;
    }
}

class VaultSecretsManager implements SecretsManagerInterface
{
    private VaultClient $vault;
    private string $basePath;
    
    public function __construct(VaultClient $vault, string $basePath = 'secret/data')
    {
        $this->vault = $vault;
        $this->basePath = $basePath;
    }
    
    public function getSecret(string $key): mixed
    {
        $path = "{$this->basePath}/{$key}";
        $data = $this->vault->getSecret($path);
        return $data[$key] ?? null;
    }
    
    public function setSecret(string $key, mixed $value): void
    {
        $path = "{$this->basePath}/{$key}";
        $this->vault->setSecret($path, [$key => $value]);
    }
}

// 使用工厂模式选择实现
class SecretsManagerFactory
{
    public static function create(string $type, array $config): SecretsManagerInterface
    {
        return match ($type) {
            'env' => new EnvSecretsManager(),
            'vault' => new VaultSecretsManager(
                new VaultClient($config['url'], $config['token']),
                $config['base_path'] ?? 'secret/data'
            ),
            'aws' => new AwsSecretsManager($config),
            default => throw new InvalidArgumentException("Unknown secrets manager type: {$type}"),
        };
    }
}
```

## 密钥轮换

### 密钥轮换策略

密钥轮换是定期更新密钥以降低安全风险的重要实践。

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
            $records = $stmt->fetchAll(PDO::FETCH_ASSOC);
            
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
    
    public function scheduleRotation(int $days): void
    {
        // 定期检查密钥是否需要轮换
        $stmt = $this->db->query("SELECT rotated_at FROM key_versions WHERE current_key = 'current'");
        $lastRotation = $stmt->fetchColumn();
        
        if ($lastRotation && (time() - strtotime($lastRotation)) > ($days * 86400)) {
            $this->rotateKey($this->getCurrentKey(), $this->generateNewKey());
        }
    }
    
    private function getCurrentKey(): string
    {
        // 从密钥管理服务获取当前密钥
        return $_ENV['ENCRYPTION_KEY'];
    }
    
    private function generateNewKey(): string
    {
        return bin2hex(random_bytes(32));
    }
}
```

### 密钥版本管理

```php
<?php
declare(strict_types=1);

class KeyVersionManager
{
    private PDO $db;
    
    public function __construct(PDO $db)
    {
        $this->db = $db;
    }
    
    public function addKeyVersion(string $keyId, string $key, bool $isActive = false): void
    {
        $stmt = $this->db->prepare("
            INSERT INTO key_versions (key_id, key_value, is_active, created_at)
            VALUES (?, ?, ?, NOW())
        ");
        $stmt->execute([$keyId, $key, $isActive ? 1 : 0]);
    }
    
    public function activateKey(string $keyId): void
    {
        $this->db->beginTransaction();
        
        try {
            // 停用所有密钥
            $this->db->exec("UPDATE key_versions SET is_active = 0");
            
            // 激活指定密钥
            $stmt = $this->db->prepare("UPDATE key_versions SET is_active = 1 WHERE key_id = ?");
            $stmt->execute([$keyId]);
            
            $this->db->commit();
        } catch (Exception $e) {
            $this->db->rollBack();
            throw $e;
        }
    }
    
    public function getActiveKey(): ?string
    {
        $stmt = $this->db->query("SELECT key_value FROM key_versions WHERE is_active = 1 LIMIT 1");
        return $stmt->fetchColumn() ?: null;
    }
    
    public function getKeyForDecryption(string $keyId): ?string
    {
        // 获取指定版本的密钥用于解密历史数据
        $stmt = $this->db->prepare("SELECT key_value FROM key_versions WHERE key_id = ?");
        $stmt->execute([$keyId]);
        return $stmt->fetchColumn() ?: null;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Dotenv\Dotenv;

// 1. 加载环境变量
$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();
$dotenv->required(['VAULT_URL', 'VAULT_TOKEN', 'ENCRYPTION_KEY']);

// 2. 初始化密钥管理服务
$secretsManager = SecretsManagerFactory::create('vault', [
    'url' => $_ENV['VAULT_URL'],
    'token' => $_ENV['VAULT_TOKEN'],
    'base_path' => 'secret/data',
]);

// 3. 获取数据库凭证
$dbCredentials = $secretsManager->getSecret('database');

// 4. 连接数据库
$pdo = new PDO(
    "mysql:host={$dbCredentials['host']};dbname={$dbCredentials['database']}",
    $dbCredentials['username'],
    $dbCredentials['password']
);

// 5. 初始化密钥轮换管理器
$keyRotation = new KeyRotationManager($pdo, new EncryptionService($_ENV['ENCRYPTION_KEY']));

// 6. 定期检查密钥轮换
$keyRotation->scheduleRotation(90); // 每 90 天轮换一次
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

- 设置密钥过期时间
- 自动触发轮换流程
- 保留旧密钥用于解密历史数据

### 4. 限制密钥访问

- 使用最小权限原则
- 记录密钥访问日志
- 定期审计密钥使用情况

## 注意事项

1. **密钥存储**：生产环境应使用专业的密钥管理服务，而非 `.env` 文件
2. **密钥传输**：使用 HTTPS 传输密钥，避免明文传输
3. **密钥备份**：安全备份密钥，但不要与代码一起存储
4. **密钥销毁**：及时销毁不再使用的密钥

## 练习

1. 实现一个统一的 Secrets 管理接口，支持从 `.env`、Vault 和 AWS Secrets Manager 获取密钥。

2. 设计一个密钥轮换系统，安全地更新加密密钥，并支持多版本密钥并存。

3. 创建一个密钥版本管理系统，支持密钥的激活、停用和历史数据解密。

4. 实现一个配置验证工具，检查所有必需的配置项是否已设置，并给出清晰的错误提示。

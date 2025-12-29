# 5.10.4 OAuth2 实现

## 概述

OAuth2 是一种授权协议，允许第三方应用访问用户资源，而无需提供用户密码。理解 OAuth2 的概念、掌握 OAuth2 的流程、实现授权码模式等授权模式，对于构建第三方登录、API 授权、服务间授权等功能至关重要。本节详细介绍 OAuth2 的概念、OAuth2 角色、授权码模式、客户端模式、密码模式、实现方法等内容，帮助零基础学员理解 OAuth2 协议。

OAuth2 是当前最流行的授权协议，被广泛用于第三方登录（如"使用 Google 登录"、"使用 GitHub 登录"）和 API 授权。理解 OAuth2 的工作原理对于实现安全的授权系统至关重要。

**主要内容**：
- OAuth2 概念（什么是 OAuth2、OAuth2 的作用、OAuth2 的优势）
- OAuth2 角色（资源所有者、客户端、授权服务器、资源服务器）
- 授权码模式（授权流程、授权码获取、Token 交换、刷新 Token）
- 客户端模式（适用场景、实现流程、安全考虑）
- 密码模式（适用场景、实现流程、安全考虑）
- 实现方法（授权服务器实现、资源服务器实现、客户端实现）
- 实际应用示例和最佳实践

## 特性

- **安全授权**：不暴露用户密码
- **灵活模式**：支持多种授权模式
- **标准协议**：遵循 OAuth2 标准（RFC 6749）
- **广泛支持**：主流平台都支持 OAuth2
- **易于集成**：易于集成到现有系统

## OAuth2 概念

### 什么是 OAuth2

OAuth2（Open Authorization 2.0）是一种授权框架，允许第三方应用获得对用户资源的有限访问权限，而无需提供用户密码。

### OAuth2 的作用

1. **第三方登录**：允许用户使用第三方账号登录
2. **API 授权**：授权第三方应用访问 API
3. **资源访问**：控制资源访问权限
4. **服务间授权**：服务间安全通信

### OAuth2 的优势

1. **安全性**：不暴露用户密码
2. **灵活性**：支持多种授权模式
3. **标准化**：遵循标准协议
4. **可扩展**：易于扩展和集成

## OAuth2 角色

### 资源所有者（Resource Owner）

**定义**：拥有被保护资源的用户。

**作用**：授权客户端访问资源。

### 客户端（Client）

**定义**：请求访问资源的第三方应用。

**类型**：
- **公开客户端**：无法安全存储密钥（如移动应用）
- **机密客户端**：可以安全存储密钥（如 Web 应用）

### 授权服务器（Authorization Server）

**定义**：颁发访问令牌的服务器。

**作用**：
- 验证用户身份
- 颁发授权码
- 交换访问令牌

### 资源服务器（Resource Server）

**定义**：托管受保护资源的服务器。

**作用**：
- 验证访问令牌
- 提供资源访问

## 授权码模式

### 授权流程

**完整流程**：
1. 客户端重定向用户到授权服务器
2. 用户在授权服务器登录并授权
3. 授权服务器重定向回客户端（带授权码）
4. 客户端用授权码换取访问令牌
5. 客户端使用访问令牌访问资源

### 授权码获取

**步骤 1：重定向到授权服务器**

**示例**：
```php
<?php
declare(strict_types=1);

function redirectToAuthServer(string $clientId, string $redirectUri, string $state): void
{
    $authServerUrl = 'https://auth.example.com/authorize';
    $params = [
        'response_type' => 'code',
        'client_id' => $clientId,
        'redirect_uri' => $redirectUri,
        'scope' => 'read write',
        'state' => $state,  // 防止 CSRF
    ];
    
    $url = $authServerUrl . '?' . http_build_query($params);
    header("Location: {$url}");
    exit;
}
```

**步骤 2：处理授权回调**

**示例**：
```php
<?php
declare(strict_types=1);

// 授权服务器回调处理
function handleAuthCallback(): void
{
    $code = $_GET['code'] ?? '';
    $state = $_GET['state'] ?? '';
    
    // 验证 state（防止 CSRF）
    if ($state !== $_SESSION['oauth_state']) {
        http_response_code(400);
        echo json_encode(['error' => 'Invalid state']);
        exit;
    }
    
    // 用授权码换取访问令牌
    $token = exchangeCodeForToken($code);
    
    // 存储访问令牌
    $_SESSION['access_token'] = $token['access_token'];
    
    // 重定向到应用
    header('Location: /dashboard');
    exit;
}
```

### Token 交换

**示例**：
```php
<?php
declare(strict_types=1);

function exchangeCodeForToken(string $code): array
{
    $tokenEndpoint = 'https://auth.example.com/token';
    
    $data = [
        'grant_type' => 'authorization_code',
        'code' => $code,
        'redirect_uri' => 'https://client.example.com/callback',
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
    ];
    
    $ch = curl_init($tokenEndpoint);
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => http_build_query($data),
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/x-www-form-urlencoded',
        ],
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}
```

### 刷新 Token

**示例**：
```php
<?php
declare(strict_types=1);

function refreshAccessToken(string $refreshToken): array
{
    $tokenEndpoint = 'https://auth.example.com/token';
    
    $data = [
        'grant_type' => 'refresh_token',
        'refresh_token' => $refreshToken,
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
    ];
    
    $ch = curl_init($tokenEndpoint);
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => http_build_query($data),
        CURLOPT_RETURNTRANSFER => true,
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}
```

## 客户端模式

### 适用场景

**适用场景**：
- 服务间通信
- 机器对机器认证
- 不需要用户参与的授权

### 实现流程

**示例**：
```php
<?php
declare(strict_types=1);

function getClientCredentialsToken(): array
{
    $tokenEndpoint = 'https://auth.example.com/token';
    
    $data = [
        'grant_type' => 'client_credentials',
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
        'scope' => 'api:read api:write',
    ];
    
    $ch = curl_init($tokenEndpoint);
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => http_build_query($data),
        CURLOPT_RETURNTRANSFER => true,
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}
```

### 安全考虑

- **客户端密钥**：安全存储客户端密钥
- **HTTPS**：使用 HTTPS 传输
- **范围限制**：限制客户端权限范围

## 密码模式

### 适用场景

**适用场景**：
- 受信任的客户端
- 第一方应用
- 内部系统

**注意**：不推荐用于第三方应用。

### 实现流程

**示例**：
```php
<?php
declare(strict_types=1);

function getPasswordToken(string $username, string $password): array
{
    $tokenEndpoint = 'https://auth.example.com/token';
    
    $data = [
        'grant_type' => 'password',
        'username' => $username,
        'password' => $password,
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
    ];
    
    $ch = curl_init($tokenEndpoint);
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => http_build_query($data),
        CURLOPT_RETURNTRANSFER => true,
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}
```

## 实现方法

### 授权服务器实现

**示例**：
```php
<?php
declare(strict_types=1);

class AuthorizationServer
{
    public function authorize(): void
    {
        $clientId = $_GET['client_id'] ?? '';
        $redirectUri = $_GET['redirect_uri'] ?? '';
        $state = $_GET['state'] ?? '';
        
        // 验证客户端
        if (!$this->validateClient($clientId, $redirectUri)) {
            http_response_code(400);
            echo json_encode(['error' => 'Invalid client']);
            exit;
        }
        
        // 检查用户是否已登录
        session_start();
        if (!isset($_SESSION['user_id'])) {
            // 重定向到登录页
            $_SESSION['oauth_redirect'] = $_SERVER['REQUEST_URI'];
            header('Location: /login.php');
            exit;
        }
        
        // 显示授权页面
        // 用户确认授权后，生成授权码
        $code = $this->generateAuthorizationCode($clientId, $_SESSION['user_id']);
        
        // 重定向回客户端
        $redirectUrl = $redirectUri . '?code=' . $code . '&state=' . $state;
        header("Location: {$redirectUrl}");
        exit;
    }

    public function token(): void
    {
        $grantType = $_POST['grant_type'] ?? '';
        
        switch ($grantType) {
            case 'authorization_code':
                $this->handleAuthorizationCode();
                break;
            case 'refresh_token':
                $this->handleRefreshToken();
                break;
            case 'client_credentials':
                $this->handleClientCredentials();
                break;
            default:
                http_response_code(400);
                echo json_encode(['error' => 'Unsupported grant type']);
                exit;
        }
    }

    private function handleAuthorizationCode(): void
    {
        $code = $_POST['code'] ?? '';
        $clientId = $_POST['client_id'] ?? '';
        $clientSecret = $_POST['client_secret'] ?? '';
        
        // 验证授权码
        $authCode = $this->validateAuthorizationCode($code, $clientId, $clientSecret);
        if ($authCode === null) {
            http_response_code(400);
            echo json_encode(['error' => 'Invalid authorization code']);
            exit;
        }
        
        // 生成访问令牌
        $accessToken = $this->generateAccessToken($authCode['user_id']);
        $refreshToken = $this->generateRefreshToken($authCode['user_id']);
        
        // 删除授权码（一次性使用）
        $this->deleteAuthorizationCode($code);
        
        http_response_code(200);
        header('Content-Type: application/json');
        echo json_encode([
            'access_token' => $accessToken,
            'refresh_token' => $refreshToken,
            'token_type' => 'Bearer',
            'expires_in' => 3600,
        ]);
    }

    private function generateAuthorizationCode(string $clientId, int $userId): string
    {
        $code = bin2hex(random_bytes(32));
        
        // 存储授权码（有效期 10 分钟）
        $this->storeAuthorizationCode($code, $clientId, $userId, time() + 600);
        
        return $code;
    }

    private function generateAccessToken(int $userId): string
    {
        // 使用 JWT 生成访问令牌
        $payload = [
            'user_id' => $userId,
            'iat' => time(),
            'exp' => time() + 3600,
        ];
        
        return JWT::encode($payload, $this->secretKey, 'HS256');
    }
}
```

### 资源服务器实现

**示例**：
```php
<?php
declare(strict_types=1);

class ResourceServer
{
    public function getResource(): void
    {
        // 获取访问令牌
        $token = $this->getAccessToken();
        
        if ($token === null) {
            http_response_code(401);
            echo json_encode(['error' => 'Access token required']);
            exit;
        }
        
        // 验证访问令牌
        $payload = $this->validateAccessToken($token);
        if ($payload === null) {
            http_response_code(401);
            echo json_encode(['error' => 'Invalid access token']);
            exit;
        }
        
        // 获取资源
        $userId = $payload['user_id'];
        $resource = $this->getUserResource($userId);
        
        http_response_code(200);
        header('Content-Type: application/json');
        echo json_encode(['data' => $resource]);
    }

    private function getAccessToken(): ?string
    {
        // 从 Authorization 头获取
        $authHeader = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
        if (preg_match('/Bearer\s+(.*)$/i', $authHeader, $matches)) {
            return $matches[1];
        }
        
        // 从查询参数获取（不推荐）
        return $_GET['access_token'] ?? null;
    }

    private function validateAccessToken(string $token): ?array
    {
        try {
            $decoded = JWT::decode($token, new Key($this->secretKey, 'HS256'));
            return (array) $decoded;
        } catch (Exception $e) {
            return null;
        }
    }
}
```

### 客户端实现

**示例**：
```php
<?php
declare(strict_types=1);

class OAuth2Client
{
    public function __construct(
        private string $clientId,
        private string $clientSecret,
        private string $redirectUri,
        private string $authServerUrl
    ) {}

    public function getAuthorizationUrl(string $state): string
    {
        $params = [
            'response_type' => 'code',
            'client_id' => $this->clientId,
            'redirect_uri' => $this->redirectUri,
            'scope' => 'read write',
            'state' => $state,
        ];
        
        return $this->authServerUrl . '/authorize?' . http_build_query($params);
    }

    public function exchangeCodeForToken(string $code): array
    {
        $tokenEndpoint = $this->authServerUrl . '/token';
        
        $data = [
            'grant_type' => 'authorization_code',
            'code' => $code,
            'redirect_uri' => $this->redirectUri,
            'client_id' => $this->clientId,
            'client_secret' => $this->clientSecret,
        ];
        
        $ch = curl_init($tokenEndpoint);
        curl_setopt_array($ch, [
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => http_build_query($data),
            CURLOPT_RETURNTRANSFER => true,
        ]);
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        return json_decode($response, true);
    }

    public function getResource(string $accessToken, string $resourceUrl): array
    {
        $ch = curl_init($resourceUrl);
        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => [
                'Authorization: Bearer ' . $accessToken,
            ],
        ]);
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        return json_decode($response, true);
    }
}
```

## 使用场景

### 第三方登录

- 使用 Google 登录
- 使用 GitHub 登录
- 使用微信登录

### API 授权

- 第三方应用访问 API
- API 密钥管理
- 权限范围控制

### 资源访问授权

- 文件访问授权
- 数据访问授权
- 服务访问授权

### 服务间授权

- 微服务间通信
- 服务间认证
- API 网关授权

## 注意事项

### 安全实现

- **HTTPS**：使用 HTTPS 传输
- **State 参数**：使用 state 参数防止 CSRF
- **客户端验证**：验证客户端身份
- **授权码一次性**：授权码只能使用一次

### 授权码有效期

- **短期有效**：授权码有效期通常为 10 分钟
- **一次性使用**：授权码使用后立即删除
- **及时交换**：及时用授权码换取访问令牌

### Token 安全存储

- **安全存储**：安全存储访问令牌
- **HTTPS 传输**：使用 HTTPS 传输令牌
- **令牌过期**：设置合理的过期时间

### 刷新 Token 机制

- **长期有效**：刷新令牌长期有效
- **安全存储**：安全存储刷新令牌
- **撤销机制**：支持撤销刷新令牌

## 常见问题

### OAuth2 的流程是什么？

1. 客户端重定向用户到授权服务器
2. 用户授权
3. 授权服务器返回授权码
4. 客户端用授权码换取访问令牌
5. 客户端使用访问令牌访问资源

### 授权码模式如何实现？

1. 实现授权端点（/authorize）
2. 实现令牌端点（/token）
3. 实现资源端点（/resource）
4. 客户端实现授权流程

### 如何选择授权模式？

- **授权码模式**：Web 应用（推荐）
- **客户端模式**：服务间通信
- **密码模式**：受信任的第一方应用
- **隐式模式**：单页应用（不推荐）

### OAuth2 的安全问题？

1. **CSRF 攻击**：使用 state 参数防护
2. **授权码泄露**：使用 HTTPS 传输
3. **令牌泄露**：安全存储令牌
4. **重定向 URI 验证**：验证重定向 URI

## 最佳实践

### 使用授权码模式（Web 应用）

- Web 应用使用授权码模式
- 验证重定向 URI
- 使用 state 参数防止 CSRF

### 实现安全的 Token 存储

- 安全存储访问令牌
- 使用 HttpOnly Cookie
- 实现令牌刷新机制

### 验证重定向 URI

- 验证重定向 URI 是否在允许列表中
- 防止开放重定向攻击
- 使用白名单机制

### 使用 HTTPS

- 所有 OAuth2 通信使用 HTTPS
- 保护授权码和令牌传输
- 防止中间人攻击

## 相关章节

- **[5.10.1 认证基础（AuthN）](section-01-authentication.md)**：了解认证的详细内容
- **[5.10.3 JWT 与 Token](section-03-jwt-token.md)**：了解 JWT Token 的详细内容
- **[5.8.2 CORS 跨域处理](../chapter-08-response-cors/section-02-cors.md)**：了解跨域处理的详细内容

## 练习任务

1. **实现 OAuth2 授权服务器**
   - 实现授权端点
   - 实现令牌端点
   - 实现授权码管理

2. **实现 OAuth2 资源服务器**
   - 验证访问令牌
   - 提供资源访问
   - 处理权限检查

3. **实现 OAuth2 客户端**
   - 实现授权流程
   - 实现令牌交换
   - 实现资源访问

4. **实现第三方登录**
   - 集成 Google OAuth2
   - 集成 GitHub OAuth2
   - 处理回调

5. **实现完整的 OAuth2 系统**
   - 授权服务器
   - 资源服务器
   - 客户端实现

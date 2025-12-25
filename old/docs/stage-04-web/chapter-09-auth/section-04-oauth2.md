# 4.9.4 OAuth2 实现

## 概述

OAuth2 是授权框架，支持第三方登录和 API 访问授权。本节详细介绍 OAuth2 流程、授权码模式、客户端凭证模式、第三方登录实现，以及安全最佳实践。

## OAuth2 流程

### 核心角色

- **资源所有者（Resource Owner）**：用户
- **客户端（Client）**：第三方应用
- **授权服务器（Authorization Server）**：提供授权
- **资源服务器（Resource Server）**：提供资源

### 授权码模式（Authorization Code）

```
1. 用户访问客户端
   ↓
2. 客户端重定向到授权服务器
   ↓
3. 用户授权
   ↓
4. 授权服务器返回授权码
   ↓
5. 客户端用授权码换取 Access Token
   ↓
6. 使用 Access Token 访问资源
```

## 授权码模式实现

### 授权端点

```php
<?php
declare(strict_types=1);

class OAuth2Server
{
    public function authorize(array $params): void
    {
        // 验证客户端
        $client = $this->validateClient($params['client_id']);
        
        // 生成授权码
        $code = $this->generateAuthorizationCode($client['id'], $params['user_id']);
        
        // 重定向到回调地址
        $redirectUri = $params['redirect_uri'] . '?code=' . $code;
        header("Location: {$redirectUri}");
        exit;
    }

    public function token(array $params): array
    {
        // 验证授权码
        $codeData = $this->validateAuthorizationCode($params['code']);
        
        // 生成 Access Token
        $accessToken = $this->generateAccessToken($codeData['client_id'], $codeData['user_id']);
        
        return [
            'access_token' => $accessToken,
            'token_type' => 'Bearer',
            'expires_in' => 3600,
        ];
    }
}
```

## 客户端凭证模式

### 实现

```php
<?php
declare(strict_types=1);

class ClientCredentialsGrant
{
    public function getToken(string $clientId, string $clientSecret): ?array
    {
        // 验证客户端凭证
        if (!$this->validateClient($clientId, $clientSecret)) {
            return null;
        }
        
        // 生成 Token
        $token = $this->generateToken($clientId);
        
        return [
            'access_token' => $token,
            'token_type' => 'Bearer',
            'expires_in' => 3600,
        ];
    }
}
```

## 第三方登录

### 实现示例

```php
<?php
declare(strict_types=1);

class OAuth2Login
{
    public function redirectToProvider(string $provider): void
    {
        $authUrl = match ($provider) {
            'github' => $this->getGithubAuthUrl(),
            'google' => $this->getGoogleAuthUrl(),
            default => throw new InvalidArgumentException('Unsupported provider'),
        };
        
        header("Location: {$authUrl}");
        exit;
    }

    public function handleCallback(string $provider, string $code): array
    {
        // 用授权码换取 Token
        $token = $this->exchangeCodeForToken($provider, $code);
        
        // 获取用户信息
        $userInfo = $this->getUserInfo($provider, $token);
        
        // 创建或更新本地用户
        $user = $this->createOrUpdateUser($provider, $userInfo);
        
        return $user;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class OAuth2Service
{
    public function handleAuthorizationRequest(): void
    {
        $clientId = $_GET['client_id'] ?? '';
        $redirectUri = $_GET['redirect_uri'] ?? '';
        $state = $_GET['state'] ?? '';
        
        // 验证客户端
        $client = $this->validateClient($clientId, $redirectUri);
        
        // 生成授权码
        $code = $this->generateCode($client['id']);
        
        // 重定向
        $url = $redirectUri . '?code=' . $code . '&state=' . $state;
        header("Location: {$url}");
        exit;
    }
}
```

## 注意事项

1. **安全性**：使用 HTTPS，验证重定向 URI
2. **状态参数**：使用 state 参数防止 CSRF
3. **Token 安全**：安全存储和传输 Token
4. **过期管理**：合理设置 Token 过期时间

## 练习

1. 实现一个简单的 OAuth2 授权服务器，支持授权码模式。

2. 创建一个第三方登录系统，支持 GitHub、Google 登录。

3. 实现 OAuth2 客户端，使用授权码获取 Token。

4. 设计一个 OAuth2 Token 管理系统，支持 Token 刷新和撤销。

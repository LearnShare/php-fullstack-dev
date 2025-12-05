# 7.3.4 认证与授权

## 概述

Laravel 提供了完整的认证和授权系统。本节介绍认证系统、授权系统、多因素认证、API 认证等内容。

## 认证系统

### 用户认证

```php
<?php
declare(strict_types=1);

use Illuminate\Support\Facades\Auth;

// 登录
if (Auth::attempt(['email' => $email, 'password' => $password])) {
    // 认证成功
    return redirect()->intended('/dashboard');
}

// 手动登录
Auth::login($user);

// 登出
Auth::logout();

// 检查认证
if (Auth::check()) {
    $user = Auth::user();
}
```

### 认证中间件

```php
<?php
declare(strict_types=1);

// routes/web.php
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});

// 控制器中使用
class DashboardController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }
}
```

## 授权系统

### 策略（Policy）

```php
<?php
declare(strict_types=1);

namespace App\Policies;

use App\Models\User;
use App\Models\Post;

class PostPolicy
{
    public function viewAny(User $user): bool
    {
        return true;
    }
    
    public function view(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
    
    public function create(User $user): bool
    {
        return $user->isAdmin();
    }
    
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
    
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }
}

// 使用
if ($user->can('view', $post)) {
    // 可以查看
}

// 在控制器中
$this->authorize('update', $post);
```

### 门面（Gate）

```php
<?php
declare(strict_types=1);

use Illuminate\Support\Facades\Gate;

// 定义 Gate
Gate::define('update-post', function (User $user, Post $post) {
    return $user->id === $post->user_id;
});

// 使用
if (Gate::allows('update-post', $post)) {
    // 允许更新
}

if (Gate::denies('update-post', $post)) {
    // 拒绝更新
}
```

## 多因素认证

### TOTP 认证

```php
<?php
declare(strict_types=1);

use PragmaRX\Google2FA\Google2FA;

class TwoFactorAuth
{
    private Google2FA $google2fa;
    
    public function __construct()
    {
        $this->google2fa = new Google2FA();
    }
    
    public function generateSecret(): string
    {
        return $this->google2fa->generateSecretKey();
    }
    
    public function generateQRCode(string $email, string $secret): string
    {
        return $this->google2fa->getQRCodeUrl(
            config('app.name'),
            $email,
            $secret
        );
    }
    
    public function verify(string $secret, string $code): bool
    {
        return $this->google2fa->verifyKey($secret, $code);
    }
}
```

## API 认证

### Sanctum

```php
<?php
declare(strict_types=1);

// 生成 token
$token = $user->createToken('api-token')->plainTextToken;

// 使用 token
// Authorization: Bearer {token}

// 撤销 token
$user->tokens()->delete();
```

### Passport

```php
<?php
declare(strict_types=1);

// 安装 Passport
php artisan passport:install

// 创建 token
$token = $user->createToken('My Token', ['read', 'write'])->accessToken;

// 使用 token
// Authorization: Bearer {token}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);
        
        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();
            return redirect()->intended('/dashboard');
        }
        
        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ]);
    }
    
    public function logout(Request $request)
    {
        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();
        return redirect('/');
    }
}

class PostController extends Controller
{
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);
        
        $post->update($request->validated());
        return response()->json($post);
    }
}
```

## 最佳实践

1. **使用策略**：复杂授权逻辑使用策略
2. **API 认证**：API 使用 token 认证
3. **会话安全**：注意会话安全配置
4. **多因素认证**：敏感操作使用多因素认证

## 注意事项

1. 密码应该哈希存储
2. 使用 HTTPS 传输认证信息
3. 实现登录限制，防止暴力破解
4. 定期更新 token

## 练习

1. 实现用户登录和登出功能。

2. 创建授权策略，控制资源访问。

3. 实现多因素认证。

4. 配置 API 认证，使用 Sanctum 或 Passport。

# 5.15.1 请求验证概述

## 概述

请求验证是确保数据安全和完整性的重要环节。理解验证的概念、掌握验证时机、设计验证规则，对于构建安全、可靠的 Web 应用至关重要。本节详细介绍验证概念、验证的重要性、验证时机、验证规则类型、验证错误处理等内容，帮助零基础学员理解数据验证的重要性。

数据验证是 Web 应用安全的第一道防线，可以有效防止恶意数据、保证数据完整性、提升用户体验。理解验证的工作原理对于正确实现验证系统至关重要。

**主要内容**：
- 验证概念（什么是验证、验证的目的、验证的类型）
- 验证的重要性（数据安全、数据完整性、用户体验、系统稳定性）
- 验证时机（客户端验证、服务器端验证、双重验证、验证顺序）
- 验证规则类型（必填验证、类型验证、格式验证、范围验证、自定义验证）
- 验证错误处理（错误收集、错误格式化、错误返回、错误提示）
- 实际应用示例和最佳实践

## 特性

- **多层验证**：支持客户端和服务器端验证
- **灵活规则**：支持多种验证规则
- **清晰错误**：提供清晰的错误信息
- **易于扩展**：易于扩展新规则
- **性能优化**：高效的验证机制

## 验证概念

### 什么是验证

验证（Validation）是检查数据是否符合预期规则的过程，确保数据的正确性、完整性和安全性。

### 验证的目的

1. **数据安全**：防止恶意数据输入
2. **数据完整性**：确保数据完整和正确
3. **用户体验**：提供及时的错误反馈
4. **系统稳定**：防止系统错误和崩溃

### 验证的类型

**主要类型**：
- **输入验证**：验证用户输入
- **格式验证**：验证数据格式
- **业务验证**：验证业务规则
- **安全验证**：验证安全性

## 验证的重要性

### 数据安全

**示例**：
```php
<?php
declare(strict_types=1);

// 验证邮箱格式，防止注入攻击
function validateEmail(string $email): bool
{
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}

// 验证整数，防止 SQL 注入
function validateInteger(mixed $value): ?int
{
    return filter_var($value, FILTER_VALIDATE_INT);
}
```

### 数据完整性

**示例**：
```php
<?php
declare(strict_types=1);

function validateUserData(array $data): array
{
    $errors = [];
    
    // 必填字段验证
    if (empty($data['name'])) {
        $errors['name'] = '姓名是必填项';
    }
    
    if (empty($data['email'])) {
        $errors['email'] = '邮箱是必填项';
    }
    
    // 格式验证
    if (!empty($data['email']) && !filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = '邮箱格式不正确';
    }
    
    return $errors;
}
```

### 用户体验

**示例**：
```php
<?php
declare(strict_types=1);

function validateAndReturnErrors(array $data): array
{
    $errors = [];
    
    // 实时验证，提供即时反馈
    if (empty($data['password'])) {
        $errors['password'] = '密码不能为空';
    } elseif (strlen($data['password']) < 8) {
        $errors['password'] = '密码长度至少 8 位';
    }
    
    return $errors;
}
```

### 系统稳定性

**示例**：
```php
<?php
declare(strict_types=1);

function validateBeforeProcess(array $data): void
{
    $errors = validateUserData($data);
    
    if (!empty($errors)) {
        throw new ValidationException('数据验证失败', $errors);
    }
    
    // 验证通过后处理数据
    processData($data);
}
```

## 验证时机

### 客户端验证

**示例**：
```html
<!-- HTML5 验证 -->
<form>
    <input type="email" name="email" required>
    <input type="password" name="password" minlength="8" required>
    <button type="submit">提交</button>
</form>
```

```javascript
// JavaScript 验证
function validateForm() {
    const email = document.getElementById('email').value;
    if (!email.includes('@')) {
        alert('邮箱格式不正确');
        return false;
    }
    return true;
}
```

### 服务器端验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateOnServer(array $data): array
{
    $errors = [];
    
    // 服务器端验证（必需）
    if (empty($data['email'])) {
        $errors['email'] = '邮箱是必填项';
    } elseif (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = '邮箱格式不正确';
    }
    
    return $errors;
}
```

### 双重验证

**原则**：客户端验证提升用户体验，服务器端验证保证安全性。

**示例**：
```php
<?php
declare(strict_types=1);

// 客户端验证（JavaScript）+ 服务器端验证（PHP）
// 客户端：即时反馈，提升用户体验
// 服务器端：安全保证，防止绕过
```

### 验证顺序

**推荐顺序**：
1. **客户端验证**：快速反馈
2. **服务器端验证**：安全保证
3. **业务验证**：业务规则检查
4. **数据库约束**：最后防线

## 验证规则类型

### 必填验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateRequired(array $data, array $requiredFields): array
{
    $errors = [];
    
    foreach ($requiredFields as $field) {
        if (!isset($data[$field]) || $data[$field] === '') {
            $errors[$field] = "{$field} 是必填项";
        }
    }
    
    return $errors;
}

// 使用
$errors = validateRequired($_POST, ['name', 'email', 'password']);
```

### 类型验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateType(mixed $value, string $type): bool
{
    return match ($type) {
        'int' => filter_var($value, FILTER_VALIDATE_INT) !== false,
        'float' => filter_var($value, FILTER_VALIDATE_FLOAT) !== false,
        'string' => is_string($value),
        'email' => filter_var($value, FILTER_VALIDATE_EMAIL) !== false,
        'url' => filter_var($value, FILTER_VALIDATE_URL) !== false,
        'boolean' => filter_var($value, FILTER_VALIDATE_BOOLEAN) !== false,
        default => false,
    };
}
```

### 格式验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateFormat(string $value, string $pattern): bool
{
    return (bool) preg_match($pattern, $value);
}

// 使用
$isValidPhone = validateFormat($phone, '/^1[3-9]\d{9}$/');  // 手机号
$isValidSlug = validateFormat($slug, '/^[a-z0-9-]+$/');      // Slug
```

### 范围验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateRange(mixed $value, int $min, int $max): bool
{
    $num = filter_var($value, FILTER_VALIDATE_INT);
    if ($num === false) {
        return false;
    }
    return $num >= $min && $num <= $max;
}

// 使用
$isValidAge = validateRange($age, 1, 120);
$isValidScore = validateRange($score, 0, 100);
```

### 自定义验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateCustom(mixed $value, callable $validator): bool
{
    return $validator($value);
}

// 使用
$isValid = validateCustom($password, function($pwd) {
    return strlen($pwd) >= 8 && 
           preg_match('/[A-Z]/', $pwd) && 
           preg_match('/[a-z]/', $pwd) && 
           preg_match('/[0-9]/', $pwd);
});
```

## 验证错误处理

### 错误收集

**示例**：
```php
<?php
declare(strict_types=1);

class Validator
{
    private array $errors = [];

    public function validate(array $data, array $rules): bool
    {
        $this->errors = [];
        
        foreach ($rules as $field => $fieldRules) {
            $value = $data[$field] ?? null;
            
            foreach ($fieldRules as $rule) {
                if (!$this->checkRule($value, $rule)) {
                    $this->errors[$field][] = $this->getErrorMessage($field, $rule);
                }
            }
        }
        
        return empty($this->errors);
    }

    public function getErrors(): array
    {
        return $this->errors;
    }

    private function checkRule(mixed $value, string $rule): bool
    {
        // 验证规则检查逻辑
        return true;
    }

    private function getErrorMessage(string $field, string $rule): string
    {
        // 生成错误消息
        return "{$field} 验证失败: {$rule}";
    }
}
```

### 错误格式化

**示例**：
```php
<?php
declare(strict_types=1);

function formatErrors(array $errors): array
{
    $formatted = [];
    
    foreach ($errors as $field => $fieldErrors) {
        $formatted[$field] = [
            'field' => $field,
            'errors' => $fieldErrors,
            'first' => $fieldErrors[0] ?? null,
        ];
    }
    
    return $formatted;
}
```

### 错误返回

**示例**：
```php
<?php
declare(strict_types=1);

function validateAndRespond(array $data, array $rules): void
{
    $validator = new Validator();
    
    if (!$validator->validate($data, $rules)) {
        http_response_code(422);  // Unprocessable Entity
        header('Content-Type: application/json');
        echo json_encode([
            'success' => false,
            'errors' => $validator->getErrors(),
        ]);
        exit;
    }
}
```

### 错误提示

**示例**：
```php
<?php
declare(strict_types=1);

function getErrorMessage(string $field, string $rule, array $params = []): string
{
    $messages = [
        'required' => "{$field} 是必填项",
        'email' => "{$field} 必须是有效的邮箱地址",
        'min' => "{$field} 最小长度为 {$params['min']}",
        'max' => "{$field} 最大长度为 {$params['max']}",
        'numeric' => "{$field} 必须是数字",
    ];
    
    return $messages[$rule] ?? "{$field} 验证失败";
}
```

## 使用场景

### 表单提交

- 用户注册表单
- 用户登录表单
- 数据编辑表单
- 搜索表单

### API 请求

- RESTful API 请求
- GraphQL 请求
- Webhook 请求
- 第三方 API 调用

### 数据导入

- CSV 数据导入
- Excel 数据导入
- JSON 数据导入
- 批量数据处理

### 用户输入

- 搜索输入
- 评论输入
- 文件上传
- 配置输入

## 注意事项

### 服务器端验证是必需的

- **重要**：客户端验证可以被绕过
- **安全保证**：服务器端验证是安全保证
- **双重验证**：客户端 + 服务器端

### 验证规则的完整性

- **全面验证**：验证所有必要字段
- **格式验证**：验证数据格式
- **业务验证**：验证业务规则

### 错误信息的清晰性

- **清晰明确**：错误信息要清晰明确
- **用户友好**：使用用户友好的语言
- **可操作**：提供可操作的建议

### 性能考虑

- **及时验证**：及时验证，快速反馈
- **批量验证**：批量验证提高效率
- **缓存验证**：缓存验证结果

## 常见问题

### 为什么需要验证？

1. **数据安全**：防止恶意数据
2. **数据完整性**：确保数据正确
3. **用户体验**：提供及时反馈
4. **系统稳定**：防止系统错误

### 何时进行验证？

1. **客户端验证**：提交前验证
2. **服务器端验证**：接收数据后立即验证
3. **业务验证**：处理数据前验证
4. **数据库约束**：最后防线

### 如何设计验证规则？

1. **必填规则**：定义必填字段
2. **类型规则**：定义数据类型
3. **格式规则**：定义数据格式
4. **业务规则**：定义业务逻辑

### 如何提供错误信息？

1. **清晰明确**：错误信息要清晰
2. **用户友好**：使用友好语言
3. **可操作**：提供操作建议
4. **统一格式**：统一错误格式

## 最佳实践

### 始终进行服务器端验证

- 服务器端验证是必需的
- 客户端验证只是辅助
- 双重验证提供最佳体验

### 提供清晰的错误信息

- 使用用户友好的语言
- 提供具体的错误原因
- 给出可操作的建议

### 使用验证库简化验证

- 使用成熟的验证库
- 减少重复代码
- 提高验证效率

### 实现统一的验证机制

- 统一的验证接口
- 统一的错误格式
- 统一的验证流程

## 相关章节

- **[5.15.2 Symfony Validator](section-02-symfony-validator.md)**：了解 Symfony Validator 的详细内容
- **[5.15.3 Respect/Validation](section-03-respect-validation.md)**：了解 Respect/Validation 的详细内容
- **[5.15.4 自定义验证规则](section-04-custom-rules.md)**：了解自定义验证规则的详细内容
- **[5.3.1 $_GET、$_POST 与 $_REQUEST](../chapter-03-superglobals/section-01-get-post-request.md)**：了解请求数据的获取

## 练习任务

1. **实现基本验证函数**
   - 必填验证
   - 类型验证
   - 格式验证

2. **实现验证类**
   - 验证规则定义
   - 错误收集
   - 错误格式化

3. **实现表单验证**
   - 表单数据验证
   - 错误显示
   - 用户反馈

4. **实现 API 验证**
   - API 请求验证
   - 错误响应
   - 验证中间件

5. **实现完整的验证系统**
   - 多种验证规则
   - 错误处理
   - 验证库集成

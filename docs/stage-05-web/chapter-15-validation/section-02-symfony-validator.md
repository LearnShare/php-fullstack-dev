# 5.15.2 Symfony Validator

## 概述

Symfony Validator 是功能强大的验证组件，提供了丰富的验证规则和灵活的验证机制。理解 Symfony Validator 的使用方法、掌握验证规则定义、实现验证执行和错误处理，对于构建健壮的验证系统至关重要。本节详细介绍 Symfony Validator 概述、安装配置、验证规则、验证执行、错误处理等内容，帮助零基础学员掌握 Symfony Validator 的使用。

Symfony Validator 是 Symfony 框架的验证组件，也可以独立使用。它提供了丰富的验证规则、清晰的错误信息、灵活的验证机制，是 PHP 生态中最流行的验证库之一。

**主要内容**：
- Symfony Validator 概述（Validator 的功能、Validator 的优势、使用场景）
- 安装配置（Composer 安装、基本配置、验证器创建）
- 验证规则（约束定义、内置约束、约束组合、自定义约束）
- 验证执行（验证方法、验证结果、错误处理、批量验证）
- 错误处理（错误获取、错误格式化、错误消息、错误翻译）
- 实际应用示例和最佳实践

## 特性

- **丰富规则**：提供丰富的内置验证规则
- **灵活配置**：支持灵活的验证配置
- **清晰错误**：提供清晰的错误信息
- **易于扩展**：易于扩展自定义规则
- **高性能**：高效的验证机制

## Symfony Validator 概述

### Validator 的功能

1. **数据验证**：验证各种数据类型
2. **约束定义**：灵活的约束定义方式
3. **错误处理**：清晰的错误信息
4. **国际化**：支持错误消息翻译

### Validator 的优势

1. **功能完整**：功能完整且强大
2. **易于使用**：简洁的 API
3. **可扩展**：易于扩展
4. **标准化**：遵循标准规范
5. **文档完善**：文档完善

### 使用场景

1. **表单验证**：表单数据验证
2. **API 验证**：API 请求验证
3. **数据导入**：数据导入验证
4. **业务验证**：业务规则验证

## 安装配置

### Composer 安装

**安装命令**：
```bash
composer require symfony/validator
```

### 基本配置

**示例**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Symfony\Component\Validator\Validation;
use Symfony\Component\Validator\Validator\ValidatorInterface;

$validator = Validation::createValidator();
```

### 验证器创建

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Validation;
use Symfony\Component\Validator\Validator\ValidatorInterface;

class ValidatorFactory
{
    public static function create(): ValidatorInterface
    {
        return Validation::createValidator();
    }
}

$validator = ValidatorFactory::create();
```

## 验证规则

### 约束定义

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Constraints as Assert;

class User
{
    #[Assert\NotBlank(message: '姓名不能为空')]
    #[Assert\Length(min: 2, max: 50, minMessage: '姓名至少 2 个字符', maxMessage: '姓名最多 50 个字符')]
    public string $name;

    #[Assert\NotBlank(message: '邮箱不能为空')]
    #[Assert\Email(message: '邮箱格式不正确')]
    public string $email;

    #[Assert\NotBlank(message: '密码不能为空')]
    #[Assert\Length(min: 8, minMessage: '密码至少 8 个字符')]
    public string $password;
}
```

### 内置约束

**常用约束**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Constraints as Assert;

// 非空约束
#[Assert\NotBlank]
#[Assert\NotNull]

// 类型约束
#[Assert\Type('string')]
#[Assert\Type('int')]

// 长度约束
#[Assert\Length(min: 1, max: 100)]

// 范围约束
#[Assert\Range(min: 1, max: 100)]

// 格式约束
#[Assert\Email]
#[Assert\Url]
#[Assert\Regex('/^[a-z0-9-]+$/')]

// 比较约束
#[Assert\EqualTo('value')]
#[Assert\NotEqualTo('value')]
#[Assert\GreaterThan(0)]
#[Assert\LessThan(100)]
```

### 约束组合

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Constraints as Assert;

class Product
{
    #[Assert\NotBlank]
    #[Assert\Length(min: 1, max: 200)]
    public string $name;

    #[Assert\NotBlank]
    #[Assert\Type('float')]
    #[Assert\Range(min: 0.01, max: 999999.99)]
    public float $price;

    #[Assert\NotBlank]
    #[Assert\Choice(['active', 'inactive', 'pending'])]
    public string $status;
}
```

### 自定义约束

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Constraint;
use Symfony\Component\Validator\ConstraintValidator;

#[\Attribute]
class StrongPassword extends Constraint
{
    public string $message = '密码强度不足';
}

class StrongPasswordValidator extends ConstraintValidator
{
    public function validate($value, Constraint $constraint): void
    {
        if ($value === null || $value === '') {
            return;
        }

        $hasUpper = preg_match('/[A-Z]/', $value);
        $hasLower = preg_match('/[a-z]/', $value);
        $hasNumber = preg_match('/[0-9]/', $value);
        $hasSpecial = preg_match('/[^A-Za-z0-9]/', $value);

        if (!$hasUpper || !$hasLower || !$hasNumber || !$hasSpecial) {
            $this->context->buildViolation($constraint->message)
                ->addViolation();
        }
    }
}

// 使用
class User
{
    #[StrongPassword]
    public string $password;
}
```

## 验证执行

### 验证方法

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Validation;
use Symfony\Component\Validator\Validator\ValidatorInterface;

$validator = Validation::createValidator();

$user = new User();
$user->name = 'John';
$user->email = 'john@example.com';
$user->password = 'password123';

$violations = $validator->validate($user);
```

### 验证结果

**示例**：
```php
<?php
declare(strict_types=1);

$violations = $validator->validate($user);

if (count($violations) > 0) {
    foreach ($violations as $violation) {
        echo $violation->getPropertyPath() . ': ' . $violation->getMessage() . "\n";
    }
} else {
    echo "验证通过\n";
}
```

### 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

function validateUser(User $user): array
{
    $validator = Validation::createValidator();
    $violations = $validator->validate($user);
    
    $errors = [];
    foreach ($violations as $violation) {
        $field = $violation->getPropertyPath();
        $errors[$field][] = $violation->getMessage();
    }
    
    return $errors;
}
```

### 批量验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateUsers(array $users): array
{
    $validator = Validation::createValidator();
    $allErrors = [];
    
    foreach ($users as $index => $user) {
        $violations = $validator->validate($user);
        
        if (count($violations) > 0) {
            $allErrors[$index] = [];
            foreach ($violations as $violation) {
                $field = $violation->getPropertyPath();
                $allErrors[$index][$field][] = $violation->getMessage();
            }
        }
    }
    
    return $allErrors;
}
```

## 错误处理

### 错误获取

**示例**：
```php
<?php
declare(strict_types=1);

$violations = $validator->validate($user);

foreach ($violations as $violation) {
    $propertyPath = $violation->getPropertyPath();  // 字段路径
    $message = $violation->getMessage();            // 错误消息
    $code = $violation->getCode();                  // 错误代码
    $invalidValue = $violation->getInvalidValue();  // 无效值
}
```

### 错误格式化

**示例**：
```php
<?php
declare(strict_types=1);

function formatViolations($violations): array
{
    $errors = [];
    
    foreach ($violations as $violation) {
        $field = $violation->getPropertyPath();
        $errors[$field] = [
            'message' => $violation->getMessage(),
            'code' => $violation->getCode(),
            'value' => $violation->getInvalidValue(),
        ];
    }
    
    return $errors;
}
```

### 错误消息

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Constraints as Assert;

class User
{
    #[Assert\NotBlank(message: '姓名不能为空')]
    public string $name;

    #[Assert\Email(message: '邮箱格式不正确')]
    public string $email;

    #[Assert\Length(
        min: 8,
        minMessage: '密码至少 {{ limit }} 个字符',
        max: 100,
        maxMessage: '密码最多 {{ limit }} 个字符'
    )]
    public string $password;
}
```

### 错误翻译

**示例**：
```php
<?php
declare(strict_types=1);

use Symfony\Component\Validator\Validation;
use Symfony\Component\Translation\Translator;
use Symfony\Component\Translation\Loader\XliffFileLoader;

$translator = new Translator('zh_CN');
$translator->addLoader('xlf', new XliffFileLoader());
$translator->addResource('xlf', 'path/to/translations/validators.zh_CN.xlf', 'zh_CN', 'validators');

$validator = Validation::createValidatorBuilder()
    ->setTranslator($translator)
    ->setTranslationDomain('validators')
    ->getValidator();
```

## 使用场景

### 表单验证

- 用户注册表单
- 用户登录表单
- 数据编辑表单
- 搜索表单

### API 验证

- RESTful API 请求
- GraphQL 请求
- Webhook 请求
- 第三方 API 调用

### 数据导入

- CSV 数据导入
- Excel 数据导入
- JSON 数据导入
- 批量数据处理

### 业务验证

- 业务规则验证
- 数据完整性验证
- 数据一致性验证

## 注意事项

### 约束定义

- **清晰明确**：约束定义要清晰明确
- **完整覆盖**：覆盖所有需要验证的字段
- **合理约束**：设置合理的约束条件

### 错误处理

- **及时处理**：及时处理验证错误
- **清晰错误**：提供清晰的错误信息
- **用户友好**：使用用户友好的语言

### 性能考虑

- **批量验证**：批量验证提高效率
- **缓存约束**：缓存约束定义
- **优化验证**：优化验证逻辑

### 国际化

- **错误翻译**：支持错误消息翻译
- **多语言支持**：支持多语言
- **本地化**：本地化错误消息

## 常见问题

### 如何安装 Symfony Validator？

使用 Composer 安装：
```bash
composer require symfony/validator
```

### 如何定义验证规则？

使用属性（Attributes）或注解（Annotations）定义约束：
```php
#[Assert\NotBlank]
#[Assert\Email]
public string $email;
```

### 如何执行验证？

使用 `validate()` 方法：
```php
$violations = $validator->validate($object);
```

### 如何处理验证错误？

遍历 violations 获取错误信息：
```php
foreach ($violations as $violation) {
    echo $violation->getMessage();
}
```

## 最佳实践

### 使用属性定义约束

- 使用 PHP 8 属性定义约束
- 清晰直观
- 易于维护

### 提供清晰的错误消息

- 使用自定义错误消息
- 使用参数化消息
- 支持国际化

### 实现统一的验证接口

- 统一的验证方法
- 统一的错误格式
- 统一的错误处理

### 使用验证组

- 使用验证组组织约束
- 支持不同场景的验证
- 提高验证灵活性

## 相关章节

- **[5.15.1 请求验证概述](section-01-overview.md)**：了解请求验证概述的详细内容
- **[5.15.3 Respect/Validation](section-03-respect-validation.md)**：了解 Respect/Validation 的详细内容
- **[5.15.4 自定义验证规则](section-04-custom-rules.md)**：了解自定义验证规则的详细内容

## 练习任务

1. **安装和配置 Symfony Validator**
   - Composer 安装
   - 基本配置
   - 验证器创建

2. **实现基本验证**
   - 定义约束
   - 执行验证
   - 处理错误

3. **实现复杂验证**
   - 组合约束
   - 自定义约束
   - 验证组

4. **实现错误处理**
   - 错误格式化
   - 错误翻译
   - 错误响应

5. **实现完整的验证系统**
   - 多种验证规则
   - 错误处理
   - 验证中间件

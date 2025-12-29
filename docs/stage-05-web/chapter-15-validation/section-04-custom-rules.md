# 5.15.4 自定义验证规则

## 概述

自定义验证规则是满足特定业务需求的重要方式。理解自定义规则的编写方法、掌握规则注册和复用、实现规则测试，对于构建灵活的验证系统至关重要。本节详细介绍自定义规则概述、规则编写、规则注册、规则复用、规则测试等内容，帮助零基础学员掌握自定义验证规则的开发。

自定义验证规则可以满足特定的业务需求，提供灵活的验证机制。掌握自定义规则的编写方法对于构建完整的验证系统至关重要。

**主要内容**：
- 自定义规则概述（为什么需要自定义规则、自定义规则的优势、使用场景）
- 规则编写（规则接口、规则实现、规则逻辑、规则消息）
- 规则注册（规则注册方法、规则命名、规则配置、规则管理）
- 规则复用（规则封装、规则组合、规则继承、规则库）
- 规则测试（单元测试、集成测试、测试覆盖）
- 实际应用示例和最佳实践

## 特性

- **灵活定制**：满足特定业务需求
- **易于扩展**：易于扩展新规则
- **可重用**：可在多个场景中重用
- **可测试**：易于单元测试
- **标准化**：遵循标准接口

## 自定义规则概述

### 为什么需要自定义规则

1. **业务需求**：满足特定业务需求
2. **复杂验证**：实现复杂验证逻辑
3. **规则复用**：提高规则复用性
4. **代码组织**：更好的代码组织

### 自定义规则的优势

1. **灵活性**：灵活的验证逻辑
2. **可维护**：易于维护和扩展
3. **可测试**：易于单元测试
4. **可重用**：可在多个场景中重用

### 使用场景

1. **业务规则**：业务逻辑验证
2. **复杂验证**：复杂验证逻辑
3. **特定格式**：特定格式验证
4. **数据一致性**：数据一致性验证

## 规则编写

### 规则接口

**示例**：
```php
<?php
declare(strict_types=1);

interface ValidationRuleInterface
{
    public function validate($value): bool;
    public function getErrorMessage(): string;
}
```

### 规则实现

**示例**：
```php
<?php
declare(strict_types=1);

class StrongPasswordRule implements ValidationRuleInterface
{
    public function validate($value): bool
    {
        if (!is_string($value)) {
            return false;
        }

        $hasUpper = preg_match('/[A-Z]/', $value);
        $hasLower = preg_match('/[a-z]/', $value);
        $hasNumber = preg_match('/[0-9]/', $value);
        $hasSpecial = preg_match('/[^A-Za-z0-9]/', $value);

        return strlen($value) >= 8 && $hasUpper && $hasLower && $hasNumber && $hasSpecial;
    }

    public function getErrorMessage(): string
    {
        return '密码必须至少 8 个字符，包含大写字母、小写字母、数字和特殊字符';
    }
}
```

### 规则逻辑

**示例**：
```php
<?php
declare(strict_types=1);

class PhoneNumberRule implements ValidationRuleInterface
{
    private string $country;

    public function __construct(string $country = 'CN')
    {
        $this->country = $country;
    }

    public function validate($value): bool
    {
        if (!is_string($value)) {
            return false;
        }

        return match ($this->country) {
            'CN' => preg_match('/^1[3-9]\d{9}$/', $value),
            'US' => preg_match('/^\+1\d{10}$/', $value),
            default => false,
        };
    }

    public function getErrorMessage(): string
    {
        return "手机号格式不正确（{$this->country}）";
    }
}
```

### 规则消息

**示例**：
```php
<?php
declare(strict_types=1);

class CustomRule implements ValidationRuleInterface
{
    private string $fieldName;

    public function __construct(string $fieldName = '字段')
    {
        $this->fieldName = $fieldName;
    }

    public function validate($value): bool
    {
        // 验证逻辑
        return true;
    }

    public function getErrorMessage(): string
    {
        return "{$this->fieldName} 验证失败";
    }
}
```

## 规则注册

### 规则注册方法

**示例**：
```php
<?php
declare(strict_types=1);

class RuleRegistry
{
    private array $rules = [];

    public function register(string $name, ValidationRuleInterface $rule): void
    {
        $this->rules[$name] = $rule;
    }

    public function get(string $name): ?ValidationRuleInterface
    {
        return $this->rules[$name] ?? null;
    }

    public function has(string $name): bool
    {
        return isset($this->rules[$name]);
    }
}

// 使用
$registry = new RuleRegistry();
$registry->register('strongPassword', new StrongPasswordRule());
$registry->register('phoneCN', new PhoneNumberRule('CN'));
```

### 规则命名

**命名规范**：
- 使用描述性名称
- 使用驼峰命名
- 避免缩写

**示例**：
```php
<?php
declare(strict_types=1);

$registry->register('strongPassword', new StrongPasswordRule());
$registry->register('phoneNumberCN', new PhoneNumberRule('CN'));
$registry->register('emailDomain', new EmailDomainRule());
```

### 规则配置

**示例**：
```php
<?php
declare(strict_types=1);

class ConfigurableRule implements ValidationRuleInterface
{
    private array $config;

    public function __construct(array $config = [])
    {
        $this->config = array_merge([
            'minLength' => 8,
            'requireUpper' => true,
            'requireLower' => true,
            'requireNumber' => true,
            'requireSpecial' => true,
        ], $config);
    }

    public function validate($value): bool
    {
        if (!is_string($value)) {
            return false;
        }

        if (strlen($value) < $this->config['minLength']) {
            return false;
        }

        if ($this->config['requireUpper'] && !preg_match('/[A-Z]/', $value)) {
            return false;
        }

        if ($this->config['requireLower'] && !preg_match('/[a-z]/', $value)) {
            return false;
        }

        if ($this->config['requireNumber'] && !preg_match('/[0-9]/', $value)) {
            return false;
        }

        if ($this->config['requireSpecial'] && !preg_match('/[^A-Za-z0-9]/', $value)) {
            return false;
        }

        return true;
    }

    public function getErrorMessage(): string
    {
        return '密码不符合要求';
    }
}
```

### 规则管理

**示例**：
```php
<?php
declare(strict_types=1);

class RuleManager
{
    private RuleRegistry $registry;

    public function __construct()
    {
        $this->registry = new RuleRegistry();
        $this->registerDefaultRules();
    }

    private function registerDefaultRules(): void
    {
        $this->registry->register('strongPassword', new StrongPasswordRule());
        $this->registry->register('phoneCN', new PhoneNumberRule('CN'));
    }

    public function validate(string $ruleName, $value): bool
    {
        $rule = $this->registry->get($ruleName);
        if ($rule === null) {
            throw new \InvalidArgumentException("规则 {$ruleName} 不存在");
        }
        return $rule->validate($value);
    }
}
```

## 规则复用

### 规则封装

**示例**：
```php
<?php
declare(strict_types=1);

class UserValidationRules
{
    public static function email(): ValidationRuleInterface
    {
        return new EmailRule();
    }

    public static function password(): ValidationRuleInterface
    {
        return new StrongPasswordRule();
    }

    public static function phone(): ValidationRuleInterface
    {
        return new PhoneNumberRule('CN');
    }

    public static function name(): ValidationRuleInterface
    {
        return new NameRule();
    }
}

// 使用
$emailRule = UserValidationRules::email();
$passwordRule = UserValidationRules::password();
```

### 规则组合

**示例**：
```php
<?php
declare(strict_types=1);

class CompositeRule implements ValidationRuleInterface
{
    private array $rules;

    public function __construct(array $rules)
    {
        $this->rules = $rules;
    }

    public function validate($value): bool
    {
        foreach ($this->rules as $rule) {
            if (!$rule->validate($value)) {
                return false;
            }
        }
        return true;
    }

    public function getErrorMessage(): string
    {
        $messages = [];
        foreach ($this->rules as $rule) {
            $messages[] = $rule->getErrorMessage();
        }
        return implode('; ', $messages);
    }
}

// 使用
$rule = new CompositeRule([
    new NotEmptyRule(),
    new StringRule(),
    new LengthRule(8, 100),
]);
```

### 规则继承

**示例**：
```php
<?php
declare(strict_types=1);

abstract class BaseRule implements ValidationRuleInterface
{
    protected string $fieldName;

    public function __construct(string $fieldName = '字段')
    {
        $this->fieldName = $fieldName;
    }

    abstract public function validate($value): bool;

    public function getErrorMessage(): string
    {
        return "{$this->fieldName} 验证失败";
    }
}

class CustomPasswordRule extends BaseRule
{
    public function validate($value): bool
    {
        // 验证逻辑
        return true;
    }

    public function getErrorMessage(): string
    {
        return "{$this->fieldName} 必须符合密码要求";
    }
}
```

### 规则库

**示例**：
```php
<?php
declare(strict_types=1);

class RuleLibrary
{
    private static array $rules = [];

    public static function register(string $name, ValidationRuleInterface $rule): void
    {
        self::$rules[$name] = $rule;
    }

    public static function get(string $name): ?ValidationRuleInterface
    {
        return self::$rules[$name] ?? null;
    }

    public static function getAll(): array
    {
        return self::$rules;
    }
}

// 注册规则
RuleLibrary::register('strongPassword', new StrongPasswordRule());
RuleLibrary::register('phoneCN', new PhoneNumberRule('CN'));

// 使用
$rule = RuleLibrary::get('strongPassword');
```

## 规则测试

### 单元测试

**示例**：
```php
<?php
declare(strict_types=1);

class StrongPasswordRuleTest extends TestCase
{
    public function testValidPassword()
    {
        $rule = new StrongPasswordRule();
        $this->assertTrue($rule->validate('Password123!'));
    }

    public function testInvalidPassword()
    {
        $rule = new StrongPasswordRule();
        $this->assertFalse($rule->validate('password'));  // 缺少大写、数字、特殊字符
        $this->assertFalse($rule->validate('PASSWORD123!'));  // 缺少小写
        $this->assertFalse($rule->validate('Password!'));  // 缺少数字
    }
}
```

### 集成测试

**示例**：
```php
<?php
declare(strict_types=1);

class ValidationIntegrationTest extends TestCase
{
    public function testUserValidation()
    {
        $validator = new Validator();
        $validator->addRule('email', UserValidationRules::email());
        $validator->addRule('password', UserValidationRules::password());

        $data = [
            'email' => 'john@example.com',
            'password' => 'Password123!',
        ];

        $this->assertTrue($validator->validate($data));
    }
}
```

### 测试覆盖

**示例**：
```php
<?php
declare(strict_types=1);

class RuleTestCoverage
{
    public function testAllCases()
    {
        $rule = new StrongPasswordRule();

        // 测试有效情况
        $this->assertTrue($rule->validate('Password123!'));

        // 测试无效情况
        $this->assertFalse($rule->validate(''));  // 空字符串
        $this->assertFalse($rule->validate('short'));  // 太短
        $this->assertFalse($rule->validate('password123!'));  // 缺少大写
        $this->assertFalse($rule->validate('PASSWORD123!'));  // 缺少小写
        $this->assertFalse($rule->validate('Password!'));  // 缺少数字
        $this->assertFalse($rule->validate('Password123'));  // 缺少特殊字符
    }
}
```

## 使用场景

### 业务规则

- 业务逻辑验证
- 数据一致性验证
- 业务约束验证

### 复杂验证

- 复杂格式验证
- 多条件验证
- 依赖验证

### 特定格式

- 特定格式验证
- 自定义格式验证
- 行业标准验证

### 数据一致性

- 数据关联验证
- 数据完整性验证
- 数据一致性验证

## 注意事项

### 规则设计

- **单一职责**：每个规则只做一件事
- **清晰逻辑**：验证逻辑要清晰
- **易于测试**：易于单元测试

### 规则注册

- **命名规范**：使用清晰的命名
- **规则管理**：统一管理规则
- **规则文档**：文档化规则

### 规则复用

- **封装规则**：封装常用规则
- **规则组合**：组合规则提高复用
- **规则库**：建立规则库

### 规则测试

- **单元测试**：编写单元测试
- **测试覆盖**：确保测试覆盖
- **边界测试**：测试边界情况

## 常见问题

### 如何编写自定义规则？

实现 ValidationRuleInterface 接口，实现 validate 和 getErrorMessage 方法。

### 如何注册自定义规则？

使用规则注册表注册规则，或使用验证库的注册机制。

### 如何复用规则？

封装规则类，创建规则库，使用规则组合。

### 如何测试规则？

编写单元测试，测试有效和无效情况，确保测试覆盖。

## 最佳实践

### 单一职责原则

- 每个规则只做一件事
- 保持规则简单
- 易于理解和维护

### 规则封装

- 封装常用规则
- 创建规则库
- 提高代码复用

### 规则测试

- 编写完整的单元测试
- 测试所有边界情况
- 确保测试覆盖

### 规则文档

- 文档化规则用途
- 说明规则参数
- 提供使用示例

## 相关章节

- **[5.15.1 请求验证概述](section-01-overview.md)**：了解请求验证概述的详细内容
- **[5.15.2 Symfony Validator](section-02-symfony-validator.md)**：了解 Symfony Validator 的详细内容
- **[5.15.3 Respect/Validation](section-03-respect-validation.md)**：了解 Respect/Validation 的详细内容

## 练习任务

1. **编写基本自定义规则**
   - 实现规则接口
   - 编写验证逻辑
   - 提供错误消息

2. **实现复杂规则**
   - 多条件验证
   - 依赖验证
   - 业务规则验证

3. **实现规则注册系统**
   - 规则注册表
   - 规则管理
   - 规则查找

4. **实现规则复用**
   - 规则封装
   - 规则组合
   - 规则库

5. **实现完整的自定义规则系统**
   - 规则编写和注册
   - 规则测试
   - 规则文档

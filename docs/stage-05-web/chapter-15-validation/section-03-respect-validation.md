# 5.15.3 Respect/Validation

## 概述

Respect/Validation 是简洁优雅的验证库，提供了链式验证语法和丰富的验证规则。理解 Respect/Validation 的使用方法、掌握链式验证、实现自定义规则，对于构建灵活的验证系统至关重要。本节详细介绍 Respect/Validation 概述、安装配置、验证规则、链式验证、自定义规则等内容，帮助零基础学员掌握 Respect/Validation 的使用。

Respect/Validation 以其简洁的链式语法和丰富的验证规则而闻名，是 PHP 生态中流行的验证库之一。掌握 Respect/Validation 的使用可以大大提高验证代码的可读性和可维护性。

**主要内容**：
- Respect/Validation 概述（Validation 的功能、Validation 的优势、使用场景）
- 安装配置（Composer 安装、基本使用、验证器创建）
- 验证规则（内置规则、规则组合、规则参数、规则消息）
- 链式验证（链式语法、规则链、条件验证、嵌套验证）
- 自定义规则（规则编写、规则注册、规则复用）
- 实际应用示例和最佳实践

## 特性

- **链式语法**：优雅的链式验证语法
- **丰富规则**：提供丰富的内置规则
- **易于使用**：简洁的 API
- **可扩展**：易于扩展自定义规则
- **高性能**：高效的验证机制

## Respect/Validation 概述

### Validation 的功能

1. **数据验证**：验证各种数据类型
2. **链式验证**：优雅的链式验证语法
3. **自定义规则**：支持自定义验证规则
4. **错误处理**：清晰的错误信息

### Validation 的优势

1. **语法优雅**：链式语法清晰易读
2. **功能完整**：提供丰富的验证规则
3. **易于使用**：简洁的 API
4. **可扩展**：易于扩展
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
composer require respect/validation
```

### 基本使用

**示例**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Respect\Validation\Validator as v;

$email = 'john@example.com';

if (v::email()->validate($email)) {
    echo "邮箱有效\n";
} else {
    echo "邮箱无效\n";
}
```

### 验证器创建

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

function validateEmail(string $email): bool
{
    return v::email()->validate($email);
}

function validatePhone(string $phone): bool
{
    return v::phone()->validate($phone);
}
```

## 验证规则

### 内置规则

**常用规则**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 类型规则
v::string()
v::int()
v::float()
v::bool()
v::array()

// 格式规则
v::email()
v::url()
v::ip()
v::phone()
v::date()

// 长度规则
v::length(1, 100)
v::minLength(8)
v::maxLength(100)

// 范围规则
v::between(1, 100)
v::min(0)
v::max(100)

// 其他规则
v::notEmpty()
v::notBlank()
v::notNull()
v::alnum()
v::alpha()
v::digit()
v::numeric()
```

### 规则组合

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 链式组合
$validator = v::string()
    ->notEmpty()
    ->length(2, 50)
    ->alpha();

// 使用
if ($validator->validate($name)) {
    echo "验证通过\n";
}
```

### 规则参数

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 带参数的规则
$validator = v::length(8, 100);  // 长度范围
$validator = v::between(1, 100);  // 数值范围
$validator = v::date('Y-m-d');    // 日期格式
$validator = v::regex('/^[a-z0-9-]+$/');  // 正则表达式
```

### 规则消息

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

$validator = v::email()->setName('邮箱');

try {
    $validator->assert('invalid-email');
} catch (\Respect\Validation\Exceptions\ValidationException $e) {
    echo $e->getFullMessage();
    // 输出: 邮箱 必须是有效的邮箱地址
}
```

## 链式验证

### 链式语法

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 链式验证
$validator = v::string()
    ->notEmpty()
    ->length(2, 50)
    ->alpha();

if ($validator->validate($name)) {
    echo "验证通过\n";
}
```

### 规则链

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 复杂链式验证
$emailValidator = v::email()
    ->notEmpty()
    ->setName('邮箱');

$passwordValidator = v::string()
    ->notEmpty()
    ->length(8, 100)
    ->regex('/[A-Z]/')
    ->regex('/[a-z]/')
    ->regex('/[0-9]/')
    ->setName('密码');

// 使用
if ($emailValidator->validate($email) && $passwordValidator->validate($password)) {
    echo "验证通过\n";
}
```

### 条件验证

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 条件验证
$validator = v::when(
    v::int()->positive(),
    v::between(1, 100),
    v::string()->notEmpty()
);

// 如果值是整数且为正数，验证范围；否则验证字符串
```

### 嵌套验证

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 嵌套验证
$validator = v::key('name', v::string()->notEmpty()->length(2, 50))
    ->key('email', v::email()->notEmpty())
    ->key('age', v::int()->min(18)->max(120));

$data = [
    'name' => 'John',
    'email' => 'john@example.com',
    'age' => 25,
];

if ($validator->validate($data)) {
    echo "验证通过\n";
}
```

## 自定义规则

### 规则编写

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Rules\AbstractRule;

class StrongPassword extends AbstractRule
{
    public function validate($input): bool
    {
        if (!is_string($input)) {
            return false;
        }

        $hasUpper = preg_match('/[A-Z]/', $input);
        $hasLower = preg_match('/[a-z]/', $input);
        $hasNumber = preg_match('/[0-9]/', $input);
        $hasSpecial = preg_match('/[^A-Za-z0-9]/', $input);

        return $hasUpper && $hasLower && $hasNumber && $hasSpecial;
    }
}

// 使用
use Respect\Validation\Validator as v;

$validator = v::string()
    ->notEmpty()
    ->length(8, 100)
    ->addRule(new StrongPassword())
    ->setName('密码');
```

### 规则注册

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;
use Respect\Validation\Factory;

// 注册自定义规则
$factory = new Factory();
$factory->register('strongPassword', function() {
    return new StrongPassword();
});

// 使用
$validator = v::strongPassword();
```

### 规则复用

**示例**：
```php
<?php
declare(strict_types=1);

use Respect\Validation\Validator as v;

// 创建可复用的验证器
class UserValidators
{
    public static function email(): \Respect\Validation\Validator
    {
        return v::email()->notEmpty()->setName('邮箱');
    }

    public static function password(): \Respect\Validation\Validator
    {
        return v::string()
            ->notEmpty()
            ->length(8, 100)
            ->regex('/[A-Z]/')
            ->regex('/[a-z]/')
            ->regex('/[0-9]/')
            ->setName('密码');
    }

    public static function name(): \Respect\Validation\Validator
    {
        return v::string()
            ->notEmpty()
            ->length(2, 50)
            ->alpha()
            ->setName('姓名');
    }
}

// 使用
$emailValidator = UserValidators::email();
$passwordValidator = UserValidators::password();
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

### 链式语法

- **可读性**：链式语法提高可读性
- **顺序**：注意规则的执行顺序
- **性能**：链式验证可能有性能影响

### 错误处理

- **异常捕获**：使用 try-catch 捕获异常
- **错误消息**：使用 getFullMessage() 获取完整错误
- **错误格式化**：格式化错误信息

### 性能考虑

- **及时验证**：及时验证，避免不必要的验证
- **规则优化**：优化验证规则顺序
- **缓存验证**：缓存验证结果

### 自定义规则

- **规则设计**：设计清晰的规则
- **规则测试**：编写规则测试
- **规则文档**：文档化规则

## 常见问题

### 如何安装 Respect/Validation？

使用 Composer 安装：
```bash
composer require respect/validation
```

### 如何使用链式验证？

使用链式语法组合多个规则：
```php
v::string()->notEmpty()->length(2, 50)->alpha();
```

### 如何自定义验证规则？

继承 AbstractRule 类，实现 validate 方法。

### 如何处理验证错误？

使用 try-catch 捕获 ValidationException：
```php
try {
    $validator->assert($value);
} catch (ValidationException $e) {
    echo $e->getFullMessage();
}
```

## 最佳实践

### 使用链式语法

- 使用链式语法提高可读性
- 合理组织规则顺序
- 保持链式简洁

### 提供清晰的错误消息

- 使用 setName() 设置字段名
- 使用自定义错误消息
- 格式化错误信息

### 实现可复用的验证器

- 创建验证器类
- 封装常用验证规则
- 提高代码复用

### 编写自定义规则

- 继承 AbstractRule
- 实现 validate 方法
- 编写规则测试

## 相关章节

- **[5.15.1 请求验证概述](section-01-overview.md)**：了解请求验证概述的详细内容
- **[5.15.2 Symfony Validator](section-02-symfony-validator.md)**：了解 Symfony Validator 的详细内容
- **[5.15.4 自定义验证规则](section-04-custom-rules.md)**：了解自定义验证规则的详细内容

## 练习任务

1. **安装和配置 Respect/Validation**
   - Composer 安装
   - 基本使用
   - 验证器创建

2. **实现基本验证**
   - 使用内置规则
   - 链式验证
   - 错误处理

3. **实现复杂验证**
   - 规则组合
   - 条件验证
   - 嵌套验证

4. **实现自定义规则**
   - 规则编写
   - 规则注册
   - 规则复用

5. **实现完整的验证系统**
   - 多种验证规则
   - 错误处理
   - 验证中间件

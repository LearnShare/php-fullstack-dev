# 3.1.2 可见性修饰符

## 概述

可见性修饰符控制类成员的访问范围，是封装（Encapsulation）的基础。理解可见性的作用对于设计良好的类至关重要。

## 可见性修饰符

### 三种可见性

| 修饰符     | 说明                                           | 访问范围                           |
| :--------- | :--------------------------------------------- | :--------------------------------- |
| `public`   | 公开访问                                       | 类内、子类、外部均可访问           |
| `protected` | 受保护访问                                     | 类内、子类可访问，外部不可访问     |
| `private`  | 私有访问                                       | 仅类内可访问，子类与外部均不可访问 |

### public（公开）

```php
class User
{
    public int $id;
    public string $name;

    public function getName(): string
    {
        return $this->name;
    }
}

$user = new User();
$user->id = 1;           // 可以访问
$user->name = 'Alice';   // 可以访问
echo $user->getName();   // 可以调用
```

### protected（受保护）

```php
class User
{
    protected string $email;

    protected function validateEmail(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}

class Admin extends User
{
    public function setEmail(string $email): void
    {
        if ($this->validateEmail($email)) {  // 可以访问父类的 protected 方法
            $this->email = $email;            // 可以访问父类的 protected 属性
        }
    }
}

$admin = new Admin();
$admin->setEmail('admin@example.com');
// $admin->email;  // 错误：无法从外部访问 protected 属性
```

### private（私有）

```php
class BankAccount
{
    private float $balance = 0.0;
    private string $pin;

    private function validatePin(string $input): bool
    {
        return $this->pin === $input;
    }

    public function withdraw(float $amount, string $pin): bool
    {
        if (!$this->validatePin($pin)) {  // 可以访问 private 方法
            return false;
        }
        if ($amount > $this->balance) {
            return false;
        }
        $this->balance -= $amount;  // 可以访问 private 属性
        return true;
    }
}

$account = new BankAccount();
// $account->balance;  // 错误：无法访问 private 属性
// $account->validatePin('1234');  // 错误：无法调用 private 方法
```

## 可见性最佳实践

### 属性封装

- 属性通常设为 `private` 或 `protected`，通过公开方法（getter/setter）控制访问。

```php
class User
{
    private string $email;

    public function getEmail(): string
    {
        return $this->email;
    }

    public function setEmail(string $email): void
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email format');
        }
        $this->email = $email;
    }
}
```

### 方法可见性

- 方法根据业务需求选择可见性，避免过度暴露内部实现。

```php
class PaymentProcessor
{
    public function processPayment(float $amount): bool
    {
        $this->validateAmount($amount);
        $this->checkBalance($amount);
        return $this->charge($amount);
    }

    private function validateAmount(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
    }

    private function checkBalance(float $amount): void
    {
        // 检查余额逻辑
    }

    private function charge(float $amount): bool
    {
        // 扣款逻辑
        return true;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class BankAccount
{
    private float $balance = 0.0;
    private string $accountNumber;
    protected string $bankName = 'MyBank';

    public function __construct(string $accountNumber)
    {
        $this->accountNumber = $accountNumber;
    }

    public function deposit(float $amount): void
    {
        $this->validateAmount($amount);
        $this->balance += $amount;
        $this->logTransaction('deposit', $amount);
    }

    public function withdraw(float $amount): void
    {
        $this->validateAmount($amount);
        if ($amount > $this->balance) {
            throw new RuntimeException('Insufficient funds');
        }
        $this->balance -= $amount;
        $this->logTransaction('withdraw', $amount);
    }

    public function getBalance(): float
    {
        return $this->balance;
    }

    public function getAccountNumber(): string
    {
        return $this->accountNumber;
    }

    private function validateAmount(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
    }

    private function logTransaction(string $type, float $amount): void
    {
        // 记录交易日志
        error_log("Transaction: {$type} {$amount}");
    }
}

class SavingsAccount extends BankAccount
{
    private float $interestRate = 0.02;

    public function applyInterest(): void
    {
        $interest = $this->getBalance() * $this->interestRate;
        $this->deposit($interest);
    }

    public function getBankName(): string
    {
        return $this->bankName;  // 可以访问 protected 属性
    }
}
```

## 注意事项

1. **默认可见性**：类成员的默认可见性是 `public`，但建议显式声明。

2. **封装原则**：优先使用 `private`，只在需要时使用 `protected` 或 `public`。

3. **Getter/Setter**：对于需要外部访问的属性，提供 getter/setter 方法而不是直接暴露属性。

4. **继承考虑**：如果属性或方法可能被子类使用，使用 `protected` 而不是 `private`。

5. **接口实现**：实现接口的方法必须是 `public`。

## 练习

1. 创建一个 `Product` 类，使用 `private` 属性存储价格，提供 `getPrice()` 和 `setPrice()` 方法，在 `setPrice()` 中验证价格必须为正数。

2. 实现一个 `User` 类，包含 `private` 的密码哈希，提供 `verifyPassword()` 方法验证密码，但不允许直接访问密码哈希。

3. 创建一个 `Logger` 类，使用 `private` 方法 `formatMessage()` 格式化日志消息，提供 `public` 方法 `log()` 记录日志。

4. 实现一个 `Shape` 基类，包含 `protected` 属性 `color`，创建 `Rectangle` 和 `Circle` 子类，子类可以访问 `color` 属性但外部不能。

5. 创建一个 `DatabaseConnection` 类，使用 `private` 方法处理连接细节，只暴露必要的 `public` 方法给外部使用。

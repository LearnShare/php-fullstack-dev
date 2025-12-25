# 2.18.5 升级指南与最佳实践

## 概述

本节提供从 PHP 8.1 到 8.5 的逐版本升级路线、兼容性检测脚本、最佳实践与练习任务，确保生产环境平滑过渡。

## 版本升级路线

### 从 PHP 8.1 升级到 8.2

1. **检查废弃功能**
   ```bash
   php -l your-file.php
   ```

2. **更新只读类**
   - 将需要不可变的对象标记为 `readonly class`

3. **使用新的 Random 扩展**
   - 考虑迁移到新的 `Random` 扩展

### 从 PHP 8.2 升级到 8.3

1. **使用 json_validate()**
   - 在解码前验证 JSON，提升性能

2. **添加类常量类型**
   - 为类常量添加类型声明

3. **使用 #[\Override] 属性**
   - 明确标记覆盖的方法

### 从 PHP 8.3 升级到 8.4

1. **使用属性钩子**
   - 将 getter/setter 逻辑迁移到属性钩子

2. **使用不对称可见性**
   - 利用不对称可见性简化属性访问控制

3. **使用 array_first() 和 array_last()**
   - 替换 `reset()` 和 `end()` 的使用

4. **添加 #[\NoDiscard] 属性**
   - 标记不应忽略返回值的函数

### 从 PHP 8.4 升级到 8.5

1. **使用管道操作符**
   - 简化函数调用链，提升可读性

2. **使用 clone with 语法**
   - 简化对象克隆和属性更新

3. **迁移到内置 URI 扩展**
   - 使用新的 URI API 替代旧的解析方法

4. **更新类型转换**
   - 将 `(integer)`、`(double)` 等替换为 `(int)`、`(float)`

5. **利用 OPcache 必选特性**
   - OPcache 现在总是可用，无需检查扩展是否存在

## 兼容性检查

```php
<?php
// 检查 PHP 版本
if (version_compare(PHP_VERSION, '8.3.0', '>=')) {
    // 使用 PHP 8.3+ 特性
    if (json_validate($json)) {
        // ...
    }
} else {
    // 降级处理
    $data = json_decode($json, true);
    if (json_last_error() === JSON_ERROR_NONE) {
        // ...
    }
}
```

## 最佳实践

### 1. 使用 json_validate() 提升性能

```php
// 推荐：先验证再解码
if (json_validate($json)) {
    $data = json_decode($json, true);
}

// 不推荐：直接解码
$data = json_decode($json, true);
if (json_last_error() !== JSON_ERROR_NONE) {
    // 处理错误
}
```

### 2. 使用只读类创建不可变对象

```php
readonly class Money
{
    public function __construct(
        public float $amount,
        public string $currency,
    ) {}

    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Currency mismatch');
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
}
```

### 3. 使用 #[\Override] 提高代码可维护性

```php
class BaseRepository
{
    public function find(int $id): ?Entity
    {
        // 基础实现
    }
}

class UserRepository extends BaseRepository
{
    #[\Override]
    public function find(int $id): ?User
    {
        // 如果父类方法签名改变，这里会报错
        return parent::find($id);
    }
}
```

## 升级检查清单

- [ ] 运行 `php -l` 检查所有文件语法
- [ ] 运行静态分析工具（PHPStan/Psalm）检查类型错误
- [ ] 运行完整测试套件，确保功能正常
- [ ] 检查第三方依赖是否支持目标 PHP 版本
- [ ] 更新 CI/CD 配置，使用新版本 PHP
- [ ] 在生产环境前，先在 staging 环境验证
- [ ] 监控错误日志和性能指标

## 练习

1. 编写一个版本兼容性检查工具，根据 PHP 版本选择不同的实现。
2. 将现有项目从 PHP 8.1 升级到 8.3，记录遇到的问题和解决方案。
3. 使用新特性重构现有代码，提升代码质量和可读性。
4. 创建升级文档，记录升级步骤和注意事项。

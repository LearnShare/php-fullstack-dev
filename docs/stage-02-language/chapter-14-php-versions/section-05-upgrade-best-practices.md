# 2.14.5 升级指南与最佳实践

## 概述

PHP 版本升级需要系统的方法。本节详细介绍逐版本升级路线、兼容性检测脚本、最佳实践与练习任务，帮助顺利完成版本升级。

理解升级策略和最佳实践对于安全、顺利地升级 PHP 版本非常重要。掌握升级方法可以帮助避免问题、减少风险、提高升级成功率。

## 特性

- **系统化方法**：系统化的升级方法
- **风险控制**：降低升级风险
- **兼容性检测**：自动检测兼容性问题
- **回滚策略**：准备回滚方案

## 语法/定义

### 升级策略

**逐步升级**：不要跨版本升级，逐步升级

**测试环境**：先在测试环境升级

**备份数据**：升级前备份数据和代码

### 兼容性检测

**工具**：使用 PHPCompatibility 等工具检测兼容性

**方法**：运行兼容性检测脚本

**修复**：修复不兼容的代码

## 基本用法

### 示例 1：版本检查脚本

```php
<?php
declare(strict_types=1);

function checkPhpVersion(string $requiredVersion): void
{
    if (version_compare(PHP_VERSION, $requiredVersion, '<')) {
        echo "PHP {$requiredVersion}+ required, current: " . PHP_VERSION . "\n";
        exit(1);
    }
    
    echo "PHP version OK: " . PHP_VERSION . "\n";
}

// 使用
checkPhpVersion('8.2.0');
```

### 示例 2：扩展检查脚本

```php
<?php
declare(strict_types=1);

function checkExtensions(array $required): void
{
    $missing = [];
    
    foreach ($required as $ext) {
        if (!extension_loaded($ext)) {
            $missing[] = $ext;
        }
    }
    
    if (!empty($missing)) {
        echo "Missing extensions: " . implode(', ', $missing) . "\n";
        exit(1);
    }
    
    echo "All required extensions are loaded\n";
}

// 使用
checkExtensions(['mbstring', 'json', 'pdo', 'opcache']);
```

### 示例 3：兼容性检测

```php
<?php
declare(strict_types=1);

class CompatibilityChecker
{
    public static function checkDeprecatedFeatures(): array
    {
        $issues = [];
        
        // 检查废弃的函数
        $deprecated = [
            'mysql_connect',
            'ereg',
            'split'
        ];
        
        foreach ($deprecated as $func) {
            if (function_exists($func)) {
                $issues[] = "Deprecated function: {$func}";
            }
        }
        
        return $issues;
    }
    
    public static function checkIniSettings(): array
    {
        $issues = [];
        
        // 检查废弃的 ini 设置
        $deprecated = [
            'allow_url_include',
            'magic_quotes_gpc'
        ];
        
        foreach ($deprecated as $setting) {
            if (ini_get($setting) !== false) {
                $issues[] = "Deprecated ini setting: {$setting}";
            }
        }
        
        return $issues;
    }
}

// 使用
$issues = CompatibilityChecker::checkDeprecatedFeatures();
if (!empty($issues)) {
    echo "Compatibility issues found:\n";
    foreach ($issues as $issue) {
        echo "  - {$issue}\n";
    }
}
```

## 完整代码示例

### 示例 1：升级检查清单

```php
<?php
declare(strict_types=1);

class UpgradeChecklist
{
    public static function run(): void
    {
        echo "=== PHP Upgrade Checklist ===\n\n";
        
        // 1. 检查 PHP 版本
        echo "1. PHP Version: " . PHP_VERSION . "\n";
        
        // 2. 检查必需扩展
        echo "2. Required Extensions:\n";
        $required = ['mbstring', 'json', 'pdo', 'opcache'];
        foreach ($required as $ext) {
            $status = extension_loaded($ext) ? '✓' : '✗';
            echo "   {$status} {$ext}\n";
        }
        
        // 3. 检查配置
        echo "3. Configuration:\n";
        echo "   - error_reporting: " . error_reporting() . "\n";
        echo "   - display_errors: " . ini_get('display_errors') . "\n";
        echo "   - opcache.enable: " . ini_get('opcache.enable') . "\n";
        
        // 4. 检查废弃特性
        echo "4. Deprecated Features:\n";
        $deprecated = self::checkDeprecated();
        if (empty($deprecated)) {
            echo "   ✓ No deprecated features found\n";
        } else {
            foreach ($deprecated as $item) {
                echo "   ✗ {$item}\n";
            }
        }
        
        echo "\n=== Checklist Complete ===\n";
    }
    
    private static function checkDeprecated(): array
    {
        $issues = [];
        
        // 检查废弃函数
        if (function_exists('mysql_connect')) {
            $issues[] = "mysql_* functions are deprecated";
        }
        
        return $issues;
    }
}

// 使用
UpgradeChecklist::run();
```

### 示例 2：升级步骤脚本

```php
<?php
declare(strict_types=1);

class UpgradeManager
{
    public static function upgrade(string $targetVersion): void
    {
        echo "Starting upgrade to PHP {$targetVersion}\n\n";
        
        // 步骤 1：备份
        echo "Step 1: Backup\n";
        self::backup();
        
        // 步骤 2：兼容性检测
        echo "Step 2: Compatibility Check\n";
        $issues = self::checkCompatibility($targetVersion);
        if (!empty($issues)) {
            echo "Issues found:\n";
            foreach ($issues as $issue) {
                echo "  - {$issue}\n";
            }
            echo "Please fix issues before continuing\n";
            return;
        }
        
        // 步骤 3：测试环境升级
        echo "Step 3: Test Environment Upgrade\n";
        echo "Please upgrade PHP in test environment and run tests\n";
        
        // 步骤 4：生产环境升级
        echo "Step 4: Production Environment Upgrade\n";
        echo "After successful testing, upgrade production environment\n";
        
        echo "\nUpgrade process completed\n";
    }
    
    private static function backup(): void
    {
        echo "  - Backing up code...\n";
        echo "  - Backing up database...\n";
        echo "  - Backup complete\n";
    }
    
    private static function checkCompatibility(string $version): array
    {
        $issues = [];
        
        // 检查版本要求
        if (version_compare(PHP_VERSION, $version, '<')) {
            $issues[] = "Current PHP version is lower than target";
        }
        
        // 检查扩展
        $required = ['mbstring', 'json'];
        foreach ($required as $ext) {
            if (!extension_loaded($ext)) {
                $issues[] = "Required extension missing: {$ext}";
            }
        }
        
        return $issues;
    }
}

// 使用
UpgradeManager::upgrade('8.2.0');
```

### 示例 3：回滚策略

```php
<?php
declare(strict_types=1);

class RollbackManager
{
    public static function prepareRollback(): void
    {
        echo "=== Rollback Preparation ===\n\n";
        
        // 1. 备份当前版本
        echo "1. Backup current version\n";
        self::backupVersion(PHP_VERSION);
        
        // 2. 记录配置
        echo "2. Record configuration\n";
        self::recordConfig();
        
        // 3. 创建回滚脚本
        echo "3. Create rollback script\n";
        self::createRollbackScript();
        
        echo "\nRollback preparation complete\n";
    }
    
    private static function backupVersion(string $version): void
    {
        $backupDir = __DIR__ . "/backups/php-{$version}";
        if (!is_dir($backupDir)) {
            mkdir($backupDir, 0755, true);
        }
        
        // 备份关键文件
        $files = ['composer.json', 'composer.lock', '.env'];
        foreach ($files as $file) {
            if (file_exists($file)) {
                copy($file, "{$backupDir}/{$file}");
            }
        }
        
        echo "  - Version backed up to {$backupDir}\n";
    }
    
    private static function recordConfig(): void
    {
        $config = [
            'php_version' => PHP_VERSION,
            'extensions' => get_loaded_extensions(),
            'ini_settings' => [
                'error_reporting' => error_reporting(),
                'display_errors' => ini_get('display_errors'),
                'memory_limit' => ini_get('memory_limit')
            ]
        ];
        
        file_put_contents(
            __DIR__ . '/backups/config.json',
            json_encode($config, JSON_PRETTY_PRINT)
        );
        
        echo "  - Configuration recorded\n";
    }
    
    private static function createRollbackScript(): void
    {
        $script = <<<'PHP'
<?php
// Rollback script
// Restore from backup if needed
PHP;
        
        file_put_contents(__DIR__ . '/rollback.php', $script);
        echo "  - Rollback script created\n";
    }
}

// 使用
RollbackManager::prepareRollback();
```

## 使用场景

### 版本升级

- **功能需求**：需要使用新版本特性
- **性能优化**：新版本性能更好
- **安全更新**：新版本安全修复

### 兼容性检测

- **升级前检查**：升级前检测兼容性
- **问题发现**：及早发现兼容性问题
- **修复指导**：指导修复不兼容代码

### 回滚准备

- **风险控制**：降低升级风险
- **快速恢复**：快速恢复到旧版本
- **数据保护**：保护数据和配置

## 注意事项

### 升级策略

- **逐步升级**：不要跨版本升级
- **测试环境**：先在测试环境升级
- **备份数据**：升级前备份数据和代码

### 兼容性检测

- **全面检测**：全面检测兼容性问题
- **及时修复**：及时修复不兼容代码
- **工具使用**：使用专业工具检测

### 回滚策略

- **准备充分**：充分准备回滚方案
- **快速响应**：能够快速回滚
- **数据保护**：保护数据和配置

## 常见问题

### 问题 1：兼容性问题

**症状**：升级后代码无法运行

**原因**：代码使用了废弃特性或语法

**解决方法**：

```php
<?php
declare(strict_types=1);

// 使用兼容性检测工具
// composer require --dev phpcompatibility/php-compatibility

// 运行检测
// vendor/bin/phpcs --standard=PHPCompatibility --runtime-set testVersion 8.2 src/

// 修复问题
// 1. 替换废弃函数
// 2. 更新语法
// 3. 修复类型问题
```

### 问题 2：性能问题

**症状**：升级后性能下降

**原因**：新版本行为变化或配置不当

**解决方法**：

```php
<?php
declare(strict_types=1);

// 1. 检查 OPcache 配置
if (function_exists('opcache_get_status')) {
    $status = opcache_get_status();
    if ($status === false) {
        echo "OPcache is not enabled\n";
    }
}

// 2. 检查内存限制
echo "Memory limit: " . ini_get('memory_limit') . "\n";

// 3. 性能测试
$start = microtime(true);
// 执行代码
$end = microtime(true);
echo "Execution time: " . ($end - $start) . "s\n";
```

## 最佳实践

### 升级前准备

- **制定计划**：制定详细的升级计划
- **备份数据**：备份所有数据和代码
- **测试环境**：在测试环境充分测试

### 升级过程

- **逐步升级**：逐步升级，不要跨版本
- **兼容性检测**：使用工具检测兼容性
- **全面测试**：全面测试应用功能

### 升级后验证

- **功能验证**：验证所有功能正常
- **性能监控**：监控性能指标
- **错误监控**：监控错误日志

### 回滚准备

- **回滚方案**：准备回滚方案
- **快速响应**：能够快速回滚
- **数据保护**：保护数据和配置

## 对比分析

### 逐步升级 vs 跨版本升级

| 特性 | 逐步升级 | 跨版本升级 |
|:-----|:---------|:-----------|
| 风险 | 低 | 高 |
| 时间 | 长 | 短 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **所有场景**：使用逐步升级
- **避免跨版本**：避免跨版本升级

## 相关章节

- **1.2 PHP 运行环境**：了解 PHP 安装和配置
- **1.5 PHP 配置与扩展**：了解 PHP 配置
- **2.13.3 静态代码分析**：了解代码检查工具

## 练习任务

1. **升级准备练习**：
   - 制定升级计划
   - 备份数据和代码
   - 准备测试环境
   - 运行兼容性检测

2. **兼容性检测练习**：
   - 使用兼容性检测工具
   - 修复不兼容代码
   - 测试修复效果
   - 验证兼容性

3. **升级执行练习**：
   - 在测试环境升级
   - 运行测试套件
   - 验证功能正常
   - 监控性能

4. **回滚准备练习**：
   - 准备回滚方案
   - 创建回滚脚本
   - 测试回滚流程
   - 验证回滚效果

5. **综合练习**：
   - 完成一次完整的版本升级
   - 记录升级过程
   - 总结经验和教训
   - 建立升级最佳实践

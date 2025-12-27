# 4.8.4 排查清单与最佳实践

## 概述

系统化的排查方法能够提高问题解决的效率。在实际开发中，遇到问题时需要系统化地收集信息、分析问题、定位原因、解决问题。理解并应用系统化的排查方法，对于快速解决问题至关重要。

本节提供问题排查的清单和最佳实践，包括问题描述、环境信息收集、排查步骤、问题记录等，帮助零基础学员建立系统化的问题排查流程。

**主要内容**：
- 排查清单概述
- 问题描述和复现
- 环境信息收集
- 排查步骤
- 日志分析
- 代码审查
- 测试验证
- 问题记录
- 实际应用场景和最佳实践

## 特性

- **系统化**：提供系统化的排查方法
- **全面性**：覆盖排查的各个方面
- **可操作性**：提供可操作的清单
- **经验积累**：帮助积累排查经验

## 语法/定义

### 排查清单要点

排查清单没有单一的语法定义，而是系统化的方法和流程。

## 基本用法

### 示例 1：问题排查清单

```php
<?php
declare(strict_types=1);

class TroubleshootingChecklist
{
    /**
     * 问题排查清单
     */
    public static function getChecklist(): array
    {
        return [
            '问题描述' => [
                '问题现象是什么？',
                '问题何时发生？',
                '问题的复现步骤？',
                '预期行为是什么？',
                '实际行为是什么？',
            ],
            '环境信息' => [
                'PHP 版本是什么？',
                '相关扩展是否安装？',
                '配置文件是否正确？',
                '系统环境是什么？',
            ],
            '错误信息' => [
                '错误消息是什么？',
                '错误位置在哪里？',
                '堆栈跟踪是什么？',
                '相关日志内容？',
            ],
            '排查步骤' => [
                '检查错误日志',
                '检查代码逻辑',
                '检查配置文件',
                '检查环境配置',
                '测试验证',
            ],
        ];
    }
    
    /**
     * 环境信息收集
     */
    public static function collectEnvironmentInfo(): array
    {
        return [
            'php_version' => PHP_VERSION,
            'php_sapi' => php_sapi_name(),
            'extensions' => get_loaded_extensions(),
            'ini_settings' => [
                'error_reporting' => error_reporting(),
                'display_errors' => ini_get('display_errors'),
                'memory_limit' => ini_get('memory_limit'),
            ],
            'system' => [
                'os' => PHP_OS,
                'os_family' => PHP_OS_FAMILY,
            ],
        ];
    }
    
    /**
     * 打印排查清单
     */
    public static function printChecklist(): void
    {
        $checklist = self::getChecklist();
        
        echo "=== 问题排查清单 ===\n\n";
        
        foreach ($checklist as $section => $items) {
            echo "【{$section}】\n";
            foreach ($items as $index => $item) {
                echo "  " . ($index + 1) . ". {$item}\n";
            }
            echo "\n";
        }
    }
}

// 使用
TroubleshootingChecklist::printChecklist();

$envInfo = TroubleshootingChecklist::collectEnvironmentInfo();
echo "环境信息:\n";
print_r($envInfo);
```

**说明**：
- 提供系统化的排查清单
- 收集环境信息，便于问题定位

### 示例 2：问题记录模板

```php
<?php
declare(strict_types=1);

class ProblemReport
{
    private string $title;
    private array $description;
    private array $environment;
    private array $steps;
    private array $errors;
    private ?string $solution = null;
    
    public function __construct(string $title)
    {
        $this->title = $title;
        $this->description = [];
        $this->environment = [];
        $this->steps = [];
        $this->errors = [];
    }
    
    public function setDescription(string $phenomenon, string $expected, string $actual): void
    {
        $this->description = [
            'phenomenon' => $phenomenon,
            'expected' => $expected,
            'actual' => $actual,
        ];
    }
    
    public function setEnvironment(array $info): void
    {
        $this->environment = $info;
    }
    
    public function addStep(string $step): void
    {
        $this->steps[] = $step;
    }
    
    public function addError(string $message, ?string $file = null, ?int $line = null): void
    {
        $this->errors[] = [
            'message' => $message,
            'file' => $file,
            'line' => $line,
        ];
    }
    
    public function setSolution(string $solution): void
    {
        $this->solution = $solution;
    }
    
    public function toMarkdown(): string
    {
        $md = "# {$this->title}\n\n";
        
        $md .= "## 问题描述\n\n";
        $md .= "- **现象**: {$this->description['phenomenon']}\n";
        $md .= "- **预期**: {$this->description['expected']}\n";
        $md .= "- **实际**: {$this->description['actual']}\n\n";
        
        $md .= "## 环境信息\n\n";
        $md .= "```\n";
        $md .= print_r($this->environment, true);
        $md .= "```\n\n";
        
        $md .= "## 复现步骤\n\n";
        foreach ($this->steps as $index => $step) {
            $md .= ($index + 1) . ". {$step}\n";
        }
        $md .= "\n";
        
        $md .= "## 错误信息\n\n";
        foreach ($this->errors as $error) {
            $md .= "- **消息**: {$error['message']}\n";
            if ($error['file']) {
                $md .= "  - **文件**: {$error['file']}\n";
            }
            if ($error['line']) {
                $md .= "  - **行号**: {$error['line']}\n";
            }
        }
        $md .= "\n";
        
        if ($this->solution) {
            $md .= "## 解决方案\n\n";
            $md .= $this->solution . "\n\n";
        }
        
        return $md;
    }
}

// 使用
$report = new ProblemReport('数据库连接失败');

$report->setDescription(
    '无法连接到数据库',
    '应该能够正常连接',
    '连接超时'
);

$report->setEnvironment([
    'php_version' => PHP_VERSION,
    'mysql_extension' => extension_loaded('pdo_mysql'),
]);

$report->addStep('启动应用');
$report->addStep('尝试连接数据库');
$report->addStep('观察错误信息');

$report->addError('Connection timeout', '/path/to/file.php', 42);

$report->setSolution('检查数据库配置和网络连接');

echo $report->toMarkdown();
```

**说明**：
- 提供问题记录模板
- 系统化记录问题信息

### 示例 3：系统化排查流程

```php
<?php
declare(strict_types=1);

class TroubleshootingWorkflow
{
    /**
     * 系统化排查流程
     */
    public static function troubleshoot(callable $problemFunction): void
    {
        echo "=== 开始排查问题 ===\n\n";
        
        // 1. 收集环境信息
        echo "1. 收集环境信息...\n";
        $envInfo = TroubleshootingChecklist::collectEnvironmentInfo();
        self::printSection('环境信息', $envInfo);
        
        // 2. 启用详细错误报告
        echo "\n2. 启用详细错误报告...\n";
        error_reporting(E_ALL);
        ini_set('display_errors', '1');
        
        // 3. 执行问题代码
        echo "\n3. 执行问题代码...\n";
        try {
            $problemFunction();
        } catch (Throwable $e) {
            self::handleError($e);
        }
        
        // 4. 分析结果
        echo "\n4. 分析结果...\n";
        self::analyze();
        
        echo "\n=== 排查完成 ===\n";
    }
    
    private static function handleError(Throwable $e): void
    {
        echo "错误信息:\n";
        echo "  类型: " . get_class($e) . "\n";
        echo "  消息: " . $e->getMessage() . "\n";
        echo "  文件: " . $e->getFile() . "\n";
        echo "  行号: " . $e->getLine() . "\n";
        echo "  堆栈:\n";
        echo $e->getTraceAsString() . "\n";
    }
    
    private static function analyze(): void
    {
        echo "  检查错误日志\n";
        echo "  检查代码逻辑\n";
        echo "  检查配置文件\n";
    }
    
    private static function printSection(string $title, mixed $data): void
    {
        echo "  【{$title}】\n";
        print_r($data);
    }
}

// 使用
TroubleshootingWorkflow::troubleshoot(function () {
    // 问题代码
    $result = 10 / 0;  // 会触发错误
});
```

**说明**：
- 提供系统化的排查流程
- 自动化排查步骤

## 使用场景

### 场景 1：复杂问题排查

使用排查清单系统化排查复杂问题。

**示例**：见"示例 1：问题排查清单"

### 场景 2：问题记录

记录问题信息，便于后续参考。

**示例**：见"示例 2：问题记录模板"

### 场景 3：团队协作

使用标准化的排查流程，便于团队协作。

**示例**：见"示例 3：系统化排查流程"

## 注意事项

### 信息的完整性

收集完整的问题信息，有助于快速定位问题。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 收集完整信息
$info = [
    'php_version' => PHP_VERSION,
    'error_message' => $e->getMessage(),
    'error_file' => $e->getFile(),
    'error_line' => $e->getLine(),
    'stack_trace' => $e->getTraceAsString(),
];

// ❌ 信息不完整
// $info = ['error' => 'something wrong'];
```

### 排查的系统性

按照系统化的流程排查，避免遗漏。

**示例**：见"示例 3：系统化排查流程"

### 记录的详细性

详细记录排查过程和结果，便于后续参考。

**示例**：见"示例 2：问题记录模板"

## 常见问题

### 问题 1：如何系统化排查问题？

**回答**：使用排查清单，按照步骤逐一排查。

**示例**：见"示例 1：问题排查清单"

### 问题 2：如何记录排查过程？

**回答**：使用问题记录模板，详细记录问题信息。

**示例**：见"示例 2：问题记录模板"

### 问题 3：如何提高排查效率？

**回答**：
- 使用排查清单
- 系统化排查
- 积累排查经验
- 使用工具辅助

**示例**：见"示例 3：系统化排查流程"

### 问题 4：如何避免重复排查？

**回答**：记录排查过程和解决方案，建立知识库。

**示例**：见"示例 2：问题记录模板"

## 最佳实践

### 1. 建立排查清单

建立标准化的排查清单，确保不遗漏重要信息。

**示例**：见"示例 1：问题排查清单"

### 2. 详细记录问题

详细记录问题信息，便于后续参考和知识积累。

**示例**：见"示例 2：问题记录模板"

### 3. 系统化排查

按照系统化的流程排查，避免遗漏和重复。

**示例**：见"示例 3：系统化排查流程"

### 4. 积累排查经验

记录排查过程和解决方案，积累经验。

**示例**：

```php
<?php
declare(strict_types=1);

// 记录常见问题和解决方案
$commonProblems = [
    '数据库连接失败' => [
        '原因' => '配置错误或网络问题',
        '解决方案' => '检查配置和网络连接',
    ],
    '内存不足' => [
        '原因' => '内存限制设置过小',
        '解决方案' => '增加 memory_limit',
    ],
];
```

## 对比分析

### 排查方法对比

| 方法           | 系统性 | 效率 | 适用场景           |
|:---------------|:-------|:-----|:-------------------|
| **清单排查**   | ✅ 高  | ✅ 高| 复杂问题           |
| **经验排查**   | ⚠️ 中  | ✅ 高| 常见问题           |
| **工具排查**   | ✅ 高  | ✅ 高| 技术问题           |
| **随机排查**   | ❌ 低  | ❌ 低| 不推荐             |

## 练习任务

1. **排查清单工具**：创建一个工具，生成和跟踪排查清单。

2. **问题记录系统**：实现一个系统，记录和管理问题信息。

3. **排查工作流工具**：创建一个工具，自动化排查流程。

4. **知识库系统**：编写一个系统，积累和查询排查经验。

5. **排查最佳实践指南**：创建一个完整的指南，总结排查的最佳实践。

## 相关章节

- **[4.8.1 常见错误类型与修复流程](section-01-common-errors.md)**：了解常见错误的相关内容
- **[4.8.3 调试技巧与工具链](section-03-debugging-techniques.md)**：了解调试技巧的相关内容
- **[4.9.1 日志体系设计](../chapter-09-logging/section-01-logging-system.md)**：了解日志系统的相关内容

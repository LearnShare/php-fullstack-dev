# 4.4.7 CLI 最佳实践

## 概述

CLI 程序开发需要遵循一定的最佳实践，以确保代码质量、可维护性和用户体验。本节总结 CLI 编程的最佳实践，包括代码组织、错误处理、日志记录、测试策略、性能优化等，帮助零基础学员开发高质量的 CLI 工具。

理解并应用这些最佳实践，能够帮助开发者编写更加健壮、易用、易维护的 CLI 工具。虽然本节主要是总结性的内容，但对于开发生产级别的 CLI 工具至关重要。

**主要内容**：
- 代码组织结构（单一职责、模块化设计）
- 错误处理策略（异常处理、错误码、用户友好的错误信息）
- 日志记录（日志级别、格式、输出位置）
- 测试策略（单元测试、集成测试、命令行测试）
- 性能优化（内存使用、执行时间、资源管理）
- 用户体验优化（清晰的提示、帮助信息、进度显示）
- 安全性考虑（输入验证、权限检查）
- 实际应用场景和完整示例

## 特性

- **代码质量**：遵循最佳实践，确保代码质量
- **可维护性**：良好的代码组织，易于维护
- **用户体验**：友好的用户界面和清晰的提示
- **健壮性**：完善的错误处理和日志记录
- **性能**：优化的性能和资源使用

## 语法/定义

### CLI 最佳实践要点

CLI 最佳实践包括多个方面，没有单一的语法定义，而是综合性的指导原则。

## 基本用法

### 示例 1：代码组织结构

```php
<?php
declare(strict_types=1);

// 良好的代码组织示例
namespace App\Cli;

// 命令类（单一职责）
class ProcessCommand
{
    private Config $config;
    private Logger $logger;
    
    public function __construct(Config $config, Logger $logger)
    {
        $this->config = $config;
        $this->logger = $logger;
    }
    
    public function execute(array $args): int
    {
        try {
            $this->validateArgs($args);
            $this->process($args);
            return 0;
        } catch (Exception $e) {
            $this->handleError($e);
            return 1;
        }
    }
    
    private function validateArgs(array $args): void
    {
        // 验证参数
    }
    
    private function process(array $args): void
    {
        // 处理逻辑
    }
    
    private function handleError(Exception $e): void
    {
        // 错误处理
    }
}
```

**说明**：
- 单一职责：每个类负责一个功能
- 依赖注入：通过构造函数注入依赖
- 错误处理：统一的错误处理逻辑

### 示例 2：错误处理策略

```php
<?php
declare(strict_types=1);

class CliApp
{
    private const EXIT_SUCCESS = 0;
    private const EXIT_ERROR = 1;
    private const EXIT_INVALID_ARGS = 2;
    private const EXIT_FILE_NOT_FOUND = 3;
    
    public function run(array $argv): int
    {
        try {
            $this->validateArgs($argv);
            $this->execute($argv);
            return self::EXIT_SUCCESS;
        } catch (InvalidArgumentException $e) {
            $this->error("参数错误: " . $e->getMessage());
            return self::EXIT_INVALID_ARGS;
        } catch (FileNotFoundException $e) {
            $this->error("文件不存在: " . $e->getMessage());
            return self::EXIT_FILE_NOT_FOUND;
        } catch (Exception $e) {
            $this->error("错误: " . $e->getMessage());
            $this->logger->error('Unexpected error', ['exception' => $e]);
            return self::EXIT_ERROR;
        }
    }
    
    private function error(string $message): void
    {
        fwrite(STDERR, "错误: {$message}\n");
    }
}
```

**说明**：
- 定义退出码常量
- 捕获不同类型的异常
- 输出用户友好的错误信息
- 记录详细错误日志

### 示例 3：日志记录

```php
<?php
declare(strict_types=1);

class Logger
{
    private string $logFile;
    private int $logLevel;
    
    private const LEVEL_DEBUG = 0;
    private const LEVEL_INFO = 1;
    private const LEVEL_WARNING = 2;
    private const LEVEL_ERROR = 3;
    
    public function __construct(string $logFile, int $logLevel = self::LEVEL_INFO)
    {
        $this->logFile = $logFile;
        $this->logLevel = $logLevel;
    }
    
    public function debug(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_DEBUG, $message, $context);
    }
    
    public function info(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_INFO, $message, $context);
    }
    
    public function warning(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_WARNING, $message, $context);
    }
    
    public function error(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_ERROR, $message, $context);
    }
    
    private function log(int $level, string $message, array $context): void
    {
        if ($level < $this->logLevel) {
            return;
        }
        
        $levelName = match ($level) {
            self::LEVEL_DEBUG => 'DEBUG',
            self::LEVEL_INFO => 'INFO',
            self::LEVEL_WARNING => 'WARNING',
            self::LEVEL_ERROR => 'ERROR',
            default => 'UNKNOWN',
        };
        
        $timestamp = date('Y-m-d H:i:s');
        $contextStr = !empty($context) ? ' ' . json_encode($context) : '';
        $logMessage = "[{$timestamp}] [{$levelName}] {$message}{$contextStr}\n";
        
        file_put_contents($this->logFile, $logMessage, FILE_APPEND);
    }
}

// 使用
$logger = new Logger('/tmp/app.log', Logger::LEVEL_INFO);
$logger->info('应用启动');
$logger->error('处理失败', ['file' => 'data.txt']);
```

**说明**：
- 支持不同的日志级别
- 包含时间戳和上下文信息
- 可以配置日志级别

### 示例 4：输入验证

```php
<?php
declare(strict_types=1);

class InputValidator
{
    public static function validateFile(string $file): void
    {
        if (empty($file)) {
            throw new InvalidArgumentException('文件路径不能为空');
        }
        
        if (!file_exists($file)) {
            throw new FileNotFoundException("文件不存在: {$file}");
        }
        
        if (!is_readable($file)) {
            throw new RuntimeException("文件不可读: {$file}");
        }
    }
    
    public static function validateInteger(string $value, ?int $min = null, ?int $max = null): int
    {
        if (!is_numeric($value)) {
            throw new InvalidArgumentException("无效的数字: {$value}");
        }
        
        $intValue = (int)$value;
        
        if ($min !== null && $intValue < $min) {
            throw new InvalidArgumentException("值不能小于 {$min}");
        }
        
        if ($max !== null && $intValue > $max) {
            throw new InvalidArgumentException("值不能大于 {$max}");
        }
        
        return $intValue;
    }
}

// 使用
try {
    InputValidator::validateFile($argv[1]);
    $port = InputValidator::validateInteger($argv[2], 1, 65535);
} catch (InvalidArgumentException $e) {
    echo "参数错误: " . $e->getMessage() . "\n";
    exit(2);
}
```

**说明**：
- 验证输入的有效性
- 提供清晰的错误信息
- 使用异常处理错误

### 示例 5：帮助信息

```php
<?php
declare(strict_types=1);

class CliTool
{
    public function showHelp(): void
    {
        echo "用法: php tool.php <命令> [选项]\n\n";
        echo "命令:\n";
        echo "  process <文件>    处理文件\n";
        echo "  list              列出文件\n";
        echo "  help              显示帮助信息\n\n";
        echo "选项:\n";
        echo "  -v, --verbose     详细输出\n";
        echo "  -h, --help        显示帮助信息\n";
    }
    
    public function run(array $argv): int
    {
        if (count($argv) < 2 || in_array($argv[1], ['-h', '--help', 'help'])) {
            $this->showHelp();
            return 0;
        }
        
        // 处理命令
        return 0;
    }
}
```

**说明**：
- 提供清晰的帮助信息
- 支持 `-h` 和 `--help` 选项
- 格式化的输出

### 示例 6：进度显示

```php
<?php
declare(strict_types=1);

class ProgressBar
{
    private int $total;
    private int $current = 0;
    
    public function __construct(int $total)
    {
        $this->total = $total;
    }
    
    public function update(int $increment = 1): void
    {
        $this->current += $increment;
        $this->display();
    }
    
    public function set(int $current): void
    {
        $this->current = $current;
        $this->display();
    }
    
    private function display(): void
    {
        $percent = ($this->current / $this->total) * 100;
        $barWidth = 50;
        $filled = (int)(($this->current / $this->total) * $barWidth);
        $empty = $barWidth - $filled;
        
        $bar = str_repeat('=', $filled) . str_repeat(' ', $empty);
        printf("\r[%s] %d/%d (%.1f%%)", $bar, $this->current, $this->total, $percent);
        
        if ($this->current >= $this->total) {
            echo "\n";
        }
    }
}

// 使用
$progress = new ProgressBar(100);
for ($i = 0; $i < 100; $i++) {
    // 处理任务
    usleep(50000);
    $progress->update();
}
```

**说明**：
- 显示处理进度
- 提供视觉反馈

## 使用场景

### 场景 1：生产级 CLI 工具

开发生产环境使用的 CLI 工具。

**示例**：

```php
<?php
declare(strict_types=1);

// 完整的 CLI 工具应该包括：
// 1. 清晰的代码组织
// 2. 完善的错误处理
// 3. 日志记录
// 4. 输入验证
// 5. 帮助信息
// 6. 进度显示
// 7. 测试覆盖
```

### 场景 2：错误处理和日志

实现完善的错误处理和日志记录。

**示例**：见"示例 2：错误处理策略"和"示例 3：日志记录"

## 注意事项

### 遵循 PSR 标准

遵循 PSR-1、PSR-12 等编码标准。

**示例**：

```php
<?php
declare(strict_types=1);

// 遵循 PSR 标准
// - 使用命名空间
// - 使用类型声明
// - 遵循命名规范
```

### 代码可读性

编写清晰、易读的代码，添加必要的注释。

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 处理文件命令
 * 
 * @param array $args 命令行参数
 * @return int 退出码
 */
public function processFile(array $args): int
{
    // 验证参数
    // 处理文件
    // 返回结果
}
```

### 文档完整性

提供完整的文档，包括使用说明、示例等。

## 常见问题

### 问题 1：如何组织 CLI 代码？

**回答**：遵循单一职责原则，使用模块化设计，分离配置和逻辑。

**示例**：见"示例 1：代码组织结构"

### 问题 2：如何处理 CLI 错误？

**回答**：使用异常处理，定义错误码，输出用户友好的错误信息，记录详细日志。

**示例**：见"示例 2：错误处理策略"

### 问题 3：如何测试 CLI 工具？

**回答**：编写单元测试、集成测试，使用测试工具模拟命令行输入。

**示例**：

```php
<?php
declare(strict_types=1);

// 单元测试示例
class ProcessCommandTest extends TestCase
{
    public function testProcessFile(): void
    {
        $command = new ProcessCommand();
        $result = $command->execute(['file.txt']);
        $this->assertEquals(0, $result);
    }
}
```

### 问题 4：如何优化 CLI 性能？

**回答**：优化内存使用（流式处理）、优化执行时间（并行处理）、及时释放资源。

## 最佳实践

### 1. 使用框架简化开发

对于复杂的 CLI 工具，使用框架（如 Symfony Console）简化开发。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 Symfony Console 可以自动获得：
// - 命令组织
// - 参数验证
// - 帮助生成
// - 样式输出
```

### 2. 实现完善的错误处理

定义错误码，捕获异常，输出友好的错误信息。

**示例**：见"示例 2：错误处理策略"

### 3. 提供清晰的帮助信息

为工具提供详细的帮助信息，包括用法、选项说明等。

**示例**：见"示例 5：帮助信息"

### 4. 编写完整的文档

提供 README、使用说明、示例等文档。

### 5. 进行充分的测试

编写单元测试、集成测试，确保代码质量。

**示例**：

```php
<?php
declare(strict_types=1);

// 测试应该覆盖：
// - 正常流程
// - 错误情况
// - 边界条件
```

## 对比分析

### 最佳实践 vs 不良实践

| 方面         | 最佳实践                                    | 不良实践                                  |
|:-------------|:--------------------------------------------|:------------------------------------------|
| **代码组织** | ✅ 单一职责、模块化                         | ❌ 所有代码在一个文件中                    |
| **错误处理** | ✅ 异常处理、错误码、友好提示               | ❌ 忽略错误、技术性错误信息                |
| **日志记录** | ✅ 多级别日志、结构化日志                   | ❌ 使用 echo 输出调试信息                  |
| **输入验证** | ✅ 验证所有输入                             | ❌ 假设输入总是有效的                      |
| **帮助信息** | ✅ 清晰的帮助信息                           | ❌ 没有帮助信息                            |

## 练习任务

1. **CLI 工具重构**：选择一个现有的 CLI 脚本，按照最佳实践重构。

2. **错误处理框架**：实现一个错误处理框架，统一处理 CLI 错误。

3. **日志系统**：创建一个日志系统，支持多级别日志和日志轮转。

4. **测试套件**：为 CLI 工具编写完整的测试套件。

5. **CLI 工具模板**：创建一个 CLI 工具模板，包含最佳实践的实现。

## 相关章节

- **[4.4.1 CLI 基础与参数处理](section-01-cli-basics.md)**：了解 CLI 编程基础
- **[4.4.5 CLI 工具开发框架](section-05-cli-frameworks.md)**：了解 CLI 框架的使用
- **[4.7.3 错误和异常的最佳实践](../chapter-07-errors/section-03-best-practices.md)**：了解错误处理的最佳实践
- **[4.9.1 日志体系设计](../chapter-09-logging/section-01-logging-system.md)**：了解日志系统的设计

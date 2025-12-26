# 2.13.1 PSR 标准

## 概述

PSR（PHP Standards Recommendations）是 PHP 社区制定的编码标准，由 PHP-FIG（PHP Framework Interop Group）组织制定。遵循 PSR 标准可以提高代码质量、团队协作效率和代码可读性。

**注意**：本节仅提供 PSR 标准的简要介绍。详细的 PSR 标准规范、实施方法和最佳实践，请参考 [阶段十二：附言 - 12.1 PSR 标准规范](../../stage-12-appendix/chapter-01-psr-standards/readme.md)。

## 核心标准

### PSR-1 基础编码标准

**基本要求**：
- 纯 PHP 文件必须使用 `<?php` 标签，不得使用结束标签 `?>`
- 必须使用 UTF-8 编码（无 BOM）
- 类名使用 `StudlyCase`（大驼峰）
- 方法名使用 `camelCase`（小驼峰）
- 常量名使用 `UPPER_SNAKE_CASE`（大写下划线）

**详细内容**：详见 [12.1.2 PSR-1 基础编码标准](../../stage-12-appendix/chapter-01-psr-standards/section-02-psr-1.md)

### PSR-4 自动加载标准

**基本要求**：
- 命名空间必须与目录结构对应
- 一个文件一个类，文件名必须与类名匹配
- 通过 Composer 自动加载

**详细内容**：详见 [12.1.4 PSR-4 自动加载标准](../../stage-12-appendix/chapter-01-psr-standards/section-04-psr-4.md)

### PSR-12 编码风格扩展

**基本要求**：
- 使用 4 个空格缩进，不使用 Tab
- 控制结构的左花括号必须在同一行
- 关键字和类型（`true`、`false`、`null`）必须小写
- 不同部分之间使用空行分隔

**详细内容**：详见 [12.1.3 PSR-12 扩展编码风格指南](../../stage-12-appendix/chapter-01-psr-standards/section-03-psr-12.md)

### 其他 PSR 标准

- **PSR-3**：日志接口
- **PSR-7**：HTTP 消息接口
- **PSR-11**：容器接口
- 其他 PSR 标准

**详细内容**：详见 [12.1 PSR 标准规范](../../stage-12-appendix/chapter-01-psr-standards/readme.md)

## 基本示例

```php
<?php
declare(strict_types=1);

namespace App\Example;

class UserService
{
    public const MAX_USERS = 100;
    
    public function getUserName(): string
    {
        return "John";
    }
}
```

**要点**：
- 使用 `<?php` 标签，不使用结束标签 `?>`
- 类名使用 `StudlyCase`，方法名使用 `camelCase`
- 常量名使用 `UPPER_SNAKE_CASE`
- 4 个空格缩进，控制结构左花括号在同一行

## 相关章节

- **2.1.2 文件结构与编码**：了解文件结构要求
- **2.11 文件引入与模块化**：了解 PSR-4 自动加载基础
- **阶段十二：附言 - 12.1 PSR 标准规范**：深入学习 PSR 标准规范
  - [12.1.1 PSR 标准概述](../../stage-12-appendix/chapter-01-psr-standards/section-01-introduction.md)
  - [12.1.2 PSR-1 基础编码标准](../../stage-12-appendix/chapter-01-psr-standards/section-02-psr-1.md)
  - [12.1.3 PSR-12 扩展编码风格指南](../../stage-12-appendix/chapter-01-psr-standards/section-03-psr-12.md)
  - [12.1.4 PSR-4 自动加载标准](../../stage-12-appendix/chapter-01-psr-standards/section-04-psr-4.md)
  - [12.1.5 PSR-3 日志接口](../../stage-12-appendix/chapter-01-psr-standards/section-05-psr-3.md)
  - [12.1.6 PSR-7 HTTP 消息接口](../../stage-12-appendix/chapter-01-psr-standards/section-06-psr-7.md)
  - [12.1.7 PSR-11 容器接口](../../stage-12-appendix/chapter-01-psr-standards/section-07-psr-11.md)
  - [12.1.8 其他 PSR 标准](../../stage-12-appendix/chapter-01-psr-standards/section-08-other-psr.md)
  - [12.1.9 PSR 标准实施](../../stage-12-appendix/chapter-01-psr-standards/section-09-implementation.md)

## 练习任务

1. 创建符合 PSR-1 标准的类，遵循命名规范
2. 配置 PSR-4 自动加载，测试自动加载功能
3. 编写符合 PSR-12 的代码，使用代码格式化工具检查
4. 参考 [阶段十二：附言 - 12.1 PSR 标准规范](../../stage-12-appendix/chapter-01-psr-standards/readme.md) 深入学习

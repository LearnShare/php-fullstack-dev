# 2.13.1 PSR 标准

## 概述

PSR（PHP Standards Recommendations）是 PHP 社区制定的编码标准，由 PHP-FIG（PHP Framework Interop Group）组织维护。遵循 PSR 标准可以提高代码的可读性、可维护性和互操作性，使不同框架和库之间能够更好地协作。

## 核心 PSR 标准

### PSR-1：基本编码标准

PSR-1 定义了 PHP 代码的基本编码标准，包括文件格式、类命名、方法命名等基本规则。

**主要内容**：
- 文件必须使用 `<?php` 或 `<?=` 标签
- 文件必须使用 UTF-8 无 BOM 编码
- 类名使用 `StudlyCaps`（大驼峰）
- 方法名使用 `camelCase`（小驼峰）

> **详细内容**：请参考 [10.1.2 PSR-1 基础编码标准](../../../stage-10-appendix/chapter-01-psr-standards/section-02-psr-1.md)

### PSR-4：自动加载标准

PSR-4 定义了自动加载标准，规定了类名与文件路径的映射关系。

**主要内容**：
- 完全限定的类名格式规范
- 命名空间与目录结构的对应关系
- Composer 自动加载配置

> **详细内容**：请参考 [10.1.4 PSR-4 自动加载标准](../../../stage-10-appendix/chapter-01-psr-standards/section-04-psr-4.md)
>
> **使用说明**：关于如何在 Composer 中配置和使用 PSR-4 自动加载，请参考 [1.3.2 Composer 自动加载](../../stage-01-foundation/chapter-04-toolchain/section-02-composer-autoload.md)

### PSR-12：编码风格扩展

PSR-12 扩展了 PSR-1，定义了更详细的编码风格规范。

**主要内容**：
- 代码缩进（4 个空格）
- 行长度建议（不超过 120 个字符）
- 关键字大小写规范
- 类和方法声明格式

> **详细内容**：请参考 [10.1.3 PSR-12 扩展编码风格指南](../../../stage-10-appendix/chapter-01-psr-standards/section-03-psr-12.md)

## 其他重要 PSR 标准

### PSR-3：日志接口

定义了日志记录器的通用接口，使不同日志库可以互换使用。

> **详细内容**：请参考 [10.1.5 PSR-3 日志接口](../../../stage-10-appendix/chapter-01-psr-standards/section-05-psr-3.md)

### PSR-7：HTTP 消息接口

定义了 HTTP 请求和响应的标准接口，使不同 HTTP 库可以互换使用。

> **详细内容**：请参考 [10.1.6 PSR-7 HTTP 消息接口](../../../stage-10-appendix/chapter-01-psr-standards/section-06-psr-7.md)

### PSR-11：容器接口

定义了依赖注入容器的标准接口。

> **详细内容**：请参考 [10.1.7 PSR-11 容器接口](../../../stage-10-appendix/chapter-01-psr-standards/section-07-psr-11.md)

### 其他 PSR 标准

还包括 PSR-6（缓存接口）、PSR-13（超媒体链接）、PSR-14（事件调度器）等。

> **详细内容**：请参考 [10.1.8 其他 PSR 标准](../../../stage-10-appendix/chapter-01-psr-standards/section-08-other-psr.md)

## 快速参考

### 基本示例

```php
<?php
// UserService.php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;

class UserService
{
    private UserRepository $repository;
    
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    public function getUserById(int $id): ?User
    {
        return $this->repository->find($id);
    }
}
```

**说明**：
- 文件使用 `<?php` 标签和 `declare(strict_types=1)`
- 类名使用 `StudlyCaps`（`UserService`）
- 方法名使用 `camelCase`（`getUserById`）
- 使用 4 个空格缩进
- 符合 PSR-1 和 PSR-12 规范

## 学习建议

1. **基础学习**：先掌握 PSR-1 和 PSR-12，这是最常用的编码标准。

2. **工具使用**：使用 PHP CS Fixer 或 PHP_CodeSniffer 自动检查和修复代码。

3. **深入学习**：参考附录阶段的详细内容，深入学习各个 PSR 标准的细节。

4. **实践应用**：在实际项目中应用 PSR 标准，逐步形成编码习惯。

## 相关章节

### 附录阶段详细内容

- **[10.1 PSR 标准规范](../../../stage-10-appendix/chapter-01-psr-standards/readme.md)**：PSR 标准完整指南
  - [10.1.1 PSR 标准概述](../../../stage-10-appendix/chapter-01-psr-standards/section-01-introduction.md)
  - [10.1.2 PSR-1 基础编码标准](../../../stage-10-appendix/chapter-01-psr-standards/section-02-psr-1.md)
  - [10.1.3 PSR-12 扩展编码风格指南](../../../stage-10-appendix/chapter-01-psr-standards/section-03-psr-12.md)
  - [10.1.4 PSR-4 自动加载标准](../../../stage-10-appendix/chapter-01-psr-standards/section-04-psr-4.md)
  - [10.1.5 PSR-3 日志接口](../../../stage-10-appendix/chapter-01-psr-standards/section-05-psr-3.md)
  - [10.1.6 PSR-7 HTTP 消息接口](../../../stage-10-appendix/chapter-01-psr-standards/section-06-psr-7.md)
  - [10.1.7 PSR-11 容器接口](../../../stage-10-appendix/chapter-01-psr-standards/section-07-psr-11.md)
  - [10.1.8 其他 PSR 标准](../../../stage-10-appendix/chapter-01-psr-standards/section-08-other-psr.md)
  - [10.1.9 PSR 标准实施](../../../stage-10-appendix/chapter-01-psr-standards/section-09-implementation.md)

### 相关工具章节

- **[1.3.2 Composer 自动加载](../../stage-01-foundation/chapter-04-toolchain/section-02-composer-autoload.md)**：PSR-4 自动加载的配置和使用

## 注意事项

1. **一致性**：在整个项目中保持一致的编码风格。

2. **工具支持**：使用 PHP CS Fixer 或 PHP_CodeSniffer 自动检查。

3. **团队协作**：在团队中统一使用 PSR 标准。

4. **逐步采用**：对于现有项目，可以逐步采用 PSR 标准。

5. **深入学习**：本小节仅提供概述，详细内容请参考附录阶段的 PSR 标准规范。

## 练习

1. 阅读附录阶段的 PSR-1 和 PSR-12 详细内容，理解编码规范。

2. 配置 PHP CS Fixer，使用 PSR-12 标准格式化代码。

3. 编写一个脚本，检查项目代码是否符合 PSR 标准。

4. 重构现有代码，使其符合 PSR-12 标准。

5. 创建一个代码审查清单，包含 PSR 标准检查项。

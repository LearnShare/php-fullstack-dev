# PHP + MySQL 全栈开发指南

## 开始学习

**如果你是零基础学员，请先阅读：**
- **[全局学习指南](docs/learning-guide.md)**：完整的学习路径、时间估算、检查点、学习方法

**各阶段前置知识要求：**
- 每个阶段的 README 都包含详细的前置知识要求和学习时间估算
- 请在学习前仔细阅读，确保具备必要的知识基础

## 文档说明

本指南采用分阶段、分章节的结构，每个阶段和章节都是独立的文档文件，便于学习和查阅。所有内容面向零基础学员设计，提供详细的概念解释、语法说明、参数列表、完整示例代码和练习任务。

**重要说明**：本指南正在重组中，当前版本为新的结构大纲。实际文档内容正在从旧结构迁移到新结构，请参考 [整改方案](.docs/reviews/structure-reorganization-plan.md) 了解详细的重组计划。

## 目录结构

### 阶段一：基础入门（Foundation）

**定位**：环境搭建、工具配置、基础概念

- [阶段一总览](docs/stage-01-foundation/readme.md)（待创建）
- [1.1 PHP 发展史](docs/stage-01-foundation/chapter-01-history/readme.md)（待创建）
  - [1.1.1 PHP 起源与发展历程](docs/stage-01-foundation/chapter-01-history/section-01-history.md)（待创建）
  - [1.1.2 PHP 生态系统](docs/stage-01-foundation/chapter-01-history/section-02-ecosystem.md)（待创建）
- [1.2 PHP 安装与运行基础](docs/stage-01-foundation/chapter-02-runtime/readme.md)（待创建）
  - [1.2.1 PHP 安装与版本管理](docs/stage-01-foundation/chapter-02-runtime/section-01-installation.md)（待创建）
  - [1.2.2 运行模式与验证](docs/stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)（待创建）
- [1.3 MySQL 环境搭建与工具](docs/stage-01-foundation/chapter-03-mysql/readme.md)（待创建）
  - [1.3.1 MySQL 安装与配置](docs/stage-01-foundation/chapter-03-mysql/section-01-installation.md)（待创建）
  - [1.3.2 客户端工具与命令](docs/stage-01-foundation/chapter-03-mysql/section-02-tools-commands.md)（待创建）
- [1.4 核心工具链](docs/stage-01-foundation/chapter-04-toolchain/readme.md)（待创建）
  - [1.4.1 Composer 基础](docs/stage-01-foundation/chapter-04-toolchain/section-01-composer-basics.md)（待创建）
  - [1.4.2 IDE 与扩展配置](docs/stage-01-foundation/chapter-04-toolchain/section-02-ide-extensions.md)（待创建）
  - [1.4.3 Git 与任务管理](docs/stage-01-foundation/chapter-04-toolchain/section-03-git-tasks.md)（待创建）
  - [1.4.4 容器与环境管理](docs/stage-01-foundation/chapter-04-toolchain/section-04-docker-compose.md)（待创建）
  - [1.4.5 自动化脚本](docs/stage-01-foundation/chapter-04-toolchain/section-05-automation.md)（待创建）
- [1.5 配置、扩展与调试基础](docs/stage-01-foundation/chapter-05-config-debug/readme.md)（待创建）
  - [1.5.1 php.ini 配置](docs/stage-01-foundation/chapter-05-config-debug/section-01-php-ini.md)（待创建）
  - [1.5.2 扩展管理](docs/stage-01-foundation/chapter-05-config-debug/section-02-extensions.md)（待创建）
  - [1.5.3 调试基础介绍](docs/stage-01-foundation/chapter-05-config-debug/section-03-debugging-basics.md)（待创建）

### 阶段二：PHP 语言特性（Language）

**定位**：纯语言特性，不涉及 Web、文件系统、网络等

- [阶段二总览](docs/stage-02-language/readme.md)（待创建）
- [2.1 PHP 基本语法结构](docs/stage-02-language/chapter-01-syntax/readme.md)（待创建）
  - [2.1.1 第一个 PHP 程序：Hello World](docs/stage-02-language/chapter-01-syntax/section-01-hello-world.md)（待创建）
  - [2.1.2 文件结构与编码](docs/stage-02-language/chapter-01-syntax/section-02-file-structure.md)（待创建）
  - [2.1.3 语句与注释](docs/stage-02-language/chapter-01-syntax/section-03-statements-comments.md)（待创建）
  - [2.1.4 文件执行方式](docs/stage-02-language/chapter-01-syntax/section-04-execution.md)（待创建）
  - [2.1.5 常见错误与解决方案](docs/stage-02-language/chapter-01-syntax/section-05-common-errors.md)（待创建）
  - [2.1.6 实践建议与示例](docs/stage-02-language/chapter-01-syntax/section-06-best-practices.md)（待创建）
- [2.2 输出与调试基础](docs/stage-02-language/chapter-02-output/readme.md)（待创建）
  - [2.2.1 输出 API](docs/stage-02-language/chapter-02-output/section-01-output-api.md)（待创建）
  - [2.2.2 格式化占位符详解](docs/stage-02-language/chapter-02-output/section-02-format-placeholders.md)（待创建）
  - [2.2.3 调试函数和策略](docs/stage-02-language/chapter-02-output/section-03-debugging.md)（待创建）
- [2.3 变量与常量](docs/stage-02-language/chapter-03-variables/readme.md)（待创建）
  - [2.3.1 变量基础](docs/stage-02-language/chapter-03-variables/section-01-variable-basics.md)（待创建）
  - [2.3.2 引用与可变变量](docs/stage-02-language/chapter-03-variables/section-02-references.md)（待创建）
  - [2.3.3 常量](docs/stage-02-language/chapter-03-variables/section-03-constants.md)（待创建）
  - [2.3.4 全局与静态](docs/stage-02-language/chapter-03-variables/section-04-scope-static.md)（待创建）
  - [2.3.5 最佳实践与综合练习](docs/stage-02-language/chapter-03-variables/section-05-best-practices.md)（待创建）
- [2.4 数据类型](docs/stage-02-language/chapter-04-types/readme.md)（待创建）
  - [2.4.1 标量类型](docs/stage-02-language/chapter-04-types/section-01-scalar-types.md)（待创建）
  - [2.4.2 复合类型](docs/stage-02-language/chapter-04-types/section-02-composite-types.md)（待创建）
  - [2.4.3 特殊类型](docs/stage-02-language/chapter-04-types/section-03-special-types.md)（待创建）
  - [2.4.4 类型检测](docs/stage-02-language/chapter-04-types/section-04-type-detection.md)（待创建）
  - [2.4.5 类型声明与联合类型](docs/stage-02-language/chapter-04-types/section-05-type-declarations.md)（待创建）
  - [2.4.6 JSON 与数组互转](docs/stage-02-language/chapter-04-types/section-06-json.md)（待创建）
- [2.5 类型转换与比较](docs/stage-02-language/chapter-05-type-casting/readme.md)（待创建）
  - [2.5.1 隐式转换](docs/stage-02-language/chapter-05-type-casting/section-01-implicit-conversion.md)（待创建）
  - [2.5.2 显式转换](docs/stage-02-language/chapter-05-type-casting/section-02-explicit-conversion.md)（待创建）
  - [2.5.3 转换函数](docs/stage-02-language/chapter-05-type-casting/section-03-conversion-functions.md)（待创建）
  - [2.5.4 比较运算符与函数](docs/stage-02-language/chapter-05-type-casting/section-04-comparison.md)（待创建）
- [2.6 表达式与运算符](docs/stage-02-language/chapter-06-expressions/readme.md)（待创建）
  - [2.6.1 算术运算符](docs/stage-02-language/chapter-06-expressions/section-01-arithmetic-operators.md)（待创建）
  - [2.6.2 赋值运算符](docs/stage-02-language/chapter-06-expressions/section-02-assignment-operators.md)（待创建）
  - [2.6.3 逻辑运算符](docs/stage-02-language/chapter-06-expressions/section-03-logical-operators.md)（待创建）
  - [2.6.4 三元运算符与空合并运算符](docs/stage-02-language/chapter-06-expressions/section-04-ternary-null-coalescing.md)（待创建）
  - [2.6.5 match 表达式](docs/stage-02-language/chapter-06-expressions/section-05-match-expression.md)（待创建）
  - [2.6.6 运算符优先级与结合性](docs/stage-02-language/chapter-06-expressions/section-06-operator-precedence.md)（待创建）
- [2.7 字符串操作](docs/stage-02-language/chapter-07-strings/readme.md)（待创建）
  - [2.7.1 字符串创建方式](docs/stage-02-language/chapter-07-strings/section-01-string-creation.md)（待创建）
  - [2.7.2 常用字符串函数](docs/stage-02-language/chapter-07-strings/section-02-string-functions.md)（待创建）
- [2.8 数组完整指南](docs/stage-02-language/chapter-08-arrays/readme.md)（待创建）
  - [2.8.1 数组基础](docs/stage-02-language/chapter-08-arrays/section-01-array-basics.md)（待创建）
  - [2.8.2 数组遍历](docs/stage-02-language/chapter-08-arrays/section-02-array-iteration.md)（待创建）
  - [2.8.3 数组操作函数](docs/stage-02-language/chapter-08-arrays/section-03-array-functions.md)（待创建）
  - [2.8.4 数组排序](docs/stage-02-language/chapter-08-arrays/section-04-array-sorting.md)（待创建）
  - [2.8.5 数组解构](docs/stage-02-language/chapter-08-arrays/section-05-array-destructuring.md)（待创建）
- [2.9 控制结构](docs/stage-02-language/chapter-09-control-flow/readme.md)（待创建）
  - [2.9.1 条件语句](docs/stage-02-language/chapter-09-control-flow/section-01-conditional-statements.md)（待创建）
  - [2.9.2 循环结构](docs/stage-02-language/chapter-09-control-flow/section-02-loops.md)（待创建）
  - [2.9.3 跳转语句](docs/stage-02-language/chapter-09-control-flow/section-03-jump-statements.md)（待创建）
- [2.10 函数与作用域](docs/stage-02-language/chapter-10-functions/readme.md)（待创建）
  - [2.10.1 函数基础](docs/stage-02-language/chapter-10-functions/section-01-function-basics.md)（待创建）
  - [2.10.2 可变参数](docs/stage-02-language/chapter-10-functions/section-02-variable-arguments.md)（待创建）
  - [2.10.3 引用、箭头函数与闭包](docs/stage-02-language/chapter-10-functions/section-03-references-closures.md)（待创建）
  - [2.10.4 匿名函数](docs/stage-02-language/chapter-10-functions/section-04-anonymous-functions.md)（待创建）
  - [2.10.5 可调用类型与内置函数](docs/stage-02-language/chapter-10-functions/section-05-callable-types.md)（待创建）
- [2.11 文件引入与模块化](docs/stage-02-language/chapter-11-modularity/readme.md)（待创建）
  - [2.11.1 模块基础](docs/stage-02-language/chapter-11-modularity/section-01-module-basics.md)（待创建）
  - [2.11.2 include 与 require](docs/stage-02-language/chapter-11-modularity/section-02-include-require.md)（待创建）
  - [2.11.3 使用导入的内容](docs/stage-02-language/chapter-11-modularity/section-03-using-imported.md)（待创建）
  - [2.11.4 避免冲突和循环导入](docs/stage-02-language/chapter-11-modularity/section-04-conflicts-circular.md)（待创建）
  - [2.11.5 路径处理和返回值](docs/stage-02-language/chapter-11-modularity/section-05-paths-returns.md)（待创建）
- [2.12 isset / empty / Null 体系](docs/stage-02-language/chapter-12-null-system/readme.md)（待创建）
  - [2.12.1 isset、empty 与 is_null](docs/stage-02-language/chapter-12-null-system/section-01-isset-empty.md)（待创建）
  - [2.12.2 空合并运算符](docs/stage-02-language/chapter-12-null-system/section-02-null-coalescing.md)（待创建）
- [2.13 代码规范](docs/stage-02-language/chapter-13-standards/readme.md)（待创建）
  - [2.13.1 PSR 标准](docs/stage-02-language/chapter-13-standards/section-01-psr-standards.md)（待创建）
  - [2.13.2 PHPDoc](docs/stage-02-language/chapter-13-standards/section-02-phpdoc.md)（待创建）
  - [2.13.3 静态代码分析](docs/stage-02-language/chapter-13-standards/section-03-static-analysis.md)（待创建）
    - [2.13.3.1 PHPStan 配置与使用](docs/stage-02-language/chapter-13-standards/section-03-static-analysis/subsection-01-phpstan.md)（待创建）
    - [2.13.3.2 Psalm 配置与使用](docs/stage-02-language/chapter-13-standards/section-03-static-analysis/subsection-02-psalm.md)（待创建）
    - [2.13.3.3 静态分析与 CI/CD 集成](docs/stage-02-language/chapter-13-standards/section-03-static-analysis/subsection-03-cicd-integration.md)（待创建）
    - [2.13.3.4 代码质量提升实践](docs/stage-02-language/chapter-13-standards/section-03-static-analysis/subsection-04-best-practices.md)（待创建）
- [2.14 PHP 版本新特性](docs/stage-02-language/chapter-14-php-versions/readme.md)（待创建）
  - [2.14.1 PHP 8.2 新特性与示例](docs/stage-02-language/chapter-14-php-versions/section-01-php82.md)（待创建）
  - [2.14.2 PHP 8.3 新特性与示例](docs/stage-02-language/chapter-14-php-versions/section-02-php83.md)（待创建）
  - [2.14.3 PHP 8.4 新特性与示例](docs/stage-02-language/chapter-14-php-versions/section-03-php84.md)（待创建）
  - [2.14.4 PHP 8.5 新特性与示例](docs/stage-02-language/chapter-14-php-versions/section-04-php85.md)（待创建）
  - [2.14.5 升级指南与最佳实践](docs/stage-02-language/chapter-14-php-versions/section-05-upgrade-best-practices.md)（待创建）

### 阶段三：面向对象编程基础（OOP Basics）

**定位**：基础 OOP 编程（类、对象、继承、接口、Traits、命名空间）

- [阶段三总览](docs/stage-03-oop/readme.md)（待创建）
- [3.1 类、对象与基础 OOP](docs/stage-03-oop/chapter-01-classes/readme.md)（待创建）
  - [3.1.1 类与对象基础](docs/stage-03-oop/chapter-01-classes/section-01-basics.md)（待创建）
  - [3.1.2 可见性修饰符](docs/stage-03-oop/chapter-01-classes/section-02-visibility.md)（待创建）
  - [3.1.3 构造函数与析构函数](docs/stage-03-oop/chapter-01-classes/section-03-constructors-destructors.md)（待创建）
  - [3.1.4 对象克隆与引用](docs/stage-03-oop/chapter-01-classes/section-04-clone-references.md)（待创建）
  - [3.1.5 静态属性与方法、魔术方法](docs/stage-03-oop/chapter-01-classes/section-05-static-magic.md)（待创建）
- [3.2 现代 PHP 8+ OOP 能力](docs/stage-03-oop/chapter-02-modern-oop/readme.md)（待创建）
  - [3.2.1 构造器属性提升](docs/stage-03-oop/chapter-02-modern-oop/section-01-constructor-promotion.md)（待创建）
  - [3.2.2 Readonly 属性](docs/stage-03-oop/chapter-02-modern-oop/section-02-readonly.md)（待创建）
  - [3.2.3 枚举（Enums）](docs/stage-03-oop/chapter-02-modern-oop/section-03-enums.md)（待创建）
  - [3.2.4 属性钩子与静态方法](docs/stage-03-oop/chapter-02-modern-oop/section-04-property-hooks.md)（待创建）
- [3.3 OOP 三大特性](docs/stage-03-oop/chapter-03-oop-features/readme.md)（待创建）
  - [3.3.1 继承（Inheritance）](docs/stage-03-oop/chapter-03-oop-features/section-01-inheritance.md)（待创建）
  - [3.3.2 接口（Interface）](docs/stage-03-oop/chapter-03-oop-features/section-02-interfaces.md)（待创建）
  - [3.3.3 抽象类（Abstract Class）](docs/stage-03-oop/chapter-03-oop-features/section-03-abstract-classes.md)（待创建）
  - [3.3.4 多态（Polymorphism）](docs/stage-03-oop/chapter-03-oop-features/section-04-polymorphism.md)（待创建）
- [3.4 代码模块化与元编程](docs/stage-03-oop/chapter-04-metaprogramming/readme.md)（待创建）
  - [3.4.1 Traits](docs/stage-03-oop/chapter-04-metaprogramming/section-01-traits.md)（待创建）
  - [3.4.2 Attributes（注解）](docs/stage-03-oop/chapter-04-metaprogramming/section-02-attributes.md)（待创建）
- [3.5 命名空间与自动加载](docs/stage-03-oop/chapter-05-namespaces/readme.md)（待创建）
  - [3.5.1 命名空间基础](docs/stage-03-oop/chapter-05-namespaces/section-01-basics.md)（待创建）
  - [3.5.2 use 语句](docs/stage-03-oop/chapter-05-namespaces/section-02-use-statements.md)（待创建）
  - [3.5.3 自动加载基础](docs/stage-03-oop/chapter-05-namespaces/section-03-autoloading-basics.md)（待创建）
  - [3.5.4 Composer 自动加载与 PSR-4](docs/stage-03-oop/chapter-05-namespaces/section-04-composer-psr4.md)（待创建）

### 阶段四：系统编程（System Programming）

**定位**：文件系统、网络、CLI 编程、错误处理、调试、日志、内存管理、加密与序列化等系统级编程

- [阶段四总览](docs/stage-04-system/readme.md)（待创建）
- [4.1 时间与日期处理](docs/stage-04-system/chapter-01-datetime/readme.md)（待创建）
  - [4.1.1 DateTime 基础](docs/stage-04-system/chapter-01-datetime/section-01-datetime-basics.md)（待创建）
  - [4.1.2 时区处理](docs/stage-04-system/chapter-01-datetime/section-02-timezone.md)（待创建）
  - [4.1.3 格式化与解析](docs/stage-04-system/chapter-01-datetime/section-03-formatting-parsing.md)（待创建）
  - [4.1.4 时间计算与比较](docs/stage-04-system/chapter-01-datetime/section-04-calculations-comparisons.md)（待创建）
- [4.2 文件系统操作](docs/stage-04-system/chapter-02-filesystem/readme.md)（待创建）
  - [4.2.1 文件读取](docs/stage-04-system/chapter-02-filesystem/section-01-file-reading.md)（待创建）
  - [4.2.2 文件写入](docs/stage-04-system/chapter-02-filesystem/section-02-file-writing.md)（待创建）
  - [4.2.3 文件操作](docs/stage-04-system/chapter-02-filesystem/section-03-file-operations.md)（待创建）
  - [4.2.4 文件指针操作](docs/stage-04-system/chapter-02-filesystem/section-04-file-pointer.md)（待创建）
  - [4.2.5 二进制文件处理](docs/stage-04-system/chapter-02-filesystem/section-05-binary-files.md)（待创建）
  - [4.2.6 文件锁和临时文件](docs/stage-04-system/chapter-02-filesystem/section-06-file-locks-temp.md)（待创建）
  - [4.2.7 目录操作](docs/stage-04-system/chapter-02-filesystem/section-07-directory-operations.md)（待创建）
  - [4.2.8 文件信息](docs/stage-04-system/chapter-02-filesystem/section-08-file-info.md)（待创建）
  - [4.2.9 流式操作与流包装器](docs/stage-04-system/chapter-02-filesystem/section-09-streaming.md)（待创建）
  - [4.2.10 大文件处理](docs/stage-04-system/chapter-02-filesystem/section-10-large-files.md)（待创建）
  - [4.2.11 文件权限详解](docs/stage-04-system/chapter-02-filesystem/section-11-file-permissions.md)（待创建）
  - [4.2.12 路径处理](docs/stage-04-system/chapter-02-filesystem/section-12-path-handling.md)（待创建）
  - [4.2.13 图片处理与文件操作库](docs/stage-04-system/chapter-02-filesystem/section-13-image-processing.md)（待创建）
- [4.3 网络编程基础](docs/stage-04-system/chapter-03-networking/readme.md)（待创建）
  - [4.3.1 Socket 编程基础](docs/stage-04-system/chapter-03-networking/section-01-socket-basics.md)（待创建）
  - [4.3.2 HTTP 客户端编程](docs/stage-04-system/chapter-03-networking/section-02-http-client.md)（待创建）
  - [4.3.3 网络协议理解](docs/stage-04-system/chapter-03-networking/section-03-network-protocols.md)（待创建）
- [4.4 CLI 编程](docs/stage-04-system/chapter-04-cli/readme.md)（待创建）
  - [4.4.1 CLI 基础与参数处理](docs/stage-04-system/chapter-04-cli/section-01-cli-basics.md)（待创建）
  - [4.4.2 执行外部命令](docs/stage-04-system/chapter-04-cli/section-02-exec-commands.md)（待创建）
  - [4.4.3 交互式 CLI 与输出格式化](docs/stage-04-system/chapter-04-cli/section-03-interactive-cli.md)（待创建）
  - [4.4.4 进程管理与守护进程](docs/stage-04-system/chapter-04-cli/section-04-process-management.md)（待创建）
  - [4.4.5 CLI 工具开发框架](docs/stage-04-system/chapter-04-cli/section-05-cli-frameworks.md)（待创建）
  - [4.4.6 环境变量与系统信息](docs/stage-04-system/chapter-04-cli/section-06-environment-system.md)（待创建）
  - [4.4.7 CLI 最佳实践](docs/stage-04-system/chapter-04-cli/section-07-best-practices.md)（待创建）
- [4.5 内存管理](docs/stage-04-system/chapter-05-memory/readme.md)（待创建）
  - [4.5.1 内存使用监控](docs/stage-04-system/chapter-05-memory/section-01-memory-monitoring.md)（待创建）
  - [4.5.2 内存限制设置](docs/stage-04-system/chapter-05-memory/section-02-memory-limits.md)（待创建）
  - [4.5.3 垃圾回收机制](docs/stage-04-system/chapter-05-memory/section-03-garbage-collection.md)（待创建）
  - [4.5.4 内存优化实践](docs/stage-04-system/chapter-05-memory/section-04-memory-optimization.md)（待创建）
- [4.6 加密、哈希与序列化](docs/stage-04-system/chapter-06-crypto-serialization/readme.md)（待创建）
  - [4.6.1 密码哈希](docs/stage-04-system/chapter-06-crypto-serialization/section-01-password-hashing.md)（待创建）
  - [4.6.2 数据加密](docs/stage-04-system/chapter-06-crypto-serialization/section-02-data-encryption.md)（待创建）
  - [4.6.3 哈希函数](docs/stage-04-system/chapter-06-crypto-serialization/section-03-hash-functions.md)（待创建）
  - [4.6.4 序列化与反序列化](docs/stage-04-system/chapter-06-crypto-serialization/section-04-serialization.md)（待创建）
  - [4.6.5 数据持久化](docs/stage-04-system/chapter-06-crypto-serialization/section-05-data-persistence.md)（待创建）
- [4.7 错误与异常处理](docs/stage-04-system/chapter-07-errors/readme.md)（待创建）
  - [4.7.1 错误处理机制](docs/stage-04-system/chapter-07-errors/section-01-error-handling.md)（待创建）
  - [4.7.2 异常处理机制](docs/stage-04-system/chapter-07-errors/section-02-exceptions.md)（待创建）
  - [4.7.3 错误和异常的最佳实践](docs/stage-04-system/chapter-07-errors/section-03-best-practices.md)（待创建）
- [4.8 调试与问题排查](docs/stage-04-system/chapter-08-debugging/readme.md)（待创建）
  - [4.8.1 常见错误类型与修复流程](docs/stage-04-system/chapter-08-debugging/section-01-common-errors.md)（待创建）
  - [4.8.2 Xdebug 配置与使用](docs/stage-04-system/chapter-08-debugging/section-02-xdebug-config.md)（待创建）
  - [4.8.3 调试技巧与工具链](docs/stage-04-system/chapter-08-debugging/section-03-debugging-techniques.md)（待创建）
  - [4.8.4 排查清单与最佳实践](docs/stage-04-system/chapter-08-debugging/section-04-troubleshooting-checklist.md)（待创建）
- [4.9 日志系统](docs/stage-04-system/chapter-09-logging/readme.md)（待创建）
  - [4.9.1 日志体系设计](docs/stage-04-system/chapter-09-logging/section-01-logging-system.md)（待创建）
  - [4.9.2 结构化日志](docs/stage-04-system/chapter-09-logging/section-02-structured-logs.md)（待创建）
  - [4.9.3 链路追踪](docs/stage-04-system/chapter-09-logging/section-03-tracing.md)（待创建）
  - [4.9.4 日志分析与监控](docs/stage-04-system/chapter-09-logging/section-04-log-analysis.md)（待创建）

### 阶段五：Web/API 开发（Web Development）

**定位**：Web 开发、API 设计、HTTP 协议、会话管理等

- [阶段五总览](docs/stage-05-web/readme.md)（待创建）
- [5.1 Web 交互基础：从请求到响应](docs/stage-05-web/chapter-01-request-response/readme.md)（待创建）
  - [5.1.1 HTTP 请求响应流程](docs/stage-05-web/chapter-01-request-response/section-01-http-flow.md)（待创建）
  - [5.1.2 Web Server 与 PHP-FPM 协作](docs/stage-05-web/chapter-01-request-response/section-02-web-server-fpm.md)（待创建）
  - [5.1.3 Shared Nothing 架构](docs/stage-05-web/chapter-01-request-response/section-03-shared-nothing.md)（待创建）
- [5.2 HTML 渲染与输出控制](docs/stage-05-web/chapter-02-html-rendering/readme.md)（待创建）
  - [5.2.1 HTML 渲染与模板引擎](docs/stage-05-web/chapter-02-html-rendering/section-01-rendering-templates.md)（待创建）
  - [5.2.2 输出缓冲与控制](docs/stage-05-web/chapter-02-html-rendering/section-02-output-buffering.md)（待创建）
- [5.3 超全局变量：Web 输入的核心](docs/stage-05-web/chapter-03-superglobals/readme.md)（待创建）
  - [5.3.1 $_GET、$_POST 与 $_REQUEST](docs/stage-05-web/chapter-03-superglobals/section-01-get-post-request.md)（待创建）
  - [5.3.2 $_SERVER、$_SESSION 与 $_COOKIE](docs/stage-05-web/chapter-03-superglobals/section-02-server-session-cookie.md)（待创建）
  - [5.3.3 $_FILES 与安全处理](docs/stage-05-web/chapter-03-superglobals/section-03-files-security.md)（待创建）
- [5.4 URL 处理](docs/stage-05-web/chapter-04-url-handling/readme.md)（待创建）
  - [5.4.1 URL 解析与构建](docs/stage-05-web/chapter-04-url-handling/section-01-url-parsing.md)（待创建）
  - [5.4.2 URL 编码与解码](docs/stage-05-web/chapter-04-url-handling/section-02-url-encoding.md)（待创建）
  - [5.4.3 查询字符串处理](docs/stage-05-web/chapter-04-url-handling/section-03-query-string.md)（待创建）
  - [5.4.4 URL 路由与重写](docs/stage-05-web/chapter-04-url-handling/section-04-url-routing.md)（待创建）
- [5.5 请求体解析：处理 JSON / API 请求](docs/stage-05-web/chapter-05-json-requests/readme.md)（待创建）
  - [5.5.1 JSON 请求解析](docs/stage-05-web/chapter-05-json-requests/section-01-json-parsing.md)（待创建）
  - [5.5.2 API 请求处理](docs/stage-05-web/chapter-05-json-requests/section-02-api-requests.md)（待创建）
- [5.6 文件上传处理](docs/stage-05-web/chapter-06-file-upload/readme.md)（待创建）
  - [5.6.1 文件上传基础](docs/stage-05-web/chapter-06-file-upload/section-01-upload-basics.md)（待创建）
  - [5.6.2 文件验证与安全](docs/stage-05-web/chapter-06-file-upload/section-02-validation-security.md)（待创建）
  - [5.6.3 文件存储策略](docs/stage-05-web/chapter-06-file-upload/section-03-storage-strategy.md)（待创建）
- [5.7 RESTful API 设计与接口规范](docs/stage-05-web/chapter-07-restful-api/readme.md)（待创建）
  - [5.7.1 RESTful 设计原则](docs/stage-05-web/chapter-07-restful-api/section-01-restful-principles.md)（待创建）
  - [5.7.2 API 路由与版本控制](docs/stage-05-web/chapter-07-restful-api/section-02-routing-versioning.md)（待创建）
  - [5.7.3 API 文档与测试](docs/stage-05-web/chapter-07-restful-api/section-03-documentation-testing.md)（待创建）
  - [5.7.4 API 测试工具](docs/stage-05-web/chapter-07-restful-api/section-04-api-testing.md)（待创建）
  - [5.7.5 API 文档](docs/stage-05-web/chapter-07-restful-api/section-05-api-documentation.md)（待创建）
- [5.8 响应处理与跨域（CORS）](docs/stage-05-web/chapter-08-response-cors/readme.md)（待创建）
  - [5.8.1 响应处理](docs/stage-05-web/chapter-08-response-cors/section-01-response-handling.md)（待创建）
  - [5.8.2 CORS 跨域处理](docs/stage-05-web/chapter-08-response-cors/section-02-cors.md)（待创建）
- [5.9 会话与状态管理](docs/stage-05-web/chapter-09-session/readme.md)（待创建）
  - [5.9.1 Session 基础](docs/stage-05-web/chapter-09-session/section-01-session-basics.md)（待创建）
  - [5.9.2 Cookie 管理](docs/stage-05-web/chapter-09-session/section-02-cookie-management.md)（待创建）
  - [5.9.3 状态管理最佳实践](docs/stage-05-web/chapter-09-session/section-03-state-management.md)（待创建）
- [5.10 鉴权与授权模型（AuthN & AuthZ）](docs/stage-05-web/chapter-10-auth/readme.md)（待创建）
  - [5.10.1 认证基础（AuthN）](docs/stage-05-web/chapter-10-auth/section-01-authentication.md)（待创建）
  - [5.10.2 授权模型（AuthZ）](docs/stage-05-web/chapter-10-auth/section-02-authorization.md)（待创建）
  - [5.10.3 JWT 与 Token](docs/stage-05-web/chapter-10-auth/section-03-jwt-token.md)（待创建）
  - [5.10.4 OAuth2 实现](docs/stage-05-web/chapter-10-auth/section-04-oauth2.md)（待创建）
- [5.11 流量治理与安全](docs/stage-05-web/chapter-11-traffic-security/readme.md)（待创建）
  - [5.11.1 Rate Limiting](docs/stage-05-web/chapter-11-traffic-security/section-01-rate-limiting.md)（待创建）
  - [5.11.2 请求签名与验证](docs/stage-05-web/chapter-11-traffic-security/section-02-request-signing.md)（待创建）
  - [5.11.3 安全最佳实践](docs/stage-05-web/chapter-11-traffic-security/section-03-security-best-practices.md)（待创建）
- [5.12 HTTP 客户端](docs/stage-05-web/chapter-12-http-client/readme.md)（待创建）
  - [5.12.1 Guzzle 基础](docs/stage-05-web/chapter-12-http-client/section-01-guzzle-basics.md)（待创建）
  - [5.12.2 异步请求](docs/stage-05-web/chapter-12-http-client/section-02-async-requests.md)（待创建）
  - [5.12.3 错误处理与重试](docs/stage-05-web/chapter-12-http-client/section-03-error-handling-retry.md)（待创建）
- [5.13 中间件深入](docs/stage-05-web/chapter-13-middleware/readme.md)（待创建）
  - [5.13.1 中间件概述](docs/stage-05-web/chapter-13-middleware/section-01-overview.md)（待创建）
  - [5.13.2 中间件模式](docs/stage-05-web/chapter-13-middleware/section-02-patterns.md)（待创建）
  - [5.13.3 编写自定义中间件](docs/stage-05-web/chapter-13-middleware/section-03-custom.md)（待创建）
  - [5.13.4 中间件最佳实践](docs/stage-05-web/chapter-13-middleware/section-04-best-practices.md)（待创建）
- [5.14 路由设计](docs/stage-05-web/chapter-14-routing/readme.md)（待创建）
  - [5.14.1 路由设计概述](docs/stage-05-web/chapter-14-routing/section-01-overview.md)（待创建）
  - [5.14.2 路由组织与模块化](docs/stage-05-web/chapter-14-routing/section-02-organization.md)（待创建）
  - [5.14.3 动态路由与参数](docs/stage-05-web/chapter-14-routing/section-03-dynamic.md)（待创建）
  - [5.14.4 路由守卫与权限控制](docs/stage-05-web/chapter-14-routing/section-04-guards.md)（待创建）
- [5.15 请求验证与数据校验](docs/stage-05-web/chapter-15-validation/readme.md)（待创建）
  - [5.15.1 请求验证概述](docs/stage-05-web/chapter-15-validation/section-01-overview.md)（待创建）
  - [5.15.2 Symfony Validator](docs/stage-05-web/chapter-15-validation/section-02-symfony-validator.md)（待创建）
  - [5.15.3 Respect/Validation](docs/stage-05-web/chapter-15-validation/section-03-respect-validation.md)（待创建）
  - [5.15.4 自定义验证规则](docs/stage-05-web/chapter-15-validation/section-04-custom-rules.md)（待创建）
- [5.16 错误处理最佳实践（Web 层面）](docs/stage-05-web/chapter-16-error-handling/readme.md)（待创建）
  - [5.16.1 错误处理概述](docs/stage-05-web/chapter-16-error-handling/section-01-overview.md)（待创建）
  - [5.16.2 错误分类与自定义错误](docs/stage-05-web/chapter-16-error-handling/section-02-error-types.md)（待创建）
  - [5.16.3 错误中间件](docs/stage-05-web/chapter-16-error-handling/section-03-error-middleware.md)（待创建）
  - [5.16.4 错误响应格式](docs/stage-05-web/chapter-16-error-handling/section-04-response-format.md)（待创建）
- [5.17 WebSocket 服务器](docs/stage-05-web/chapter-17-websocket/readme.md)（待创建）
  - [5.17.1 WebSocket 概述](docs/stage-05-web/chapter-17-websocket/section-01-overview.md)（待创建）
  - [5.17.2 Ratchet](docs/stage-05-web/chapter-17-websocket/section-02-ratchet.md)（待创建）
  - [5.17.3 ReactPHP WebSocket](docs/stage-05-web/chapter-17-websocket/section-03-reactphp.md)（待创建）
  - [5.17.4 WebSocket 最佳实践](docs/stage-05-web/chapter-17-websocket/section-04-best-practices.md)（待创建）
- [5.18 GraphQL API](docs/stage-05-web/chapter-18-graphql/readme.md)（待创建）
  - [5.18.1 GraphQL 概述](docs/stage-05-web/chapter-18-graphql/section-01-overview.md)（待创建）
  - [5.18.2 GraphQLite](docs/stage-05-web/chapter-18-graphql/section-02-graphqlite.md)（待创建）
  - [5.18.3 Lighthouse](docs/stage-05-web/chapter-18-graphql/section-03-lighthouse.md)（待创建）
  - [5.18.4 GraphQL 最佳实践](docs/stage-05-web/chapter-18-graphql/section-04-best-practices.md)（待创建）

### 阶段六：数据库与缓存系统（Database & Cache）

**定位**：数据库设计、操作、优化、缓存系统

- [阶段六总览](docs/stage-06-database/readme.md)（待创建）
- [6.1 数据库设计基础](docs/stage-06-database/chapter-01-database-design/readme.md)（待创建）
  - [6.1.1 数据库设计原则](docs/stage-06-database/chapter-01-database-design/section-01-design-principles.md)（待创建）
  - [6.1.2 范式与反范式](docs/stage-06-database/chapter-01-database-design/section-02-normalization.md)（待创建）
  - [6.1.3 索引设计](docs/stage-06-database/chapter-01-database-design/section-03-index-design.md)（待创建）
- [6.2 PDO 入门与高安全模式](docs/stage-06-database/chapter-02-pdo/readme.md)（待创建）
  - [6.2.1 PDO 基础与连接](docs/stage-06-database/chapter-02-pdo/section-01-pdo-basics.md)（待创建）
  - [6.2.2 预处理语句与防注入](docs/stage-06-database/chapter-02-pdo/section-02-prepared-statements.md)（待创建）
  - [6.2.3 CRUD 操作](docs/stage-06-database/chapter-02-pdo/section-03-crud-operations.md)（待创建）
  - [6.2.4 高级特性](docs/stage-06-database/chapter-02-pdo/section-04-advanced-features.md)（待创建）
- [6.3 并发控制与 MVCC](docs/stage-06-database/chapter-03-concurrency/readme.md)（待创建）
  - [6.3.1 并发控制](docs/stage-06-database/chapter-03-concurrency/section-01-concurrency-control.md)（待创建）
  - [6.3.2 MVCC 机制](docs/stage-06-database/chapter-03-concurrency/section-02-mvcc.md)（待创建）
  - [6.3.3 锁机制](docs/stage-06-database/chapter-03-concurrency/section-03-locking.md)（待创建）
- [6.4 ORM 框架与数据迁移](docs/stage-06-database/chapter-04-orm/readme.md)（待创建）
  - [6.4.1 ORM 基础](docs/stage-06-database/chapter-04-orm/section-01-orm-basics.md)（待创建）
  - [6.4.2 Eloquent 使用](docs/stage-06-database/chapter-04-orm/section-02-eloquent.md)（待创建）
  - [6.4.3 数据迁移](docs/stage-06-database/chapter-04-orm/section-03-migrations.md)（待创建）
  - [6.4.4 关系映射](docs/stage-06-database/chapter-04-orm/section-04-relationships.md)（待创建）
- [6.5 Redis 缓存策略与性能优化](docs/stage-06-database/chapter-05-redis/readme.md)（待创建）
  - [6.5.1 Redis 基础](docs/stage-06-database/chapter-05-redis/section-01-redis-basics.md)（待创建）
  - [6.5.2 缓存策略](docs/stage-06-database/chapter-05-redis/section-02-cache-strategies.md)（待创建）
  - [6.5.3 缓存问题与解决方案](docs/stage-06-database/chapter-05-redis/section-03-cache-problems.md)（待创建）
  - [6.5.4 高级特性](docs/stage-06-database/chapter-05-redis/section-04-advanced-features.md)（待创建）
- [6.6 数据库运维与管理](docs/stage-06-database/chapter-06-operations/readme.md)（待创建）
  - [6.6.1 数据库连接池](docs/stage-06-database/chapter-06-operations/section-01-connection-pool.md)（待创建）
  - [6.6.2 数据库备份与恢复](docs/stage-06-database/chapter-06-operations/section-02-backup-restore.md)（待创建）
  - [6.6.3 消息队列基础](docs/stage-06-database/chapter-06-operations/section-03-message-queue.md)（待创建）
- [6.7 数据库性能监控](docs/stage-06-database/chapter-07-monitoring/readme.md)（待创建）
  - [6.7.1 数据库性能监控概述](docs/stage-06-database/chapter-07-monitoring/section-01-overview.md)（待创建）
  - [6.7.2 慢查询分析](docs/stage-06-database/chapter-07-monitoring/section-02-slow-query.md)（待创建）
  - [6.7.3 性能指标监控](docs/stage-06-database/chapter-07-monitoring/section-03-metrics.md)（待创建）
- [6.8 其他数据库](docs/stage-06-database/chapter-08-other-databases/readme.md)（待创建）
  - [6.8.1 其他关系型数据库](docs/stage-06-database/chapter-08-other-databases/section-01-other-relational.md)（待创建）
  - [6.8.2 NoSQL 数据库](docs/stage-06-database/chapter-08-other-databases/section-02-nosql.md)（待创建）

### 阶段七：高级架构与设计模式（Advanced Architecture & Design Patterns）

**定位**：设计原则、设计模式、架构模式、高级架构设计

- [阶段七总览](docs/stage-07-architecture/readme.md)（待创建）
- [7.1 SOLID 原则](docs/stage-07-architecture/chapter-01-solid/readme.md)（待创建）
  - [7.1.1 SOLID 原则概述](docs/stage-07-architecture/chapter-01-solid/section-01-overview.md)（待创建）
  - [7.1.2 单一职责原则（SRP）](docs/stage-07-architecture/chapter-01-solid/section-02-srp.md)（待创建）
  - [7.1.3 开闭原则（OCP）](docs/stage-07-architecture/chapter-01-solid/section-03-ocp.md)（待创建）
  - [7.1.4 里氏替换原则（LSP）](docs/stage-07-architecture/chapter-01-solid/section-04-lsp.md)（待创建）
  - [7.1.5 接口隔离原则（ISP）](docs/stage-07-architecture/chapter-01-solid/section-05-isp.md)（待创建）
  - [7.1.6 依赖倒置原则（DIP）](docs/stage-07-architecture/chapter-01-solid/section-06-dip.md)（待创建）
  - [7.1.7 SOLID 原则实践](docs/stage-07-architecture/chapter-01-solid/section-07-practice.md)（待创建）
- [7.2 设计模式（Design Patterns）](docs/stage-07-architecture/chapter-02-design-patterns/readme.md)（待创建）
  - [7.2.1 设计模式概述](docs/stage-07-architecture/chapter-02-design-patterns/section-01-overview.md)（待创建）
  - [7.2.2 创建型模式](docs/stage-07-architecture/chapter-02-design-patterns/section-02-creational.md)（待创建）
  - [7.2.3 结构型模式](docs/stage-07-architecture/chapter-02-design-patterns/section-03-structural.md)（待创建）
  - [7.2.4 行为型模式](docs/stage-07-architecture/chapter-02-design-patterns/section-04-behavioral.md)（待创建）
  - [7.2.5 设计模式实践](docs/stage-07-architecture/chapter-02-design-patterns/section-05-practice.md)（待创建）
- [7.3 应用架构模式](docs/stage-07-architecture/chapter-03-architecture-patterns/readme.md)（待创建）
  - [7.3.1 MVC 架构模式](docs/stage-07-architecture/chapter-03-architecture-patterns/section-01-mvc.md)（待创建）
  - [7.3.2 ADR 架构模式](docs/stage-07-architecture/chapter-03-architecture-patterns/section-02-adr.md)（待创建）
  - [7.3.3 模块化单体架构](docs/stage-07-architecture/chapter-03-architecture-patterns/section-03-modular-monolith.md)（待创建）
  - [7.3.4 架构模式对比与选择](docs/stage-07-architecture/chapter-03-architecture-patterns/section-04-comparison.md)（待创建）
- [7.4 异常体系与框架级设计](docs/stage-07-architecture/chapter-04-exceptions/readme.md)（待创建）
  - [7.4.1 异常体系层次](docs/stage-07-architecture/chapter-04-exceptions/section-01-exception-hierarchy.md)（待创建）
  - [7.4.2 自定义异常](docs/stage-07-architecture/chapter-04-exceptions/section-02-custom-exceptions.md)（待创建）
  - [7.4.3 框架级异常设计](docs/stage-07-architecture/chapter-04-exceptions/section-03-framework-design.md)（待创建）
  - [7.4.4 异常处理最佳实践](docs/stage-07-architecture/chapter-04-exceptions/section-04-best-practices.md)（待创建）
- [7.5 六边形架构](docs/stage-07-architecture/chapter-05-hexagonal/readme.md)（待创建）
  - [7.5.1 六边形架构概述](docs/stage-07-architecture/chapter-05-hexagonal/section-01-overview.md)（待创建）
  - [7.5.2 Ports（端口）](docs/stage-07-architecture/chapter-05-hexagonal/section-02-ports.md)（待创建）
  - [7.5.3 Adapters（适配器）](docs/stage-07-architecture/chapter-05-hexagonal/section-03-adapters.md)（待创建）
  - [7.5.4 六边形架构实践](docs/stage-07-architecture/chapter-05-hexagonal/section-04-practice.md)（待创建）
- [7.6 DDD 领域驱动设计初级入门](docs/stage-07-architecture/chapter-06-ddd/readme.md)（待创建）
  - [7.6.1 DDD 概述与分层](docs/stage-07-architecture/chapter-06-ddd/section-01-overview.md)（待创建）
  - [7.6.2 实体与值对象](docs/stage-07-architecture/chapter-06-ddd/section-02-entities-value-objects.md)（待创建）
  - [7.6.3 聚合根（Aggregate Root）](docs/stage-07-architecture/chapter-06-ddd/section-03-aggregates.md)（待创建）
  - [7.6.4 仓储模式](docs/stage-07-architecture/chapter-06-ddd/section-04-repositories.md)（待创建）
  - [7.6.5 领域服务与领域事件](docs/stage-07-architecture/chapter-06-ddd/section-05-domain-services-events.md)（待创建）
  - [7.6.6 DDD 实践指南](docs/stage-07-architecture/chapter-06-ddd/section-06-practice.md)（待创建）
- [7.7 数据流设计进阶](docs/stage-07-architecture/chapter-07-data-flow/readme.md)（待创建）
  - [7.7.1 CQRS 模式](docs/stage-07-architecture/chapter-07-data-flow/section-01-cqrs.md)（待创建）
  - [7.7.2 事件溯源（Event Sourcing）](docs/stage-07-architecture/chapter-07-data-flow/section-02-event-sourcing.md)（待创建）
  - [7.7.3 事件总线（Event Bus）](docs/stage-07-architecture/chapter-07-data-flow/section-03-event-bus.md)（待创建）
  - [7.7.4 数据流设计实践](docs/stage-07-architecture/chapter-07-data-flow/section-04-practice.md)（待创建）
- [7.8 微服务架构基础](docs/stage-07-architecture/chapter-08-microservices/readme.md)（待创建）
  - [7.8.1 微服务概述与服务拆分](docs/stage-07-architecture/chapter-08-microservices/section-01-overview-splitting.md)（待创建）
  - [7.8.2 服务间通信](docs/stage-07-architecture/chapter-08-microservices/section-02-communication.md)（待创建）
  - [7.8.3 微服务挑战与解决方案](docs/stage-07-architecture/chapter-08-microservices/section-03-challenges-solutions.md)（待创建）
  - [7.8.4 微服务实践指南](docs/stage-07-architecture/chapter-08-microservices/section-04-practice.md)（待创建）

### 阶段八：性能优化与安全（Performance & Security）

**定位**：性能优化、安全防护、测试体系

- [阶段八总览](docs/stage-08-performance-security/readme.md)（待创建）
- [8.1 安全防护](docs/stage-08-performance-security/chapter-01-security/readme.md)（待创建）
  - [8.1.1 OWASP Top 10（2025）风险模型与防御](docs/stage-08-performance-security/chapter-01-security/section-01-owasp.md)（待创建）
  - [8.1.2 密码与加密安全](docs/stage-08-performance-security/chapter-01-security/section-02-encryption.md)（待创建）
  - [8.1.3 Secrets 管理](docs/stage-08-performance-security/chapter-01-security/section-03-secrets.md)（待创建）
  - [8.1.4 安全 HTTP 头](docs/stage-08-performance-security/chapter-01-security/section-04-http-headers.md)（待创建）
  - [8.1.5 API 安全](docs/stage-08-performance-security/chapter-01-security/section-05-api-security.md)（待创建）
  - [8.1.6 安全审计与日志](docs/stage-08-performance-security/chapter-01-security/section-06-audit.md)（待创建）
  - [8.1.7 依赖安全](docs/stage-08-performance-security/chapter-01-security/section-07-dependency-security.md)（待创建）
- [8.2 性能优化](docs/stage-08-performance-security/chapter-02-performance/readme.md)（待创建）
  - [8.2.1 PHP 性能优化](docs/stage-08-performance-security/chapter-02-performance/section-01-php-optimization.md)（待创建）
  - [8.2.2 OPcache 配置](docs/stage-08-performance-security/chapter-02-performance/section-02-opcache.md)（待创建）
  - [8.2.3 代码优化技巧](docs/stage-08-performance-security/chapter-02-performance/section-03-code-optimization.md)（待创建）
  - [8.2.4 数据库深度优化](docs/stage-08-performance-security/chapter-02-performance/section-04-db-optimization.md)（待创建）
  - [8.2.5 性能监控](docs/stage-08-performance-security/chapter-02-performance/section-05-monitoring.md)（待创建）
  - [8.2.6 性能测试](docs/stage-08-performance-security/chapter-02-performance/section-06-performance-testing.md)（待创建）
- [8.3 现代 PHP 运行时](docs/stage-08-performance-security/chapter-03-runtime/readme.md)（待创建）
  - [8.3.1 RoadRunner](docs/stage-08-performance-security/chapter-03-runtime/section-01-roadrunner.md)（待创建）
  - [8.3.2 FrankenPHP](docs/stage-08-performance-security/chapter-03-runtime/section-02-frankenphp.md)（待创建）
  - [8.3.3 OpenSwoole](docs/stage-08-performance-security/chapter-03-runtime/section-03-openswoole.md)（待创建）
  - [8.3.4 Laravel Octane](docs/stage-08-performance-security/chapter-03-runtime/section-04-laravel-octane.md)（待创建）
  - [8.3.5 运行时对比与选择](docs/stage-08-performance-security/chapter-03-runtime/section-05-comparison.md)（待创建）
- [8.4 Worker、协程、Event Loop](docs/stage-08-performance-security/chapter-04-workers/readme.md)（待创建）
  - [8.4.1 Worker 模式](docs/stage-08-performance-security/chapter-04-workers/section-01-worker-mode.md)（待创建）
  - [8.4.2 协程（Coroutine）](docs/stage-08-performance-security/chapter-04-workers/section-02-coroutines.md)（待创建）
  - [8.4.3 Event Loop（事件循环）](docs/stage-08-performance-security/chapter-04-workers/section-03-event-loop.md)（待创建）
  - [8.4.4 PHP 8 Fibers](docs/stage-08-performance-security/chapter-04-workers/section-04-fibers.md)（待创建）
  - [8.4.5 实际应用与最佳实践](docs/stage-08-performance-security/chapter-04-workers/section-05-best-practices.md)（待创建）
- [8.5 异步与分布式处理](docs/stage-08-performance-security/chapter-05-async/readme.md)（待创建）
  - [8.5.1 异步处理基础](docs/stage-08-performance-security/chapter-05-async/section-01-async-basics.md)（待创建）
  - [8.5.2 消息队列](docs/stage-08-performance-security/chapter-05-async/section-02-message-queue.md)（待创建）
  - [8.5.3 分布式处理](docs/stage-08-performance-security/chapter-05-async/section-03-distributed.md)（待创建）
- [8.6 测试体系建设](docs/stage-08-performance-security/chapter-06-testing/readme.md)（待创建）
  - [8.6.1 测试金字塔](docs/stage-08-performance-security/chapter-06-testing/section-01-testing-pyramid.md)（待创建）
  - [8.6.2 单元测试](docs/stage-08-performance-security/chapter-06-testing/section-02-unit-testing.md)（待创建）
  - [8.6.3 集成测试](docs/stage-08-performance-security/chapter-06-testing/section-03-integration-testing.md)（待创建）
  - [8.6.4 E2E 测试](docs/stage-08-performance-security/chapter-06-testing/section-04-e2e-testing.md)（待创建）
- [8.7 Mock、Stub、Fakes 深度讲解](docs/stage-08-performance-security/chapter-07-mocking/readme.md)（待创建）
  - [8.7.1 Mock 基础](docs/stage-08-performance-security/chapter-07-mocking/section-01-mock-basics.md)（待创建）
  - [8.7.2 Stub 与 Fakes](docs/stage-08-performance-security/chapter-07-mocking/section-02-stub-fakes.md)（待创建）
  - [8.7.3 测试替身实践](docs/stage-08-performance-security/chapter-07-mocking/section-03-test-doubles.md)（待创建）
- [8.8 TDD（测试驱动开发）](docs/stage-08-performance-security/chapter-08-tdd/readme.md)（待创建）
  - [8.8.1 TDD 概述](docs/stage-08-performance-security/chapter-08-tdd/section-01-overview.md)（待创建）
  - [8.8.2 TDD 工作流程](docs/stage-08-performance-security/chapter-08-tdd/section-02-workflow.md)（待创建）
  - [8.8.3 TDD 实践](docs/stage-08-performance-security/chapter-08-tdd/section-03-practice.md)（待创建）
- [8.9 BDD（行为驱动开发）](docs/stage-08-performance-security/chapter-09-bdd/readme.md)（待创建）
  - [8.9.1 BDD 概述](docs/stage-08-performance-security/chapter-09-bdd/section-01-overview.md)（待创建）
  - [8.9.2 BDD 工具（Behat、Codeception）](docs/stage-08-performance-security/chapter-09-bdd/section-02-tools.md)（待创建）
  - [8.9.3 BDD 实践](docs/stage-08-performance-security/chapter-09-bdd/section-03-practice.md)（待创建）
- [8.10 代码覆盖率深入](docs/stage-08-performance-security/chapter-10-coverage/readme.md)（待创建）
  - [8.10.1 代码覆盖率概述](docs/stage-08-performance-security/chapter-10-coverage/section-01-overview.md)（待创建）
  - [8.10.2 PHPUnit 覆盖率](docs/stage-08-performance-security/chapter-10-coverage/section-02-phpunit.md)（待创建）
  - [8.10.3 Xdebug 覆盖率](docs/stage-08-performance-security/chapter-10-coverage/section-03-xdebug.md)（待创建）
  - [8.10.4 覆盖率报告与分析](docs/stage-08-performance-security/chapter-10-coverage/section-04-reports.md)（待创建）
- [8.11 可观测性体系](docs/stage-08-performance-security/chapter-11-observability/readme.md)（待创建）
  - [8.11.1 可观测性概述](docs/stage-08-performance-security/chapter-11-observability/section-01-overview.md)（待创建）
  - [8.11.2 日志、指标、追踪](docs/stage-08-performance-security/chapter-11-observability/section-02-logs-metrics-traces.md)（待创建）
  - [8.11.3 监控与告警](docs/stage-08-performance-security/chapter-11-observability/section-03-monitoring-alerts.md)（待创建）

### 阶段九：现代框架深度应用（Framework）

**定位**：框架应用、IoC/DI、路由、中间件等

- [阶段九总览](docs/stage-09-frameworks/readme.md)（待创建）
- [9.1 IoC 与 DI 设计模式深度理解](docs/stage-09-frameworks/chapter-01-ioc-di/readme.md)（待创建）
  - [9.1.1 IoC 容器原理](docs/stage-09-frameworks/chapter-01-ioc-di/section-01-ioc-container.md)（待创建）
  - [9.1.2 依赖注入](docs/stage-09-frameworks/chapter-01-ioc-di/section-02-dependency-injection.md)（待创建）
  - [9.1.3 服务提供者](docs/stage-09-frameworks/chapter-01-ioc-di/section-03-service-providers.md)（待创建）
- [9.2 路由、中间件、Pipeline](docs/stage-09-frameworks/chapter-02-middleware/readme.md)（待创建）
  - [9.2.1 路由系统](docs/stage-09-frameworks/chapter-02-middleware/section-01-routing.md)（待创建）
  - [9.2.2 中间件](docs/stage-09-frameworks/chapter-02-middleware/section-02-middleware.md)（待创建）
  - [9.2.3 Pipeline 模式](docs/stage-09-frameworks/chapter-02-middleware/section-03-pipeline.md)（待创建）
- [9.3 Laravel（2025 最新特性）](docs/stage-09-frameworks/chapter-03-laravel/readme.md)（待创建）
  - [9.3.1 Laravel 基础](docs/stage-09-frameworks/chapter-03-laravel/section-01-laravel-basics.md)（待创建）
  - [9.3.2 核心组件](docs/stage-09-frameworks/chapter-03-laravel/section-02-core-components.md)（待创建）
  - [9.3.3 数据库与 ORM](docs/stage-09-frameworks/chapter-03-laravel/section-03-database-orm.md)（待创建）
  - [9.3.4 认证与授权](docs/stage-09-frameworks/chapter-03-laravel/section-04-auth.md)（待创建）
  - [9.3.5 2025 最新特性](docs/stage-09-frameworks/chapter-03-laravel/section-05-latest-features.md)（待创建）
- [9.4 Symfony](docs/stage-09-frameworks/chapter-04-symfony/readme.md)（待创建）
  - [9.4.1 Symfony 基础](docs/stage-09-frameworks/chapter-04-symfony/section-01-symfony-basics.md)（待创建）
  - [9.4.2 组件系统](docs/stage-09-frameworks/chapter-04-symfony/section-02-components.md)（待创建）
  - [9.4.3 最佳实践](docs/stage-09-frameworks/chapter-04-symfony/section-03-best-practices.md)（待创建）
  - [9.4.4 与 Laravel 对比](docs/stage-09-frameworks/chapter-04-symfony/section-04-laravel-comparison.md)（待创建）
- [9.5 其他现代框架](docs/stage-09-frameworks/chapter-05-other-frameworks/readme.md)（待创建）
  - [9.5.1 CodeIgniter](docs/stage-09-frameworks/chapter-05-other-frameworks/section-01-codeigniter.md)（待创建）
  - [9.5.2 Phalcon](docs/stage-09-frameworks/chapter-05-other-frameworks/section-02-phalcon.md)（待创建）
  - [9.5.3 Slim Framework](docs/stage-09-frameworks/chapter-05-other-frameworks/section-03-slim.md)（待创建）
- [9.6 框架对比与选型指南](docs/stage-09-frameworks/chapter-06-comparison/readme.md)（待创建）
  - [9.6.1 框架对比概述](docs/stage-09-frameworks/chapter-06-comparison/section-01-overview.md)（待创建）
  - [9.6.2 框架选型标准](docs/stage-09-frameworks/chapter-06-comparison/section-02-criteria.md)（待创建）
  - [9.6.3 框架选型决策](docs/stage-09-frameworks/chapter-06-comparison/section-03-decision.md)（待创建）
- [9.7 框架性能优化](docs/stage-09-frameworks/chapter-07-performance/readme.md)（待创建）
  - [9.7.1 框架性能优化概述](docs/stage-09-frameworks/chapter-07-performance/section-01-overview.md)（待创建）
  - [9.7.2 Laravel 性能优化](docs/stage-09-frameworks/chapter-07-performance/section-02-laravel.md)（待创建）
  - [9.7.3 Symfony 性能优化](docs/stage-09-frameworks/chapter-07-performance/section-03-symfony.md)（待创建）
- [9.8 框架最佳实践总结](docs/stage-09-frameworks/chapter-08-best-practices/readme.md)（待创建）
  - [9.8.1 框架最佳实践概述](docs/stage-09-frameworks/chapter-08-best-practices/section-01-overview.md)（待创建）
  - [9.8.2 代码组织与结构](docs/stage-09-frameworks/chapter-08-best-practices/section-02-structure.md)（待创建）
  - [9.8.3 性能与安全](docs/stage-09-frameworks/chapter-08-best-practices/section-03-performance-security.md)（待创建）

### 阶段十：部署、云原生与 DevOps（DevOps）

**定位**：部署、容器、CI/CD、监控等

- [阶段十总览](docs/stage-10-devops/readme.md)（待创建）
- [10.1 专业 Dockerfile](docs/stage-10-devops/chapter-01-dockerfile/readme.md)（待创建）
  - [10.1.1 Dockerfile 基础](docs/stage-10-devops/chapter-01-dockerfile/section-01-dockerfile-basics.md)（待创建）
  - [10.1.2 多阶段构建](docs/stage-10-devops/chapter-01-dockerfile/section-02-multi-stage.md)（待创建）
  - [10.1.3 优化技巧](docs/stage-10-devops/chapter-01-dockerfile/section-03-optimization.md)（待创建）
- [10.2 docker-compose 与本地开发环境](docs/stage-10-devops/chapter-02-docker-compose/readme.md)（待创建）
  - [10.2.1 docker-compose 基础](docs/stage-10-devops/chapter-02-docker-compose/section-01-compose-basics.md)（待创建）
  - [10.2.2 本地开发环境](docs/stage-10-devops/chapter-02-docker-compose/section-02-dev-environment.md)（待创建）
- [10.3 镜像安全](docs/stage-10-devops/chapter-03-security/readme.md)（待创建）
  - [10.3.1 镜像安全](docs/stage-10-devops/chapter-03-security/section-01-image-security.md)（待创建）
  - [10.3.2 容器安全](docs/stage-10-devops/chapter-03-security/section-02-container-security.md)（待创建）
- [10.4 部署选择](docs/stage-10-devops/chapter-04-deployment/readme.md)（待创建）
  - [10.4.1 部署选择](docs/stage-10-devops/chapter-04-deployment/section-01-deployment-options.md)（待创建）
  - [10.4.2 传统部署](docs/stage-10-devops/chapter-04-deployment/section-02-traditional-deployment.md)（待创建）
  - [10.4.3 云平台部署](docs/stage-10-devops/chapter-04-deployment/section-03-cloud-deployment.md)（待创建）
- [10.5 Kubernetes (K8s)](docs/stage-10-devops/chapter-05-kubernetes/readme.md)（待创建）
  - [10.5.1 K8s 基础](docs/stage-10-devops/chapter-05-kubernetes/section-01-k8s-basics.md)（待创建）
  - [10.5.2 Pod 与 Service](docs/stage-10-devops/chapter-05-kubernetes/section-02-pod-service.md)（待创建）
  - [10.5.3 Deployment 与 Scaling](docs/stage-10-devops/chapter-05-kubernetes/section-03-deployment-scaling.md)（待创建）
  - [10.5.4 配置与存储](docs/stage-10-devops/chapter-05-kubernetes/section-04-config-storage.md)（待创建）
- [10.6 Serverless PHP](docs/stage-10-devops/chapter-06-serverless/readme.md)（待创建）
  - [10.6.1 Serverless 概念](docs/stage-10-devops/chapter-06-serverless/section-01-serverless-concept.md)（待创建）
  - [10.6.2 PHP Serverless 实践](docs/stage-10-devops/chapter-06-serverless/section-02-php-serverless.md)（待创建）
- [10.7 CI/CD 与 GitHub Actions](docs/stage-10-devops/chapter-07-cicd/readme.md)（待创建）
  - [10.7.1 CI/CD 基础](docs/stage-10-devops/chapter-07-cicd/section-01-cicd-basics.md)（待创建）
  - [10.7.2 GitHub Actions](docs/stage-10-devops/chapter-07-cicd/section-02-github-actions.md)（待创建）
  - [10.7.3 GitLab CI](docs/stage-10-devops/chapter-07-cicd/section-03-gitlab-ci.md)（待创建）
- [10.8 Deploy 策略](docs/stage-10-devops/chapter-08-strategies/readme.md)（待创建）
  - [10.8.1 部署策略](docs/stage-10-devops/chapter-08-strategies/section-01-deployment-strategies.md)（待创建）
  - [10.8.2 蓝绿部署](docs/stage-10-devops/chapter-08-strategies/section-02-blue-green.md)（待创建）
  - [10.8.3 金丝雀发布](docs/stage-10-devops/chapter-08-strategies/section-03-canary.md)（待创建）
- [10.9 GitOps](docs/stage-10-devops/chapter-09-gitops/readme.md)（待创建）
  - [10.9.1 GitOps 概念](docs/stage-10-devops/chapter-09-gitops/section-01-gitops-concept.md)（待创建）
  - [10.9.2 GitOps 实践](docs/stage-10-devops/chapter-09-gitops/section-02-gitops-practice.md)（待创建）
- [10.10 基础设施即代码（IaC）](docs/stage-10-devops/chapter-10-iac/readme.md)（待创建）
  - [10.10.1 基础设施即代码概述](docs/stage-10-devops/chapter-10-iac/section-01-overview.md)（待创建）
  - [10.10.2 Terraform](docs/stage-10-devops/chapter-10-iac/section-02-terraform.md)（待创建）
  - [10.10.3 Pulumi](docs/stage-10-devops/chapter-10-iac/section-03-pulumi.md)（待创建）
- [10.11 回滚与灾难恢复](docs/stage-10-devops/chapter-11-rollback/readme.md)（待创建）
  - [10.11.1 回滚策略](docs/stage-10-devops/chapter-11-rollback/section-01-rollback.md)（待创建）
  - [10.11.2 灾难恢复](docs/stage-10-devops/chapter-11-rollback/section-02-disaster-recovery.md)（待创建）
  - [10.11.3 备份与恢复流程](docs/stage-10-devops/chapter-11-rollback/section-03-backup-restore.md)（待创建）
- [10.12 监控与告警](docs/stage-10-devops/chapter-12-monitoring/readme.md)（待创建）
  - [10.12.1 监控与告警概述](docs/stage-10-devops/chapter-12-monitoring/section-01-overview.md)（待创建）
  - [10.12.2 应用监控](docs/stage-10-devops/chapter-12-monitoring/section-02-application.md)（待创建）
  - [10.12.3 基础设施监控](docs/stage-10-devops/chapter-12-monitoring/section-03-infrastructure.md)（待创建）
  - [10.12.4 告警系统](docs/stage-10-devops/chapter-12-monitoring/section-04-alerts.md)（待创建）
- [10.13 API 网关](docs/stage-10-devops/chapter-13-api-gateway/readme.md)（待创建）
  - [10.13.1 API 网关概述](docs/stage-10-devops/chapter-13-api-gateway/section-01-overview.md)（待创建）
  - [10.13.2 Kong](docs/stage-10-devops/chapter-13-api-gateway/section-02-kong.md)（待创建）
  - [10.13.3 Traefik](docs/stage-10-devops/chapter-13-api-gateway/section-03-traefik.md)（待创建）
- [10.14 负载均衡](docs/stage-10-devops/chapter-14-load-balancing/readme.md)（待创建）
  - [10.14.1 负载均衡概述](docs/stage-10-devops/chapter-14-load-balancing/section-01-overview.md)（待创建）
  - [10.14.2 负载均衡算法](docs/stage-10-devops/chapter-14-load-balancing/section-02-algorithms.md)（待创建）
  - [10.14.3 Nginx 负载均衡](docs/stage-10-devops/chapter-14-load-balancing/section-03-nginx.md)（待创建）
  - [10.14.4 HAProxy](docs/stage-10-devops/chapter-14-load-balancing/section-04-haproxy.md)（待创建）
- [10.15 CDN 深入](docs/stage-10-devops/chapter-15-cdn/readme.md)（待创建）
  - [10.15.1 CDN 原理与架构](docs/stage-10-devops/chapter-15-cdn/section-01-principles.md)（待创建）
  - [10.15.2 CDN 缓存策略](docs/stage-10-devops/chapter-15-cdn/section-02-cache-strategies.md)（待创建）
  - [10.15.3 CDN 性能优化](docs/stage-10-devops/chapter-15-cdn/section-03-optimization.md)（待创建）

### 阶段十一：实战开发项目（Project Practice）

**定位**：综合项目实践

- [阶段十一总览](docs/stage-11-projects/readme.md)
- [11.1 多用户博客系统](docs/stage-11-projects/chapter-01-multi-user-blog/readme.md)
  - [11.1.1 需求分析与功能规划](docs/stage-11-projects/chapter-01-multi-user-blog/section-01-requirements.md)
  - [11.1.2 技术架构设计](docs/stage-11-projects/chapter-01-multi-user-blog/section-02-architecture.md)
  - [11.1.3 技术栈选型](docs/stage-11-projects/chapter-01-multi-user-blog/section-03-technology-stack.md)
  - [11.1.4 数据库设计](docs/stage-11-projects/chapter-01-multi-user-blog/section-04-database-design.md)
  - [11.1.5 API 接口设计](docs/stage-11-projects/chapter-01-multi-user-blog/section-05-api-design.md)
  - [11.1.6 模块划分与实现指南](docs/stage-11-projects/chapter-01-multi-user-blog/section-06-module-design.md)
  - [11.1.7 测试策略与实现](docs/stage-11-projects/chapter-01-multi-user-blog/section-07-testing.md)
  - [11.1.8 部署与运维](docs/stage-11-projects/chapter-01-multi-user-blog/section-08-deployment.md)
- [11.2 网站访问统计服务](docs/stage-11-projects/chapter-02-analytics-service/readme.md)
  - [11.2.1 需求分析与功能规划](docs/stage-11-projects/chapter-02-analytics-service/section-01-requirements.md)
  - [11.2.2 技术架构设计](docs/stage-11-projects/chapter-02-analytics-service/section-02-architecture.md)
  - [11.2.3 技术栈选型](docs/stage-11-projects/chapter-02-analytics-service/section-03-technology-stack.md)
  - [11.2.4 数据库设计](docs/stage-11-projects/chapter-02-analytics-service/section-04-database-design.md)
  - [11.2.5 API 接口设计](docs/stage-11-projects/chapter-02-analytics-service/section-05-api-design.md)
  - [11.2.6 模块划分与实现指南](docs/stage-11-projects/chapter-02-analytics-service/section-06-module-design.md)
  - [11.2.7 测试策略与实现](docs/stage-11-projects/chapter-02-analytics-service/section-07-testing.md)
  - [11.2.8 部署与运维](docs/stage-11-projects/chapter-02-analytics-service/section-08-deployment.md)

### 阶段十二：附言（Appendix）

**定位**：补充参考资料

- [阶段十二总览](docs/stage-12-appendix/readme.md)（待创建）
- [12.1 PSR 标准规范](docs/stage-12-appendix/chapter-01-psr-standards/readme.md)（待创建）
  - [12.1.1 PSR 标准概述](docs/stage-12-appendix/chapter-01-psr-standards/section-01-introduction.md)（待创建）
  - [12.1.2 PSR-1 基础编码标准](docs/stage-12-appendix/chapter-01-psr-standards/section-02-psr-1.md)（待创建）
  - [12.1.3 PSR-12 扩展编码风格指南](docs/stage-12-appendix/chapter-01-psr-standards/section-03-psr-12.md)（待创建）
  - [12.1.4 PSR-4 自动加载标准](docs/stage-12-appendix/chapter-01-psr-standards/section-04-psr-4.md)（待创建）
  - [12.1.5 PSR-3 日志接口](docs/stage-12-appendix/chapter-01-psr-standards/section-05-psr-3.md)（待创建）
  - [12.1.6 PSR-7 HTTP 消息接口](docs/stage-12-appendix/chapter-01-psr-standards/section-06-psr-7.md)（待创建）
  - [12.1.7 PSR-11 容器接口](docs/stage-12-appendix/chapter-01-psr-standards/section-07-psr-11.md)（待创建）
  - [12.1.8 其他 PSR 标准](docs/stage-12-appendix/chapter-01-psr-standards/section-08-other-psr.md)（待创建）
  - [12.1.9 PSR 标准实施](docs/stage-12-appendix/chapter-01-psr-standards/section-09-implementation.md)（待创建）
- [12.2 Composer 高级应用](docs/stage-12-appendix/chapter-02-composer-advanced/readme.md)（待创建）
  - [12.2.1 编写和发布 Composer 包](docs/stage-12-appendix/chapter-02-composer-advanced/section-01-composer-packages.md)（待创建）
  - [12.2.2 Composer 脚本与插件](docs/stage-12-appendix/chapter-02-composer-advanced/section-02-scripts-plugins.md)（待创建）
  - [12.2.3 Composer 最佳实践](docs/stage-12-appendix/chapter-02-composer-advanced/section-03-best-practices.md)（待创建）

## 文档特点

- **面向零基础**：所有内容从基础概念开始，循序渐进
- **内容详实**：提供完整的语法、参数说明和使用示例
- **示例丰富**：每个知识点都配有完整的代码示例
- **实践导向**：每章都包含练习任务，帮助巩固学习
- **结构清晰**：按阶段、章节、小节分层组织，便于查阅
- **标准规范**：附言阶段包含 PSR 标准规范等补充内容，帮助提升代码质量
- **章节类型化**：根据章节类型（概念性、配置性、语法性、工具性、实践性、参考性、最佳实践、对比性）确定必需要素，灵活选择可选要素

**详细的学习路径、时间估算、检查点、学习方法请参考 [全局学习指南](docs/learning-guide.md)。**

## 贡献与反馈

本指南持续更新中，如有问题或建议，欢迎提出反馈。

## 版本信息

- **版本**：3.0（重组中）
- **创建日期**：2025-12-24
- **适用 PHP 版本**：PHP 8.2+
- **重组说明**：当前版本为新的结构大纲，实际文档内容正在从旧结构迁移到新结构

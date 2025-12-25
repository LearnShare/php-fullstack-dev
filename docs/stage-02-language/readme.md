# 阶段二：PHP 语言特性（Language）

本阶段覆盖 PHP 语言从语法、类型系统到代码规范的核心知识，面向零基础学员设计。每章提供概念解释、示例代码、练习建议与常见错误对照，帮助你迅速建立坚实的 PHP 语言基础。建议按顺序学习，每完成一章即编写 3~5 个小练习。

## 定位

纯语言特性，不涉及 Web、文件系统、网络等

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握（阶段一）

- **PHP 安装**：能够安装和配置 PHP 8.2+
- **MySQL 安装**：能够安装和配置 MySQL 8.0+
- **开发工具**：能够使用 IDE、Composer、Git
- **环境验证**：能够运行 PHP 程序

### 不需要掌握

以下知识**不需要**提前掌握，会在学习过程中逐步学习：

- PHP 语言语法（从零开始学习）
- Web 开发知识（将在阶段五学习）
- 文件系统操作（将在阶段四学习）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **环境检查**
   - [ ] PHP 已安装并可以运行
   - [ ] MySQL 已安装并可以连接
   - [ ] IDE 已配置并可以编写代码
   - [ ] 能够运行 Hello World 程序

2. **工具检查**
   - [ ] 能够使用 Composer 安装依赖
   - [ ] 能够使用 Git 管理代码
   - [ ] 能够使用命令行执行 PHP 程序

**如果以上检查点都通过，可以开始阶段二的学习。**

**如果某些检查点未通过，建议：**
- 完成阶段一的所有章节
- 确保环境搭建正确
- 参考阶段一的文档解决问题

## 学习时间估算

**建议学习时间：** 4-6 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含练习和实践时间

**时间分配建议：**
- 语法基础（2.1-2.3）：1 周
- 类型系统（2.4-2.5）：1 周
- 表达式和字符串（2.6-2.7）：1 周
- 数组（2.8）：1 周
- 控制结构和函数（2.9-2.10）：1 周
- 模块化和规范（2.11-2.13）：1 周
- PHP 版本新特性（2.14）：1 周
- 总结和练习：1 周

## 练习建议

### 基础练习（每章完成后）

每完成一章，建议完成 3-5 个小练习，例如：

1. **语法基础**
   - 编写 Hello World 程序
   - 练习变量和常量使用
   - 练习基本的数据类型操作

2. **控制结构**
   - 编写条件判断程序
   - 编写循环程序
   - 练习函数定义和调用

3. **数组和字符串**
   - 练习数组操作
   - 练习字符串处理
   - 练习数组和字符串的组合使用

### 综合练习（阶段完成后）

4. **综合项目**
   - 编写计算器程序
   - 编写待办事项列表程序
   - 编写简单的数据处理程序

**项目要求：**
- 使用本阶段学到的所有知识点
- 代码规范、注释清晰
- 能够处理基本错误

## 章节内容

1. **[2.1 PHP 基本语法结构](chapter-01-syntax/readme.md)**：第一个 PHP 程序、文件结构、语句与注释、文件执行方式、常见错误与解决方案、实践建议与示例。
   - [2.1.1 第一个 PHP 程序：Hello World](chapter-01-syntax/section-01-hello-world.md)
   - [2.1.2 文件结构与编码](chapter-01-syntax/section-02-file-structure.md)
   - [2.1.3 语句与注释](chapter-01-syntax/section-03-statements-comments.md)
   - [2.1.4 文件执行方式](chapter-01-syntax/section-04-execution.md)
   - [2.1.5 常见错误与解决方案](chapter-01-syntax/section-05-common-errors.md)
   - [2.1.6 实践建议与示例](chapter-01-syntax/section-06-best-practices.md)

2. **[2.2 输出与调试基础](chapter-02-output/readme.md)**：输出 API、格式化占位符详解、调试函数和策略（仅 CLI 环境）。
   - [2.2.1 输出 API](chapter-02-output/section-01-output-api.md)
   - [2.2.2 格式化占位符详解](chapter-02-output/section-02-format-placeholders.md)
   - [2.2.3 调试函数和策略](chapter-02-output/section-03-debugging.md)

3. **[2.3 变量与常量](chapter-03-variables/readme.md)**：变量基础、引用与可变变量、常量、全局与静态、最佳实践与综合练习。
   - [2.3.1 变量基础](chapter-03-variables/section-01-variable-basics.md)
   - [2.3.2 引用与可变变量](chapter-03-variables/section-02-references.md)
   - [2.3.3 常量](chapter-03-variables/section-03-constants.md)
   - [2.3.4 全局与静态](chapter-03-variables/section-04-scope-static.md)
   - [2.3.5 最佳实践与综合练习](chapter-03-variables/section-05-best-practices.md)

4. **[2.4 数据类型](chapter-04-types/readme.md)**：标量类型、复合类型、特殊类型、类型检测、类型声明与联合类型、JSON 与数组互转。
   - [2.4.1 标量类型](chapter-04-types/section-01-scalar-types.md)
   - [2.4.2 复合类型](chapter-04-types/section-02-composite-types.md)
   - [2.4.3 特殊类型](chapter-04-types/section-03-special-types.md)
   - [2.4.4 类型检测](chapter-04-types/section-04-type-detection.md)
   - [2.4.5 类型声明与联合类型](chapter-04-types/section-05-type-declarations.md)
   - [2.4.6 JSON 与数组互转](chapter-04-types/section-06-json.md)

5. **[2.5 类型转换与比较](chapter-05-type-casting/readme.md)**：隐式转换、显式转换、转换函数、比较运算符与函数。
   - [2.5.1 隐式转换](chapter-05-type-casting/section-01-implicit-conversion.md)
   - [2.5.2 显式转换](chapter-05-type-casting/section-02-explicit-conversion.md)
   - [2.5.3 转换函数](chapter-05-type-casting/section-03-conversion-functions.md)
   - [2.5.4 比较运算符与函数](chapter-05-type-casting/section-04-comparison.md)

6. **[2.6 表达式与运算符](chapter-06-expressions/readme.md)**：算术运算符、赋值运算符、逻辑运算符、三元运算符与空合并运算符、match 表达式、运算符优先级与结合性。
   - [2.6.1 算术运算符](chapter-06-expressions/section-01-arithmetic-operators.md)
   - [2.6.2 赋值运算符](chapter-06-expressions/section-02-assignment-operators.md)
   - [2.6.3 逻辑运算符](chapter-06-expressions/section-03-logical-operators.md)
   - [2.6.4 三元运算符与空合并运算符](chapter-06-expressions/section-04-ternary-null-coalescing.md)
   - [2.6.5 match 表达式](chapter-06-expressions/section-05-match-expression.md)
   - [2.6.6 运算符优先级与结合性](chapter-06-expressions/section-06-operator-precedence.md)

7. **[2.7 字符串操作](chapter-07-strings/readme.md)**：字符串创建方式、常用字符串函数。
   - [2.7.1 字符串创建方式](chapter-07-strings/section-01-string-creation.md)
   - [2.7.2 常用字符串函数](chapter-07-strings/section-02-string-functions.md)

8. **[2.8 数组完整指南](chapter-08-arrays/readme.md)**：数组基础、数组遍历、数组操作函数、数组排序、数组解构。
   - [2.8.1 数组基础](chapter-08-arrays/section-01-array-basics.md)
   - [2.8.2 数组遍历](chapter-08-arrays/section-02-array-iteration.md)
   - [2.8.3 数组操作函数](chapter-08-arrays/section-03-array-functions.md)
   - [2.8.4 数组排序](chapter-08-arrays/section-04-array-sorting.md)
   - [2.8.5 数组解构](chapter-08-arrays/section-05-array-destructuring.md)

9. **[2.9 控制结构](chapter-09-control-flow/readme.md)**：条件语句、循环结构、跳转语句。
   - [2.9.1 条件语句](chapter-09-control-flow/section-01-conditional-statements.md)
   - [2.9.2 循环结构](chapter-09-control-flow/section-02-loops.md)
   - [2.9.3 跳转语句](chapter-09-control-flow/section-03-jump-statements.md)

10. **[2.10 函数与作用域](chapter-10-functions/readme.md)**：函数基础、可变参数、引用、箭头函数与闭包、匿名函数、可调用类型与内置函数。
    - [2.10.1 函数基础](chapter-10-functions/section-01-function-basics.md)
    - [2.10.2 可变参数](chapter-10-functions/section-02-variable-arguments.md)
    - [2.10.3 引用、箭头函数与闭包](chapter-10-functions/section-03-references-closures.md)
    - [2.10.4 匿名函数](chapter-10-functions/section-04-anonymous-functions.md)
    - [2.10.5 可调用类型与内置函数](chapter-10-functions/section-05-callable-types.md)

11. **[2.11 文件引入与模块化](chapter-11-modularity/readme.md)**：模块基础、include 与 require、使用导入的内容、避免冲突和循环导入、路径处理和返回值（仅 include/require，不含命名空间）。
    - [2.11.1 模块基础](chapter-11-modularity/section-01-module-basics.md)
    - [2.11.2 include 与 require](chapter-11-modularity/section-02-include-require.md)
    - [2.11.3 使用导入的内容](chapter-11-modularity/section-03-using-imported.md)
    - [2.11.4 避免冲突和循环导入](chapter-11-modularity/section-04-conflicts-circular.md)
    - [2.11.5 路径处理和返回值](chapter-11-modularity/section-05-paths-returns.md)

12. **[2.12 isset / empty / Null 体系](chapter-12-null-system/readme.md)**：isset、empty 与 is_null、空合并运算符。
    - [2.12.1 isset、empty 与 is_null](chapter-12-null-system/section-01-isset-empty.md)
    - [2.12.2 空合并运算符](chapter-12-null-system/section-02-null-coalescing.md)

13. **[2.13 代码规范](chapter-13-standards/readme.md)**：PSR 标准、PHPDoc、静态代码分析。
    - [2.13.1 PSR 标准](chapter-13-standards/section-01-psr-standards.md)
    - [2.13.2 PHPDoc](chapter-13-standards/section-02-phpdoc.md)
    - [2.13.3 静态代码分析](chapter-13-standards/section-03-static-analysis.md)
      - [2.13.3.1 PHPStan 配置与使用](chapter-13-standards/section-03-static-analysis/subsection-01-phpstan.md)
      - [2.13.3.2 Psalm 配置与使用](chapter-13-standards/section-03-static-analysis/subsection-02-psalm.md)
      - [2.13.3.3 静态分析与 CI/CD 集成](chapter-13-standards/section-03-static-analysis/subsection-03-cicd-integration.md)
      - [2.13.3.4 代码质量提升实践](chapter-13-standards/section-03-static-analysis/subsection-04-best-practices.md)

14. **[2.14 PHP 版本新特性](chapter-14-php-versions/readme.md)**：PHP 8.2-8.5 版本新特性与示例、升级指南与最佳实践。
    - [2.14.1 PHP 8.2 新特性与示例](chapter-14-php-versions/section-01-php82.md)
    - [2.14.2 PHP 8.3 新特性与示例](chapter-14-php-versions/section-02-php83.md)
    - [2.14.3 PHP 8.4 新特性与示例](chapter-14-php-versions/section-03-php84.md)
    - [2.14.4 PHP 8.5 新特性与示例](chapter-14-php-versions/section-04-php85.md)
    - [2.14.5 升级指南与最佳实践](chapter-14-php-versions/section-05-upgrade-best-practices.md)

完成本阶段后，你将具备：

- 扎实的 PHP 语言基础，能够独立编写 PHP 程序
- 对类型系统、内置函数及运行机制的系统理解，能准确选择合适的数据结构
- 具备规范意识：使用 PHPDoc、PSR-12，并借助静态分析工具自查
- 形成"阅读 → 实验 → 笔记 → 练习 → 复盘"的学习闭环，提升代码阅读与重构速度

## 相关章节

- **阶段一：基础入门**：环境搭建和工具配置
- **阶段三：面向对象编程基础**：学习 OOP 和命名空间（包含从本阶段移出的命名空间内容）
- **阶段四：系统编程**：学习时间日期处理、错误处理、调试（包含从本阶段移出的相关内容）
- **阶段五：Web/API 开发**：学习超级全局变量（包含从本阶段移出的超级全局变量内容）

# 10.1 PSR 标准规范

## 目标

- 深入理解 PHP-FIG（PHP Framework Interop Group）制定的 PSR 标准。
- 掌握核心 PSR 标准（PSR-1、PSR-12、PSR-4、PSR-3、PSR-7 等）的规范要求。
- 能够在实际项目中应用 PSR 标准，编写符合规范的代码。
- 理解 PSR 标准对 PHP 生态系统的重要意义。

## 章节内容

本章分为多个小节，系统讲解各类 PSR 标准：

1. **[PSR 标准概述](section-01-introduction.md)**：PHP-FIG 组织介绍、标准分类与版本、标准的意义。
2. **[PSR-1 基础编码标准](section-02-psr-1.md)**：文件格式、类命名、方法命名、常量命名。
3. **[PSR-12 扩展编码风格指南](section-03-psr-12.md)**：缩进、花括号、关键字、命名空间、类、方法、属性、控制结构。
4. **[PSR-4 自动加载标准](section-04-psr-4.md)**：命名空间与目录映射、Composer 实现、最佳实践。
5. **[PSR-3 日志接口](section-05-psr-3.md)**：LoggerInterface、日志级别、上下文、Monolog 实现。
6. **[PSR-7 HTTP 消息接口](section-06-psr-7.md)**：Request、Response、Stream、Uri、ServerRequest。
7. **[PSR-11 容器接口](section-07-psr-11.md)**：ContainerInterface、依赖注入容器、服务定位器模式。
8. **[其他 PSR 标准](section-08-other-psr.md)**：PSR-15/16/17/18 等标准概述。
9. **[PSR 标准实施](section-09-implementation.md)**：工具链、CI/CD 集成、代码审查、团队协作。

## 学习建议

1. 按顺序学习各小节，重点掌握 PSR-1、PSR-12 和 PSR-4，这些是最基础和常用的标准。
2. 每个标准都配有完整的代码示例和对比说明，帮助理解规范要求。
3. 结合实际项目练习，使用工具自动检查和修复代码风格。
4. 理解 PSR 标准的意义，不仅仅是代码风格，更是团队协作和生态系统互操作的基础。

## 完成本章后

- 对 PSR 标准体系有全面理解，能够选择和应用合适的标准。
- 编写符合 PSR-1 和 PSR-12 规范的代码，使用工具自动检查和修复。
- 理解 PSR-4 自动加载机制，能够正确配置 Composer autoload。
- 使用 PSR-3 日志接口进行结构化日志记录。
- 理解 PSR-7 HTTP 消息接口，能够在框架间互操作。
- 掌握 PSR-11 容器接口，实现依赖注入和服务定位。
- 在实际项目中应用 PSR 标准，提升代码质量和团队协作效率。

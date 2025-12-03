# 阶段三：面向对象、架构与设计模式

本阶段深入讲解 PHP 的面向对象编程（OOP）、现代 PHP 8+ 特性、架构模式与设计原则。从基础的类与对象开始，逐步掌握继承、接口、Traits、命名空间等核心概念，最终能够设计模块化、可扩展、符合 DDD 原则的企业级应用架构。每章提供完整示例、最佳实践与常见陷阱说明，帮助你从过程式编程平滑过渡到面向对象思维。

1. **[类、对象与基础 OOP](chapter-01-classes/README.md)**：类、对象与基础 OOP 概念。
   - [3.1.1 类与对象基础](chapter-01-classes/section-01-basics.md)
   - [3.1.2 可见性修饰符](chapter-01-classes/section-02-visibility.md)
   - [3.1.3 构造函数与析构函数](chapter-01-classes/section-03-constructors-destructors.md)
   - [3.1.4 对象克隆与引用](chapter-01-classes/section-04-clone-references.md)
   - [3.1.5 静态属性与方法、魔术方法](chapter-01-classes/section-05-static-magic.md)

2. **[现代 PHP 8+ OOP 能力](chapter-02-modern-oop/README.md)**：现代 PHP 8+ OOP 能力（构造器属性提升、Readonly、Enums）。
   - [3.2.1 构造器属性提升](chapter-02-modern-oop/section-01-constructor-promotion.md)
   - [3.2.2 Readonly 属性](chapter-02-modern-oop/section-02-readonly.md)
   - [3.2.3 枚举（Enums）](chapter-02-modern-oop/section-03-enums.md)
   - [3.2.4 属性钩子与静态方法](chapter-02-modern-oop/section-04-property-hooks.md)

3. **[OOP 三大特性](chapter-03-oop-features/README.md)**：OOP 三大特性（继承、接口、抽象类）。
   - [3.3.1 继承（Inheritance）](chapter-03-oop-features/section-01-inheritance.md)
   - [3.3.2 接口（Interface）](chapter-03-oop-features/section-02-interfaces.md)
   - [3.3.3 抽象类（Abstract Class）](chapter-03-oop-features/section-03-abstract-classes.md)
   - [3.3.4 多态（Polymorphism）](chapter-03-oop-features/section-04-polymorphism.md)

4. **[代码模块化与元编程](chapter-04-metaprogramming/README.md)**：代码模块化与元编程（Traits、Attributes）。
   - [3.4.1 Traits](chapter-04-metaprogramming/section-01-traits.md)
   - [3.4.2 Attributes（注解）](chapter-04-metaprogramming/section-02-attributes.md)

5. **[命名空间与自动加载](chapter-05-namespaces/README.md)**：命名空间与自动加载（PSR-4）。
   - [3.5.1 命名空间基础](chapter-05-namespaces/section-01-basics.md)
   - [3.5.2 use 语句](chapter-05-namespaces/section-02-use-statements.md)
   - [3.5.3 自动加载基础](chapter-05-namespaces/section-03-autoloading-basics.md)
   - [3.5.4 PSR-4 自动加载标准](chapter-05-namespaces/section-04-psr4.md)

6. **[异常体系与框架级设计](chapter-06-exceptions/README.md)**：异常体系与框架级设计。
   - [3.6.1 异常体系层次](chapter-06-exceptions/section-01-exception-hierarchy.md)
   - [3.6.2 自定义异常](chapter-06-exceptions/section-02-custom-exceptions.md)
   - [3.6.3 框架级异常设计](chapter-06-exceptions/section-03-framework-design.md)

7. **[应用架构模式](chapter-07-architecture/README.md)**：应用架构模式（MVC、ADR）。
   - [3.7.1 MVC 架构模式](chapter-07-architecture/section-01-mvc.md)
   - [3.7.2 ADR 架构模式](chapter-07-architecture/section-02-adr.md)
   - [3.7.3 模块化单体架构](chapter-07-architecture/section-03-modular-monolith.md)

8. **[六边形架构](chapter-08-hexagonal/README.md)**：六边形架构（Ports & Adapters）。
   - [3.8.1 六边形架构概述与 Ports](chapter-08-hexagonal/section-01-overview-ports.md)
   - [3.8.2 Adapters（适配器）](chapter-08-hexagonal/section-02-adapters.md)
   - [3.8.3 六边形架构实践](chapter-08-hexagonal/section-03-practice.md)

9. **[DDD 领域驱动设计初级入门](chapter-09-ddd/README.md)**：DDD 领域驱动设计初级入门。
   - [3.9.1 DDD 概述与分层](chapter-09-ddd/section-01-overview.md)
   - [3.9.2 实体与值对象](chapter-09-ddd/section-02-entities-value-objects.md)
   - [3.9.3 聚合根（Aggregate Root）](chapter-09-ddd/section-03-aggregates.md)
   - [3.9.4 仓储模式](chapter-09-ddd/section-04-repositories.md)

10. **[数据流设计进阶](chapter-10-dataflow/README.md)**：数据流设计进阶（CQRS、事件溯源）。
    - [3.10.1 CQRS 模式](chapter-10-dataflow/section-01-cqrs.md)
    - [3.10.2 事件溯源（Event Sourcing）](chapter-10-dataflow/section-02-event-sourcing.md)
    - [3.10.3 事件总线（Event Bus）](chapter-10-dataflow/section-03-event-bus.md)

11. **[微服务架构](chapter-12-microservices/README.md)**：微服务架构。
    - [3.12.1 微服务概述与服务拆分](chapter-12-microservices/section-01-overview-splitting.md)
    - [3.12.2 服务间通信](chapter-12-microservices/section-02-communication.md)
    - [3.12.3 微服务挑战与解决方案](chapter-12-microservices/section-03-challenges-solutions.md)

12. **[阶段总结](chapter-11-summary/README.md)**：阶段总结与练习指引。

完成阶段三后，你将具备：

- 熟练使用类、对象、继承、接口、Traits 等 OOP 核心特性，能够设计清晰的类层次结构。
- 掌握现代 PHP 8+ 特性（Readonly、Enums、Attributes），编写类型安全、不可变的代码。
- 理解命名空间、PSR-4 自动加载机制，能够组织大型项目的代码结构。
- 具备架构设计能力：MVC、ADR、六边形架构、DDD 基础，能够设计可扩展、可测试的应用架构。
- 形成面向对象思维：封装、多态、依赖注入、单一职责原则，为后续框架学习打下坚实基础。

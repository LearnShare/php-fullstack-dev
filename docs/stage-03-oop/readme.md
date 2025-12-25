# 阶段三：面向对象编程基础（OOP Basics）

本阶段深入讲解 PHP 的面向对象编程（OOP），从基础的类与对象开始，逐步掌握继承、接口、Traits、命名空间等核心概念。每章提供完整示例、最佳实践与常见陷阱说明，帮助你从过程式编程平滑过渡到面向对象思维。

## 定位

基础 OOP 编程（类、对象、继承、接口、Traits、命名空间）

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握（阶段二）

- **PHP 基础语法**：变量、常量、数据类型、运算符、表达式
- **控制结构**：条件语句、循环结构、跳转语句
- **函数**：函数定义、参数传递、返回值、作用域
- **数组和字符串**：数组操作、字符串处理
- **文件引入**：include、require 的基本使用

### 不需要掌握

以下知识**不需要**提前掌握，会在学习过程中逐步学习：

- 面向对象编程（从零开始学习）
- 命名空间（从零开始学习）
- 设计模式（将在后续阶段学习）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **语法检查**
   - [ ] 能够编写包含变量、函数、控制结构的 PHP 程序
   - [ ] 能够使用数组和字符串进行数据处理
   - [ ] 能够使用 include/require 引入文件

2. **理解检查**
   - [ ] 理解函数的作用域和参数传递
   - [ ] 能够阅读和理解中等复杂度的 PHP 代码
   - [ ] 理解代码模块化的基本概念

3. **实践检查**
   - [ ] 完成阶段二的所有练习
   - [ ] 能够独立编写简单的 PHP 程序（如：计算器、待办事项列表）

**如果以上检查点都通过，可以开始阶段三的学习。**

**如果某些检查点未通过，建议：**
- 复习阶段二的相关章节
- 完成阶段二的练习
- 确保理解核心概念后再继续

## 学习时间估算

**建议学习时间：** 4-6 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含练习和实践时间

**时间分配建议：**
- 基础 OOP（3.1-3.2）：2 周
- OOP 特性（3.3）：1 周
- 模块化与元编程（3.4）：1 周
- 命名空间与自动加载（3.5）：1 周
- 阶段总结与实践：1 周

## 练习建议

### 基础练习（每章完成后）

每完成一章，建议完成 3-5 个小练习，例如：

1. **类与对象**
   - 创建简单的类和使用对象
   - 练习构造函数和析构函数
   - 练习可见性修饰符

2. **继承与接口**
   - 创建继承关系
   - 实现接口
   - 练习多态

3. **命名空间**
   - 使用命名空间组织代码
   - 练习 use 语句
   - 配置 Composer 自动加载

### 综合练习（阶段完成后）

4. **综合项目**
   - 设计一个简单的类库（如：图书管理系统）
   - 使用命名空间组织代码
   - 实现继承和接口
   - 使用 Traits 复用代码

**项目要求：**
- 使用本阶段学到的所有知识点
- 代码规范、注释清晰
- 使用 PSR-4 自动加载标准

## 章节内容

1. **[3.1 类、对象与基础 OOP](chapter-01-classes/readme.md)**：类与对象基础、可见性修饰符、构造函数与析构函数、对象克隆与引用、静态属性与方法、魔术方法。
   - [3.1.1 类与对象基础](chapter-01-classes/section-01-basics.md)
   - [3.1.2 可见性修饰符](chapter-01-classes/section-02-visibility.md)
   - [3.1.3 构造函数与析构函数](chapter-01-classes/section-03-constructors-destructors.md)
   - [3.1.4 对象克隆与引用](chapter-01-classes/section-04-clone-references.md)
   - [3.1.5 静态属性与方法、魔术方法](chapter-01-classes/section-05-static-magic.md)

2. **[3.2 现代 PHP 8+ OOP 能力](chapter-02-modern-oop/readme.md)**：构造器属性提升、Readonly 属性、枚举（Enums）、属性钩子与静态方法。
   - [3.2.1 构造器属性提升](chapter-02-modern-oop/section-01-constructor-promotion.md)
   - [3.2.2 Readonly 属性](chapter-02-modern-oop/section-02-readonly.md)
   - [3.2.3 枚举（Enums）](chapter-02-modern-oop/section-03-enums.md)
   - [3.2.4 属性钩子与静态方法](chapter-02-modern-oop/section-04-property-hooks.md)

3. **[3.3 OOP 三大特性](chapter-03-oop-features/readme.md)**：继承（Inheritance）、接口（Interface）、抽象类（Abstract Class）、多态（Polymorphism）。
   - [3.3.1 继承（Inheritance）](chapter-03-oop-features/section-01-inheritance.md)
   - [3.3.2 接口（Interface）](chapter-03-oop-features/section-02-interfaces.md)
   - [3.3.3 抽象类（Abstract Class）](chapter-03-oop-features/section-03-abstract-classes.md)
   - [3.3.4 多态（Polymorphism）](chapter-03-oop-features/section-04-polymorphism.md)

4. **[3.4 代码模块化与元编程](chapter-04-metaprogramming/readme.md)**：Traits、Attributes（注解）。
   - [3.4.1 Traits](chapter-04-metaprogramming/section-01-traits.md)
   - [3.4.2 Attributes（注解）](chapter-04-metaprogramming/section-02-attributes.md)

5. **[3.5 命名空间与自动加载](chapter-05-namespaces/readme.md)**：命名空间基础、use 语句、自动加载基础、Composer 自动加载与 PSR-4。
   - [3.5.1 命名空间基础](chapter-05-namespaces/section-01-basics.md)
   - [3.5.2 use 语句](chapter-05-namespaces/section-02-use-statements.md)
   - [3.5.3 自动加载基础](chapter-05-namespaces/section-03-autoloading-basics.md)
   - [3.5.4 Composer 自动加载与 PSR-4](chapter-05-namespaces/section-04-composer-psr4.md)

## 完成本阶段后，你将具备：

- 扎实的面向对象编程基础，能够设计和实现类、对象、继承、接口
- 理解现代 PHP 8+ 的 OOP 特性，能够使用构造器属性提升、Readonly、枚举等新特性
- 掌握代码模块化方法，能够使用 Traits 和 Attributes 组织代码
- 掌握命名空间和自动加载，能够使用 PSR-4 标准组织项目结构
- 形成面向对象的编程思维，能够将过程式代码重构为面向对象代码

## 相关章节

- **阶段二：PHP 语言特性**：PHP 基础语法和函数
- **阶段四：系统编程**：文件系统、网络、CLI 编程等系统级编程
- **阶段五：Web/API 开发**：Web 开发中的 OOP 应用
- **阶段八：框架**：框架中的 OOP 设计模式

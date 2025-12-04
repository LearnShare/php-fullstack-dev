# 5.6 ORM 框架与数据迁移

## 目标

- 理解 ORM（Object-Relational Mapping）的概念与优势。
- 掌握 Eloquent ORM 的基本用法。
- 了解 Doctrine ORM 的特点与使用。
- 熟悉数据库迁移（Migration）的概念与实现。
- 掌握 Seeder 数据填充的使用。

## 章节内容

本章分为四个独立小节，每节提供详细的概念解释、代码示例和最佳实践：

1. **[ORM 基础](section-01-orm-basics.md)**：什么是 ORM、ORM 优势、ORM vs 原生 SQL、ORM 选择、完整示例。

2. **[Eloquent 使用](section-02-eloquent.md)**：Eloquent 基础、CRUD 操作、查询构建器、关联关系、完整示例。

3. **[数据迁移](section-03-migrations.md)**：迁移基础、创建迁移、运行迁移、回滚迁移、迁移最佳实践、完整示例。

4. **[关系映射](section-04-relationships.md)**：一对一、一对多、多对多、关联查询、关联操作、完整示例。

## 核心概念

- **ORM**：对象关系映射
- **Eloquent**：Laravel 的 ORM
- **迁移**：版本化数据库结构
- **关联**：表之间的关系映射

## 学习建议

1. **重点掌握**：
   - Eloquent 的使用
   - 数据迁移
   - 关联关系

2. **实践练习**：
   - 完成每小节后的练习题目
   - 实现完整的 ORM 模型
   - 创建数据库迁移

## 完成本章后

- 能够使用 ORM 进行数据库操作。
- 掌握数据迁移，能够管理数据库结构。
- 理解关联关系，能够处理复杂查询。
- 具备构建生产级 ORM 应用的能力。

## 相关章节

- **5.2 PDO 入门**：数据库操作基础
- **7.3 Laravel**：Laravel 框架

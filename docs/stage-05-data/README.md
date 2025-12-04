# 阶段五：数据持久化与日志管理

本阶段深入讲解 PHP 与数据库的交互、数据持久化策略、缓存机制和日志管理。从数据库设计基础开始，逐步掌握 PDO、事务处理、现代 MySQL 特性、ORM 框架、Redis 缓存、文件系统操作和日志体系。每章提供完整示例、最佳实践与性能优化建议，帮助你构建高效、可靠的数据层。

1. **[数据库设计基础](chapter-01-database-design/README.md)**：数据库设计基础。
   - [5.1.1 数据库设计原则](chapter-01-database-design/section-01-design-principles.md)
   - [5.1.2 范式与反范式](chapter-01-database-design/section-02-normalization.md)
   - [5.1.3 索引设计](chapter-01-database-design/section-03-index-design.md)
2. **[PDO 入门与高安全模式](chapter-02-pdo/README.md)**：PDO 入门与高安全模式。
   - [5.2.1 PDO 基础与连接](chapter-02-pdo/section-01-pdo-basics.md)
   - [5.2.2 预处理语句与防注入](chapter-02-pdo/section-02-prepared-statements.md)
   - [5.2.3 CRUD 操作](chapter-02-pdo/section-03-crud-operations.md)
   - [5.2.4 高级特性](chapter-02-pdo/section-04-advanced-features.md)
3. **[MySQL 事务处理](chapter-03-transactions/README.md)**：MySQL 事务处理。
   - [5.3.1 事务基础](chapter-03-transactions/section-01-transaction-basics.md)
   - [5.3.2 ACID 特性](chapter-03-transactions/section-02-acid.md)
   - [5.3.3 隔离级别](chapter-03-transactions/section-03-isolation-levels.md)
4. **[现代 MySQL 高级特性](chapter-04-mysql-advanced/README.md)**：现代 MySQL 高级特性。
   - [5.4.1 JSON 字段](chapter-04-mysql-advanced/section-01-json-fields.md)
   - [5.4.2 窗口函数](chapter-04-mysql-advanced/section-02-window-functions.md)
   - [5.4.3 CTE 与递归查询](chapter-04-mysql-advanced/section-03-cte-recursive.md)
   - [5.4.4 性能优化](chapter-04-mysql-advanced/section-04-performance.md)
5. **[并发控制与 MVCC](chapter-05-concurrency/README.md)**：并发控制与 MVCC。
   - [5.5.1 并发控制](chapter-05-concurrency/section-01-concurrency-control.md)
   - [5.5.2 MVCC 机制](chapter-05-concurrency/section-02-mvcc.md)
   - [5.5.3 锁机制](chapter-05-concurrency/section-03-locking.md)
6. **[ORM 框架与数据迁移](chapter-06-orm/README.md)**：ORM 框架与数据迁移。
   - [5.6.1 ORM 基础](chapter-06-orm/section-01-orm-basics.md)
   - [5.6.2 Eloquent 使用](chapter-06-orm/section-02-eloquent.md)
   - [5.6.3 数据迁移](chapter-06-orm/section-03-migrations.md)
   - [5.6.4 关系映射](chapter-06-orm/section-04-relationships.md)
7. **[Redis 缓存策略与性能优化](chapter-07-redis/README.md)**：Redis 缓存策略与性能优化。
   - [5.7.1 Redis 基础](chapter-07-redis/section-01-redis-basics.md)
   - [5.7.2 缓存策略](chapter-07-redis/section-02-cache-strategies.md)
   - [5.7.3 缓存问题与解决方案](chapter-07-redis/section-03-cache-problems.md)
   - [5.7.4 高级特性](chapter-07-redis/section-04-advanced-features.md)
8. **[文件系统流式操作](chapter-08-filesystem/README.md)**：文件系统流式操作。
   - [5.8.1 流式操作](chapter-08-filesystem/section-01-streaming.md)
   - [5.8.2 大文件处理](chapter-08-filesystem/section-02-large-files.md)
9. **[日志体系与监控](chapter-09-logging/README.md)**：日志体系与监控。
   - [5.9.1 日志体系设计](chapter-09-logging/section-01-logging-system.md)
   - [5.9.2 结构化日志](chapter-09-logging/section-02-structured-logs.md)
   - [5.9.3 链路追踪](chapter-09-logging/section-03-tracing.md)

完成阶段五后，你将具备：

- 掌握数据库设计原则，能够设计规范的数据库结构。
- 熟练使用 PDO 进行安全的数据库操作，防止 SQL 注入。
- 理解事务处理、ACID 特性、隔离级别等数据库核心概念。
- 熟悉现代 MySQL 特性（JSON 字段、窗口函数等）。
- 掌握 ORM 框架的使用，能够进行数据迁移和填充。
- 理解 Redis 缓存策略，能够处理缓存穿透、击穿、雪崩等问题。
- 掌握文件系统流式操作，能够高效处理大文件。
- 建立完整的日志体系，支持结构化日志和链路追踪。

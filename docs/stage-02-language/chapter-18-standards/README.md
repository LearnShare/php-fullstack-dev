# 2.18 代码规范与文档

## 目标

- 熟悉 PHP 社区常用规范（PSR 系列、PHPDoc、命名约定）。
- 掌握自动化工具（PHP CS Fixer、PHP_CodeSniffer、PHPStan/Psalm）的使用。
- 建立统一的代码审查与文档约定。

## 命名与结构

- **命名空间**：遵循 PSR-4，目录结构与命名空间一致。
- **类/接口**：`StudlyCase`（`UserService`）。
- **方法/属性**：`camelCase`（`getTotal`）。
- **常量**：`UPPER_SNAKE_CASE`。
- **文件**：单一职责，一个文件一个类。

## PSR 标准速览

- **PSR-1**：基本编码规范
  - 文件使用 UTF-8 无 BOM。
  - 声明命名空间、类时遵循单一职责。
- **PSR-4**：自动加载标准。
- **PSR-12**：编码风格扩展，包含缩进、换行、`use` 排序等。
- **PSR-3**：日志接口。
- **PSR-7/15/17**：HTTP 消息与中间件规范。

## PHPDoc

- 注释示例：

```php
/**
 * @param string $email
 * @param UserRepository $repository
 * @return User|null
 */
function findUser(string $email, UserRepository $repository): ?User
{
    // ...
}
```

- 常用标签：`@param`、`@return`、`@throws`、`@deprecated`、`@see`。
- 通过 PHPStorm、VS Code 插件自动生成。

## 静态分析

- **PHPStan** / **Psalm**：检查类型错误。
  ```bash
  vendor/bin/phpstan analyse src --level=7
  ```
- **Rector**：自动升级/重构。
- **Phan**：另一个静态分析器。

## 代码格式化

- **PHP CS Fixer**、**PHP_CodeSniffer**：
  ```bash
  vendor/bin/php-cs-fixer fix
  vendor/bin/phpcs --standard=PSR12 src
  ```
- 建议在 CI 中执行，并配置 pre-commit 钩子。

## Commit 与审查

- 使用 Conventional Commits：
  - `feat(controller): add user signup`
  - `fix(auth): handle token expiry`
- Code Review 关注点：
  1. 功能是否符合需求。
  2. 代码是否遵循规范。
  3. 安全、性能、可维护性。

## 文档与注释

- README 说明项目目标、环境搭建、运行方式。
- `docs/` 中存放架构说明、API 手册、学习笔记。
- 复杂函数前添加说明性注释，解释算法或业务规则。

## 自动化检查流程

```
composer validate
composer run lint
composer run stan
composer run test
```

- 在 CI（GitHub Actions、GitLab CI）中串联以上步骤，阻止不合规代码进入主分支。

## 练习

1. 配置 `phpcs.xml` 与 `.php_cs.dist`，让团队本地/CI 使用相同规则。
2. 为现有项目添加 PHPStan，设定 Level 5，逐步提升到 Level 8。
3. 制定代码审查清单，包含命名、错误处理、测试覆盖率等检查项。

# 2.13 文件引入与模块化

## 目标

- 理解 `include`、`require` 及其 `_once` 变体的差异与适用场景。
- 掌握基于 Composer autoload 的现代模块化方式。
- 学会组织项目结构、拆分配置、模板、业务逻辑文件。

## include / require

| 关键字      | 失败时行为         |
| :---------- | :----------------- |
| `include`   | 发出警告 (`E_WARNING`)，脚本继续执行 |
| `require`   | 抛出致命错误 (`E_ERROR`)，脚本终止 |
| `include_once` | 仅在第一次包含文件时执行，避免重复定义 |
| `require_once` | 同上，但致命错误行为不变 |

示例：

```php
require_once __DIR__ . '/vendor/autoload.php';
include __DIR__ . '/views/header.php';
```

> **建议**：对关键依赖使用 `require_once`，模板或可选模块用 `include`.

## 动态路径

- 使用 `__DIR__` 组合路径，避免依赖当前工作目录 (`getcwd()`).
- 利用 `dirname(__DIR__)` 向上寻找根目录。

```php
$config = require __DIR__ . '/../config/app.php';
```

## Composer Autoload

- 在 `composer.json` 中配置 `autoload`：
  ```json
  {
    "autoload": {
      "psr-4": {
        "App\\": "src/"
      }
    }
  }
  ```
- 运行 `composer dump-autoload` 生成 `vendor/autoload.php`。
- 项目入口引入：`require __DIR__ . '/../vendor/autoload.php';`

## 组织结构建议

```
project/
├── public/          # Web 根目录
├── src/             # 业务代码
│   ├── Domain/
│   ├── Application/
│   └── Infrastructure/
├── config/
├── resources/views/
├── tests/
└── vendor/
```

## 配置文件拆分

- 使用 `config/*.php` 返回数组：

```php
return [
    'name' => env('APP_NAME', 'Learning Path'),
    'debug' => env('APP_DEBUG', false),
];
```

- 通过 `array_merge` 或自定义 `config()` 助手加载。

## 模板与组件

- 纯 PHP 模板：`include` 头尾文件，传入数据数组。
- Blade/Twig 等模板引擎：根据需要引入依赖。
- 建议在模板中仅做展示逻辑，将业务逻辑放在控制器/服务中。

## 环境配置

- `.env` 文件存储敏感配置，与 `vlucas/phpdotenv` 配合使用。
- 示例：
  ```php
  Dotenv\Dotenv::createImmutable(__DIR__)->load();
  $dbHost = $_ENV['DB_HOST'] ?? 'localhost';
  ```

## 模块化最佳实践

1. **单一职责**：每个文件只关注一类功能（配置、路由、控制器等）。
2. **命名空间**：文件路径与命名空间保持一致，便于 autoload。
3. **依赖注入**：通过构造函数或容器传递依赖，避免 `require` 深层嵌套。
4. **PSR 标准**：遵循 PSR-4 Autoload、PSR-12 代码风格。

## 练习

1. 以 `src/Services`、`src/Repositories`、`src/Http` 组织代码，配置对应的 PSR-4 自动加载，并验证 `composer dump-autoload` 后类可被自动加载。
2. 编写一个 `config()` 助手，支持延迟加载配置文件并缓存结果。
3. 将旧项目中的 `require` 链重构为 Composer autoload + 命名空间结构，确保功能一致。

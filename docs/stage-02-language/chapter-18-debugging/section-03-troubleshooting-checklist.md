# 2.19.3 排查清单与最佳实践

## 概述

当问题爆发时，拥有清晰的排查清单、最佳实践和练习任务，能够显著缩短定位与修复时间。本节将第二节中的技巧流程化，形成可执行的 SOP。

## 常见问题排查清单

### 1. 代码未执行

- [ ] PHP 起始和结束标签是否正确？
- [ ] 文件是否通过 `require`/`autoload` 加载？
- [ ] 分支条件是否命中？使用 `var_dump(__LINE__)` 快速验证。
- [ ] 是否使用了 `@` 抑制错误？如有，请移除并查看真实报错。

### 2. 变量值为空

- [ ] 是否初始化变量或使用空合并运算符？
- [ ] 变量作用域是否在函数内部被重新声明？
- [ ] 是否被 `unset()` 删除？
- [ ] 是否发生隐式类型转换导致 `falsey`？

### 3. 数据库查询失败

- [ ] 数据库连接是否建立？`PDO::errorInfo()` 是什么？
- [ ] SQL 是否存在语法/大小写问题？
- [ ] 参数绑定和数据类型是否匹配？
- [ ] 当前用户是否拥有对应权限？
- [ ] 数据库/框架日志中是否有更详细的错误？

### 4. 性能问题

- [ ] 是否存在 N+1 查询（循环内发起 SQL）？
- [ ] 是否有深层循环或递归？
- [ ] 内存是否持续上涨（`memory_get_usage()`）？
- [ ] 是否启用了 OPcache/缓存？
- [ ] 是否有阻塞型网络调用/外部 API？

## 最佳实践

### 防御性编程

```php
<?php
declare(strict_types=1);

function processUser(array $data): User
{
    if (empty($data['name'])) {
        throw new InvalidArgumentException('Name is required');
    }

    if (!filter_var($data['email'] ?? '', FILTER_VALIDATE_EMAIL)) {
        throw new InvalidArgumentException('Invalid email');
    }

    return new User($data);
}
```

- 所有外部输入都要验证，避免脏数据淹没核心逻辑。

### 详细日志与追踪

```php
try {
    riskyOperation();
} catch (Throwable $e) {
    error_log(sprintf(
        'Error in %s:%d: %s\nStack trace:\n%s',
        $e->getFile(),
        $e->getLine(),
        $e->getMessage(),
        $e->getTraceAsString()
    ));
    throw $e;
}
```

- 日志中包含文件、行号、上下文信息，方便快速定位。

### 类型提示与静态分析

```php
function calculateTotal(array $items): float
{
    $total = 0.0;
    foreach ($items as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    return $total;
}
```

- 使用严格类型、Psalm/PHPStan，开发阶段提前暴露问题。

## 实战练习

1. **统一错误处理器**：实现一个 `ErrorHandler` 类，集中处理异常、错误、日志。
2. **查询日志系统**：记录所有 SQL、耗时、来源堆栈，并输出报表。
3. **性能分析工具**：封装 `PerformanceProfiler`，支持起止计时、内存对比、自动输出。
4. **调试助手库**：封装 `dump()`、`dd()`、`trace_id()` 等日常调试函数。
5. **错误报告系统**：将异常推送到飞书/Slack，包含上下文信息。
6. **自检清单**：将本节清单写入 ISSUE 模板或项目 Wiki，形成复盘标准。

## 结语

- 把“排查清单”落地到团队协作工具中，作为上线前/复盘后的必填项。
- 将调试工具封装到可复用的组件或 Composer 包，减少重复工作。
- 每次线上故障复盘时更新本节清单，持续迭代调试体系。
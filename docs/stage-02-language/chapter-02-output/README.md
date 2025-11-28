# 2.2 输出与调试

## 目标

- 掌握 `echo`、`print`、`printf`、`sprintf` 的语义差异，新手可以在 CLI 和浏览器中正确输出信息。
- 理解 `var_dump`、`print_r`、`debug_zval_dump` 等调试函数，能读懂输出结构。
- 构建“开发输出 vs 生产日志”的思维，避免把调试信息泄漏到用户界面。

## 输出 API

### `echo`

- **语法**：`echo expression [, expression ... ]`
- **参数**：任意数量的字符串或变量，不会自动换行。
- **返回值**：无。
- **场景**：输出 HTML 片段、调试文本、模板数据。
- **示例**：
  ```php
  echo "Hello ", $userName, "!";
  ```

### `print`

- **语法**：`print expression`
- **参数**：仅接受一个表达式。
- **返回值**：`1`（成功），因此可用于条件表达式。
- **场景**：需要在三元、`if` 中嵌入输出时。
- **示例**：
  ```php
  if (print("Ready? ")) {
      echo "Go!";
  }
  ```

### `printf`

- **语法**：`printf(string $format, mixed ...$values): int`
- **参数**：`$format` 含占位符；`$values` 是依次填充的值。
- **返回值**：输出字节数。
- **场景**：直接输出带格式的文本（金额、时间、编号）。
- **示例**：
  ```php
  printf("Order %05d costs %.2f\n", $orderId, $amount);
  ```

### `sprintf`

- **语法**：`sprintf(string $format, mixed ...$values): string`
- **参数**：与 `printf` 相同，但结果以字符串返回。
- **返回值**：格式化后的字符串。
- **场景**：需要拼接、存储、记录日志时。
- **示例**：
  ```php
  $line = sprintf("User %s has %.1f credits", $user->name, $user->balance());
  logger()->info($line);
  ```

### `ob_start`

- **语法**：`ob_start(?callable $callback = null, int $chunk_size = 0, int $flags = PHP_OUTPUT_HANDLER_STDFLAGS): bool`
- **参数**：
  - `$callback` 可对缓冲内容做二次处理。
  - `$chunk_size` 控制缓冲区大小。
  - `$flags` 通常保持默认。
- **返回值**：成功返回 `true`。
- **场景**：捕获模板输出、生成邮件正文、测试渲染结果。

### `ob_get_clean`

- **语法**：`ob_get_clean(): string|false`
- **参数**：无。
- **返回值**：当前缓冲区内容，并清空该缓冲。
- **场景**：与 `ob_start()` 搭配，把输出保存为字符串。
- **示例**：
  ```php
  ob_start();
  include __DIR__ . '/view.php';
  $html = ob_get_clean();
  mail($user->email, 'Welcome', $html);
  ```

常见占位符：

| 占位符     | 含义             | 示例                                         |
| :--------- | :--------------- | :------------------------------------------- |
| `%s`       | 字符串           | `printf("Hello %s", $name);`                 |
| `%d` / `%u` | 有符号/无符号整数 | `printf("ID: %05d", $id);`                   |
| `%f`       | 浮点数           | `printf("Price: %.2f", $price);`             |
| `%02d`     | 左侧补零         | `printf("Order-%02d", $index);`              |
| `%1$`      | 指定参数顺序     | `printf("%2$s, %1$s", "World", "Hello");`    |

组合示例：

```php
$orderId = 7;
$amount = 199.9;
$output = sprintf("Order-%04d total %.2f CNY", $orderId, $amount);
echo $output; // Order-0007 total 199.90 CNY
```

## 调试函数

| 函数                | 语法                                                                 | 参数说明                                         | 用途                 |
| :------------------ | :------------------------------------------------------------------- | :----------------------------------------------- | :------------------- |
| `var_dump`          | `var_dump(mixed ...$values): void`                                   | 可传多个值；输出类型、长度与结构                | 排查类型/结构问题    |
| `print_r`           | `print_r(mixed $value, bool $return = false): string|bool`          | `$return=true` 返回字符串                        | 快速查看数组/对象    |
| `var_export`        | `var_export(mixed $value, bool $return = false): string|bool`       | 生成可执行的 PHP 表达式                          | 记录配置、生成代码   |
| `debug_backtrace`   | `debug_backtrace(int $options = DEBUG_BACKTRACE_PROVIDE_OBJECT, int $limit = 0): array` | `$options` 控制是否包含参数；`$limit` 限制层级 | 查看调用栈           |
| `debug_zval_dump`   | `debug_zval_dump(mixed ...$values): void`                            | 显示 zval 引用计数，学习引用语义                 | 深入理解变量引用     |
| `dump`（VarDumper） | `dump(mixed ...$vars): void`                                         | 需要安装 `symfony/var-dumper`                    | 更友好的 CLI/HTML 输出 |

示例：

```php
$payload = ['id' => 1, 'tags' => ['php', 'mysql']];
var_dump($payload);

$trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5);
print_r($trace, true);
```

## 调试策略

- **开发环境**：开启 `display_errors=On`，允许临时 `var_dump`，但结束前要清理。
- **生产环境**：关闭 `display_errors`，使用 PSR-3 Logger（Monolog）记录调试信息，敏感数据必须脱敏。
- **日志上下文**：`logger()->info('payment.created', ['order_id' => $id, 'amount' => $amount]);`，使用数组上下文便于检索。
- **CI 检查**：配置 PHP CS Fixer 或 Rector，检测 `var_dump`、`dd()` 等调试函数，防止提交到主干。

## 示例

```php
ob_start();
printf("User %s has %.2f credits\n", $user->name, $user->wallet->balance());
$line = ob_get_clean();
logger()->info('wallet.debug', ['output' => $line]);
```

## 练习

1. 使用 `printf` 输出订单号（左侧补零）、总金额（保留两位）、下单时间（`date('Y-m-d H:i:s')`）。
2. 编写 `debug.php`，接受命令行参数，将 `json_decode` 的结果通过 `var_dump` 与 `print_r` 分别展示。
3. 配置一个简单的 `monolog.php`，将 `sprintf` 结果写入 `storage/logs/app.log`，观察日志格式。

通过区分不同输出函数的语义，并结合缓冲、日志与断点调试，可以在保证安全性的前提下高效定位问题。

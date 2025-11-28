# 1.4 配置、扩展与调试

## php.ini

- 使用 `php --ini` 定位“Loaded Configuration File”，CLI 与 FPM 常见不一致，新手可在 `phpinfo()` 页面确认。
- 建议在仓库中保存一份 `php.ini.dev` 模板，标注关键配置与注释来源。
- 常用配置：

| 项目             | 开发值           | 生产值                  | 说明                         |
| :--------------- | :--------------- | :---------------------- | :--------------------------- |
| `display_errors` | On               | Off                     | 线上禁止直接输出错误。       |
| `error_reporting` | `E_ALL`         | `E_ALL & ~E_DEPRECATED` | 开发阶段捕获所有错误。       |
| `memory_limit`   | 512M             | 256M                    | 根据业务调整，避免无限制。   |
| `opcache.enable` | 1                | 1                       | 生产必须开启。               |
| `date.timezone`  | `Asia/Shanghai`  | `Asia/Shanghai`         | 如未设置将触发 Warning。     |

## 扩展管理

- 通过 `php -m` 或 `php --ri xdebug` 检查状态。
- 核心扩展：
  - `pdo_mysql`、`mysqli`（数据库连接）
  - `mbstring`（多字节字符串）
  - `curl`（HTTP 调用）
  - `intl`（国际化）
  - `gd` 或 `imagick`（图像处理）
- 安装方式：
  - Windows：在 `php.ini` 中取消注释 `extension=pdo_mysql` 等条目。
  - macOS/Linux：使用包管理器，如 `brew install php@8.3-imagick` 或 `apt install php8.2-mysql`.
  - 在容器中使用 `docker-php-ext-install`，例如：
  ```bash
  docker-php-ext-install pdo_mysql intl opcache
  ```
  - 如需 PECL 扩展（redis、xdebug），执行 `pecl install redis` 并在 `php.ini` 中通过 `extension=redis` 启用。

## Xdebug 调试

1. 安装：`pecl install xdebug` 或直接选择带 Xdebug 的 Docker 镜像（如 `sail-8.3/app`）。
2. 创建 `xdebug.ini`，示例内容：
   ```
   zend_extension=xdebug
   xdebug.mode=debug,develop,trace
   xdebug.start_with_request=yes
   xdebug.client_host=host.docker.internal
   xdebug.client_port=9003
   xdebug.log=/var/log/php/xdebug.log
   ```
3. IDE 配置：
   - PhpStorm：Preferences → PHP → Servers，绑定项目根路径与 URL。
   - VS Code：安装 PHP Debug 扩展，创建 `.vscode/launch.json`。
4. 测试流程：
   - 在 `public/index.php` 设置断点。
   - 浏览器安装 Xdebug helper 插件，设置为 Debug 模式。
   - 访问页面，IDE 若自动停在断点说明调试成功。
5. 常见问题：
   - CLI 模式未触发：执行 `XDEBUG_MODE=debug php artisan migrate`。
   - Docker 环境 IP 问题：Windows/macOS 使用 `host.docker.internal`，Linux 需填写宿主机 IP。
   - 性能下降：仅在需要时开启 `xdebug.mode=trace` 或 `profile`。

## 配置差异追踪

- 使用 `php -i | grep php.ini` 确保 CLI/FPM 共用配置。
- 通过 `ini_set()` 在入口脚本中追加临时配置，便于差异对比。
- 生产环境将 `php.ini` 存入版本库（或管理在 Ansible、Chef）确保可追溯。

## 调试技巧

- `xdebug_info()`：快速诊断 Xdebug 是否加载。
- `strace -p <pid>`：分析 FPM 进程阻塞。
- `request_slowlog_timeout` + `slowlog`：捕获慢请求调用栈。

## 常用函数与命令语法

| 名称           | 语法                                                   | 参数说明                            | 示例                              |
| :------------- | :----------------------------------------------------- | :---------------------------------- | :-------------------------------- |
| `ini_get`      | `ini_get(string $option): string|false`                | `$option` 为配置项名称               | `ini_get('memory_limit')`         |
| `ini_set`      | `ini_set(string $option, string|int|float|bool $value): string|false` | 返回旧值；失败时返回 `false` | `ini_set('display_errors', '1');` |
| `xdebug_info`  | `xdebug_info(?string $category = null): array|string`  | `$category` 可为 `mode`、`functions` 等 | `var_dump(xdebug_info('mode'));` |
| `php --ri`     | `php --ri extension`                                   | 显示扩展详细配置                     | `php --ri xdebug`                 |
| `strace`       | `strace -p <pid>`                                      | Linux 下追踪系统调用，需 root 权限   | `sudo strace -p 12345 -tt`        |

示例：在引导文件中根据环境切换配置

```php
<?php
declare(strict_types=1);

if (getenv('APP_ENV') === 'local') {
    ini_set('display_errors', '1');
    ini_set('xdebug.mode', 'debug,develop');
}
```

本章确保你能定位真正生效的配置，管理常用扩展，并熟练使用 Xdebug 完成交互式调试。建议将所有配置、安装命令与故障排查步骤记录在团队 Wiki，形成“环境基线”文档。

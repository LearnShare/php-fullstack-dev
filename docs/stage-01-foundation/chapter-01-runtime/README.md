# 1.1 PHP 安装与运行基础

## 目标

- 了解 PHP 在不同平台的安装方式与命令行工具，初学者也能独立完成部署。
- 选择合适的 PHP 8.2+ 版本，并掌握并行维护 CLI、FPM、容器运行时的方法。
- 通过验证命令、示例脚本与排错指南，建立“安装→验证→记录”的闭环流程。

## 版本策略

| 场景     | 推荐版本 | 说明                                               |
| :------- | :------- | :------------------------------------------------- |
| 本地开发 | 8.2 LTS  | 与主流框架兼容，扩展生态成熟。                     |
| 新项目   | 8.3      | 获得最新语法（`readonly class`、改进的 `json_validate` 等）。 |
| 旧项目维护 | 8.1    | 如依赖旧扩展，可暂时保留，但需规划升级窗口。       |

> 提示：确保 CLI `php -v`、FPM `php-fpm -v`、容器镜像标签保持一致，否则会出现“线上/线下不一致”问题。

## 安装方式

### Windows：Herd / Laragon

1. 访问 Herd / Laragon 官网，下载面向新手的一键安装包并完成安装向导。
2. 打开控制面板，勾选所需 PHP 版本与 Nginx/MySQL 组件，点击“Start All”。
3. 打开全新的 CMD / PowerShell，执行 `php -v`，确认输出中包含“CLI (built: …)”。
4. 若需要多个版本并行：
   - 在 Herd 中添加额外版本（如 8.3）。
   - 将 `C:\Program Files\Herd\php83\` 添加到 PATH，或者创建 `php83.bat` 指向该目录。
   - 使用 `php82 -v`、`php83 -v` 分别验证。
5. 记录安装日志（版本、路径、日期）存入 `docs/setup-log.md`，便于团队复现。

### macOS：Homebrew / php-tap

```bash
brew install php@8.3
brew link --overwrite php@8.3
php -v
```

额外安装 `valet` 或 `Laravel Herd` 可快速调试 HTTPS 站点。新手可按以下顺序操作：

1. 安装 Homebrew 后运行 `brew update`，确保软件源可用。
2. 使用 `brew doctor` 检查潜在问题（权限、路径冲突）。
3. 安装完成后将 `export PATH="/opt/homebrew/opt/php@8.3/bin:$PATH"` 写入 `~/.zshrc`。
4. 执行 `php -i | grep "Loaded Configuration"` 找到生效的 `php.ini`。

### Linux：包管理器 / 源码 / Docker

- Debian/Ubuntu：`sudo apt install php8.2 php8.2-fpm`
- RHEL：使用 Remi repo。
- 若需要高度隔离，直接使用 `docker run --rm -it php:8.3-cli`。

> 入门者在 Linux 安装前可先运行 `lsb_release -a` 或 `cat /etc/os-release` 确认发行版，再选择对应命令。

## 运行模式对比

- **CLI**：用于执行脚本、迁移、队列；命令为 `php script.php`。
- **PHP-FPM**：与 Nginx/Apache 配合的常规 Web 模式，需关注 `php-fpm.conf`、pool 配置。
- **内置服务器**：`php -S 127.0.0.1:8000`，适合教学和 Demo。
- **常驻运行时**：FrankenPHP、RoadRunner、Swoole 通过常驻内存提升性能，本章先了解概念，后续章节深入。

## 多版本管理

- `update-alternatives`（Linux）或 `scoop`（Windows）设置优先级。
- 使用 `asdf`、`mise` 等版本管理工具可跨语言统一管理。
- Docker 场景下以 `php:8.3-fpm-alpine` 为基础镜像，借助 `docker compose` 对 FPM、CLI、Queue 容器分拆。

## 运行验证

1. `php -i | head -n 20` 检查配置来源。
2. `php -m` 确认扩展列表。
3. `php -S 127.0.0.1:8000` 启动内置服务器，访问 `phpinfo()` 页验证。
4. 编写 `hello.php`：
   ```php
   <?php
   declare(strict_types=1);
   echo "Env OK: " . PHP_VERSION;
   ```
   运行 `php hello.php`，确保输出正确并记录版本。

## 常用命令速查

| 命令                        | 作用                           | 关键参数                           |
| :-------------------------- | :----------------------------- | :--------------------------------- |
| `php -v`                    | 显示当前 CLI 版本与构建信息   | 无                                 |
| `php -i`                    | 查看完整配置，与 `phpinfo()` 一致 | 可结合 `grep` 过滤                |
| `php -m`                    | 列出已启用扩展               | 无                                 |
| `php-fpm -v`                | 查看 FPM 版本                 | 配置多个版本时用于核对             |
| `where php` / `which php`   | 找到实际可执行文件路径       | Windows 使用 `where`，Unix 使用 `which` |
| `php -S host:port -t public` | 启动内置服务器并指定根目录 | `-t` 指定文档根                    |

> 所有命令建议记录在 `docs/cheatsheet-runtime.md`，并标注“适用平台+操作人+日期”，便于团队溯源。

## 常见故障

- **缺少扩展**：通过包管理器安装 `php8.2-xml`、`pecl install xdebug`。
- **Conflicting PATH**：检查 `where php` 输出，移除旧版本。
- **无法切换 FPM 版本**：在 Nginx 配置中显式指定 `fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;`，重启服务。
- **权限问题**：Windows 需以管理员身份运行安装器；macOS/Linux 先执行 `sudo chown -R $USER /usr/local/php`。
- **证书错误**：`curl` 或 Composer 下载失败时，确认系统证书或使用 `git config --global http.sslBackend schannel`。

## 自检清单

- [ ] 能够描述 CLI 与 FPM 的差异，并说出各自启动方式。
- [ ] 至少掌握一种图形化工具（Herd/Laragon）和一种命令行安装方式（Homebrew/apt）。
- [ ] 知道 `php.ini`、扩展目录与日志所在位置。
- [ ] 具备安装脚本或文档，团队成员可根据文档重现环境。

完成本章后，你应能够在任意操作系统上安装、切换、验证 PHP 运行时，为后续课程提供稳定基础。

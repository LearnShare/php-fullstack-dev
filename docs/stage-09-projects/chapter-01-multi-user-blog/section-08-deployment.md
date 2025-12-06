# 9.1.8 部署与运维

## 一、Docker 部署

### 1.1 Dockerfile 多阶段构建

**文件**：`Dockerfile`

```dockerfile
# 阶段一：构建阶段
FROM php:8.2-fpm AS builder

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd opcache

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www

# 复制 composer 文件
COPY composer.json composer.lock ./

# 安装依赖（生产环境）
RUN composer install --no-dev --optimize-autoloader --no-scripts --no-interaction

# 复制应用代码
COPY . .

# 运行 Composer 脚本
RUN composer dump-autoload --optimize

# 阶段二：运行阶段
FROM php:8.2-fpm

# 安装运行时依赖
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd opcache

# 配置 OPcache
RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.memory_consumption=256" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.max_accelerated_files=20000" >> /usr/local/etc/php/conf.d/opcache.ini

# 设置工作目录
WORKDIR /var/www

# 从构建阶段复制文件
COPY --from=builder /var/www /var/www

# 设置权限
RUN chown -R www-data:www-data /var/www \
    && chmod -R 755 /var/www/storage \
    && chmod -R 755 /var/www/bootstrap/cache

# 暴露端口
EXPOSE 9000

# 启动 PHP-FPM
CMD ["php-fpm"]
```

### 1.2 Docker Compose 配置

**文件**：`docker-compose.yml`

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: blog_app
    restart: unless-stopped
    working_dir: /var/www
    volumes:
      - ./storage:/var/www/storage
      - ./bootstrap/cache:/var/www/bootstrap/cache
    networks:
      - blog-network
    depends_on:
      - mysql
      - redis

  nginx:
    image: nginx:alpine
    container_name: blog_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./:/var/www
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./docker/nginx/ssl:/etc/nginx/ssl
    networks:
      - blog-network
    depends_on:
      - app

  mysql:
    image: mysql:8.0
    container_name: blog_mysql
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_USER: ${DB_USERNAME}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
      - ./backups:/backups
    networks:
      - blog-network
    command: --default-authentication-plugin=mysql_native_password

  redis:
    image: redis:7-alpine
    container_name: blog_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
      - ./docker/redis/redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - blog-network
    command: redis-server /usr/local/etc/redis/redis.conf

  queue:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: blog_queue
    restart: unless-stopped
    working_dir: /var/www
    command: php artisan queue:work --sleep=3 --tries=3 --max-time=3600
    volumes:
      - ./:/var/www
    networks:
      - blog-network
    depends_on:
      - app
      - mysql
      - redis

volumes:
  mysql-data:
  redis-data:

networks:
  blog-network:
    driver: bridge
```

## 二、数据备份方案

### 2.1 数据库备份策略

#### 2.1.1 备份类型

1. **全量备份**：每天凌晨 2:00 执行
2. **增量备份**：每 6 小时执行一次
3. **实时备份**：使用 MySQL 主从复制

#### 2.1.2 备份脚本

**文件**：`scripts/backup-database.sh`

```bash
#!/bin/bash

# 配置
BACKUP_DIR="/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="blog"
DB_USER="root"
DB_PASSWORD="${MYSQL_ROOT_PASSWORD}"
RETENTION_DAYS=30

# 创建备份目录
mkdir -p ${BACKUP_DIR}

# 全量备份
mysqldump -u${DB_USER} -p${DB_PASSWORD} \
    --single-transaction \
    --routines \
    --triggers \
    --events \
    --quick \
    --lock-tables=false \
    ${DB_NAME} | gzip > ${BACKUP_DIR}/backup_${DATE}.sql.gz

# 删除过期备份
find ${BACKUP_DIR} -name "backup_*.sql.gz" -mtime +${RETENTION_DAYS} -delete

# 记录备份日志
echo "$(date): Database backup completed: backup_${DATE}.sql.gz" >> ${BACKUP_DIR}/backup.log

# 发送备份通知（可选）
# curl -X POST https://your-webhook-url -d "Backup completed: backup_${DATE}.sql.gz"
```

#### 2.1.3 定时备份（Cron）

**文件**：`docker/cron/backup-cron`

```cron
# 每天凌晨 2:00 全量备份
0 2 * * * /scripts/backup-database.sh

# 每 6 小时增量备份
0 */6 * * * /scripts/backup-incremental.sh
```

#### 2.1.4 Laravel 备份包

**使用 Laravel Backup 包**：

```bash
composer require spatie/laravel-backup
```

**配置**：`config/backup.php`

```php
<?php

return [
    'backup' => [
        'name' => env('APP_NAME', 'blog'),
        'source' => [
            'databases' => [
                'mysql',
            ],
            'files' => [
                'include' => [
                    storage_path('app'),
                    storage_path('logs'),
                ],
            ],
        ],
        'destination' => [
            'disks' => [
                'local',
                's3', // 可选：备份到 S3
            ],
        ],
    ],
];
```

**定时任务**：`app/Console/Kernel.php`

```php
protected function schedule(Schedule $schedule)
{
    // 每天凌晨 2:00 备份
    $schedule->command('backup:run')->daily()->at('02:00');
    
    // 清理旧备份（保留 30 天）
    $schedule->command('backup:clean')->daily();
}
```

### 2.2 文件备份策略

#### 2.2.1 备份内容

- 上传的文件（`storage/app/public`）
- 日志文件（`storage/logs`）
- 配置文件（`.env`，加密存储）

#### 2.2.2 文件备份脚本

**文件**：`scripts/backup-files.sh`

```bash
#!/bin/bash

BACKUP_DIR="/backups/files"
DATE=$(date +%Y%m%d_%H%M%S)
SOURCE_DIR="/var/www/storage"
RETENTION_DAYS=30

mkdir -p ${BACKUP_DIR}

# 备份文件
tar -czf ${BACKUP_DIR}/files_${DATE}.tar.gz \
    -C ${SOURCE_DIR} \
    app logs

# 删除过期备份
find ${BACKUP_DIR} -name "files_*.tar.gz" -mtime +${RETENTION_DAYS} -delete

echo "$(date): Files backup completed: files_${DATE}.tar.gz" >> ${BACKUP_DIR}/backup.log
```

### 2.3 Redis 备份策略

#### 2.3.1 RDB 快照备份

**配置**：`docker/redis/redis.conf`

```conf
# 每 900 秒至少 1 个 key 变化时保存
save 900 1

# 每 300 秒至少 10 个 key 变化时保存
save 300 10

# 每 60 秒至少 10000 个 key 变化时保存
save 60 10000

# RDB 文件保存路径
dir /data

# RDB 文件名
dbfilename dump.rdb
```

#### 2.3.2 AOF 持久化（可选）

```conf
# 启用 AOF
appendonly yes

# AOF 文件名
appendfilename "appendonly.aof"

# 同步策略：每秒同步
appendfsync everysec
```

### 2.4 备份存储策略

#### 2.4.1 本地存储

- 备份文件存储在 `/backups` 目录
- 定期清理过期备份

#### 2.4.2 远程存储（推荐）

**备份到云存储**：

1. **AWS S3**
```php
// config/filesystems.php
's3' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
],
```

2. **阿里云 OSS**
```php
'oss' => [
    'driver' => 'oss',
    'access_id' => env('OSS_ACCESS_ID'),
    'access_key' => env('OSS_ACCESS_KEY'),
    'bucket' => env('OSS_BUCKET'),
    'endpoint' => env('OSS_ENDPOINT'),
],
```

### 2.5 备份恢复流程

#### 2.5.1 数据库恢复

```bash
# 1. 停止应用
docker-compose stop app

# 2. 恢复数据库
gunzip < /backups/mysql/backup_20250101_020000.sql.gz | \
    mysql -u${DB_USER} -p${DB_PASSWORD} ${DB_NAME}

# 3. 验证数据
mysql -u${DB_USER} -p${DB_PASSWORD} -e "USE ${DB_NAME}; SELECT COUNT(*) FROM articles;"

# 4. 启动应用
docker-compose start app
```

#### 2.5.2 文件恢复

```bash
# 1. 停止应用
docker-compose stop app

# 2. 恢复文件
tar -xzf /backups/files/files_20250101_020000.tar.gz -C /var/www/storage

# 3. 设置权限
chown -R www-data:www-data /var/www/storage
chmod -R 755 /var/www/storage

# 4. 启动应用
docker-compose start app
```

## 三、项目更新方案

### 3.1 版本管理

#### 3.1.1 Git 版本控制

**分支策略**：
- `main`：生产环境代码
- `develop`：开发环境代码
- `feature/*`：功能分支
- `hotfix/*`：紧急修复分支

**标签管理**：
```bash
# 创建版本标签
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# 查看版本
git tag -l
```

#### 3.1.2 版本号规范

遵循语义化版本（Semantic Versioning）：
- **主版本号**：不兼容的 API 修改
- **次版本号**：向后兼容的功能新增
- **修订号**：向后兼容的问题修正

示例：`v1.2.3`

### 3.2 更新流程

#### 3.2.1 开发环境更新

```bash
# 1. 拉取最新代码
git pull origin develop

# 2. 安装/更新依赖
composer install

# 3. 运行数据库迁移
php artisan migrate

# 4. 清除缓存
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear

# 5. 重新构建缓存
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 6. 重启服务
docker-compose restart app
```

#### 3.2.2 生产环境更新（零停机部署）

**方案一：蓝绿部署**

```bash
# 1. 备份数据库
./scripts/backup-database.sh

# 2. 部署到新环境（绿色环境）
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 3. 切换流量（Nginx 配置）
# 修改 Nginx upstream 指向新环境

# 4. 验证新环境
curl https://api.example.com/health

# 5. 停止旧环境（蓝色环境）
docker-compose stop app_old

# 6. 清理旧环境
docker-compose rm app_old
```

**方案二：滚动更新**

```bash
# 1. 备份数据库
./scripts/backup-database.sh

# 2. 更新第一个实例
docker-compose up -d --no-deps --build app1

# 3. 等待健康检查通过
sleep 30
curl https://api.example.com/health

# 4. 更新第二个实例
docker-compose up -d --no-deps --build app2

# 5. 验证所有实例
docker-compose ps
```

#### 3.2.3 数据库迁移更新

**安全迁移流程**：

```bash
# 1. 备份数据库
./scripts/backup-database.sh

# 2. 在测试环境验证迁移
php artisan migrate --pretend

# 3. 执行迁移（生产环境）
php artisan migrate --force

# 4. 验证数据完整性
php artisan db:check

# 5. 如有问题，回滚迁移
php artisan migrate:rollback --step=1
```

### 3.3 回滚方案

#### 3.3.1 代码回滚

```bash
# 1. 回滚到上一个版本
git checkout v1.0.0

# 2. 清除缓存
php artisan config:clear
php artisan cache:clear

# 3. 重新构建缓存
php artisan config:cache
php artisan route:cache

# 4. 重启服务
docker-compose restart app
```

#### 3.3.2 数据库回滚

```bash
# 1. 停止应用
docker-compose stop app

# 2. 恢复数据库备份
gunzip < /backups/mysql/backup_20250101_020000.sql.gz | \
    mysql -u${DB_USER} -p${DB_PASSWORD} ${DB_NAME}

# 3. 回滚迁移
php artisan migrate:rollback --step=1

# 4. 启动应用
docker-compose start app
```

#### 3.3.3 快速回滚脚本

**文件**：`scripts/rollback.sh`

```bash
#!/bin/bash

set -e

VERSION=$1
BACKUP_DATE=$2

if [ -z "$VERSION" ] || [ -z "$BACKUP_DATE" ]; then
    echo "Usage: ./rollback.sh <version> <backup_date>"
    echo "Example: ./rollback.sh v1.0.0 20250101_020000"
    exit 1
fi

echo "Starting rollback to version $VERSION..."

# 1. 停止应用
echo "Stopping application..."
docker-compose stop app

# 2. 恢复代码
echo "Restoring code to version $VERSION..."
git checkout $VERSION

# 3. 恢复数据库
echo "Restoring database from backup $BACKUP_DATE..."
gunzip < /backups/mysql/backup_${BACKUP_DATE}.sql.gz | \
    mysql -u${DB_USER} -p${DB_PASSWORD} ${DB_NAME}

# 4. 清除缓存
echo "Clearing cache..."
php artisan config:clear
php artisan cache:clear
php artisan route:clear

# 5. 重建缓存
echo "Rebuilding cache..."
php artisan config:cache
php artisan route:cache

# 6. 启动应用
echo "Starting application..."
docker-compose start app

echo "Rollback completed successfully!"
```

## 四、CI/CD 配置

### 4.1 GitHub Actions 工作流

**文件**：`.github/workflows/deploy.yml`

```yaml
name: Deploy to Production

on:
  push:
    tags:
      - 'v*'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install Dependencies
        run: composer install
      - name: Run Tests
        run: php artisan test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref_type == 'tag'
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      
      - name: Deploy to Server
        run: |
          ssh user@server << 'EOF'
            cd /var/www/blog
            git fetch origin
            git checkout ${{ github.ref_name }}
            composer install --no-dev --optimize-autoloader
            php artisan migrate --force
            php artisan config:cache
            php artisan route:cache
            php artisan view:cache
            docker-compose restart app
          EOF
      
      - name: Health Check
        run: |
          sleep 10
          curl -f https://api.example.com/health || exit 1
```

### 4.2 部署前检查清单

**文件**：`scripts/pre-deploy-check.sh`

```bash
#!/bin/bash

set -e

echo "Running pre-deployment checks..."

# 1. 检查 PHP 版本
php -v | grep -q "PHP 8.2" || { echo "PHP version check failed"; exit 1; }

# 2. 检查 Composer 依赖
composer validate --no-check-publish || { echo "Composer validation failed"; exit 1; }

# 3. 运行测试
php artisan test || { echo "Tests failed"; exit 1; }

# 4. 检查数据库连接
php artisan db:check || { echo "Database connection check failed"; exit 1; }

# 5. 检查 Redis 连接
php artisan tinker --execute="Redis::ping()" || { echo "Redis connection check failed"; exit 1; }

# 6. 检查磁盘空间
df -h | awk '$5 > 90 {print "Disk space warning: " $0; exit 1}'

# 7. 检查备份是否最新
LAST_BACKUP=$(ls -t /backups/mysql/backup_*.sql.gz | head -1)
BACKUP_AGE=$(find $LAST_BACKUP -mtime +1)
if [ -n "$BACKUP_AGE" ]; then
    echo "Warning: Last backup is older than 1 day"
fi

echo "All pre-deployment checks passed!"
```

## 五、监控和日志

### 5.1 日志系统

#### 5.1.1 Laravel 日志配置

**配置**：`config/logging.php`

```php
<?php

return [
    'default' => env('LOG_CHANNEL', 'stack'),
    
    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['daily', 'slack'],
            'ignore_exceptions' => false,
        ],
        
        'daily' => [
            'driver' => 'daily',
            'path' => storage_path('logs/laravel.log'),
            'level' => env('LOG_LEVEL', 'debug'),
            'days' => 14,
        ],
        
        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Blog Bot',
            'emoji' => ':boom:',
            'level' => 'error',
        ],
    ],
];
```

#### 5.1.2 结构化日志

```php
// 记录结构化日志
Log::info('Article created', [
    'article_id' => $article->id(),
    'author_id' => $article->authorId()->value(),
    'title' => $article->title(),
    'ip' => request()->ip(),
    'user_agent' => request()->userAgent(),
]);
```

#### 5.1.3 日志轮转

**配置 Logrotate**：`/etc/logrotate.d/laravel`

```
/var/www/storage/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0644 www-data www-data
    sharedscripts
    postrotate
        docker-compose exec app php artisan log:clear
    endscript
}
```

### 5.2 性能监控

#### 5.2.1 Laravel Telescope（开发环境）

```bash
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate
```

**配置**：`config/telescope.php`

```php
'enabled' => env('TELESCOPE_ENABLED', false),
```

#### 5.2.2 APM 监控（生产环境）

**New Relic 配置**：

```php
// config/newrelic.php
return [
    'application_name' => env('NEW_RELIC_APP_NAME', 'Blog Platform'),
    'license' => env('NEW_RELIC_LICENSE_KEY'),
    'browser_monitoring' => [
        'auto_instrument' => true,
    ],
];
```

#### 5.2.3 自定义监控指标

**文件**：`app/Providers/AppServiceProvider.php`

```php
public function boot()
{
    // 记录请求响应时间
    $this->app['router']->matched(function ($route, $request) {
        $startTime = microtime(true);
        
        $this->app->terminating(function () use ($startTime, $route) {
            $duration = (microtime(true) - $startTime) * 1000;
            
            if ($duration > 1000) { // 超过 1 秒
                Log::warning('Slow request detected', [
                    'route' => $route->getName(),
                    'duration' => $duration,
                    'memory' => memory_get_peak_usage(true),
                ]);
            }
        });
    });
}
```

### 5.3 健康检查

#### 5.3.1 健康检查端点

**路由**：`routes/api.php`

```php
Route::get('/health', function () {
    $checks = [
        'database' => DB::connection()->getPdo() ? 'ok' : 'fail',
        'redis' => Redis::ping() ? 'ok' : 'fail',
        'disk' => disk_free_space('/') > 1024 * 1024 * 1024 ? 'ok' : 'fail', // 1GB
    ];
    
    $status = in_array('fail', $checks) ? 503 : 200;
    
    return response()->json([
        'status' => $status === 200 ? 'healthy' : 'unhealthy',
        'checks' => $checks,
        'timestamp' => now()->toIso8601String(),
    ], $status);
});
```

#### 5.3.2 Nginx 健康检查

**配置**：`docker/nginx/default.conf`

```nginx
upstream app {
    server app:9000;
}

server {
    location /health {
        access_log off;
        proxy_pass http://app;
        proxy_set_header Host $host;
    }
}
```

### 5.4 告警规则

#### 5.4.1 错误率告警

```php
// 监控错误率
if ($errorRate > 5) { // 错误率超过 5%
    // 发送告警
    Log::critical('High error rate detected', [
        'error_rate' => $errorRate,
        'time_window' => '5 minutes',
    ]);
}
```

#### 5.4.2 响应时间告警

```php
// 监控响应时间
if ($avgResponseTime > 1000) { // 平均响应时间超过 1 秒
    Log::warning('High response time detected', [
        'avg_response_time' => $avgResponseTime,
        'p95_response_time' => $p95ResponseTime,
    ]);
}
```

#### 5.4.3 资源使用告警

```bash
# 监控 CPU 和内存使用
#!/bin/bash
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM_USAGE=$(free | grep Mem | awk '{printf("%.2f", $3/$2 * 100)}')

if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
    echo "CPU usage is high: ${CPU_USAGE}%"
fi

if (( $(echo "$MEM_USAGE > 80" | bc -l) )); then
    echo "Memory usage is high: ${MEM_USAGE}%"
fi
```

## 六、问题排查流程

### 6.1 问题分类

1. **性能问题**：响应慢、高 CPU、高内存
2. **错误问题**：500 错误、数据库错误、缓存错误
3. **功能问题**：功能异常、数据不一致
4. **安全问题**：安全漏洞、异常访问

### 6.2 排查步骤

#### 6.2.1 查看日志

```bash
# 查看应用日志
tail -f storage/logs/laravel.log

# 查看错误日志
tail -f storage/logs/laravel-error.log

# 查看 Nginx 日志
docker-compose logs -f nginx

# 查看 PHP-FPM 日志
docker-compose logs -f app
```

#### 6.2.2 检查数据库

```bash
# 连接数据库
docker-compose exec mysql mysql -uroot -p

# 查看慢查询
SHOW PROCESSLIST;
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;

# 查看表状态
SHOW TABLE STATUS;
```

#### 6.2.3 检查缓存

```bash
# 连接 Redis
docker-compose exec redis redis-cli

# 查看键
KEYS *

# 查看内存使用
INFO memory

# 清空缓存（谨慎使用）
FLUSHALL
```

#### 6.2.4 检查队列

```bash
# 查看队列状态
php artisan queue:monitor

# 查看失败任务
php artisan queue:failed

# 重试失败任务
php artisan queue:retry all
```

### 6.3 常见问题处理

#### 6.3.1 数据库连接失败

```bash
# 检查数据库服务
docker-compose ps mysql

# 检查连接配置
php artisan tinker
>>> DB::connection()->getPdo();

# 重启数据库
docker-compose restart mysql
```

#### 6.3.2 缓存问题

```bash
# 清除所有缓存
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# 重建缓存
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

#### 6.3.3 队列积压

```bash
# 增加队列 Worker
docker-compose scale queue=3

# 查看队列统计
php artisan queue:stats

# 清理失败任务
php artisan queue:flush
```

## 七、运维最佳实践

### 7.1 定期维护任务

**定时任务**：`app/Console/Kernel.php`

```php
protected function schedule(Schedule $schedule)
{
    // 每天凌晨执行数据库备份
    $schedule->command('backup:run')->daily()->at('02:00');
    
    // 清理过期备份
    $schedule->command('backup:clean')->daily();
    
    // 清理过期日志
    $schedule->command('log:clear')->daily();
    
    // 清理过期会话
    $schedule->command('session:gc')->hourly();
    
    // 优化数据库
    $schedule->command('db:optimize')->weekly();
}
```

### 7.2 安全加固

1. **定期更新依赖**：`composer update`
2. **检查安全漏洞**：`composer audit`
3. **更新系统补丁**：`apt-get update && apt-get upgrade`
4. **检查文件权限**：确保敏感文件权限正确

### 7.3 性能优化

1. **启用 OPcache**：PHP 配置中启用
2. **使用 Redis 缓存**：缓存热点数据
3. **数据库索引优化**：定期分析慢查询
4. **CDN 加速**：静态资源使用 CDN

## 八、下一步

完成部署和运维文档后，项目设计阶段完成。可以开始编码实现。

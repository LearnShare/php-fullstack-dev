# 8.4.2 传统部署

## 概述

传统部署方式仍然广泛使用。本节介绍 VPS 部署、Laravel Forge、部署脚本等传统部署方法。

## VPS 部署

### 服务器准备

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装 Nginx
sudo apt install nginx -y

# 安装 PHP-FPM
sudo apt install php8.2-fpm php8.2-mysql php8.2-xml php8.2-mbstring -y

# 安装 MySQL
sudo apt install mysql-server -y

# 安装 Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

### Nginx 配置

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name example.com;
    root /var/www/myapp/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Laravel Forge

### Forge 部署流程

1. **连接服务器**：在 Forge 中添加服务器
2. **创建站点**：创建新站点
3. **配置环境**：设置环境变量
4. **部署脚本**：配置部署脚本

### 部署脚本示例

```bash
# Forge 部署脚本
cd /home/forge/myapp
git pull origin main
composer install --no-interaction --prefer-dist --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## 部署脚本

### 自动化部署脚本

```bash
#!/bin/bash
# deploy.sh

set -e

APP_DIR="/var/www/myapp"
REPO_URL="https://github.com/user/repo.git"
BRANCH="main"

echo "Starting deployment..."

# 备份当前版本
if [ -d "$APP_DIR" ]; then
    cp -r $APP_DIR $APP_DIR.backup.$(date +%Y%m%d_%H%M%S)
fi

# 克隆或更新代码
if [ -d "$APP_DIR/.git" ]; then
    cd $APP_DIR
    git pull origin $BRANCH
else
    git clone -b $BRANCH $REPO_URL $APP_DIR
    cd $APP_DIR
fi

# 安装依赖
composer install --no-dev --optimize-autoloader

# 运行迁移
php artisan migrate --force

# 清理缓存
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 设置权限
chown -R www-data:www-data $APP_DIR
chmod -R 755 $APP_DIR/storage

echo "Deployment completed!"
```

## 完整示例

```bash
#!/bin/bash
# 完整的部署脚本

set -e

APP_NAME="myapp"
APP_DIR="/var/www/${APP_NAME}"
BACKUP_DIR="/var/backups/${APP_NAME}"
REPO_URL="https://github.com/user/repo.git"
BRANCH="main"

# 创建备份
echo "Creating backup..."
mkdir -p $BACKUP_DIR
if [ -d "$APP_DIR" ]; then
    tar -czf "${BACKUP_DIR}/backup-$(date +%Y%m%d_%H%M%S).tar.gz" -C $(dirname $APP_DIR) $(basename $APP_DIR)
fi

# 部署新版本
echo "Deploying new version..."
if [ -d "$APP_DIR/.git" ]; then
    cd $APP_DIR
    git fetch origin
    git reset --hard origin/$BRANCH
else
    git clone -b $BRANCH $REPO_URL $APP_DIR
    cd $APP_DIR
fi

# 安装依赖
echo "Installing dependencies..."
composer install --no-dev --optimize-autoloader --no-interaction

# 运行迁移
echo "Running migrations..."
php artisan migrate --force

# 优化
echo "Optimizing..."
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 设置权限
echo "Setting permissions..."
chown -R www-data:www-data $APP_DIR
chmod -R 755 $APP_DIR/storage
chmod -R 755 $APP_DIR/bootstrap/cache

# 重启服务
echo "Restarting services..."
systemctl restart php8.2-fpm
systemctl reload nginx

echo "Deployment completed successfully!"
```

## 最佳实践

1. **自动化部署**：使用脚本自动化部署流程
2. **备份策略**：部署前备份当前版本
3. **零停机部署**：使用蓝绿部署或滚动更新
4. **回滚机制**：准备快速回滚方案

## 注意事项

1. 测试部署脚本
2. 注意文件权限
3. 处理数据库迁移
4. 监控部署过程

## 练习

1. 编写一个自动化部署脚本。

2. 配置 Nginx 和 PHP-FPM。

3. 实现零停机部署。

4. 创建回滚机制。

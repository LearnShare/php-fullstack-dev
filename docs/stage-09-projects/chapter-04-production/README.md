# 9.4 生产级部署

## 目标

- 将应用部署到生产环境。
- 配置监控和日志系统。
- 实现自动化运维。
- 建立完整的 DevOps 流程。

## 项目概述

本项目展示如何将 PHP 应用部署到生产环境，包括 Docker 容器化、Kubernetes 编排、监控告警、日志收集、CI/CD 流程等完整的生产级部署方案。

## 部署架构

### 整体架构

```
用户请求
  ↓
负载均衡器（ALB/NLB）
  ↓
Kubernetes Ingress
  ↓
PHP 应用 Pods（多副本）
  ├── PHP-FPM 容器
  ├── Nginx 容器（可选）
  └── 应用代码
  ↓
数据库（RDS/Cloud SQL）
  ↓
缓存（Redis/ElastiCache）
  ↓
消息队列（RabbitMQ/SQS）
```

### Docker 容器化

#### 多阶段构建 Dockerfile

```dockerfile
# 阶段一：构建阶段
FROM php:8.3-fpm-alpine AS builder

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 安装系统依赖
RUN apk add --no-cache \
    git \
    unzip \
    libzip-dev \
    && docker-php-ext-install zip opcache

# 复制依赖文件
WORKDIR /app
COPY composer.json composer.lock ./

# 安装依赖（只安装生产依赖）
RUN composer install --no-dev --no-scripts --no-autoloader --optimize-autoloader

# 复制应用代码
COPY . .

# 生成自动加载文件
RUN composer dump-autoload --optimize --classmap-authoritative

# 阶段二：生产阶段
FROM php:8.3-fpm-alpine

# 安装运行时依赖
RUN apk add --no-cache \
    nginx \
    supervisor \
    && docker-php-ext-install opcache pdo_mysql

# 配置 PHP
COPY docker/php/php.ini /usr/local/etc/php/conf.d/app.ini
COPY docker/php/php-fpm.conf /usr/local/etc/php-fpm.d/www.conf

# 配置 Nginx
COPY docker/nginx/nginx.conf /etc/nginx/nginx.conf
COPY docker/nginx/default.conf /etc/nginx/http.d/default.conf

# 配置 Supervisor
COPY docker/supervisor/supervisord.conf /etc/supervisord.conf

# 从构建阶段复制应用代码
COPY --from=builder /app /var/www/html

# 设置权限
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/storage

# 暴露端口
EXPOSE 80

# 启动 Supervisor（管理 PHP-FPM 和 Nginx）
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
```

#### docker-compose.yml（生产环境）

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: myapp:latest
    container_name: myapp
    restart: unless-stopped
    environment:
      - APP_ENV=production
      - APP_DEBUG=false
      - DB_HOST=db
      - DB_DATABASE=myapp
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis
    networks:
      - app-network
    volumes:
      - ./storage:/var/www/html/storage
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: mysql:8.0
    container_name: myapp-db
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_DATABASE}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./docker/nginx/ssl:/etc/nginx/ssl
    depends_on:
      - app
    networks:
      - app-network

volumes:
  db-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### Kubernetes 部署

#### Deployment 配置

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3  # 3 个副本
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: php-fpm
        image: myapp:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        env:
        - name: APP_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-password
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
      imagePullSecrets:
      - name: registry-secret
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### Ingress 配置

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.example.com
    secretName: myapp-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

## 监控配置

### Prometheus 指标收集

```php
<?php
declare(strict_types=1);

namespace App\Services;

/**
 * Prometheus 指标收集器
 * 
 * 收集应用指标并暴露给 Prometheus
 */
class MetricsCollector
{
    private array $counters = [];
    private array $gauges = [];
    private array $histograms = [];
    
    /**
     * 增加计数器
     */
    public function incrementCounter(string $name, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        $this->counters[$key] = ($this->counters[$key] ?? 0) + 1;
    }
    
    /**
     * 设置仪表盘值
     */
    public function setGauge(string $name, float $value, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        $this->gauges[$key] = $value;
    }
    
    /**
     * 记录直方图
     */
    public function observeHistogram(string $name, float $value, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        if (!isset($this->histograms[$key])) {
            $this->histograms[$key] = [];
        }
        $this->histograms[$key][] = $value;
    }
    
    /**
     * 导出 Prometheus 格式
     */
    public function export(): string
    {
        $output = [];
        
        // 导出计数器
        foreach ($this->counters as $key => $value) {
            $output[] = "# TYPE {$key} counter";
            $output[] = "{$key} {$value}";
        }
        
        // 导出仪表盘
        foreach ($this->gauges as $key => $value) {
            $output[] = "# TYPE {$key} gauge";
            $output[] = "{$key} {$value}";
        }
        
        // 导出直方图
        foreach ($this->histograms as $key => $values) {
            $output[] = "# TYPE {$key} histogram";
            $count = count($values);
            $sum = array_sum($values);
            $output[] = "{$key}_count {$count}";
            $output[] = "{$key}_sum {$sum}";
            $output[] = "{$key}_bucket{le=\"+Inf\"} {$count}";
        }
        
        return implode("\n", $output);
    }
    
    private function getKey(string $name, array $labels): string
    {
        if (empty($labels)) {
            return $name;
        }
        
        $labelStr = implode(',', array_map(
            fn($k, $v) => "{$k}=\"{$v}\"",
            array_keys($labels),
            $labels
        ));
        
        return "{$name}{{$labelStr}}";
    }
}

// 在应用中收集指标
$metrics = new MetricsCollector();

// 记录 HTTP 请求
$metrics->incrementCounter('http_requests_total', [
    'method' => $_SERVER['REQUEST_METHOD'],
    'status' => http_response_code(),
]);

// 记录响应时间
$startTime = microtime(true);
// ... 处理请求 ...
$duration = microtime(true) - $startTime;
$metrics->observeHistogram('http_request_duration_seconds', $duration);

// 记录内存使用
$metrics->setGauge('memory_usage_bytes', memory_get_usage());

// 暴露指标端点
if ($_SERVER['REQUEST_URI'] === '/metrics') {
    header('Content-Type: text/plain');
    echo $metrics->export();
    exit;
}
```

### Grafana 仪表板配置

```json
{
  "dashboard": {
    "title": "PHP Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{status}}"
          }
        ]
      },
      {
        "title": "Response Time",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, http_request_duration_seconds_bucket)",
            "legendFormat": "95th percentile"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "5xx errors"
          }
        ]
      }
    ]
  }
}
```

## 日志收集

### ELK Stack 配置

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Monolog\Logger;
use Monolog\Handler\ElasticsearchHandler;
use Elasticsearch\ClientBuilder;

/**
 * ELK 日志集成
 * 
 * 将日志发送到 Elasticsearch
 */
class ElkLogger
{
    private Logger $logger;
    
    public function __construct()
    {
        $this->logger = new Logger('app');
        
        // 创建 Elasticsearch 客户端
        $client = ClientBuilder::create()
            ->setHosts(['elasticsearch:9200'])
            ->build();
        
        // 创建 Elasticsearch Handler
        $handler = new ElasticsearchHandler(
            $client,
            [
                'index' => 'php-app-logs',
                'type' => '_doc',
            ],
            Logger::INFO
        );
        
        $this->logger->pushHandler($handler);
    }
    
    public function log(string $level, string $message, array $context = []): void
    {
        $this->logger->log($level, $message, $context);
    }
}

// 使用
$elkLogger = new ElkLogger();
$elkLogger->log('info', 'User logged in', [
    'user_id' => 123,
    'ip' => $_SERVER['REMOTE_ADDR'],
]);
```

### Loki 日志配置

```yaml
# docker-compose.yml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - ./promtail-config.yaml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
    command: -config.file=/etc/promtail/config.yml
```

## CI/CD 流程

### GitHub Actions 完整配置

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          extensions: pdo_mysql, redis, opcache
          coverage: xdebug
      
      - name: Install dependencies
        run: composer install --prefer-dist --no-progress
      
      - name: Run tests
        run: composer test
      
      - name: Generate coverage
        run: composer test-coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml

  build:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Set up Kustomize
        uses: imranismail/setup-kustomize@v2
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp \
            myapp=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production
          
          kubectl rollout status deployment/myapp -n production
      
      - name: Run health check
        run: |
          sleep 30
          curl -f https://api.example.com/health || exit 1
```

## 告警配置

### Prometheus Alertmanager 规则

```yaml
groups:
  - name: php_app_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors/sec"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 1
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High response time"
          description: "95th percentile response time is {{ $value }}s"
      
      - alert: HighMemoryUsage
        expr: memory_usage_bytes / 1024 / 1024 > 400
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}MB"
```

## 灾难恢复

### 备份策略

```bash
#!/bin/bash
# 数据库备份脚本

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="myapp"
DB_USER="backup_user"
DB_PASSWORD="backup_password"

# 创建备份
mysqldump -u $DB_USER -p$DB_PASSWORD $DB_NAME | gzip > "$BACKUP_DIR/db_$DATE.sql.gz"

# 保留最近 7 天的备份
find $BACKUP_DIR -name "db_*.sql.gz" -mtime +7 -delete

# 上传到 S3
aws s3 cp "$BACKUP_DIR/db_$DATE.sql.gz" s3://myapp-backups/database/
```

### 恢复流程

```bash
#!/bin/bash
# 数据库恢复脚本

BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: restore.sh <backup_file>"
    exit 1
fi

# 下载备份文件
aws s3 cp "s3://myapp-backups/database/$BACKUP_FILE" /tmp/

# 恢复数据库
gunzip -c /tmp/$BACKUP_FILE | mysql -u root -p myapp

echo "Database restored from $BACKUP_FILE"
```

## 练习

1. **部署应用到生产环境**
   - 使用 Docker 容器化应用
   - 部署到 Kubernetes 集群
   - 配置 Ingress 和 Service

2. **配置完整的监控体系**
   - 集成 Prometheus 收集指标
   - 创建 Grafana 仪表板
   - 配置告警规则

3. **实现日志收集和分析**
   - 配置 ELK Stack 或 Loki
   - 实现结构化日志
   - 建立日志查询和分析流程

4. **建立告警机制**
   - 配置 Prometheus Alertmanager
   - 设置告警规则（错误率、响应时间、资源使用）
   - 集成通知渠道（Email、Slack、PagerDuty）

5. **设计灾难恢复方案**
   - 实现数据库自动备份
   - 设计备份恢复流程
   - 测试灾难恢复流程

6. **实现自动化运维流程**
   - 配置 CI/CD 流水线
   - 实现自动化测试和部署
   - 建立回滚机制

7. **性能优化**
   - 配置 Horizontal Pod Autoscaler
   - 优化容器资源限制
   - 实现缓存策略

8. **安全加固**
   - 配置 TLS/SSL 证书
   - 实现 Secrets 管理
   - 配置网络策略

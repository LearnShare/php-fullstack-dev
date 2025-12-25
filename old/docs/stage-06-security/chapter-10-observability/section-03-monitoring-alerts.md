# 6.10.3 监控与告警

## 概述

监控与告警是可观测性的重要组成部分。本节介绍监控系统、告警配置、可视化等内容。

## 监控系统

### Prometheus + Grafana

```php
<?php
declare(strict_types=1);

// 暴露 Prometheus 指标端点
class MetricsEndpoint
{
    private PrometheusMetrics $metrics;
    
    public function handle(): void
    {
        header('Content-Type: text/plain');
        echo $this->metrics->export();
    }
}

// 路由配置
// GET /metrics -> MetricsEndpoint
```

### 自定义监控

```php
<?php
declare(strict_types=1);

class SystemMonitor
{
    private array $metrics = [];
    
    public function collect(): void
    {
        $this->metrics = [
            'cpu_usage' => $this->getCpuUsage(),
            'memory_usage' => $this->getMemoryUsage(),
            'disk_usage' => $this->getDiskUsage(),
            'request_count' => $this->getRequestCount(),
            'error_rate' => $this->getErrorRate(),
        ];
    }
    
    public function getMetrics(): array
    {
        return $this->metrics;
    }
    
    private function getCpuUsage(): float
    {
        // 获取 CPU 使用率
        return sys_getloadavg()[0];
    }
    
    private function getMemoryUsage(): float
    {
        return memory_get_usage(true) / 1024 / 1024; // MB
    }
    
    private function getDiskUsage(): float
    {
        // 获取磁盘使用率
        return disk_free_space('/') / disk_total_space('/') * 100;
    }
    
    private function getRequestCount(): int
    {
        // 从指标收集器获取
        return 0;
    }
    
    private function getErrorRate(): float
    {
        // 计算错误率
        return 0.0;
    }
}
```

## 告警配置

### 告警规则

```yaml
# alerts.yml
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: error_rate > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          
      - alert: HighMemoryUsage
        expr: memory_usage > 0.9
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage"
```

### 告警处理

```php
<?php
declare(strict_types=1);

class AlertManager
{
    private array $handlers = [];
    
    public function registerHandler(string $type, callable $handler): void
    {
        $this->handlers[$type] = $handler;
    }
    
    public function trigger(string $alert, array $data): void
    {
        foreach ($this->handlers as $type => $handler) {
            $handler($alert, $data);
        }
    }
    
    public function checkThresholds(array $metrics): void
    {
        if ($metrics['error_rate'] > 0.05) {
            $this->trigger('high_error_rate', $metrics);
        }
        
        if ($metrics['memory_usage'] > 0.9) {
            $this->trigger('high_memory_usage', $metrics);
        }
    }
}

// 使用
$alertManager = new AlertManager();
$alertManager->registerHandler('email', function($alert, $data) {
    mail('admin@example.com', "Alert: {$alert}", json_encode($data));
});

$alertManager->registerHandler('slack', function($alert, $data) {
    // 发送到 Slack
});
```

## 可视化

### Grafana 仪表板

```json
{
  "dashboard": {
    "title": "Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(requests_total[5m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(errors_total[5m])"
          }
        ]
      }
    ]
  }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class MonitoringSystem
{
    private SystemMonitor $monitor;
    private AlertManager $alerts;
    private PrometheusMetrics $metrics;
    
    public function __construct()
    {
        $this->monitor = new SystemMonitor();
        $this->alerts = new AlertManager();
        $this->metrics = new PrometheusMetrics();
    }
    
    public function run(): void
    {
        while (true) {
            // 收集指标
            $this->monitor->collect();
            $metrics = $this->monitor->getMetrics();
            
            // 更新 Prometheus 指标
            $this->metrics->setGauge('cpu_usage', $metrics['cpu_usage']);
            $this->metrics->setGauge('memory_usage', $metrics['memory_usage']);
            
            // 检查告警
            $this->alerts->checkThresholds($metrics);
            
            sleep(60); // 每分钟检查一次
        }
    }
}
```

## 最佳实践

1. **关键指标**：监控关键业务和性能指标
2. **告警阈值**：设置合理的告警阈值
3. **告警降噪**：避免告警风暴
4. **可视化**：使用 Grafana 等工具可视化

## 注意事项

1. 告警应该可操作
2. 避免告警疲劳
3. 定期审查告警规则
4. 监控系统本身

## 练习

1. 实现一个系统监控器，收集系统指标。

2. 配置告警规则，监控关键指标。

3. 创建 Grafana 仪表板，可视化指标。

4. 实现告警处理机制，发送告警通知。

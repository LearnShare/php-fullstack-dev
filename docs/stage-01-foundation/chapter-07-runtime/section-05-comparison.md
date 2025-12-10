# 1.6.5 运行时对比与选择

## 概述

选择合适的 PHP 运行时对于应用性能至关重要。本章对比各种运行时的特点，帮助选择最适合的方案。

## 运行时对比

### 功能对比

| 特性 | PHP-FPM | RoadRunner | FrankenPHP | OpenSwoole |
| :--- | :------ | :--------- | :--------- | :--------- |
| 性能 | 基准 | 高 | 高 | 极高 |
| 内存占用 | 高 | 低 | 中 | 低 |
| 部署复杂度 | 低 | 中 | 低 | 高 |
| WebSocket 支持 | 否 | 是 | 是 | 是 |
| HTTP/2 支持 | 需 Nginx | 是 | 是 | 是 |
| HTTP/3 支持 | 否 | 否 | 是 | 否 |
| 协程支持 | 否 | 否 | 否 | 是 |
| Laravel 支持 | 原生 | Octane | Octane | Octane |
| 自动 HTTPS | 否 | 否 | 是 | 否 |
| 学习曲线 | 低 | 中 | 低 | 高 |

### 性能对比

| 指标 | PHP-FPM | RoadRunner | FrankenPHP | OpenSwoole |
| :--- | :------ | :--------- | :--------- | :--------- |
| 请求/秒 | 基准 | 2-5x | 2-4x | 5-10x |
| 内存使用 | 高 | 低 | 中 | 低 |
| 启动时间 | 每次请求 | 仅启动时 | 仅启动时 | 仅启动时 |
| 连接复用 | 否 | 是 | 是 | 是 |

## 选择建议

### 使用 PHP-FPM 的场景

**适用场景**：
- 传统 Web 应用
- 低到中等流量
- 简单的部署需求
- 不需要 WebSocket
- 团队对 PHP-FPM 熟悉

**优势**：
- 成熟稳定
- 部署简单
- 文档丰富
- 社区支持好

**劣势**：
- 性能相对较低
- 不支持 WebSocket
- 每次请求需要启动

### 使用 RoadRunner 的场景

**适用场景**：
- 高并发 API 服务
- 需要性能提升
- Go 生态集成
- Laravel 应用（通过 Octane）

**优势**：
- 性能提升明显
- 内存占用低
- 支持 WebSocket
- 易于部署

**劣势**：
- 需要学习新工具
- 配置相对复杂
- 社区相对较小

### 使用 FrankenPHP 的场景

**适用场景**：
- 需要自动 HTTPS
- 需要 HTTP/2/3
- 简单的部署
- Laravel 应用（通过 Octane）

**优势**：
- 开箱即用
- 自动 HTTPS
- HTTP/2/3 支持
- 部署简单

**劣势**：
- 相对较新
- 社区较小
- 文档较少

### 使用 OpenSwoole 的场景

**适用场景**：
- 极高性能要求
- 需要 WebSocket
- 需要协程
- 实时应用

**优势**：
- 性能极高
- 协程支持
- WebSocket 支持
- 功能强大

**劣势**：
- 学习曲线陡峭
- 部署复杂
- 需要 C 扩展
- 状态管理复杂

## 决策流程

### 流程图

```
开始
  ↓
是否需要 WebSocket？
  ├─ 是 → 选择 RoadRunner / FrankenPHP / OpenSwoole
  └─ 否 → 继续
      ↓
是否需要自动 HTTPS？
  ├─ 是 → 选择 FrankenPHP
  └─ 否 → 继续
      ↓
是否需要协程？
  ├─ 是 → 选择 OpenSwoole
  └─ 否 → 继续
      ↓
是否需要极致性能？
  ├─ 是 → 选择 OpenSwoole / RoadRunner
  └─ 否 → 选择 PHP-FPM
```

### 决策矩阵

| 需求 | PHP-FPM | RoadRunner | FrankenPHP | OpenSwoole |
| :--- | :------ | :--------- | :--------- | :--------- |
| 简单部署 | ✓ | ✗ | ✓ | ✗ |
| 高性能 | ✗ | ✓ | ✓ | ✓✓ |
| WebSocket | ✗ | ✓ | ✓ | ✓ |
| 自动 HTTPS | ✗ | ✗ | ✓ | ✗ |
| 协程 | ✗ | ✗ | ✗ | ✓ |
| Laravel | ✓ | ✓ | ✓ | ✓ |

## 迁移建议

### 从 PHP-FPM 迁移到 RoadRunner

1. **安装 Octane**
   ```bash
   composer require laravel/octane
   php artisan octane:install
   ```

2. **选择 RoadRunner**
   ```bash
   # 选择选项 1
   ```

3. **配置状态重置**
   ```php
   // 在 OctaneServiceProvider 中配置
   ```

4. **测试和优化**
   ```bash
   php artisan octane:serve
   ```

### 从 PHP-FPM 迁移到 OpenSwoole

1. **安装 OpenSwoole**
   ```bash
   pecl install openswoole
   ```

2. **重写应用代码**
   - 使用协程版本的函数
   - 避免阻塞操作
   - 管理状态

3. **测试和优化**
   - 性能测试
   - 内存监控
   - 错误处理

## 完整示例

### 性能测试脚本

```php
<?php
declare(strict_types=1);

class RuntimeComparison
{
    public static function benchmark(string $url, int $requests = 1000): array
    {
        $start = microtime(true);
        $success = 0;
        $failed = 0;
        
        for ($i = 0; $i < $requests; $i++) {
            $ch = curl_init($url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($ch, CURLOPT_TIMEOUT, 5);
            
            $response = curl_exec($ch);
            $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
            
            if ($httpCode === 200) {
                $success++;
            } else {
                $failed++;
            }
            
            curl_close($ch);
        }
        
        $end = microtime(true);
        $elapsed = $end - $start;
        $rps = $requests / $elapsed;
        
        return [
            'requests' => $requests,
            'success' => $success,
            'failed' => $failed,
            'elapsed' => $elapsed,
            'rps' => round($rps, 2)
        ];
    }
}

// 测试不同运行时
$runtimes = [
    'PHP-FPM' => 'http://localhost:80',
    'RoadRunner' => 'http://localhost:8080',
    'FrankenPHP' => 'http://localhost:80',
    'OpenSwoole' => 'http://localhost:9501'
];

foreach ($runtimes as $name => $url) {
    echo "=== {$name} ===\n";
    $result = RuntimeComparison::benchmark($url, 1000);
    echo "RPS: {$result['rps']}\n";
    echo "Success: {$result['success']}\n";
    echo "Failed: {$result['failed']}\n";
    echo "\n";
}
```

## 注意事项

1. **性能测试**：在实际环境中进行性能测试，不要仅依赖理论数据。

2. **状态管理**：常驻运行时需要特别注意状态管理。

3. **错误处理**：确保错误处理机制完善，避免 Worker 崩溃。

4. **监控和日志**：建立完善的监控和日志系统。

5. **渐进迁移**：不要一次性迁移，逐步测试和优化。

## 练习

1. 对比不同运行时的性能，使用性能测试工具进行评估。

2. 根据项目需求，选择合适的运行时方案。

3. 实现从 PHP-FPM 到 RoadRunner 的迁移。

4. 配置监控系统，跟踪运行时的性能指标。

5. 编写决策文档，说明选择特定运行时的理由。

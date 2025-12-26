# 5.13.2 中间件模式

## 概述

中间件模式是处理请求响应的设计模式。本节介绍中间件模式的实现方式，包括 Pipeline 模式、装饰器模式等，帮助零基础学员理解中间件的设计原理。

**章节类型**：概念性章节

**主要内容**：
- 中间件模式概述
- Pipeline 模式
- 装饰器模式
- 中间件接口
- 模式实现
- 完整示例

## 核心内容

### 中间件模式概述

- 设计模式概念
- 模式的优势
- 应用场景

### Pipeline 模式

- Pipeline 概念
- 请求传递
- 响应传递
- 实现方法

### 装饰器模式

- 装饰器概念
- 功能增强
- 链式装饰
- 实现方法

### 中间件接口

- 接口定义
- 标准接口
- PSR-15 标准
- 接口实现

## 基本用法

### Pipeline 模式示例

```php
<?php
declare(strict_types=1);

class Pipeline {
    private array $middlewares = [];
    
    public function pipe(callable $middleware): self {
        $this->middlewares[] = $middleware;
        return $this;
    }
    
    public function process($request) {
        $next = function($req) {
            return $req;
        };
        
        foreach (array_reverse($this->middlewares) as $middleware) {
            $next = function($req) use ($middleware, $next) {
                return $middleware($req, $next);
            };
        }
        
        return $next($request);
    }
}
```

## 使用场景

- 框架设计
- 请求处理
- 功能增强
- 横切关注点

## 注意事项

- 执行顺序
- 性能考虑
- 错误处理
- 接口标准化

## 常见问题

- Pipeline 模式如何实现？
- 装饰器模式如何应用？
- PSR-15 标准是什么？
- 如何设计中间件接口？

## 最佳实践

- 遵循 PSR-15 标准
- 实现 Pipeline 模式
- 保持接口一致性
- 优化执行性能

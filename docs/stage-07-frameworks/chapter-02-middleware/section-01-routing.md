# 7.2.1 路由系统

## 概述

路由系统是 Web 框架的核心，负责将 URL 映射到处理函数。本节介绍路由概念、路由匹配、路由参数、路由组等内容。

## 路由概念

### 基础路由

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];
    
    public function get(string $path, callable $handler): void
    {
        $this->addRoute('GET', $path, $handler);
    }
    
    public function post(string $path, callable $handler): void
    {
        $this->addRoute('POST', $path, $handler);
    }
    
    private function addRoute(string $method, string $path, callable $handler): void
    {
        $this->routes[] = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
        ];
    }
    
    public function dispatch(string $method, string $path): mixed
    {
        foreach ($this->routes as $route) {
            if ($route['method'] === $method && $route['path'] === $path) {
                return call_user_func($route['handler']);
            }
        }
        
        throw new NotFoundException("Route not found: {$method} {$path}");
    }
}

// 使用
$router = new Router();
$router->get('/users', fn() => 'User list');
$router->post('/users', fn() => 'Create user');

$result = $router->dispatch('GET', '/users');
```

## 路由参数

### 参数匹配

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];
    
    public function get(string $path, callable $handler): void
    {
        $this->addRoute('GET', $path, $handler);
    }
    
    private function addRoute(string $method, string $path, callable $handler): void
    {
        // 将路径转换为正则表达式
        $pattern = $this->pathToRegex($path);
        
        $this->routes[] = [
            'method' => $method,
            'pattern' => $pattern,
            'path' => $path,
            'handler' => $handler,
        ];
    }
    
    private function pathToRegex(string $path): string
    {
        // 替换 {id} 为命名捕获组
        $pattern = preg_replace('/\{(\w+)\}/', '(?P<$1>[^/]+)', $path);
        return '#^' . $pattern . '$#';
    }
    
    public function dispatch(string $method, string $path): mixed
    {
        foreach ($this->routes as $route) {
            if ($route['method'] !== $method) {
                continue;
            }
            
            if (preg_match($route['pattern'], $path, $matches)) {
                // 提取参数
                $params = array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
                return call_user_func($route['handler'], $params);
            }
        }
        
        throw new NotFoundException("Route not found: {$method} {$path}");
    }
}

// 使用
$router = new Router();
$router->get('/users/{id}', function(array $params) {
    return "User ID: {$params['id']}";
});

$result = $router->dispatch('GET', '/users/123');
```

## 路由组

### 路由前缀和中间件

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];
    private array $groupStack = [];
    
    public function group(array $attributes, callable $callback): void
    {
        $this->groupStack[] = $attributes;
        $callback($this);
        array_pop($this->groupStack);
    }
    
    public function get(string $path, callable $handler): void
    {
        $route = $this->buildRoute('GET', $path, $handler);
        $this->routes[] = $route;
    }
    
    private function buildRoute(string $method, string $path, callable $handler): array
    {
        $route = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
            'middleware' => [],
        ];
        
        // 应用组属性
        foreach ($this->groupStack as $group) {
            if (isset($group['prefix'])) {
                $route['path'] = $group['prefix'] . $route['path'];
            }
            
            if (isset($group['middleware'])) {
                $route['middleware'] = array_merge($route['middleware'], $group['middleware']);
            }
        }
        
        return $route;
    }
}

// 使用
$router = new Router();

$router->group(['prefix' => '/api', 'middleware' => ['auth']], function($router) {
    $router->get('/users', fn() => 'User list');
    $router->get('/posts', fn() => 'Post list');
});
```

## 路由缓存

### 缓存路由表

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];
    private ?array $cachedRoutes = null;
    private string $cacheFile;
    
    public function __construct(string $cacheFile = null)
    {
        $this->cacheFile = $cacheFile ?? __DIR__ . '/routes.cache.php';
    }
    
    public function loadCachedRoutes(): bool
    {
        if (file_exists($this->cacheFile)) {
            $this->cachedRoutes = require $this->cacheFile;
            return true;
        }
        return false;
    }
    
    public function cacheRoutes(): void
    {
        $cache = "<?php\nreturn " . var_export($this->routes, true) . ";\n";
        file_put_contents($this->cacheFile, $cache);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];
    private array $groupStack = [];
    
    public function get(string $path, callable|string $handler): Route
    {
        return $this->addRoute('GET', $path, $handler);
    }
    
    public function post(string $path, callable|string $handler): Route
    {
        return $this->addRoute('POST', $path, $handler);
    }
    
    public function put(string $path, callable|string $handler): Route
    {
        return $this->addRoute('PUT', $path, $handler);
    }
    
    public function delete(string $path, callable|string $handler): Route
    {
        return $this->addRoute('DELETE', $path, $handler);
    }
    
    public function group(array $attributes, callable $callback): void
    {
        $this->groupStack[] = $attributes;
        $callback($this);
        array_pop($this->groupStack);
    }
    
    private function addRoute(string $method, string $path, callable|string $handler): Route
    {
        $route = new Route($method, $path, $handler);
        
        // 应用组属性
        foreach ($this->groupStack as $group) {
            if (isset($group['prefix'])) {
                $route->prefix($group['prefix']);
            }
            if (isset($group['middleware'])) {
                $route->middleware($group['middleware']);
            }
        }
        
        $this->routes[] = $route;
        return $route;
    }
    
    public function dispatch(Request $request): Response
    {
        foreach ($this->routes as $route) {
            if ($route->matches($request)) {
                return $route->handle($request);
            }
        }
        
        throw new NotFoundException("Route not found");
    }
}

class Route
{
    private string $method;
    private string $path;
    private callable|string $handler;
    private array $middleware = [];
    private ?string $prefix = null;
    
    public function __construct(string $method, string $path, callable|string $handler)
    {
        $this->method = $method;
        $this->path = $path;
        $this->handler = $handler;
    }
    
    public function prefix(string $prefix): self
    {
        $this->prefix = $prefix;
        return $this;
    }
    
    public function middleware(array|string $middleware): self
    {
        $this->middleware = array_merge($this->middleware, is_array($middleware) ? $middleware : [$middleware]);
        return $this;
    }
    
    public function matches(Request $request): bool
    {
        if ($request->getMethod() !== $this->method) {
            return false;
        }
        
        $path = $this->prefix ? $this->prefix . $this->path : $this->path;
        $pattern = $this->pathToRegex($path);
        
        return preg_match($pattern, $request->getPath(), $matches);
    }
    
    private function pathToRegex(string $path): string
    {
        $pattern = preg_replace('/\{(\w+)\}/', '(?P<$1>[^/]+)', $path);
        return '#^' . $pattern . '$#';
    }
    
    public function handle(Request $request): Response
    {
        // 执行中间件
        foreach ($this->middleware as $middleware) {
            $middleware->handle($request);
        }
        
        // 执行处理器
        if (is_string($this->handler)) {
            [$controller, $method] = explode('@', $this->handler);
            $instance = new $controller();
            return $instance->$method($request);
        }
        
        return call_user_func($this->handler, $request);
    }
}

// 使用
$router = new Router();

$router->get('/users', 'UserController@index');
$router->get('/users/{id}', 'UserController@show');

$router->group(['prefix' => '/api', 'middleware' => ['auth']], function($router) {
    $router->post('/users', 'UserController@store');
    $router->put('/users/{id}', 'UserController@update');
    $router->delete('/users/{id}', 'UserController@destroy');
});
```

## 最佳实践

1. **RESTful 路由**：遵循 RESTful 设计原则
2. **路由缓存**：生产环境使用路由缓存
3. **路由组**：使用路由组组织相关路由
4. **参数验证**：验证路由参数

## 注意事项

1. 路由顺序很重要，先匹配的路由优先
2. 避免路由冲突
3. 合理使用路由缓存
4. 路由参数应该验证

## 练习

1. 实现一个支持参数的路由系统。

2. 实现路由组功能，支持前缀和中间件。

3. 实现路由缓存，提升性能。

4. 创建一个 RESTful API 路由系统。

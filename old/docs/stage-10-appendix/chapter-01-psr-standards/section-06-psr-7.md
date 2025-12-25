# 10.1.6 PSR-7 HTTP 消息接口

## 概述

PSR-7（PHP Standards Recommendation 7）定义了 HTTP 消息的标准接口，实现了不同框架之间的 HTTP 消息互操作。通过使用 PSR-7 接口，可以在不同的框架和库中统一处理 HTTP 请求和响应。

## 官方文档

- **标准名称**：HTTP Message Interfaces
- **状态**：已接受（Accepted）
- **版本**：1.0.0
- **官方链接**：https://www.php-fig.org/psr/psr-7/

## 核心接口

### MessageInterface

所有 HTTP 消息的基础接口。

```php
<?php

namespace Psr\Http\Message;

interface MessageInterface
{
    public function getProtocolVersion(): string;
    public function withProtocolVersion(string $version): MessageInterface;
    public function getHeaders(): array;
    public function hasHeader(string $name): bool;
    public function getHeader(string $name): array;
    public function getHeaderLine(string $name): string;
    public function withHeader(string $name, $value): MessageInterface;
    public function withAddedHeader(string $name, $value): MessageInterface;
    public function withoutHeader(string $name): MessageInterface;
    public function getBody(): StreamInterface;
    public function withBody(StreamInterface $body): MessageInterface;
}
```

### RequestInterface

HTTP 请求接口，继承自 `MessageInterface`。

```php
<?php

namespace Psr\Http\Message;

interface RequestInterface extends MessageInterface
{
    public function getRequestTarget(): string;
    public function withRequestTarget(string $requestTarget): RequestInterface;
    public function getMethod(): string;
    public function withMethod(string $method): RequestInterface;
    public function getUri(): UriInterface;
    public function withUri(UriInterface $uri, bool $preserveHost = false): RequestInterface;
}
```

### ResponseInterface

HTTP 响应接口，继承自 `MessageInterface`。

```php
<?php

namespace Psr\Http\Message;

interface ResponseInterface extends MessageInterface
{
    public function getStatusCode(): int;
    public function withStatus(int $code, string $reasonPhrase = ''): ResponseInterface;
    public function getReasonPhrase(): string;
}
```

### ServerRequestInterface

服务器请求接口，继承自 `RequestInterface`。

```php
<?php

namespace Psr\Http\Message;

interface ServerRequestInterface extends RequestInterface
{
    public function getServerParams(): array;
    public function getCookieParams(): array;
    public function withCookieParams(array $cookies): ServerRequestInterface;
    public function getQueryParams(): array;
    public function withQueryParams(array $query): ServerRequestInterface;
    public function getUploadedFiles(): array;
    public function withUploadedFiles(array $uploadedFiles): ServerRequestInterface;
    public function getParsedBody();
    public function withParsedBody($data): ServerRequestInterface;
    public function getAttributes(): array;
    public function getAttribute(string $name, $default = null);
    public function withAttribute(string $name, $value): ServerRequestInterface;
    public function withoutAttribute(string $name): ServerRequestInterface;
}
```

### StreamInterface

流接口，用于处理消息体。

```php
<?php

namespace Psr\Http\Message;

interface StreamInterface
{
    public function __toString(): string;
    public function close(): void;
    public function detach();
    public function getSize(): ?int;
    public function tell(): int;
    public function eof(): bool;
    public function isSeekable(): bool;
    public function seek(int $offset, int $whence = SEEK_SET): void;
    public function rewind(): void;
    public function isWritable(): bool;
    public function write(string $string): int;
    public function isReadable(): bool;
    public function read(int $length): string;
    public function getContents(): string;
    public function getMetadata(?string $key = null);
}
```

### UriInterface

URI 接口。

```php
<?php

namespace Psr\Http\Message;

interface UriInterface
{
    public function getScheme(): string;
    public function getAuthority(): string;
    public function getUserInfo(): string;
    public function getHost(): string;
    public function getPort(): ?int;
    public function getPath(): string;
    public function getQuery(): string;
    public function getFragment(): string;
    public function withScheme(string $scheme): UriInterface;
    public function withUserInfo(string $user, ?string $password = null): UriInterface;
    public function withHost(string $host): UriInterface;
    public function withPort(?int $port): UriInterface;
    public function withPath(string $path): UriInterface;
    public function withQuery(string $query): UriInterface;
    public function withFragment(string $fragment): UriInterface;
    public function __toString(): string;
}
```

## 实际应用

### 创建请求

```php
<?php
declare(strict_types=1);

use Psr\Http\Message\RequestInterface;
use GuzzleHttp\Psr7\Request;

$request = new Request(
    'GET',
    'https://api.example.com/users',
    ['Content-Type' => 'application/json']
);
```

### 创建响应

```php
<?php
declare(strict_types=1);

use Psr\Http\Message\ResponseInterface;
use GuzzleHttp\Psr7\Response;

$response = new Response(
    200,
    ['Content-Type' => 'application/json'],
    json_encode(['message' => 'Success'])
);
```

### 在中间件中使用

```php
<?php
declare(strict_types=1);

use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

class AuthMiddleware
{
    public function __invoke(
        ServerRequestInterface $request,
        callable $next
    ): ResponseInterface {
        // 检查认证
        if (!$this->isAuthenticated($request)) {
            return new Response(401);
        }
        
        // 继续处理
        return $next($request);
    }
}
```

## 实现库

- **Guzzle PSR-7**：`guzzlehttp/psr7`
- **Slim PSR-7**：`slim/psr7`
- **Zend Diactoros**：`laminas/laminas-diactoros`

## 总结

PSR-7 提供了统一的 HTTP 消息接口，实现了：

- 框架和库之间的 HTTP 消息互操作
- 统一的请求和响应处理方式
- 中间件和组件的可重用性
- 更好的测试能力

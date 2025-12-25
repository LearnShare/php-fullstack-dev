# 9.1.5 API 接口设计

## 一、API 设计原则

### 1.1 RESTful 设计原则

1. **资源导向**：URL 表示资源，HTTP 方法表示操作
2. **无状态**：每个请求包含所有必要信息
3. **统一接口**：使用标准 HTTP 方法（GET、POST、PUT、DELETE）
4. **分层系统**：支持中间件和代理
5. **缓存友好**：支持 HTTP 缓存机制

### 1.2 API 版本控制

**URL 版本控制**：
- 基础路径：`/api/v1/`
- 示例：`/api/v1/articles`

**版本策略**：
- 主版本号（v1、v2）：不兼容的 API 变更
- 次版本号（v1.1、v1.2）：向后兼容的新功能

### 1.3 响应格式

**统一响应格式**：
```json
{
    "success": true,
    "data": {},
    "message": "操作成功",
    "meta": {
        "pagination": {}
    }
}
```

**错误响应格式**：
```json
{
    "success": false,
    "error": {
        "code": "ERROR_CODE",
        "message": "错误信息",
        "details": {}
    }
}
```

## 二、认证授权

### 2.1 认证方式

**JWT Token 认证**：
- 登录后获取 Access Token
- Token 放在请求头：`Authorization: Bearer {token}`
- Token 有效期：15 分钟
- Refresh Token 有效期：7 天

### 2.2 认证流程

1. **用户登录**：`POST /api/v1/auth/login`
2. **获取 Token**：返回 `access_token` 和 `refresh_token`
3. **请求 API**：在请求头携带 `Authorization: Bearer {access_token}`
4. **刷新 Token**：`POST /api/v1/auth/refresh`

## 三、用户相关 API

### 3.1 用户注册

**接口**：`POST /api/v1/auth/register`

**请求参数**：
```json
{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "nickname": "John Doe"
}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "user": {
            "id": 1,
            "username": "john_doe",
            "email": "john@example.com",
            "nickname": "John Doe",
            "avatar": null,
            "created_at": "2025-01-01T00:00:00.000000Z"
        },
        "message": "注册成功，请验证邮箱"
    }
}
```

**状态码**：
- `201 Created`：注册成功
- `422 Unprocessable Entity`：验证失败

### 3.2 用户登录

**接口**：`POST /api/v1/auth/login`

**请求参数**：
```json
{
    "email": "john@example.com",
    "password": "password123",
    "remember": false
}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "user": {
            "id": 1,
            "username": "john_doe",
            "email": "john@example.com",
            "nickname": "John Doe"
        },
        "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
        "token_type": "Bearer",
        "expires_in": 900
    }
}
```

**状态码**：
- `200 OK`：登录成功
- `401 Unauthorized`：认证失败

### 3.3 获取当前用户信息

**接口**：`GET /api/v1/auth/me`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "username": "john_doe",
        "email": "john@example.com",
        "nickname": "John Doe",
        "avatar": "https://example.com/avatar.jpg",
        "bio": "Software Developer",
        "website": "https://johndoe.com",
        "role": "user",
        "stats": {
            "articles_count": 10,
            "followers_count": 50,
            "following_count": 20,
            "likes_received": 100
        },
        "created_at": "2025-01-01T00:00:00.000000Z"
    }
}
```

**状态码**：
- `200 OK`：成功
- `401 Unauthorized`：未认证

### 3.4 更新用户资料

**接口**：`PUT /api/v1/users/{id}`

**请求头**：
```
Authorization: Bearer {access_token}
```

**请求参数**：
```json
{
    "nickname": "John Updated",
    "bio": "Updated bio",
    "website": "https://updated.com",
    "avatar": "https://example.com/new-avatar.jpg"
}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "username": "john_doe",
        "nickname": "John Updated",
        "bio": "Updated bio",
        "website": "https://updated.com",
        "avatar": "https://example.com/new-avatar.jpg",
        "updated_at": "2025-01-02T00:00:00.000000Z"
    },
    "message": "资料更新成功"
}
```

**状态码**：
- `200 OK`：更新成功
- `403 Forbidden`：无权限
- `422 Unprocessable Entity`：验证失败

### 3.5 获取用户信息

**接口**：`GET /api/v1/users/{id}`

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "username": "john_doe",
        "nickname": "John Doe",
        "avatar": "https://example.com/avatar.jpg",
        "bio": "Software Developer",
        "website": "https://johndoe.com",
        "stats": {
            "articles_count": 10,
            "followers_count": 50,
            "following_count": 20
        },
        "is_following": false,
        "created_at": "2025-01-01T00:00:00.000000Z"
    }
}
```

## 四、文章相关 API

### 4.1 获取文章列表

**接口**：`GET /api/v1/articles`

**查询参数**：
- `page`：页码（默认：1）
- `per_page`：每页数量（默认：20，最大：100）
- `category_id`：分类 ID（可选）
- `tag`：标签名称（可选）
- `author_id`：作者 ID（可选）
- `sort`：排序方式（`latest`、`popular`、`comments`，默认：`latest`）
- `search`：搜索关键词（可选）

**响应示例**：
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "title": "文章标题",
            "slug": "article-slug",
            "summary": "文章摘要",
            "cover_image": "https://example.com/cover.jpg",
            "author": {
                "id": 1,
                "username": "john_doe",
                "nickname": "John Doe",
                "avatar": "https://example.com/avatar.jpg"
            },
            "category": {
                "id": 1,
                "name": "技术",
                "slug": "tech"
            },
            "tags": [
                {
                    "id": 1,
                    "name": "PHP",
                    "slug": "php"
                }
            ],
            "stats": {
                "view_count": 100,
                "like_count": 50,
                "comment_count": 20
            },
            "published_at": "2025-01-01T00:00:00.000000Z",
            "created_at": "2025-01-01T00:00:00.000000Z"
        }
    ],
    "meta": {
        "pagination": {
            "current_page": 1,
            "per_page": 20,
            "total": 100,
            "last_page": 5,
            "from": 1,
            "to": 20
        }
    }
}
```

**状态码**：
- `200 OK`：成功

### 4.2 获取文章详情

**接口**：`GET /api/v1/articles/{id}` 或 `GET /api/v1/articles/{slug}`

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "title": "文章标题",
        "slug": "article-slug",
        "summary": "文章摘要",
        "content": "# 文章内容\n\nMarkdown 格式的内容...",
        "cover_image": "https://example.com/cover.jpg",
        "author": {
            "id": 1,
            "username": "john_doe",
            "nickname": "John Doe",
            "avatar": "https://example.com/avatar.jpg"
        },
        "category": {
            "id": 1,
            "name": "技术",
            "slug": "tech"
        },
        "tags": [
            {
                "id": 1,
                "name": "PHP",
                "slug": "php"
            }
        ],
        "status": "published",
        "visibility": "public",
        "stats": {
            "view_count": 100,
            "like_count": 50,
            "comment_count": 20
        },
        "is_liked": false,
        "is_favorited": false,
        "published_at": "2025-01-01T00:00:00.000000Z",
        "created_at": "2025-01-01T00:00:00.000000Z",
        "updated_at": "2025-01-01T00:00:00.000000Z"
    }
}
```

**状态码**：
- `200 OK`：成功
- `404 Not Found`：文章不存在

### 4.3 创建文章

**接口**：`POST /api/v1/articles`

**请求头**：
```
Authorization: Bearer {access_token}
```

**请求参数**：
```json
{
    "title": "新文章标题",
    "summary": "文章摘要",
    "content": "# 文章内容\n\nMarkdown 格式...",
    "cover_image": "https://example.com/cover.jpg",
    "category_id": 1,
    "tags": ["PHP", "Laravel"],
    "status": "draft",
    "visibility": "public"
}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "title": "新文章标题",
        "slug": "new-article-title",
        "status": "draft",
        "created_at": "2025-01-01T00:00:00.000000Z"
    },
    "message": "文章创建成功"
}
```

**状态码**：
- `201 Created`：创建成功
- `401 Unauthorized`：未认证
- `422 Unprocessable Entity`：验证失败

### 4.4 更新文章

**接口**：`PUT /api/v1/articles/{id}`

**请求头**：
```
Authorization: Bearer {access_token}
```

**请求参数**：
```json
{
    "title": "更新后的标题",
    "summary": "更新后的摘要",
    "content": "# 更新后的内容",
    "category_id": 2,
    "tags": ["PHP", "Laravel"],
    "status": "published"
}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "title": "更新后的标题",
        "updated_at": "2025-01-02T00:00:00.000000Z"
    },
    "message": "文章更新成功"
}
```

**状态码**：
- `200 OK`：更新成功
- `403 Forbidden`：无权限
- `404 Not Found`：文章不存在

### 4.5 删除文章

**接口**：`DELETE /api/v1/articles/{id}`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "message": "文章删除成功"
}
```

**状态码**：
- `200 OK`：删除成功
- `403 Forbidden`：无权限
- `404 Not Found`：文章不存在

## 五、评论相关 API

### 5.1 获取文章评论

**接口**：`GET /api/v1/articles/{id}/comments`

**查询参数**：
- `page`：页码（默认：1）
- `per_page`：每页数量（默认：20）

**响应示例**：
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "content": "这是一条评论",
            "author": {
                "id": 2,
                "username": "jane_doe",
                "nickname": "Jane Doe",
                "avatar": "https://example.com/avatar.jpg"
            },
            "parent_id": null,
            "replies": [
                {
                    "id": 2,
                    "content": "这是回复",
                    "author": {
                        "id": 1,
                        "username": "john_doe",
                        "nickname": "John Doe"
                    },
                    "parent_id": 1,
                    "created_at": "2025-01-01T01:00:00.000000Z"
                }
            ],
            "stats": {
                "like_count": 10
            },
            "created_at": "2025-01-01T00:00:00.000000Z"
        }
    ],
    "meta": {
        "pagination": {
            "current_page": 1,
            "per_page": 20,
            "total": 50
        }
    }
}
```

### 5.2 创建评论

**接口**：`POST /api/v1/articles/{id}/comments`

**请求头**：
```
Authorization: Bearer {access_token}
```

**请求参数**：
```json
{
    "content": "这是一条评论",
    "parent_id": null
}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "content": "这是一条评论",
        "author": {
            "id": 1,
            "username": "john_doe",
            "nickname": "John Doe"
        },
        "created_at": "2025-01-01T00:00:00.000000Z"
    },
    "message": "评论创建成功"
}
```

**状态码**：
- `201 Created`：创建成功
- `401 Unauthorized`：未认证
- `422 Unprocessable Entity`：验证失败

### 5.3 删除评论

**接口**：`DELETE /api/v1/comments/{id}`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "message": "评论删除成功"
}
```

## 六、互动功能 API

### 6.1 点赞文章

**接口**：`POST /api/v1/articles/{id}/like`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "liked": true,
        "like_count": 51
    },
    "message": "点赞成功"
}
```

### 6.2 取消点赞

**接口**：`DELETE /api/v1/articles/{id}/like`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "liked": false,
        "like_count": 50
    },
    "message": "取消点赞成功"
}
```

### 6.3 收藏文章

**接口**：`POST /api/v1/articles/{id}/favorite`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "favorited": true
    },
    "message": "收藏成功"
}
```

### 6.4 关注用户

**接口**：`POST /api/v1/users/{id}/follow`

**请求头**：
```
Authorization: Bearer {access_token}
```

**响应示例**：
```json
{
    "success": true,
    "data": {
        "following": true,
        "followers_count": 51
    },
    "message": "关注成功"
}
```

## 七、分类和标签 API

### 7.1 获取分类列表

**接口**：`GET /api/v1/categories`

**响应示例**：
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "name": "技术",
            "slug": "tech",
            "description": "技术相关文章",
            "icon": "https://example.com/icon.jpg",
            "article_count": 100
        }
    ]
}
```

### 7.2 获取标签列表

**接口**：`GET /api/v1/tags`

**查询参数**：
- `popular`：是否只返回热门标签（`true`/`false`）
- `limit`：返回数量（默认：50）

**响应示例**：
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "name": "PHP",
            "slug": "php",
            "color": "#777BB4",
            "usage_count": 100
        }
    ]
}
```

## 八、搜索 API

### 8.1 搜索文章

**接口**：`GET /api/v1/search/articles`

**查询参数**：
- `q`：搜索关键词（必填）
- `page`：页码（默认：1）
- `per_page`：每页数量（默认：20）

**响应示例**：
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "title": "PHP 教程",
            "summary": "PHP 学习指南",
            "author": {
                "id": 1,
                "username": "john_doe"
            },
            "highlight": {
                "title": "PHP <em>教程</em>",
                "summary": "PHP 学习<em>指南</em>"
            }
        }
    ],
    "meta": {
        "pagination": {
            "current_page": 1,
            "total": 50
        }
    }
}
```

## 九、错误处理

### 9.1 错误码定义

| HTTP 状态码 | 错误码 | 说明 |
|------------|--------|------|
| 400 | `BAD_REQUEST` | 请求参数错误 |
| 401 | `UNAUTHORIZED` | 未认证 |
| 403 | `FORBIDDEN` | 无权限 |
| 404 | `NOT_FOUND` | 资源不存在 |
| 422 | `VALIDATION_ERROR` | 验证失败 |
| 429 | `TOO_MANY_REQUESTS` | 请求过于频繁 |
| 500 | `INTERNAL_ERROR` | 服务器内部错误 |

### 9.2 错误响应示例

**验证错误**：
```json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "验证失败",
        "details": {
            "email": ["邮箱格式不正确"],
            "password": ["密码长度至少 8 位"]
        }
    }
}
```

**资源不存在**：
```json
{
    "success": false,
    "error": {
        "code": "NOT_FOUND",
        "message": "文章不存在"
    }
}
```

**权限不足**：
```json
{
    "success": false,
    "error": {
        "code": "FORBIDDEN",
        "message": "无权访问此资源"
    }
}
```

## 十、Rate Limiting

### 10.1 限流规则

- **认证接口**：每分钟最多 5 次
- **创建内容**：每分钟最多 10 次
- **其他接口**：每分钟最多 60 次

### 10.2 限流响应

**响应头**：
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1640995200
```

**响应体**：
```json
{
    "success": false,
    "error": {
        "code": "TOO_MANY_REQUESTS",
        "message": "请求过于频繁，请稍后再试"
    }
}
```

## 十一、API 文档工具

### 11.1 文档生成

**使用工具**：
- Laravel API Documentation（Laravel 内置）
- Swagger/OpenAPI（可选）
- Postman Collection（可选）

### 11.2 文档内容

1. **接口列表**：所有可用接口
2. **请求示例**：请求方法和参数
3. **响应示例**：成功和错误响应
4. **认证说明**：如何获取和使用 Token
5. **错误码说明**：所有错误码和含义

## 十二、下一步

完成 API 设计后，继续学习：
- [模块划分与实现指南](section-06-module-design.md)：模块划分和实现步骤

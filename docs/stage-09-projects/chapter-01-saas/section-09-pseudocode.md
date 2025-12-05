# 9.1.9 伪代码示例

## 一、租户识别中间件

### 1.1 伪代码

```
函数 handle(request, next):
    // 1. 从请求中提取租户信息
    tenant = 识别租户(request)
    
    // 2. 如果无法识别租户，返回错误
    如果 tenant == null:
        返回 400 错误("无法识别租户")
    
    // 3. 检查租户状态
    如果 tenant.status == "suspended":
        返回 403 错误("租户已被暂停")
    
    如果 tenant.status == "cancelled":
        返回 403 错误("租户已取消")
    
    // 4. 设置租户上下文
    设置上下文("current_tenant", tenant)
    
    // 5. 继续处理请求
    返回 next(request)

函数 识别租户(request):
    // 方式1: 从 Header 识别
    tenantId = request.headers["X-Tenant-ID"]
    如果 tenantId != null:
        返回 查询租户(tenantId)
    
    // 方式2: 从子域名识别
    host = request.headers["Host"]
    子域名 = 提取子域名(host)
    如果 子域名 != null:
        返回 查询租户(子域名)
    
    // 方式3: 从自定义域名识别
    返回 查询租户(host)

函数 提取子域名(host):
    部分 = 分割(host, ".")
    如果 部分.length >= 3:
        返回 部分[0]  // 第一个部分就是子域名
    返回 null

函数 查询租户(identifier):
    返回 租户仓储.查找(identifier)
```

### 1.2 实现要点

- 识别优先级：Header > 子域名 > 自定义域名
- 识别失败时返回明确的错误信息
- 租户上下文存储在请求上下文中，后续所有查询自动使用

## 二、用户注册流程

### 2.1 伪代码

```
函数 register(注册数据):
    // 1. 验证输入
    验证结果 = 验证输入(注册数据)
    如果 验证结果.失败:
        返回 错误(验证结果.错误信息)
    
    // 2. 获取当前租户（已通过中间件识别）
    租户 = 获取当前租户()
    
    // 3. 检查邮箱是否已存在
    如果 用户仓储.查找邮箱(注册数据.邮箱, 租户.id) != null:
        返回 错误("邮箱已被使用")
    
    // 4. 检查子域名是否已存在（如果是租户注册）
    如果 注册数据.子域名 != null:
        如果 租户仓储.查找子域名(注册数据.子域名) != null:
            返回 错误("子域名已被使用")
    
    // 5. 创建租户（如果是租户注册）
    如果 注册数据.子域名 != null:
        租户 = 创建租户(注册数据)
    
    // 6. 创建用户
    用户 = 创建用户(注册数据, 租户.id)
    
    // 7. 分配角色
    如果 注册数据.是管理员:
        角色 = 角色仓储.查找("super_admin", 租户.id)
        用户角色仓储.关联(用户.id, 角色.id)
    
    // 8. 发送验证邮件
    令牌 = 生成验证令牌()
    缓存.设置("email_verification:" + 用户.id, 令牌, 86400)
    邮件服务.发送验证邮件(用户.邮箱, 令牌)
    
    // 9. 返回结果
    返回 成功({
        "tenant_id": 租户.id,
        "user_id": 用户.id,
        "message": "注册成功，请查收验证邮件"
    })

函数 创建用户(注册数据, 租户id):
    用户 = 新建用户()
    用户.id = 生成UUID()
    用户.tenant_id = 租户id
    用户.email = 注册数据.邮箱
    用户.name = 注册数据.姓名
    用户.password_hash = 加密密码(注册数据.密码)
    用户.status = "inactive"
    用户仓储.保存(用户)
    返回 用户

函数 加密密码(明文密码):
    返回 password_hash(明文密码, "argon2id")
```

### 2.2 实现要点

- 输入验证要全面，包括格式、唯一性等
- 密码使用强加密算法
- 邮箱验证令牌存储在缓存中，有过期时间
- 新用户默认状态为未激活

## 三、用户登录流程

### 3.1 伪代码

```
函数 login(登录数据):
    // 1. 获取当前租户
    租户 = 获取当前租户()
    
    // 2. 查找用户
    用户 = 用户仓储.查找邮箱(登录数据.邮箱, 租户.id)
    如果 用户 == null:
        返回 错误("邮箱或密码错误")  // 不透露用户是否存在
    
    // 3. 验证密码
    如果 验证密码(登录数据.密码, 用户.password_hash) == false:
        增加失败次数(用户.id)
        如果 失败次数 >= 5:
            锁定账号(用户.id, 1800)  // 锁定30分钟
        返回 错误("邮箱或密码错误")
    
    // 4. 检查用户状态
    如果 用户.status == "suspended":
        返回 错误("账号已被暂停")
    
    如果 用户.status == "inactive":
        // 可以登录但功能受限
        警告 = "邮箱未验证，部分功能受限"
    
    // 5. 检查账号是否被锁定
    如果 账号被锁定(用户.id):
        剩余时间 = 获取剩余锁定时间(用户.id)
        返回 错误("账号已被锁定，请" + 剩余时间 + "分钟后重试")
    
    // 6. 生成 Token
    token = 生成JWTToken({
        "user_id": 用户.id,
        "tenant_id": 租户.id,
        "email": 用户.email,
        "roles": 获取用户角色(用户.id)
    })
    
    refreshToken = 生成RefreshToken(用户.id)
    
    // 7. 更新登录信息
    用户.last_login_at = 当前时间()
    重置失败次数(用户.id)
    用户仓储.保存(用户)
    
    // 8. 记录登录日志
    日志.记录("用户登录", {
        "user_id": 用户.id,
        "tenant_id": 租户.id,
        "ip": 请求IP(),
        "time": 当前时间()
    })
    
    // 9. 返回结果
    返回 成功({
        "token": token,
        "refresh_token": refreshToken,
        "expires_in": 86400,
        "user": {
            "id": 用户.id,
            "email": 用户.email,
            "name": 用户.name,
            "roles": 获取用户角色(用户.id)
        },
        "warning": 警告  // 如果有
    })

函数 验证密码(明文密码, 密码哈希):
    返回 password_verify(明文密码, 密码哈希)

函数 生成JWTToken(载荷):
    头部 = {
        "alg": "HS256",
        "typ": "JWT"
    }
    
    载荷.iat = 当前时间戳()
    载荷.exp = 当前时间戳() + 86400  // 24小时后过期
    
    签名 = HMAC_SHA256(base64(头部) + "." + base64(载荷), 密钥)
    
    返回 base64(头部) + "." + base64(载荷) + "." + base64(签名)
```

### 3.2 实现要点

- 密码错误时不透露用户是否存在（安全考虑）
- 失败次数达到阈值后锁定账号
- Token 包含必要信息但不过多
- 记录登录日志用于审计

## 四、权限验证中间件

### 4.1 伪代码

```
函数 handle(request, next, 需要的权限):
    // 1. 获取当前用户
    用户 = 获取当前用户()
    如果 用户 == null:
        返回 401 错误("未授权")
    
    // 2. 获取用户权限
    用户权限 = 获取用户权限(用户.id)
    
    // 3. 检查权限
    如果 检查权限(用户权限, 需要的权限) == false:
        返回 403 错误("权限不足")
    
    // 4. 如果涉及工作空间，检查工作空间权限
    如果 request.包含工作空间ID():
        工作空间ID = request.获取工作空间ID()
        工作空间权限 = 获取工作空间权限(用户.id, 工作空间ID)
        如果 工作空间权限 == null:
            返回 403 错误("无权访问此工作空间")
        
        // 检查工作空间级权限
        如果 检查权限(工作空间权限, 需要的权限) == false:
            返回 403 错误("权限不足")
    
    // 5. 继续处理请求
    返回 next(request)

函数 获取用户权限(用户ID):
    // 获取用户的所有角色
    角色列表 = 用户角色仓储.查找用户角色(用户ID)
    
    // 合并所有角色的权限（取并集）
    权限集合 = 新建集合()
    对于 每个角色 in 角色列表:
        如果 角色.permissions == ["*"]:
            返回 ["*"]  // 超级管理员，所有权限
        否则:
            权限集合.添加所有(角色.permissions)
    
    返回 权限集合.转数组()

函数 检查权限(用户权限, 需要的权限):
    // 如果用户有所有权限
    如果 用户权限.包含("*"):
        返回 true
    
    // 检查是否有需要的权限
    返回 用户权限.包含(需要的权限)

函数 获取工作空间权限(用户ID, 工作空间ID):
    成员 = 工作空间成员仓储.查找(工作空间ID, 用户ID)
    如果 成员 == null:
        返回 null
    
    // 根据工作空间角色返回权限
    返回 工作空间角色权限映射[成员.role]
```

### 4.2 实现要点

- 权限检查顺序：租户级 → 工作空间级 → 资源级
- 超级管理员拥有所有权限（`*`）
- 权限取并集（用户有多个角色时）

## 五、创建工作空间流程

### 5.1 伪代码

```
函数 create(工作空间数据):
    // 1. 获取当前用户和租户
    用户 = 获取当前用户()
    租户 = 获取当前租户()
    
    // 2. 检查权限
    如果 权限检查器.检查(用户.id, "workspaces.create") == false:
        返回 403 错误("权限不足")
    
    // 3. 检查工作空间数量限制
    当前数量 = 工作空间仓储.统计(租户.id)
    限制 = 获取套餐限制(租户.id, "max_workspaces")
    如果 限制 != -1 并且 当前数量 >= 限制:
        返回 错误("已达到工作空间数量限制，请升级套餐")
    
    // 4. 验证输入
    验证结果 = 验证工作空间数据(工作空间数据)
    如果 验证结果.失败:
        返回 错误(验证结果.错误信息)
    
    // 5. 检查名称唯一性
    如果 工作空间仓储.查找名称(租户.id, 工作空间数据.名称) != null:
        返回 错误("工作空间名称已存在")
    
    // 6. 创建工作空间
    工作空间 = 新建工作空间()
    工作空间.id = 生成UUID()
    工作空间.tenant_id = 租户.id
    工作空间.name = 工作空间数据.名称
    工作空间.description = 工作空间数据.描述
    工作空间.owner_id = 用户.id
    工作空间仓储.保存(工作空间)
    
    // 7. 创建者自动成为成员
    成员 = 新建工作空间成员()
    成员.workspace_id = 工作空间.id
    成员.user_id = 用户.id
    成员.role = "owner"
    工作空间成员仓储.保存(成员)
    
    // 8. 发布事件（可选）
    事件发布器.发布("workspace.created", {
        "workspace_id": 工作空间.id,
        "tenant_id": 租户.id,
        "owner_id": 用户.id
    })
    
    // 9. 返回结果
    返回 成功({
        "workspace": {
            "id": 工作空间.id,
            "name": 工作空间.name,
            "description": 工作空间.description,
            "owner": {
                "id": 用户.id,
                "name": 用户.name
            },
            "created_at": 工作空间.created_at
        }
    })

函数 获取套餐限制(租户ID, 限制类型):
    订阅 = 订阅仓储.查找活跃订阅(租户ID)
    如果 订阅 == null:
        返回 默认限制[限制类型]  // 免费版限制
    
    套餐 = 套餐仓储.查找(订阅.plan_id)
    返回 套餐.limits[限制类型]
```

### 5.2 实现要点

- 创建前检查权限和限制
- 创建者自动成为所有者
- 名称在租户内唯一
- 使用事件系统解耦（可选）

## 六、订阅创建流程

### 6.1 伪代码

```
函数 create(订阅数据):
    // 1. 获取当前用户和租户
    用户 = 获取当前用户()
    租户 = 获取当前租户()
    
    // 2. 检查权限（只有管理员可以管理订阅）
    如果 权限检查器.检查(用户.id, "subscriptions.manage") == false:
        返回 403 错误("权限不足")
    
    // 3. 查询套餐
    套餐 = 套餐仓储.查找(订阅数据.plan_id)
    如果 套餐 == null 或 套餐.is_active == false:
        返回 错误("套餐不存在或已停用")
    
    // 4. 检查现有订阅
    现有订阅 = 订阅仓储.查找活跃订阅(租户.id)
    如果 现有订阅 != null:
        // 如果是升级，创建新订阅，旧订阅在周期结束后取消
        如果 套餐.价格 > 现有订阅.套餐.价格:
            处理升级(租户.id, 现有订阅, 套餐, 订阅数据.payment_method_id)
        否则:
            返回 错误("只能升级套餐，降级请联系客服")
    
    // 5. 获取或创建 Stripe Customer
    stripeCustomer = Stripe适配器.获取或创建客户(租户)
    
    // 6. 创建 Stripe 订阅
    stripeSubscription = Stripe适配器.创建订阅({
        "customer_id": stripeCustomer.id,
        "plan_id": 套餐.stripe_plan_id,
        "payment_method_id": 订阅数据.payment_method_id
    })
    
    // 7. 创建本地订阅记录
    订阅 = 新建订阅()
    订阅.id = 生成UUID()
    订阅.tenant_id = 租户.id
    订阅.plan_id = 套餐.id
    订阅.stripe_subscription_id = stripeSubscription.id
    订阅.status = stripeSubscription.status
    订阅.current_period_start = 时间戳转日期(stripeSubscription.current_period_start)
    订阅.current_period_end = 时间戳转日期(stripeSubscription.current_period_end)
    订阅仓储.保存(订阅)
    
    // 8. 更新租户订阅信息
    租户.subscription_id = 订阅.id
    租户仓储.保存(租户)
    
    // 9. 发送确认邮件
    邮件服务.发送订阅确认邮件(用户.email, {
        "plan_name": 套餐.display_name,
        "price": 套餐.price_monthly,
        "period_end": 订阅.current_period_end
    })
    
    // 10. 返回结果
    返回 成功({
        "subscription": {
            "id": 订阅.id,
            "plan": {
                "id": 套餐.id,
                "name": 套餐.name,
                "display_name": 套餐.display_name
            },
            "status": 订阅.status,
            "current_period_end": 订阅.current_period_end
        }
    })

函数 处理升级(租户ID, 旧订阅, 新套餐, 支付方式ID):
    // 创建新订阅
    新订阅 = 创建订阅(租户ID, 新套餐, 支付方式ID)
    
    // 标记旧订阅在周期结束后取消
    旧订阅.cancel_at_period_end = true
    订阅仓储.保存(旧订阅)
    
    // 立即更新租户的订阅ID（使用新订阅）
    租户 = 租户仓储.查找(租户ID)
    租户.subscription_id = 新订阅.id
    租户仓储.保存(租户)
```

### 6.2 实现要点

- 订阅创建需要支付方式
- 升级时立即生效，旧订阅在周期结束后取消
- 降级需要特殊处理（通常需要客服介入）
- Stripe Webhook 会异步更新订阅状态

## 七、Stripe Webhook 处理

### 7.1 伪代码

```
函数 handleWebhook(request):
    // 1. 验证 Webhook 签名
    如果 验证签名(request) == false:
        返回 400 错误("无效签名")
    
    // 2. 解析事件
    事件 = 解析JSON(request.body)
    事件类型 = 事件.type
    事件数据 = 事件.data.object
    
    // 3. 处理不同事件类型
    匹配 事件类型:
        情况 "customer.subscription.created":
            处理订阅创建(事件数据)
        
        情况 "customer.subscription.updated":
            处理订阅更新(事件数据)
        
        情况 "customer.subscription.deleted":
            处理订阅删除(事件数据)
        
        情况 "invoice.payment_succeeded":
            处理支付成功(事件数据)
        
        情况 "invoice.payment_failed":
            处理支付失败(事件数据)
        
        默认:
            日志.记录("未知事件类型", 事件类型)
    
    // 4. 返回成功
    返回 200 成功()

函数 处理订阅更新(stripeSubscription):
    // 查找本地订阅
    订阅 = 订阅仓储.查找StripeID(stripeSubscription.id)
    如果 订阅 == null:
        日志.记录("订阅不存在", stripeSubscription.id)
        返回
    
    // 更新订阅信息
    订阅.status = stripeSubscription.status
    订阅.current_period_start = 时间戳转日期(stripeSubscription.current_period_start)
    订阅.current_period_end = 时间戳转日期(stripeSubscription.current_period_end)
    订阅.cancel_at_period_end = stripeSubscription.cancel_at_period_end
    
    如果 stripeSubscription.canceled_at != null:
        订阅.cancelled_at = 时间戳转日期(stripeSubscription.canceled_at)
    
    订阅仓储.保存(订阅)
    
    // 如果订阅被取消，更新租户状态
    如果 订阅.status == "cancelled":
        租户 = 租户仓储.查找(订阅.tenant_id)
        // 可以标记租户状态或发送通知

函数 处理支付成功(invoice):
    // 生成发票记录
    发票 = 新建发票()
    发票.id = 生成UUID()
    发票.tenant_id = 查找租户ID(invoice.customer)
    发票.stripe_invoice_id = invoice.id
    发票.amount = invoice.amount_paid
    发票.currency = invoice.currency
    发票.status = "paid"
    发票.paid_at = 当前时间()
    发票仓储.保存(发票)
    
    // 发送发票邮件
    租户 = 租户仓储.查找(发票.tenant_id)
    邮件服务.发送发票邮件(租户.管理员邮箱, 发票)

函数 处理支付失败(invoice):
    // 查找订阅
    订阅 = 订阅仓储.查找StripeID(invoice.subscription)
    如果 订阅 == null:
        返回
    
    // 更新订阅状态
    订阅.status = "past_due"
    订阅仓储.保存(订阅)
    
    // 发送支付失败通知
    租户 = 租户仓储.查找(订阅.tenant_id)
    邮件服务.发送支付失败通知(租户.管理员邮箱, {
        "subscription_id": 订阅.id,
        "amount": invoice.amount_due,
        "due_date": invoice.due_date
    })
```

### 7.2 实现要点

- Webhook 签名验证很重要，防止伪造请求
- 事件处理要幂等（多次处理结果一致）
- 支付失败时要及时通知用户
- 发票要保存记录，便于对账

## 八、数据查询自动过滤

### 8.1 伪代码

```
类 租户查询作用域:
    函数 apply(查询构建器, 模型):
        // 自动添加租户过滤条件
        租户 = 获取当前租户()
        如果 租户 != null:
            查询构建器.where("tenant_id", 租户.id)
    
    函数 extend(查询构建器):
        // 提供便捷方法
        查询构建器.宏("withoutTenant", function():
            // 移除租户过滤（仅管理员使用）
            返回 this.removeGlobalScope("tenant")
        )

// 使用示例
用户列表 = User::where("status", "active").get()
// 实际执行的 SQL:
// SELECT * FROM users 
// WHERE tenant_id = 'tenant_abc_123' AND status = 'active'

// 管理员查询所有租户的数据
所有用户 = User::withoutTenant()->get()
// 实际执行的 SQL:
// SELECT * FROM users WHERE status = 'active'
```

### 8.2 实现要点

- 所有业务模型自动应用租户作用域
- 提供方法移除作用域（仅管理员使用）
- 确保所有查询都包含租户过滤

## 九、总结

### 9.1 关键实现点

1. **租户隔离**：所有查询自动添加租户过滤
2. **权限控制**：多层权限验证（租户级、工作空间级、资源级）
3. **安全考虑**：密码加密、Token 管理、Webhook 签名验证
4. **错误处理**：明确的错误信息和错误码
5. **事件驱动**：使用事件系统解耦模块

### 9.2 注意事项

- 所有涉及租户的操作都要验证租户上下文
- 权限检查要全面，不能遗漏
- 支付相关操作要处理各种异常情况
- 日志记录要完整，便于问题排查

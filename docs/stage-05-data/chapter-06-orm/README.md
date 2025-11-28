# 5.6 ORM 框架与数据迁移

## 目标

- 理解 ORM（Object-Relational Mapping）的概念与优势。
- 掌握 Eloquent ORM 的基本用法。
- 了解 Doctrine ORM 的特点与使用。
- 熟悉数据库迁移（Migration）的概念与实现。
- 掌握 Seeder 数据填充的使用。

## ORM 基础

### 什么是 ORM

- **ORM**：Object-Relational Mapping（对象关系映射）。
- 将数据库表映射为对象，简化数据库操作。
- 提供类型安全、自动关联、查询构建等功能。

### ORM 优势

- **类型安全**：使用对象而非 SQL 字符串。
- **代码复用**：模型可在多处使用。
- **自动关联**：处理表之间的关系。
- **迁移管理**：版本化数据库结构。

### ORM vs 原生 SQL

| 特性           | ORM                    | 原生 SQL                |
| :------------- | :--------------------- | :---------------------- |
| 学习曲线       | 需要学习 ORM API       | 需要学习 SQL            |
| 类型安全       | 是                     | 否                      |
| 性能           | 可能稍慢               | 最快                    |
| 可维护性       | 高                     | 中等                    |
| 复杂查询       | 可能受限               | 完全支持                |

## Eloquent ORM

### 基础模型

```php
<?php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'users';
    protected $primaryKey = 'id';
    public $timestamps = true;
    
    protected $fillable = [
        'username',
        'email',
        'password_hash',
    ];
    
    protected $hidden = [
        'password_hash',
    ];
    
    protected $casts = [
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];
}
```

### CRUD 操作

```php
<?php
declare(strict_types=1);

use App\Models\User;

// Create
$user = new User();
$user->username = 'alice';
$user->email = 'alice@example.com';
$user->password_hash = password_hash('password', PASSWORD_DEFAULT);
$user->save();

// 或使用 create
$user = User::create([
    'username' => 'bob',
    'email' => 'bob@example.com',
    'password_hash' => password_hash('password', PASSWORD_DEFAULT),
]);

// Read
$user = User::find(1);
$user = User::where('email', 'alice@example.com')->first();
$users = User::where('status', 'active')->get();

// Update
$user = User::find(1);
$user->email = 'newemail@example.com';
$user->save();

// 或使用 update
User::where('id', 1)->update(['email' => 'newemail@example.com']);

// Delete
$user = User::find(1);
$user->delete();

// 或使用 destroy
User::destroy(1);
```

### 查询构建器

```php
<?php
declare(strict_types=1);

// 条件查询
$users = User::where('status', 'active')
    ->where('age', '>', 18)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();

// 关联查询
$users = User::whereHas('orders', function ($query) {
    $query->where('total', '>', 100);
})->get();

// 聚合查询
$count = User::where('status', 'active')->count();
$avgAge = User::avg('age');
$maxAge = User::max('age');
```

### 关联关系

```php
<?php
declare(strict_types=1);

class User extends Model
{
    // 一对多
    public function orders()
    {
        return $this->hasMany(Order::class);
    }
    
    // 多对多
    public function roles()
    {
        return $this->belongsToMany(Role::class, 'user_roles');
    }
}

class Order extends Model
{
    // 多对一
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    // 一对多
    public function items()
    {
        return $this->hasMany(OrderItem::class);
    }
}

// 使用关联
$user = User::find(1);
$orders = $user->orders;  // 获取用户的所有订单

$order = Order::find(1);
$user = $order->user;  // 获取订单的用户
```

## Doctrine ORM

### 实体定义

```php
<?php
declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ORM\Table(name: 'users')]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;
    
    #[ORM\Column(type: 'string', length: 255)]
    private string $username;
    
    #[ORM\Column(type: 'string', length: 255)]
    private string $email;
    
    #[ORM\OneToMany(targetEntity: Order::class, mappedBy: 'user')]
    private Collection $orders;
    
    public function __construct()
    {
        $this->orders = new ArrayCollection();
    }
    
    // Getters and Setters
    public function getId(): ?int
    {
        return $this->id;
    }
    
    public function getUsername(): string
    {
        return $this->username;
    }
    
    public function setUsername(string $username): self
    {
        $this->username = $username;
        return $this;
    }
}
```

### Repository 模式

```php
<?php
declare(strict_types=1);

namespace App\Repository;

use App\Entity\User;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

/**
 * 用户仓储类
 * 
 * Doctrine 的 Repository 模式提供了数据访问的封装层
 * 将查询逻辑集中在 Repository 中，保持实体类的简洁
 */
class UserRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        // 调用父类构造函数，注册实体类型
        parent::__construct($registry, User::class);
    }
    
    /**
     * 根据邮箱查找用户
     * 
     * @param string $email 用户邮箱
     * @return User|null 找到的用户，如果不存在返回 null
     */
    public function findByEmail(string $email): ?User
    {
        // 使用 QueryBuilder 构建类型安全的查询
        // 'u' 是 User 实体的别名
        return $this->createQueryBuilder('u')
            ->where('u.email = :email')           // WHERE 条件
            ->setParameter('email', $email)       // 绑定参数，防止 SQL 注入
            ->getQuery()                          // 获取 Query 对象
            ->getOneOrNullResult();               // 执行查询，返回单个结果或 null
    }
    
    /**
     * 查找所有活跃用户
     * 
     * @return array<User> 活跃用户数组
     */
    public function findActiveUsers(): array
    {
        return $this->createQueryBuilder('u')
            ->where('u.status = :status')         // 状态过滤
            ->setParameter('status', 'active')     // 参数绑定
            ->orderBy('u.createdAt', 'DESC')      // 按创建时间降序排序
            ->getQuery()
            ->getResult();                        // 返回结果数组
    }
    
    /**
     * 使用 DQL（Doctrine Query Language）查询
     * 
     * DQL 类似于 SQL，但操作的是实体而非表
     */
    public function findUsersWithOrders(): array
    {
        return $this->createQueryBuilder('u')
            ->innerJoin('u.orders', 'o')          // 内连接订单表
            ->where('o.status = :status')         // 订单状态过滤
            ->setParameter('status', 'completed')
            ->groupBy('u.id')                     // 按用户分组
            ->having('COUNT(o.id) > :minOrders')  // 订单数量过滤
            ->setParameter('minOrders', 5)
            ->getQuery()
            ->getResult();
    }
}
```

### Doctrine 查询语言（DQL）

```php
<?php
declare(strict_types=1);

namespace App\Repository;

use App\Entity\User;
use Doctrine\ORM\EntityManagerInterface;

class UserRepository
{
    public function __construct(
        private EntityManagerInterface $em
    ) {}
    
    /**
     * 使用原生 DQL 查询
     * 
     * DQL 操作的是实体和属性，而非表和列
     */
    public function findUsersByRole(string $role): array
    {
        // DQL 查询：SELECT u FROM App\Entity\User u WHERE u.role = :role
        $dql = "SELECT u FROM App\Entity\User u WHERE u.role = :role";
        
        $query = $this->em->createQuery($dql);
        $query->setParameter('role', $role);
        
        return $query->getResult();
    }
    
    /**
     * 使用原生 SQL 查询（需要时）
     * 
     * 当 DQL 无法满足复杂查询需求时，可以使用原生 SQL
     */
    public function findUsersWithComplexQuery(): array
    {
        $sql = "
            SELECT u.*, COUNT(o.id) as order_count
            FROM users u
            LEFT JOIN orders o ON u.id = o.user_id
            WHERE u.status = 'active'
            GROUP BY u.id
            HAVING order_count > 10
        ";
        
        $stmt = $this->em->getConnection()->prepare($sql);
        $result = $stmt->executeQuery();
        
        // 将结果映射到实体
        return $this->em->getRepository(User::class)
            ->createQueryBuilder('u')
            ->where('u.id IN (:ids)')
            ->setParameter('ids', array_column($result->fetchAllAssociative(), 'id'))
            ->getQuery()
            ->getResult();
    }
}
```

### Doctrine 关联关系详解

```php
<?php
declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\Collection;
use Doctrine\Common\Collections\ArrayCollection;

#[ORM\Entity]
#[ORM\Table(name: 'users')]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;
    
    #[ORM\Column(type: 'string', length: 255)]
    private string $username;
    
    /**
     * 一对多关联：一个用户有多个订单
     * 
     * @var Collection<Order> 订单集合
     */
    #[ORM\OneToMany(targetEntity: Order::class, mappedBy: 'user')]
    private Collection $orders;
    
    /**
     * 多对多关联：一个用户有多个角色
     * 
     * @var Collection<Role> 角色集合
     */
    #[ORM\ManyToMany(targetEntity: Role::class, inversedBy: 'users')]
    #[ORM\JoinTable(name: 'user_roles')]
    private Collection $roles;
    
    public function __construct()
    {
        // 初始化集合，避免 null 检查
        $this->orders = new ArrayCollection();
        $this->roles = new ArrayCollection();
    }
    
    /**
     * 获取订单集合
     * 
     * @return Collection<Order>
     */
    public function getOrders(): Collection
    {
        return $this->orders;
    }
    
    /**
     * 添加订单
     * 
     * @param Order $order 要添加的订单
     */
    public function addOrder(Order $order): void
    {
        if (!$this->orders->contains($order)) {
            $this->orders->add($order);
            $order->setUser($this);  // 维护双向关联
        }
    }
    
    /**
     * 移除订单
     * 
     * @param Order $order 要移除的订单
     */
    public function removeOrder(Order $order): void
    {
        if ($this->orders->removeElement($order)) {
            $order->setUser(null);  // 解除关联
        }
    }
}
```

### Doctrine 事务管理

```php
<?php
declare(strict_types=1);

namespace App\Service;

use Doctrine\ORM\EntityManagerInterface;
use App\Entity\User;
use App\Entity\Order;

class OrderService
{
    public function __construct(
        private EntityManagerInterface $em
    ) {}
    
    /**
     * 创建订单（事务处理）
     * 
     * Doctrine 自动管理事务，但也可以手动控制
     */
    public function createOrder(User $user, array $items): Order
    {
        // 开始事务（如果尚未开始）
        $this->em->beginTransaction();
        
        try {
            // 创建订单实体
            $order = new Order();
            $order->setUser($user);
            $order->setStatus('pending');
            
            // 添加订单项
            foreach ($items as $itemData) {
                $orderItem = new OrderItem();
                $orderItem->setProductId($itemData['product_id']);
                $orderItem->setQuantity($itemData['quantity']);
                $order->addItem($orderItem);
            }
            
            // 持久化实体（此时还未写入数据库）
            $this->em->persist($order);
            
            // 刷新到数据库（可选，用于获取生成的 ID）
            $this->em->flush();
            
            // 提交事务
            $this->em->commit();
            
            return $order;
            
        } catch (\Exception $e) {
            // 发生错误，回滚事务
            $this->em->rollback();
            throw $e;
        }
    }
    
    /**
     * 批量操作优化
     * 
     * 使用批量处理提高性能
     */
    public function batchCreateOrders(array $ordersData): void
    {
        $batchSize = 20;  // 每批处理 20 条
        
        foreach ($ordersData as $i => $orderData) {
            $order = new Order();
            // ... 设置订单属性
            
            $this->em->persist($order);
            
            // 每处理 batchSize 条记录，刷新一次
            if (($i % $batchSize) === 0) {
                $this->em->flush();
                $this->em->clear();  // 清除实体管理器，释放内存
            }
        }
        
        // 处理剩余记录
        $this->em->flush();
        $this->em->clear();
    }
}
```

## 数据库迁移

### 什么是迁移

- **迁移**：版本化数据库结构变更。
- 记录数据库的每次变更。
- 支持回滚和版本管理。

### Eloquent 迁移

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUsersTable extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('username')->unique();
            $table->string('email')->unique();
            $table->string('password_hash');
            $table->timestamps();
            $table->index('email');
        });
    }
    
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
}
```

### Doctrine 迁移

```php
<?php
declare(strict_types=1);

namespace DoctrineMigrations;

use Doctrine\DBAL\Schema\Schema;
use Doctrine\Migrations\AbstractMigration;

final class Version20250101000000 extends AbstractMigration
{
    public function up(Schema $schema): void
    {
        $this->addSql('
            CREATE TABLE users (
                id INT AUTO_INCREMENT PRIMARY KEY,
                username VARCHAR(255) NOT NULL,
                email VARCHAR(255) NOT NULL,
                password_hash VARCHAR(255) NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                UNIQUE KEY uk_username (username),
                UNIQUE KEY uk_email (email),
                INDEX idx_email (email)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
        ');
    }
    
    public function down(Schema $schema): void
    {
        $this->addSql('DROP TABLE users');
    }
}
```

### 迁移命令

```bash
# 运行迁移
php artisan migrate

# 回滚迁移
php artisan migrate:rollback

# 回滚所有迁移
php artisan migrate:reset

# 查看迁移状态
php artisan migrate:status

# Doctrine
php bin/console doctrine:migrations:migrate
php bin/console doctrine:migrations:rollback
```

## Seeder 数据填充

### Eloquent Seeder

```php
<?php
declare(strict_types=1);

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::create([
            'username' => 'admin',
            'email' => 'admin@example.com',
            'password_hash' => Hash::make('password'),
        ]);
        
        User::factory()->count(10)->create();
    }
}
```

### 运行 Seeder

```bash
# 运行所有 Seeder
php artisan db:seed

# 运行指定 Seeder
php artisan db:seed --class=UserSeeder

# 迁移并填充
php artisan migrate --seed
```

## ORM 最佳实践

### 1. 使用批量操作

```php
// 不推荐：循环插入
foreach ($users as $userData) {
    User::create($userData);
}

// 推荐：批量插入
User::insert($users);
```

### 2. 避免 N+1 查询

```php
// 不推荐：N+1 查询
$users = User::all();
foreach ($users as $user) {
    echo $user->orders->count();  // 每个用户都执行一次查询
}

// 推荐：预加载关联
$users = User::with('orders')->get();
foreach ($users as $user) {
    echo $user->orders->count();  // 只执行一次查询
}
```

### 3. 使用查询作用域

```php
<?php
class User extends Model
{
    public function scopeActive($query)
    {
        return $query->where('status', 'active');
    }
    
    public function scopeOlderThan($query, int $age)
    {
        return $query->where('age', '>', $age);
    }
}

// 使用
$users = User::active()->olderThan(18)->get();
```

### 4. 使用访问器和修改器

```php
<?php
class User extends Model
{
    // 访问器
    public function getFullNameAttribute(): string
    {
        return "{$this->first_name} {$this->last_name}";
    }
    
    // 修改器
    public function setEmailAttribute(string $value): void
    {
        $this->attributes['email'] = strtolower($value);
    }
}

// 使用
$user = User::find(1);
echo $user->full_name;  // 自动调用访问器

$user->email = 'ALICE@EXAMPLE.COM';
// 自动转换为 'alice@example.com'
```

## 练习

1. 使用 Eloquent 创建一个用户模型，实现 CRUD 操作。

2. 设计一个博客系统，使用 Eloquent 定义文章、分类、标签的关联关系。

3. 编写数据库迁移文件，创建订单系统的表结构。

4. 实现一个 Seeder，填充测试数据（用户、商品、订单等）。

5. 优化一个存在 N+1 查询问题的代码，使用预加载关联。

6. 创建一个查询作用域系统，实现常用的查询条件封装。

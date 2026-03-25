# 7.2.4 行为型模式

## 概述

行为型设计模式关注对象之间的通信和职责分配，旨在解决对象之间的交互问题。这些模式定义了对象之间的通信方式，以及如何分配职责可以使系统更加灵活和高效。

行为型模式主要解决以下问题：
- 对象之间的通信方式
- 职责的分配和传递
- 算法的封装
- 状态的切换

掌握行为型模式对于设计解耦、灵活的软件系统至关重要。本节将详细介绍观察者模式、策略模式、命令模式、责任链模式和模板方法模式的概念、实现方法和适用场景。

**主要内容**：
- 观察者模式
- 策略模式
- 命令模式
- 责任链模式
- 模板方法模式

## 观察者模式

观察者模式定义对象之间的一对多依赖关系，当一个对象状态改变时，所有依赖于它的对象都会收到通知并自动更新。

```php
<?php
declare(strict_types=1);

// 观察者接口
interface Observer
{
    public function update(Subject $subject): void;
}

// 被观察对象
class Subject
{
    private array $observers = [];
    private mixed $state;
    
    public function attach(Observer $observer): void
    {
        $this->observers[] = $observer;
    }
    
    public function detach(Observer $observer): void
    {
        $key = array_search($observer, $this->observers, true);
        if ($key !== false) {
            unset($this->observers[$key]);
        }
    }
    
    public function notify(): void
    {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }
    
    public function setState(mixed $state): void
    {
        $this->state = $state;
        $this->notify();
    }
    
    public function getState(): mixed
    {
        return $this->state;
    }
}

// 具体观察者：日志记录器
class LoggingObserver implements Observer
{
    public function update(Subject $subject): void
    {
        echo "[LOG] State changed to: " . json_encode($subject->getState()) . "\n";
    }
}

// 具体观察者：数据存储
class StorageObserver implements Observer
{
    private array $data = [];
    
    public function update(Subject $subject): void
    {
        $this->data[] = $subject->getState();
        echo "[STORAGE] Saved state. Total records: " . count($this->data) . "\n";
    }
}

// 具体观察者：缓存失效
class CacheObserver implements Observer
{
    public function update(Subject $subject): void
    {
        echo "[CACHE] Clearing cache due to state change\n";
    }
}

// 使用示例
$subject = new Subject();

$subject->attach(new LoggingObserver());
$subject->attach(new StorageObserver());
$subject->attach(new CacheObserver());

echo "=== Setting state to 'processing' ===\n";
$subject->setState('processing');

echo "\n=== Setting state to 'completed' ===\n";
$subject->setState('completed');
```

**输出**：

```
=== Setting state to 'processing' ===
[LOG] State changed to: "processing"
[STORAGE] Saved state. Total records: 1
[CACHE] Clearing cache due to state change

=== Setting state to 'completed' ===
[LOG] State changed to: "completed"
[STORAGE] Saved state. Total records: 2
[CACHE] Clearing cache due to state change
```

### 事件系统实现

```php
<?php
declare(strict_types=1);

// 事件类
class Event
{
    public function __construct(
        public readonly string $name,
        public readonly array $data = [],
        public readonly DateTime $timestamp = new DateTime()
    ) {}
}

// 事件监听器
interface EventListener
{
    public function handle(Event $event): void;
}

// 事件发射器
class EventEmitter
{
    private array $listeners = [];
    
    public function on(string $eventName, EventListener $listener): void
    {
        $this->listeners[$eventName][] = $listener;
    }
    
    public function off(string $eventName, EventListener $listener): void
    {
        if (!isset($this->listeners[$eventName])) {
            return;
        }
        
        $key = array_search($listener, $this->listeners[$eventName], true);
        if ($key !== false) {
            unset($this->listeners[$eventName][$key]);
        }
    }
    
    public function emit(string $eventName, array $data = []): void
    {
        $event = new Event($eventName, $data);
        
        if (isset($this->listeners[$eventName])) {
            foreach ($this->listeners[$eventName] as $listener) {
                $listener->handle($event);
            }
        }
    }
}

// 具体监听器
class UserEventListener implements EventListener
{
    public function handle(Event $event): void
    {
        echo "User event: {$event->name}\n";
        
        match($event->name) {
            'user.created' => $this->handleUserCreated($event),
            'user.updated' => $this->handleUserUpdated($event),
            'user.deleted' => $this->handleUserDeleted($event),
            default => null,
        };
    }
    
    private function handleUserCreated(Event $event): void
    {
        echo "  Sending welcome email to: " . ($event->data['email'] ?? 'unknown') . "\n";
    }
    
    private function handleUserUpdated(Event $event): void
    {
        echo "  Updating user profile: " . ($event->data['id'] ?? 'unknown') . "\n";
    }
    
    private function handleUserDeleted(Event $event): void
    {
        echo "  Cleaning up user data: " . ($event->data['id'] ?? 'unknown') . "\n";
    }
}

// 使用示例
$emitter = new EventEmitter();
$listener = new UserEventListener();

$emitter->on('user.created', $listener);
$emitter->on('user.updated', $listener);
$emitter->on('user.deleted', $listener);

$emitter->emit('user.created', ['email' => 'john@example.com', 'id' => 1]);
$emitter->emit('user.updated', ['id' => 1, 'name' => 'John Doe']);
$emitter->emit('user.deleted', ['id' => 1]);
```

## 策略模式

策略模式定义一系列算法，把它们一个个封装起来，并且使它们可以相互替换。策略模式使得算法可以独立于使用它的客户而变化。

```php
<?php
declare(strict_types=1);

// 策略接口
interface PaymentStrategy
{
    public function pay(float $amount): bool;
    public function getName(): string;
}

// 具体策略：信用卡支付
class CreditCardPayment implements PaymentStrategy
{
    private string $cardNumber;
    private string $cvv;
    
    public function __construct(string $cardNumber, string $cvv)
    {
        $this->cardNumber = $cardNumber;
        $this->cvv = $cvv;
    }
    
    public function pay(float $amount): bool
    {
        echo "Paying \${$amount} with Credit Card: {$this->maskCard()}\n";
        return true;
    }
    
    public function getName(): string
    {
        return 'credit_card';
    }
    
    private function maskCard(): string
    {
        return '****' . substr($this->cardNumber, -4);
    }
}

// 具体策略：PayPal 支付
class PayPalPayment implements PaymentStrategy
{
    private string $email;
    
    public function __construct(string $email)
    {
        $this->email = $email;
    }
    
    public function pay(float $amount): bool
    {
        echo "Paying \${$amount} with PayPal: {$this->email}\n";
        return true;
    }
    
    public function getName(): string
    {
        return 'paypal';
    }
}

// 具体策略：银行转账
class BankTransferPayment implements PaymentStrategy
{
    private string $accountNumber;
    
    public function __construct(string $accountNumber)
    {
        $this->accountNumber = $accountNumber;
    }
    
    public function pay(float $amount): bool
    {
        echo "Paying \${$amount} with Bank Transfer: {$this->maskAccount()}\n";
        return true;
    }
    
    public function getName(): string
    {
        return 'bank_transfer';
    }
    
    private function maskAccount(): string
    {
        return '****' . substr($this->accountNumber, -4);
    }
}

// 上下文：购物车
class ShoppingCart
{
    private array $items = [];
    private ?PaymentStrategy $paymentStrategy = null;
    
    public function addItem(string $item, float $price): void
    {
        $this->items[$item] = $price;
    }
    
    public function removeItem(string $item): void
    {
        unset($this->items[$item]);
    }
    
    public function setPaymentStrategy(PaymentStrategy $strategy): void
    {
        $this->paymentStrategy = $strategy;
    }
    
    public function getTotal(): float
    {
        return array_sum($this->items);
    }
    
    public function checkout(): void
    {
        if ($this->paymentStrategy === null) {
            throw new RuntimeException("Please select a payment method");
        }
        
        $total = $this->getTotal();
        $this->paymentStrategy->pay($total);
    }
}

// 使用示例
$cart = new ShoppingCart();
$cart->addItem('Product 1', 29.99);
$cart->addItem('Product 2', 49.99);
$cart->addItem('Product 3', 19.99);

echo "=== Total: \$" . $cart->getTotal() . " ===\n\n";

// 使用信用卡支付
$cart->setPaymentStrategy(new CreditCardPayment('4111111111111111', '123'));
$cart->checkout();

echo "\n";

// 使用 PayPal 支付
$cart->setPaymentStrategy(new PayPalPayment('user@example.com'));
$cart->checkout();
```

## 命令模式

命令模式将请求封装成对象，从而使你可以用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。

```php
<?php
declare(strict_types=1);

// 命令接口
interface Command
{
    public function execute(): void;
    public function undo(): void;
}

// 接收者：文本编辑器
class TextEditor
{
    private string $content = '';
    
    public function insertText(string $text, int $position): void
    {
        $this->content = substr($this->content, 0, $position) 
            . $text 
            . substr($this->content, $position);
        echo "Inserted '{$text}' at position {$position}\n";
    }
    
    public function deleteText(int $position, int $length): string
    {
        $deleted = substr($this->content, $position, $length);
        $this->content = substr($this->content, 0, $position) 
            . substr($this->content, $position + $length);
        echo "Deleted '{$deleted}' from position {$position}\n";
        return $deleted;
    }
    
    public function getContent(): string
    {
        return $this->content;
    }
}

// 具体命令：插入文本
class InsertTextCommand implements Command
{
    private TextEditor $editor;
    private string $text;
    private int $position;
    
    public function __construct(TextEditor $editor, string $text, int $position)
    {
        $this->editor = $editor;
        $this->text = $text;
        $this->position = $position;
    }
    
    public function execute(): void
    {
        $this->editor->insertText($this->text, $this->position);
    }
    
    public function undo(): void
    {
        $this->editor->deleteText($this->position, strlen($this->text));
    }
}

// 具体命令：删除文本
class DeleteTextCommand implements Command
{
    private TextEditor $editor;
    private int $position;
    private int $length;
    private string $deletedText = '';
    
    public function __construct(TextEditor $editor, int $position, int $length)
    {
        $this->editor = $editor;
        $this->position = $position;
        $this->length = $length;
    }
    
    public function execute(): void
    {
        $this->deletedText = $this->editor->deleteText($this->position, $this->length);
    }
    
    public function undo(): void
    {
        $this->editor->insertText($this->deletedText, $this->position);
    }
}

// 调用者：命令管理器
class CommandManager
{
    private array $history = [];
    private int $current = -1;
    
    public function execute(Command $command): void
    {
        $command->execute();
        
        // 删除当前之后的所有命令（如果存在）
        $this->history = array_slice($this->history, 0, $this->current + 1);
        
        $this->history[] = $command;
        $this->current++;
    }
    
    public function undo(): void
    {
        if ($this->current < 0) {
            echo "Nothing to undo\n";
            return;
        }
        
        $command = $this->history[$this->current];
        $command->undo();
        $this->current--;
    }
    
    public function redo(): void
    {
        if ($this->current >= count($this->history) - 1) {
            echo "Nothing to redo\n";
            return;
        }
        
        $this->current++;
        $command = $this->history[$this->current];
        $command->execute();
    }
}

// 使用示例
$editor = new TextEditor();
$manager = new CommandManager();

// 执行插入命令
$manager->execute(new InsertTextCommand($editor, "Hello ", 0));
$manager->execute(new InsertTextCommand($editor, "World", 6));
echo "Content: '" . $editor->getContent() . "'\n\n";

// 撤销
echo "=== Undo ===\n";
$manager->undo();
echo "Content: '" . $editor->getContent() . "'\n\n";

// 重做
echo "=== Redo ===\n";
$manager->redo();
echo "Content: '" . $editor->getContent() . "'\n\n";

// 再撤销
echo "=== Undo again ===\n";
$manager->undo();
echo "Content: '" . $editor->getContent() . "'\n";
```

## 责任链模式

责任链模式将请求的发送者和接收者解耦，使多个对象都有机会处理请求。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。

```php
<?php
declare(strict_types=1);

// 处理请求接口
interface RequestHandler
{
    public function setNext(?RequestHandler $handler): self;
    public function handle(Request $request): ?string;
}

// 请求类
class Request
{
    public function __construct(
        public readonly string $type,
        public readonly array $data = []
    ) {}
}

// 抽象处理者
abstract class AbstractHandler implements RequestHandler
{
    private ?RequestHandler $nextHandler = null;
    
    public function setNext(?RequestHandler $handler): self
    {
        $this->nextHandler = $handler;
        return $this;
    }
    
    public function handle(Request $request): ?string
    {
        if ($this->nextHandler !== null) {
            return $this->nextHandler->handle($request);
        }
        
        return null;
    }
}

// 具体处理者：认证
class AuthHandler extends AbstractHandler
{
    public function handle(Request $request): ?string
    {
        if (!isset($request->data['token']) || $request->data['token'] !== 'valid-token') {
            return "Error: Authentication failed\n";
        }
        
        echo "AuthHandler: Authentication passed\n";
        return parent::handle($request);
    }
}

// 具体处理者：验证
class ValidationHandler extends AbstractHandler
{
    public function handle(Request $request): ?string
    {
        if ($request->type === 'create' && empty($request->data['name'])) {
            return "Error: Name is required\n";
        }
        
        echo "ValidationHandler: Validation passed\n";
        return parent::handle($request);
    }
}

// 具体处理者：日志
class LoggingHandler extends AbstractHandler
{
    public function handle(Request $request): ?string
    {
        echo "LoggingHandler: Logging request - {$request->type}\n";
        return parent::handle($request);
    }
}

// 具体处理者：业务处理
class BusinessHandler extends AbstractHandler
{
    public function handle(Request $request): ?string
    {
        echo "BusinessHandler: Processing {$request->type} request\n";
        
        return match($request->type) {
            'create' => "Success: Resource created\n",
            'read' => "Success: Resource retrieved\n",
            'update' => "Success: Resource updated\n",
            'delete' => "Success: Resource deleted\n",
            default => "Success: Request processed\n",
        };
    }
}

// 使用示例：构建责任链
$auth = new AuthHandler();
$validation = new ValidationHandler();
$logging = new LoggingHandler();
$business = new BusinessHandler();

$auth->setNext($validation)->setNext($logging)->setNext($business);

// 测试请求
echo "=== Valid Request ===\n";
$request = new Request('create', [
    'token' => 'valid-token',
    'name' => 'Test User'
]);
echo $auth->handle($request);

echo "\n=== Invalid Token ===\n";
$request = new Request('create', [
    'token' => 'invalid-token',
    'name' => 'Test User'
]);
echo $auth->handle($request);

echo "\n=== Missing Name ===\n";
$request = new Request('create', [
    'token' => 'valid-token'
]);
echo $auth->handle($request);
```

## 模板方法模式

模板方法模式定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法的某些特定步骤。

```php
<?php
declare(strict_types=1);

// 抽象类：数据导出
abstract class DataExporter
{
    // 模板方法：定义导出流程
    public function export(array $data): void
    {
        $processed = $this->processData($data);
        $formatted = $this->formatData($processed);
        $this->writeOutput($formatted);
        $this->cleanup();
    }
    
    // 步骤1：处理数据（可由子类重写）
    protected function processData(array $data): array
    {
        echo "Processing data...\n";
        return $data;
    }
    
    // 步骤2：格式化数据（抽象方法，由子类实现）
    abstract protected function formatData(array $data): string;
    
    // 步骤3：写入输出（可由子类重写）
    protected function writeOutput(string $formatted): void
    {
        echo "Writing output: {$formatted}\n";
    }
    
    // 步骤4：清理资源
    protected function cleanup(): void
    {
        echo "Cleaning up resources...\n";
    }
}

// 具体类：JSON 导出
class JsonExporter extends DataExporter
{
    protected function formatData(array $data): string
    {
        echo "Formatting as JSON...\n";
        return json_encode($data, JSON_PRETTY_PRINT);
    }
}

// 具体类：CSV 导出
class CsvExporter extends DataExporter
{
    protected function formatData(array $data): string
    {
        echo "Formatting as CSV...\n";
        
        if (empty($data)) {
            return '';
        }
        
        $output = '';
        
        // 标题行
        $output .= implode(',', array_keys($data[0])) . "\n";
        
        // 数据行
        foreach ($data as $row) {
            $output .= implode(',', array_values($row)) . "\n";
        }
        
        return $output;
    }
}

// 具体类：XML 导出
class XmlExporter extends DataExporter
{
    protected function formatData(array $data): string
    {
        echo "Formatting as XML...\n";
        
        $xml = new SimpleXMLElement('<root/>');
        
        foreach ($data as $item) {
            $xmlItem = $xml->addChild('item');
            foreach ($item as $key => $value) {
                $xmlItem->addChild($key, htmlspecialchars($value));
            }
        }
        
        return $xml->asXML();
    }
}

// 使用示例
$sampleData = [
    ['id' => 1, 'name' => 'John', 'email' => 'john@example.com'],
    ['id' => 2, 'name' => 'Jane', 'email' => 'jane@example.com'],
];

echo "=== JSON Export ===\n";
$exporter = new JsonExporter();
$exporter->export($sampleData);

echo "\n=== CSV Export ===\n";
$exporter = new CsvExporter();
$exporter->export($sampleData);

echo "\n=== XML Export ===\n";
$exporter = new XmlExporter();
$exporter->export($sampleData);
```

**输出**：

```
=== JSON Export ===
Processing data...
Formatting as JSON...
Writing output: [
    {
        "id": 1,
        "name": "John",
        "email": "john@example.com"
    },
    {
        "id": 2,
        "name": "Jane",
        "email": "jane@example.com"
    }
]
Cleaning up resources...

=== CSV Export ===
Processing data...
Formatting as CSV...
Writing output: id,name,email
1,John,john@example.com
2,Jane,jane@example.com

Cleaning up resources...

=== XML Export ===
Processing data...
Formatting as XML...
Writing output: <?xml version="1.0" encoding="UTF-8"?>
<root><item><id>1</id><name>John</name><email>john@example.com</email></item><item><id>2</id><name>Jane</name><email>jane@example.com</email></item></root>
Cleaning up resources...
```

## 使用场景

### 观察者模式适用场景

- 一个对象的改变需要同时改变其他对象
- 需要建立对象之间的动态联系
- 需要事件系统

### 策略模式适用场景

- 需要在运行时选择算法
- 多个类只区别在行为方式
- 需要隔离业务逻辑

### 命令模式适用场景

- 需要撤销/重做功能
- 需要记录操作日志
- 需要将操作排队执行

### 责任链模式适用场景

- 多个对象可以处理同一请求
- 动态指定处理对象
- 需要解耦发送者和接收者

### 模板方法模式适用场景

- 算法骨架固定，某些步骤可变
- 需要控制子类扩展点
- 代码复用

## 常见问题

### 观察者和发布订阅的区别？

观察者模式中观察者直接订阅被观察者，发布订阅模式中有事件中心作为中介。发布订阅更松耦合。

### 策略模式和状态模式的区别？

策略模式封装不同的算法，客户端主动选择。状态模式封装不同的状态行为，状态自动转换。

### 命令模式和责任链模式的区别？

命令模式将请求封装为对象，可以撤销/重做。责任链模式将请求沿着链传递，直到被处理。

### 行为型模式如何选择？

根据需求：
- 需要通知机制 → 观察者模式
- 需要选择算法 → 策略模式
- 需要撤销功能 → 命令模式
- 需要处理链 → 责任链模式
- 需要算法骨架 → 模板方法模式

## 练习任务

1. **实现事件系统**：使用观察者模式实现一个完整的事件系统

2. **设计支付策略**：使用策略模式设计一个支持多种支付方式的支付系统

3. **实现撤销功能**：使用命令模式实现文本编辑器的撤销/重做功能

4. **构建处理链**：使用责任链模式实现一个 HTTP 请求处理管道

5. **设计导出功能**：使用模板方法模式设计支持多种格式的数据导出功能

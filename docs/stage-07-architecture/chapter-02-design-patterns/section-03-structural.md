# 7.2.3 结构型模式

## 概述

结构型设计模式关注类和对象的组合方式，旨在通过建立对象之间的结构关系来解决复杂性问题。这些模式帮助我们确定如何将类和对象组合成更大的结构，同时保持代码的灵活性和可维护性。

结构型模式主要解决以下问题：
- 如何让不兼容的接口一起工作
- 如何动态地为对象添加功能
- 如何控制对对象的访问
- 如何简化复杂子系统

掌握结构型模式对于设计良好的软件架构至关重要。本节将详细介绍适配器模式、装饰器模式、代理模式、外观模式和组合模式的概念、实现方法和适用场景。

**主要内容**：
- 适配器模式
- 装饰器模式
- 代理模式
- 外观模式
- 组合模式

## 适配器模式

适配器模式将一个类的接口转换成客户端所期望的另一个接口。适配器可以让原本不兼容的类可以合作。

```php
<?php
declare(strict_types=1);

// 目标接口：客户端期望的接口
interface PaymentGateway
{
    public function processPayment(float $amount): bool;
    public function refundPayment(string $transactionId): bool;
}

// 第三方支付 SDK（需要适配的类）
class ThirdPartyPaymentSDK
{
    public function makePayment(float $amountInCents): array
    {
        // 第三方 SDK 使用分作为单位
        return [
            'success' => true,
            'transaction_id' => 'txn_' . uniqid(),
            'amount' => $amountInCents
        ];
    }
    
    public function reversePayment(string $reference): bool
    {
        // 退款
        return true;
    }
    
    public function getPaymentStatus(string $transactionId): string
    {
        return 'completed';
    }
}

// 适配器：将第三方 SDK 适配到目标接口
class PaymentGatewayAdapter implements PaymentGateway
{
    private ThirdPartySDK $sdk;
    
    public function __construct(ThirdPartyPaymentSDK $sdk)
    {
        $this->sdk = $sdk;
    }
    
    public function processPayment(float $amount): bool
    {
        // 将金额从元转换为分
        $amountInCents = (int)($amount * 100);
        
        $result = $this->sdk->makePayment($amountInCents);
        
        return $result['success'];
    }
    
    public function refundPayment(string $transactionId): bool
    {
        return $this->sdk->reversePayment($transactionId);
    }
}

// 使用示例
$sdk = new ThirdPartyPaymentSDK();
$gateway = new PaymentGatewayAdapter($sdk);

if ($gateway->processPayment(99.99)) {
    echo "Payment processed successfully\n";
}

$gateway->refundPayment('txn_abc123');
```

### 对象适配器 vs 类适配器

PHP 中更常用的是对象适配器，因为它不要求继承被适配的类。

```php
<?php
declare(strict_types=1);

// 目标接口
interface Target
{
    public function request(): string;
}

// 被适配的类
class Adaptee
{
    public function specificRequest(): string
    {
        return "Specific request";
    }
}

// 对象适配器（推荐）
class ObjectAdapter implements Target
{
    private Adaptee $adaptee;
    
    public function __construct(Adaptee $adaptee)
    {
        $this->adaptee = $adaptee;
    }
    
    public function request(): string
    {
        return "Adapter: " . $this->adaptee->specificRequest();
    }
}

// 使用
$adapter = new ObjectAdapter(new Adaptee());
echo $adapter->request();
```

## 装饰器模式

装饰器模式允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

```php
<?php
declare(strict_types=1);

// 基础组件接口
interface DataSource
{
    public function read(): string;
    public function write(string $data): void;
}

// 具体组件：文件数据源
class FileDataSource implements DataSource
{
    private string $filename;
    private string $data = '';
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
        
        if (file_exists($filename)) {
            $this->data = file_get_contents($filename);
        }
    }
    
    public function read(): string
    {
        return $this->data;
    }
    
    public function write(string $data): void
    {
        $this->data = $data;
        file_put_contents($this->filename, $data);
    }
}

// 装饰器基类
abstract class DataSourceDecorator implements DataSource
{
    protected DataSource $wrapped;
    
    public function __construct(DataSource $source)
    {
        $this->wrapped = $source;
    }
}

// 具体装饰器：加密装饰器
class EncryptionDecorator extends DataSourceDecorator
{
    public function read(): string
    {
        $data = $this->wrapped->read();
        return $this->decrypt($data);
    }
    
    public function write(string $data): void
    {
        $encrypted = $this->encrypt($data);
        $this->wrapped->write($encrypted);
    }
    
    private function encrypt(string $data): string
    {
        // 简单的加密示例
        return base64_encode($data);
    }
    
    private function decrypt(string $data): string
    {
        return base64_decode($data);
    }
}

// 具体装饰器：压缩装饰器
class CompressionDecorator extends DataSourceDecorator
{
    public function read(): string
    {
        $data = $this->wrapped->read();
        return gzuncompress($data);
    }
    
    public function write(string $data): void
    {
        $compressed = gzcompress($data);
        $this->wrapped->write($compressed);
    }
}

// 具体装饰器：日志装饰器
class LoggingDecorator extends DataSourceDecorator
{
    public function read(): string
    {
        $data = $this->wrapped->read();
        echo "[READ] Read " . strlen($data) . " bytes\n";
        return $data;
    }
    
    public function write(string $data): void
    {
        echo "[WRITE] Writing " . strlen($data) . " bytes\n";
        $this->wrapped->write($data);
    }
}

// 使用示例：组合多个装饰器
$file = new FileDataSource('data.txt');

// 添加加密
$encrypted = new EncryptionDecorator($file);

// 添加压缩和日志
$withAll = new LoggingDecorator(
    new CompressionDecorator(
        new EncryptionDecorator(
            new FileDataSource('compressed_data.txt')
        )
    )
);

$withAll->write("Hello World");
echo $withAll->read();
```

## 代理模式

代理模式为另一个对象提供一个替身或占位符以控制对这个对象的访问。

```php
<?php
declare(strict_types=1);

// 主题接口
interface Image
{
    public function display(): void;
}

// 真实对象：大型图片
class RealImage implements Image
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
        $this->loadFromDisk();
    }
    
    private function loadFromDisk(): void
    {
        // 模拟从磁盘加载大型图片的耗时操作
        echo "Loading image: {$this->filename}\n";
    }
    
    public function display(): void
    {
        echo "Displaying image: {$this->filename}\n";
    }
}

// 代理对象：虚拟代理（延迟加载）
class ImageProxy implements Image
{
    private ?RealImage $realImage = null;
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function display(): void
    {
        // 延迟加载：只在真正需要时才加载真实对象
        if ($this->realImage === null) {
            $this->realImage = new RealImage($this->filename);
        }
        
        $this->realImage->display();
    }
}

// 保护代理：访问控制
class SecureImageProxy implements Image
{
    private ?RealImage $realImage = null;
    private string $filename;
    private string $userRole;
    
    public function __construct(string $filename, string $userRole)
    {
        $this->filename = $filename;
        $this->userRole = $userRole;
    }
    
    public function display(): void
    {
        // 权限检查
        if ($this->userRole !== 'admin' && str_contains($this->filename, 'admin')) {
            echo "Access denied: {$this->filename}\n";
            return;
        }
        
        if ($this->realImage === null) {
            $this->realImage = new RealImage($this->filename);
        }
        
        $this->realImage->display();
    }
}

// 使用示例：虚拟代理
echo "=== Virtual Proxy ===\n";
$image = new ImageProxy('photo.jpg');

// 图片不会在这里加载
echo "Proxy created\n";

// 只在调用 display 时才加载
$image->display();
// 再次调用不会重新加载
$image->display();

echo "\n=== Secure Proxy ===\n";
$adminImage = new SecureImageProxy('admin_panel.jpg', 'admin');
$adminImage->display();

$userImage = new SecureImageProxy('admin_panel.jpg', 'user');
$userImage->display();
```

**输出**：

```
=== Virtual Proxy ===
Proxy created
Loading image: photo.jpg
Displaying image: photo.jpg
Displaying image: photo.jpg

=== Secure Proxy ===
Loading image: admin_panel.jpg
Displaying image: admin_panel.jpg
Access denied: admin_panel.jpg
```

## 外观模式

外观模式为复杂的子系统提供一个统一的接口，使得子系统更容易使用。

```php
<?php
declare(strict_types=1);

// 子系统组件 1：视频文件
class VideoFile
{
    public string $filename;
    public string $codecType;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
        $this->codecType = $this->detectCodec();
    }
    
    private function detectCodec(): string
    {
        return str_ends_with($this->filename, '.mp4') ? 'mp4' : 'unknown';
    }
}

// 子系统组件 2：编解码器
class CodecFactory
{
    public static function extract(VideoFile $file): string
    {
        echo "Extracting {$file->codecType} codec\n";
        return "raw_data";
    }
}

// 子系统组件 3：音频混合
class AudioMixer
{
    public function fix(array $data): string
    {
        echo "Fixing audio\n";
        return "fixed_data";
    }
}

// 子系统组件 4：视频转换
class VideoConverter
{
    public function convert(VideoFile $file, string $format): string
    {
        echo "Converting to {$format}\n";
        return "converted_data";
    }
}

// 子系统组件 5：视频写入
class VideoWriter
{
    public function write(string $data, string $filename): void
    {
        echo "Writing to {$filename}\n";
    }
}

// 外观类：统一的视频转换接口
class VideoConversionFacade
{
    public function convertVideo(string $filename, string $format): string
    {
        $videoFile = new VideoFile($filename);
        
        $sourceCodec = CodecFactory::extract($videoFile);
        
        $audioMixer = new AudioMixer();
        $audioData = $audioMixer->fix($sourceCodec);
        
        $converter = new VideoConverter();
        $result = $converter->convert($videoFile, $format);
        
        $outputFilename = str_replace(
            pathinfo($filename, PATHINFO_EXTENSION),
            $format,
            $filename
        );
        
        $writer = new VideoWriter();
        $writer->write($result, $outputFilename);
        
        return $outputFilename;
    }
}

// 使用示例：客户端代码变得非常简单
$facade = new VideoConversionFacade();
$outputFile = $facade->convertVideo('movie.ogg', 'mp4');
echo "Converted to: {$outputFile}\n";
```

## 组合模式

组合模式将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

```php
<?php
declare(strict_types=1);

// 组件接口
interface FileSystemItem
{
    public function getName(): string;
    public function getSize(): int;
    public function render(int $depth = 0): void;
}

// 叶子节点：文件
class File implements FileSystemItem
{
    private string $name;
    private int $size;
    
    public function __construct(string $name, int $size)
    {
        $this->name = $name;
        $this->size = $size;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getSize(): int
    {
        return $this->size;
    }
    
    public function render(int $depth = 0): void
    {
        echo str_repeat('  ', $depth) . "- {$this->name} ({$this->size} bytes)\n";
    }
}

// 组合节点：目录
class Directory implements FileSystemItem
{
    private string $name;
    private array $children = [];
    
    public function __construct(string $name)
    {
        $this->name = $name;
    }
    
    public function add(FileSystemItem $item): void
    {
        $this->children[] = $item;
    }
    
    public function remove(FileSystemItem $item): void
    {
        $key = array_search($item, $this->children, true);
        if ($key !== false) {
            unset($this->children[$key]);
        }
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getSize(): int
    {
        return array_reduce(
            $this->children,
            fn(int $sum, FileSystemItem $item) => $sum + $item->getSize(),
            0
        );
    }
    
    public function render(int $depth = 0): void
    {
        echo str_repeat('  ', $depth) . "+ {$this->name}/ (" . $this->getSize() . " bytes)\n";
        
        foreach ($this->children as $child) {
            $child->render($depth + 1);
        }
    }
    
    public function getChild(int $index): ?FileSystemItem
    {
        return $this->children[$index] ?? null;
    }
}

// 使用示例：构建文件系统
$root = new Directory('root');

$docs = new Directory('documents');
$docs->add(new File('resume.pdf', 102400));
$docs->add(new File('report.docx', 51200));

$photos = new Directory('photos');
$photos->add(new File('vacation.jpg', 2048000));
$photos->add(new File('family.png', 1536000));

$downloads = new Directory('downloads');
$downloads->add(new File('installer.exe', 52428800));

$root->add($docs);
$root->add($photos);
$root->add($downloads);

// 渲染整个文件系统
echo "=== File System ===\n";
$root->render();

// 计算总大小
echo "\nTotal size: " . $root->getSize() . " bytes\n";

// 访问特定目录
echo "\n=== Documents ===\n";
$docs->render();
```

**输出**：

```
=== File System ===
+ root/ (56624400 bytes)
  + documents/ (153600 bytes)
    - resume.pdf (102400 bytes)
    - report.docx (51200 bytes)
  + photos/ (3584000 bytes)
    - vacation.jpg (2048000 bytes)
    - family.png (1536000 bytes)
  + downloads/ (52428800 bytes)
    - installer.exe (52428800 bytes)

Total size: 56624400 bytes

=== Documents ===
+ documents/ (153600 bytes)
  - resume.pdf (102400 bytes)
  - report.docx (51200 bytes)
```

## 使用场景

### 适配器模式适用场景

- 已有类的接口与新系统不兼容
- 需要集成第三方库
- 需要统一不同数据源的接口

### 装饰器模式适用场景

- 需要动态添加或撤销对象的功能
- 需要在不影响其他对象的情况下添加职责
- 需要处理可以动态添加的多个功能组合

### 代理模式适用场景

- 需要延迟加载（虚拟代理）
- 需要访问控制（保护代理）
- 需要远程访问（远程代理）
- 需要记录日志（日志代理）

### 外观模式适用场景

- 需要为复杂子系统提供简单接口
- 客户端与子系统之间存在很大的依赖
- 需要分层系统结构

### 组合模式适用场景

- 需要表示对象的部分-整体层次结构
- 需要忽略单个对象和组合对象的差异
- 需要统一处理树形结构

## 常见问题

### 适配器和装饰器的区别？

适配器改变接口使其兼容，装饰器增加功能而不改变接口。适配器解决"不兼容"问题，装饰器解决"增强功能"问题。

### 代理模式和装饰器模式的区别？

代理模式控制对对象的访问，装饰器模式为对象动态添加功能。两者都是包装器，但目的不同。

### 外观模式和适配器模式的区别？

外观模式简化复杂子系统的接口，适配器将不兼容的接口转换为兼容的接口。两者都包装现有类，但目的和范围不同。

### 何时使用组合模式？

当需要处理树形结构，或者需要统一处理单个对象和对象组合时，应该使用组合模式。

## 最佳实践

1. **区分对象适配器和类适配器**：在 PHP 中优先使用对象适配器

2. **装饰器的顺序很重要**：多个装饰器叠加时，执行顺序会影响结果

3. **代理的透明性**：代理应该对客户端透明，客户端无需知道使用的是代理

4. **外观提供便捷方法**：外观可以提供便捷方法，但不能阻止客户端访问底层组件

5. **组合的递归思维**：组合模式天然支持递归操作

## 练习任务

1. **实现数据适配器**：为一个旧的数据格式实现适配器，使其与新系统兼容

2. **构建装饰器链**：使用装饰器模式实现一个带缓存、日志、验证的数据服务

3. **设计图片加载代理**：实现一个虚拟代理来延迟加载图片

4. **简化复杂系统**：使用外观模式简化一个复杂的文件处理系统

5. **构建组织架构**：使用组合模式实现一个公司组织架构管理系统

# 4.2.13 图片处理与文件操作库

## 概述

图片处理是文件系统操作中的常见需求。本节介绍 PHP 中图片处理的基础知识，包括 GD 库的使用、图片操作函数，以及常用的第三方文件操作库，帮助零基础学员了解图片处理的基本方法。

**章节类型**：工具性章节

**主要内容**：
- 图片处理概述
- GD 库基础
- 图片读取和创建
- 图片操作（缩放、裁剪、旋转）
- 图片格式转换
- 第三方库介绍（Intervention Image、Imagine）
- 完整示例

## 核心内容

### 图片处理概述

- 常见图片格式
- 图片处理需求
- PHP 图片处理扩展

### GD 库基础

- GD 库的启用
- 图片资源（resource）
- 基本图片操作

### 图片读取和创建

- imagecreatefromjpeg()、imagecreatefrompng()
- imagecreate()、imagecreatetruecolor()
- 图片资源管理

### 图片操作

- 图片缩放（imagescale()、imagecopyresampled()）
- 图片裁剪（imagecrop()）
- 图片旋转（imagerotate()）
- 图片水印

### 第三方库

- Intervention Image
- Imagine
- 库的选择和使用

## 基本用法

### GD 库示例

```php
<?php
declare(strict_types=1);

// 创建图片
$image = imagecreatetruecolor(800, 600);
$white = imagecolorallocate($image, 255, 255, 255);
imagefill($image, 0, 0, $white);

// 保存图片
imagejpeg($image, 'output.jpg');
imagedestroy($image);
```

## 使用场景

- 图片上传处理
- 缩略图生成
- 图片水印添加
- 图片格式转换

## 注意事项

- GD 库的安装和启用
- 内存使用问题
- 图片质量设置
- 资源释放

## 常见问题

- 如何检查 GD 库是否可用？
- 如何生成缩略图？
- 如何处理大图片的内存问题？
- 第三方库如何选择？

## 最佳实践

- 使用 GD 库进行基础图片处理
- 大图片使用流式处理
- 及时释放图片资源
- 考虑使用第三方库简化操作

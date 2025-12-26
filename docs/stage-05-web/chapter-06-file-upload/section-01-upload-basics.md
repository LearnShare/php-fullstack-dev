# 5.6.1 文件上传基础

## 概述

文件上传是 Web 应用的常见功能。本节介绍文件上传的基础处理方法，包括 $_FILES 处理、move_uploaded_file() 函数、上传配置等，帮助零基础学员掌握文件上传技术。

**章节类型**：实践性章节

**主要内容**：
- 文件上传概述
- 表单配置（enctype="multipart/form-data"）
- $_FILES 处理
- move_uploaded_file() 函数
- 上传错误处理
- 上传配置（php.ini）
- 完整示例

## 核心内容

### 文件上传概述

- 文件上传流程
- 表单配置要求
- 上传限制

### $_FILES 处理

- $_FILES 数组结构
- 文件信息获取
- 临时文件位置
- 错误码处理

### move_uploaded_file()

- 语法和参数
- 文件移动
- 安全性检查
- 返回值

### 上传配置

- upload_max_filesize
- post_max_size
- max_file_uploads
- file_uploads

## 基本用法

### 文件上传示例

```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    if ($file['error'] === UPLOAD_ERR_OK) {
        $tmpName = $file['tmp_name'];
        $destination = '/path/to/upload/' . $file['name'];
        move_uploaded_file($tmpName, $destination);
    }
}
```

## 使用场景

- 用户头像上传
- 文档上传
- 图片上传
- 媒体文件上传

## 注意事项

- 表单 enctype 配置
- 上传大小限制
- 临时文件处理
- 错误处理

## 常见问题

- 如何配置文件上传表单？
- move_uploaded_file() 的作用？
- 上传文件大小如何限制？
- 如何处理上传错误？

## 最佳实践

- 检查上传错误
- 验证文件信息
- 使用 move_uploaded_file()
- 配置合理的上传限制

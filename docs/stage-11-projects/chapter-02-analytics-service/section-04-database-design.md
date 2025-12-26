# 11.2.4 数据库设计

## 概述

数据库设计是项目开发的基础。本节介绍网站访问统计服务的数据库设计，包括统计数据表设计、用户/网站管理表设计、时序数据存储设计、索引设计等，帮助零基础学员理解如何设计数据库结构。

**章节类型**：实践性章节

**主要内容**：
- 数据库设计概述
- MySQL 数据库设计
- InfluxDB 时序数据设计
- 索引设计
- 数据分区设计
- 完整示例

## 核心内容

### 数据库设计概述

- 数据库设计的作用
- 数据库设计原则
- 混合数据库方案：MySQL（主数据）+ InfluxDB（时序数据）

### MySQL 数据库设计

#### 用户表 (users)

- 字段：id、username、email、password、status、created_at、updated_at
- 索引：email、status

#### 网站表 (websites)

- 字段：id、user_id、name、domain、api_key、status、created_at、updated_at
- 外键：user_id → users(id)
- 索引：user_id、api_key、status

#### 统计数据汇总表 (statistics_summary)

- 字段：id、website_id、date、pv、uv、ip_count、avg_duration、bounce_rate、created_at、updated_at
- 外键：website_id → websites(id)
- 唯一索引：website_id + date
- 索引：date、website_id

#### 来源统计表 (referrer_statistics)

- 字段：id、website_id、date、referrer_type、referrer_domain、pv、uv、created_at、updated_at
- 索引：website_id + date、referrer_type

#### 设备统计表 (device_statistics)

- 字段：id、website_id、date、device_type、browser、os、pv、uv、created_at、updated_at
- 索引：website_id + date、device_type

#### 事件统计表 (event_statistics)

- 字段：id、website_id、event_name、date、count、created_at、updated_at
- 唯一索引：website_id + event_name + date
- 索引：website_id、event_name、date

### InfluxDB 时序数据设计

#### 数据模型

- Measurement: `page_views`
- Tags: website_id、page_path、referrer_type、device_type、browser、os、country、city
- Fields: pv、duration、is_bounce
- Timestamp: 访问时间

#### 数据保留策略

- 原始数据：保留 90 天
- 汇总数据：保留 2 年
- 统计数据：永久保留（存储在 MySQL）

### 索引设计

#### MySQL 索引策略

- 主键索引：所有表都有自增主键
- 外键索引：所有外键字段都建立索引
- 查询索引：根据查询需求建立组合索引
- 唯一索引：保证数据唯一性的字段建立唯一索引

#### InfluxDB 索引策略

- Tag 索引：所有 Tag 字段自动建立索引
- 时间索引：Timestamp 字段自动建立索引
- 查询优化：使用 Tag 进行查询，避免 Field 查询

### 数据分区设计

#### MySQL 分区

- 按日期进行分区（统计数据汇总表）
- 分区策略：RANGE 分区

#### InfluxDB 保留策略

- raw_data：90 天
- summary_data：730 天

## 基本用法

### 数据库设计流程

- 需求分析
- 数据库设计
- 优化设计

## 完整示例

### 数据库设计文档

- MySQL 数据库设计（表结构定义）
- InfluxDB 数据库设计（数据模型定义、保留策略定义）
- 索引设计说明
- 分区设计说明

## 注意事项

- 数据一致性
- 性能优化
- 数据安全
- 扩展性

## 常见问题

**Q: 为什么使用两个数据库？**

A: MySQL 用于存储关系型数据（用户、网站配置等），InfluxDB 用于存储时序数据（访问统计），各取所长。

**Q: 如何保证数据一致性？**

A: 使用消息队列异步处理，保证最终一致性。关键数据使用事务保证强一致性。

**Q: 如何处理数据量大的问题？**

A: 使用数据分区、数据归档、数据汇总等方式处理大数据量问题。

## 最佳实践

- 表设计要规范
- 索引要合理
- 分区要合理
- 备份要定期

## 对比分析

### 数据库对比

- MySQL vs InfluxDB vs PostgreSQL

## 实践任务

1. 数据库设计练习：设计一个统计服务的数据库结构，设计索引和分区策略，编写数据库设计文档
2. 数据库实现：创建数据库表结构，创建索引和分区，编写数据迁移脚本
3. 数据库优化：分析查询性能，优化索引设计，优化分区策略

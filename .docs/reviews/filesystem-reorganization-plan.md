# 文件系统章节重组方案

## 一、重组原则

### 1.1 阶段定位

- **stage-02-language**：PHP 基础语法与语言精通
  - 专注于语言本身的特性（语法、类型、函数、控制结构、基础 API）
  - **不再包含**：文件系统操作（已全部移动到 stage-05-data）

- **stage-05-data**：数据持久化与日志管理
  - 数据持久化策略、文件系统高级特性、流式操作
  - 应该包含：完整的文件系统操作（基础操作、高级特性、流式操作、系统管理）

- **stage-06-security**：安全、性能与可观测性
  - 安全防护、性能优化
  - 应该包含：路径安全、文件安全等安全相关内容

### 1.2 重组目标

1. **分离关注点**：将语言基础特性与系统编程、安全内容分离
2. **合理归类**：将相关内容归类到合适的阶段
3. **优化结构**：按学习顺序和逻辑关系重新组织 5.8 章节
4. **保持连贯性**：确保学习路径的连贯性
5. **更新引用**：更新所有相关引用和交叉链接

## 二、文件移动计划

### 2.1 移动到 stage-05-data/chapter-08-filesystem

**全部内容移动（11 个小节），重新组织为 9 个小节：**

#### 第一部分：基础文件操作（5.8.1 - 5.8.3）

1. **5.8.1 文件读取**
   - 源：`section-01-file-reading.md`（2.13.1）
   - 目标：`section-01-file-reading.md`
   - 说明：基础操作，放在最前面

2. **5.8.2 文件写入**
   - 源：`section-02-file-writing.md`（2.13.2）
   - 目标：`section-02-file-writing.md`
   - 说明：基础操作，紧随读取

3. **5.8.3 文件操作**
   - 源：`section-03-file-operations.md`（2.13.3）
   - 目标：`section-03-file-operations.md`
   - 说明：基础操作（copy、rename、unlink）

#### 第二部分：高级文件操作（5.8.4 - 5.8.6）

4. **5.8.4 文件指针操作**
   - 源：`section-04-file-pointer.md`（2.13.4）
   - 目标：`section-04-file-pointer.md`
   - 说明：需要理解文件句柄，在基础操作之后

5. **5.8.5 二进制文件处理**
   - 源：`section-05-binary-files.md`（2.13.5）
   - 目标：`section-05-binary-files.md`
   - 说明：需要理解文件模式，与指针操作相关

6. **5.8.6 文件锁和临时文件**
   - 源：`section-06-file-locks-temp.md`（2.13.6）
   - 目标：`section-06-file-locks-temp.md`
   - 说明：并发控制，在掌握基础操作后学习

#### 第三部分：目录和文件信息（5.8.7 - 5.8.8）

7. **5.8.7 目录操作**
   - 源：`section-08-directory-operations.md`（2.13.8）
   - 目标：`section-07-directory-operations.md`
   - 说明：目录相关操作，独立但相关

8. **5.8.8 文件信息**
   - 源：`section-07-file-info.md`（2.13.7）
   - 目标：`section-08-file-info.md`
   - 说明：文件元数据，与目录操作相关

#### 第四部分：流式操作（5.8.9 - 5.8.10）

9. **5.8.9 流式操作与流包装器**
   - 源：`section-01-streaming.md`（现有）+ `section-09-stream-wrappers.md`（2.13.9）
   - 目标：`section-09-streaming.md`
   - 说明：合并流式操作和流包装器，高级特性

10. **5.8.10 大文件处理**
    - 源：`section-02-large-files.md`（现有）
    - 目标：`section-10-large-files.md`
    - 说明：基于流式操作的大文件处理

#### 第五部分：系统管理（5.8.11）

11. **5.8.11 文件权限详解**
    - 源：`section-10-file-permissions.md`（2.13.10）
    - 目标：`section-11-file-permissions.md`
    - 说明：系统级特性，放在最后

**章节结构变化：**

- 原：2 个小节（5.8.1 流式操作、5.8.2 大文件处理）
- 新：11 个小节，按逻辑分组：
  - 基础操作（5.8.1 - 5.8.3）
  - 高级操作（5.8.4 - 5.8.6）
  - 目录和信息（5.8.7 - 5.8.8）
  - 流式操作（5.8.9 - 5.8.10）
  - 系统管理（5.8.11）

**注意**：`stage-02-language/chapter-13-filesystem` 目录将被完全移除

### 2.2 移动到 stage-06-security

**移动文件：路径安全**

- **源文件**：`stage-02-language/chapter-13-filesystem/section-11-path-security.md`
- **目标位置**：`stage-06-security/chapter-01-owasp/section-05-path-security.md`
- **操作**：移动文件，更新章节编号
- **新章节编号**：6.1.5

**章节结构变化：**

- 原：4 个小节（6.1.1 - 6.1.4）
- 新：5 个小节（6.1.1 - 6.1.5）

## 三、5.8 章节重组后的结构

### 3.1 章节标题

**从**：`# 5.8 文件系统流式操作`

**改为**：`# 5.8 文件系统操作`

### 3.2 章节目标

**更新为**：

- 掌握文件读写、操作等基础文件操作
- 理解文件指针、二进制处理、文件锁等高级操作
- 熟悉目录操作和文件信息获取
- 掌握流式操作和流包装器的使用
- 了解大文件处理的方法
- 理解文件权限的管理

### 3.3 学习顺序建议

1. **基础操作**（5.8.1 - 5.8.3）：必须先掌握
   - 文件读取 → 文件写入 → 文件操作

2. **高级操作**（5.8.4 - 5.8.6）：在基础之上
   - 文件指针操作 → 二进制文件处理 → 文件锁和临时文件

3. **目录和信息**（5.8.7 - 5.8.8）：独立但相关
   - 目录操作 → 文件信息

4. **流式操作**（5.8.9 - 5.8.10）：高级特性
   - 流式操作与流包装器 → 大文件处理

5. **系统管理**（5.8.11）：最后学习
   - 文件权限详解

## 四、需要更新的文件列表

### 4.1 stage-02-language 相关文件

#### 4.1.1 `stage-02-language/chapter-13-filesystem/`（整个目录将被删除）

**操作**：删除整个目录，因为所有内容都已移动到 stage-05-data

#### 4.1.2 `stage-02-language/readme.md`

**需要更新：**

1. 删除章节 13 的整个条目（第 174-185 行）
2. 后续章节编号保持不变（2.14 - 2.19）
3. 更新"学习时间估算"部分，移除文件系统相关内容
4. 更新"完成阶段二后"部分，移除文件系统相关内容

#### 4.1.3 所有引用 2.13 的文件

**需要更新：**

- 搜索所有文件中的 `2.13`、`chapter-13-filesystem` 引用
- 更新为指向 `5.8`、`stage-05-data/chapter-08-filesystem`

### 4.2 stage-05-data 相关文件

#### 4.2.1 `stage-05-data/chapter-08-filesystem/readme.md`

**需要更新：**

1. 章节标题：从"文件系统流式操作"改为"文件系统操作"
2. 章节数量：从 2 个小节改为 11 个小节
3. 更新章节列表（按新的结构）
4. 更新"目标"部分
5. 更新"学习建议"部分（按逻辑分组）
6. 更新"完成本章后"部分
7. 更新"相关章节"部分

#### 4.2.2 `stage-05-data/chapter-08-filesystem/section-09-streaming.md`（合并）

**需要更新：**

1. 合并 `section-01-streaming.md`（现有）和 `section-09-stream-wrappers.md`（2.13.9）
2. 章节编号：从 `# 5.8.1 流式操作` 改为 `# 5.8.9 流式操作与流包装器`
3. 整合流包装器的详细内容
4. 更新所有内部引用

#### 4.2.3 `stage-05-data/chapter-08-filesystem/section-10-large-files.md`（重命名）

**需要更新：**

1. 从 `section-02-large-files.md` 重命名为 `section-10-large-files.md`
2. 章节编号：从 `# 5.8.2 大文件处理` 改为 `# 5.8.10 大文件处理`
3. 更新所有内部引用

#### 4.2.4 所有移动的文件（section-01 到 section-08, section-11）

**需要更新：**

1. 更新章节编号（2.13.x → 5.8.x）
2. 更新所有内部引用（2.13.x → 5.8.x）
3. 更新交叉引用

#### 4.2.5 `stage-05-data/readme.md`

**需要更新：**

1. 第 143-145 行：更新章节 8 的小节列表（11 个小节）
2. 第 159 行：更新"完成阶段五后"部分

### 4.3 stage-06-security 相关文件

#### 4.3.1 `stage-06-security/chapter-01-owasp/readme.md`

**需要更新：**

1. 章节数量：从 4 个小节改为 5 个小节
2. 更新章节列表，添加 6.1.5 路径安全
3. 更新"目标"、"学习建议"、"完成本章后"、"相关章节"部分

#### 4.3.2 `stage-06-security/chapter-01-owasp/section-05-path-security.md`（新文件）

**需要更新：**

1. 从 `stage-02-language/chapter-13-filesystem/section-11-path-security.md` 移动
2. 更新章节编号：从 `# 2.13.11 路径安全` 改为 `# 6.1.5 路径安全`
3. 更新所有内部引用和交叉引用

#### 4.3.3 `stage-06-security/readme.md`

**需要更新：**

1. 第 113-117 行：更新章节 1 的小节列表，添加 6.1.5 路径安全

### 4.4 其他可能受影响的文件

#### 4.4.1 `php-fullstack-dev/docs/learning-guide.md`

**需要检查：**

- 检查是否有引用 2.13 的地方
- 如果有，需要更新引用

#### 4.4.2 `php-fullstack-dev/docs/stage-02-language/quick-reference.md`

**需要检查：**

- 检查是否有引用文件系统相关章节的地方
- 如果有，需要更新引用

## 五、操作步骤

### 步骤 1：备份和准备

1. 确认所有文件存在
2. 创建备份（可选，Git 已管理）

### 步骤 2：移动和重组文件

1. **移动基础文件操作（3 个文件）**：
   - `section-01-file-reading.md` → `stage-05-data/chapter-08-filesystem/section-01-file-reading.md`
   - `section-02-file-writing.md` → `stage-05-data/chapter-08-filesystem/section-02-file-writing.md`
   - `section-03-file-operations.md` → `stage-05-data/chapter-08-filesystem/section-03-file-operations.md`

2. **移动高级文件操作（3 个文件）**：
   - `section-04-file-pointer.md` → `stage-05-data/chapter-08-filesystem/section-04-file-pointer.md`
   - `section-05-binary-files.md` → `stage-05-data/chapter-08-filesystem/section-05-binary-files.md`
   - `section-06-file-locks-temp.md` → `stage-05-data/chapter-08-filesystem/section-06-file-locks-temp.md`

3. **移动目录和文件信息（2 个文件）**：
   - `section-08-directory-operations.md` → `stage-05-data/chapter-08-filesystem/section-07-directory-operations.md`
   - `section-07-file-info.md` → `stage-05-data/chapter-08-filesystem/section-08-file-info.md`

4. **合并流式操作和流包装器**：
   - 读取 `section-01-streaming.md`（现有）和 `section-09-stream-wrappers.md`（2.13.9）
   - 合并到 `stage-05-data/chapter-08-filesystem/section-09-streaming.md`
   - 删除原 `section-01-streaming.md`

5. **重命名大文件处理**：
   - `section-02-large-files.md` → `section-10-large-files.md`

6. **移动文件权限**：
   - `section-10-file-permissions.md` → `stage-05-data/chapter-08-filesystem/section-11-file-permissions.md`

7. **移动路径安全**：
   - `section-11-path-security.md` → `stage-06-security/chapter-01-owasp/section-05-path-security.md`

8. **删除原目录**：
   - 删除 `stage-02-language/chapter-13-filesystem/` 整个目录

### 步骤 3：更新所有文件的章节编号

1. 更新所有移动文件的章节编号（2.13.x → 5.8.x 或 6.1.5）
2. 更新所有移动文件的内部引用
3. 更新合并后的流式操作章节编号（5.8.1 → 5.8.9）
4. 更新大文件处理章节编号（5.8.2 → 5.8.10）

### 步骤 4：更新 readme.md 文件

1. 更新 `stage-02-language/readme.md`（删除章节 13 的引用）
2. 更新 `stage-05-data/chapter-08-filesystem/readme.md`（完全重写，按新结构）
3. 更新 `stage-05-data/readme.md`（更新章节 8 的小节列表）
4. 更新 `stage-06-security/chapter-01-owasp/readme.md`（添加 6.1.5）
5. 更新 `stage-06-security/readme.md`（更新章节 1 的小节列表）

### 步骤 5：检查其他引用

1. 搜索所有文件中的 `2.13`、`chapter-13-filesystem` 引用
2. 搜索所有文件中的 `2.13.1` 到 `2.13.11` 的引用
3. 更新找到的所有引用：
   - `2.13.1` → `5.8.1`
   - `2.13.2` → `5.8.2`
   - `2.13.3` → `5.8.3`
   - `2.13.4` → `5.8.4`
   - `2.13.5` → `5.8.5`
   - `2.13.6` → `5.8.6`
   - `2.13.7` → `5.8.8`（注意：文件信息从 7 变为 8）
   - `2.13.8` → `5.8.7`（注意：目录操作从 8 变为 7）
   - `2.13.9` → `5.8.9`（流包装器合并到流式操作）
   - `2.13.10` → `5.8.11`
   - `2.13.11` → `6.1.5`

### 步骤 6：验证

1. 检查所有文件路径是否正确
2. 检查所有章节编号是否一致
3. 检查所有交叉引用是否更新
4. 检查学习路径是否连贯

## 六、预期结果

### 6.1 stage-02-language

**变化：**

- 删除 `chapter-13-filesystem` 整个章节
- 章节总数从 19 个减少到 18 个
- 后续章节编号保持不变（2.14 - 2.19）

**定位**：专注于 PHP 语言基础特性，不包含文件系统操作

### 6.2 stage-05-data/chapter-08-filesystem

**最终结构（11 个小节，按逻辑分组）：**

**第一部分：基础文件操作**
- 5.8.1 文件读取
- 5.8.2 文件写入
- 5.8.3 文件操作

**第二部分：高级文件操作**
- 5.8.4 文件指针操作
- 5.8.5 二进制文件处理
- 5.8.6 文件锁和临时文件

**第三部分：目录和文件信息**
- 5.8.7 目录操作
- 5.8.8 文件信息

**第四部分：流式操作**
- 5.8.9 流式操作与流包装器（合并）
- 5.8.10 大文件处理

**第五部分：系统管理**
- 5.8.11 文件权限详解

**定位**：数据持久化相关的完整文件系统操作，从基础到高级，从单文件到目录，从本地到网络流，从操作到系统管理

### 6.3 stage-06-security/chapter-01-owasp

**最终结构（5 个小节）：**

1. 6.1.1 OWASP Top 10 概述
2. 6.1.2 注入攻击防护
3. 6.1.3 认证与会话管理
4. 6.1.4 其他安全风险
5. 6.1.5 路径安全（新增）

**定位**：安全防护相关内容

## 七、注意事项

1. **内容完整性**：确保移动后的内容完整，不丢失任何信息
2. **引用更新**：确保所有引用都正确更新，特别注意章节编号的变化
3. **学习路径**：确保学习路径仍然连贯，按逻辑顺序组织
4. **章节编号**：确保所有章节编号一致，特别注意文件信息和目录操作的顺序变化
5. **交叉引用**：确保交叉引用正确指向新位置
6. **合并内容**：流式操作和流包装器的合并需要仔细整合，避免重复

## 八、风险评估

### 8.1 低风险

- 文件移动操作本身是安全的
- Git 可以追踪所有变更
- 可以随时回滚

### 8.2 需要注意

- 引用更新可能遗漏某些文件
- 需要全面搜索所有可能的引用位置
- 需要验证所有链接的有效性
- 章节编号变化较多，需要仔细检查

## 九、执行检查清单

- [ ] 备份所有文件（Git 已管理）
- [ ] 移动基础文件操作（3 个文件）到 stage-05-data
- [ ] 移动高级文件操作（3 个文件）到 stage-05-data
- [ ] 移动目录和文件信息（2 个文件）到 stage-05-data
- [ ] 合并流式操作和流包装器到 section-09-streaming.md
- [ ] 重命名大文件处理为 section-10-large-files.md
- [ ] 移动文件权限到 stage-05-data（section-11-file-permissions.md）
- [ ] 移动路径安全到 stage-06-security（section-05-path-security.md）
- [ ] 更新所有移动文件的章节编号（2.13.x → 5.8.x 或 6.1.5）
- [ ] 更新所有移动文件的内部引用
- [ ] 更新合并后的流式操作章节编号（5.8.1 → 5.8.9）
- [ ] 更新大文件处理章节编号（5.8.2 → 5.8.10）
- [ ] 删除 stage-02-language/chapter-13-filesystem 目录
- [ ] 更新 stage-02-language/readme.md（删除章节 13 的引用）
- [ ] 更新 stage-05-data/chapter-08-filesystem/readme.md（完全重写，按新结构）
- [ ] 更新 stage-05-data/readme.md（更新章节 8 的小节列表）
- [ ] 更新 stage-06-security/chapter-01-owasp/readme.md（添加 6.1.5）
- [ ] 更新 stage-06-security/chapter-01-owasp/section-05-path-security.md（更新章节编号）
- [ ] 更新 stage-06-security/readme.md（更新章节 1 的小节列表）
- [ ] 搜索并更新所有 `2.13`、`chapter-13-filesystem` 引用
- [ ] 搜索并更新所有 `2.13.1` 到 `2.13.11` 的引用
- [ ] 特别注意更新 `2.13.7` → `5.8.8` 和 `2.13.8` → `5.8.7` 的引用
- [ ] 验证所有文件路径
- [ ] 验证所有章节编号
- [ ] 验证所有交叉引用
- [ ] 验证学习路径连贯性

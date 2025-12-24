# Composer 内容重组方案

## 问题分析

当前 Composer 相关内容分散在两个位置：
1. **阶段一**：`chapter-04-toolchain/section-01-composer.md` - 包含 Composer 基础内容
2. **阶段二**：`chapter-12-modularity` - 包含三个 Composer 相关小节：
   - `section-06-composer-basics.md` - Composer 基础（与阶段一内容有重复）
   - `section-07-composer-autoload.md` - Composer 自动加载（详细内容）
   - `section-08-composer-packages.md` - 编写和发布 Composer 包（详细内容）

**问题**：
- Composer 作为第三方工具，不应该放在"文件引入与模块化"章节中
- 内容重复，不利于学习
- 应该与其他工具放在一起

## 重组方案

### 方案：将 Composer 内容整合到阶段一工具链章节

**目标位置**：`stage-01-foundation/chapter-04-toolchain`

**调整计划**：

1. **扩展 `section-01-composer.md`**：
   - 保留现有的 Composer 基础内容
   - 补充 `section-06-composer-basics.md` 中的详细内容（如果阶段一没有）
   - 整合 `section-07-composer-autoload.md` 的详细自动加载内容
   - 整合 `section-08-composer-packages.md` 的包编写和发布内容

2. **或者拆分为多个小节**（推荐）：
   - `section-01-composer-basics.md` - Composer 基础（安装、基本使用、常用命令）
   - `section-02-composer-autoload.md` - Composer 自动加载（PSR-4、类映射、文件自动加载）
   - `section-03-composer-packages.md` - 编写和发布 Composer 包
   - 其他小节重新编号：
     - `section-02-ide-extensions.md` → `section-04-ide-extensions.md`
     - `section-03-git-tasks.md` → `section-05-git-tasks.md`
     - `section-04-docker-compose.md` → `section-06-docker-compose.md`
     - `section-05-automation.md` → `section-07-automation.md`

3. **从阶段二删除 Composer 相关内容**：
   - 删除 `chapter-12-modularity/section-06-composer-basics.md`
   - 删除 `chapter-12-modularity/section-07-composer-autoload.md`
   - 删除 `chapter-12-modularity/section-08-composer-packages.md`
   - 更新 `chapter-12-modularity/readme.md`，删除这三个小节的引用
   - 更新章节编号：`section-09-namespaces.md` → `section-06-namespaces.md`

4. **更新所有引用**：
   - 更新 `stage-02-language/readme.md`
   - 更新 `README.md`
   - 更新其他章节中的交叉引用

## 推荐方案：拆分为多个小节

**优势**：
- 内容组织更清晰
- 每个小节专注一个主题
- 便于学习和查阅
- 符合工具链章节的逻辑

**新的章节结构**：

```
stage-01-foundation/chapter-04-toolchain/
├── readme.md
├── section-01-composer-basics.md          # Composer 基础
├── section-02-composer-autoload.md        # Composer 自动加载
├── section-03-composer-packages.md        # 编写和发布 Composer 包
├── section-04-ide-extensions.md           # IDE 与扩展配置（原 section-02）
├── section-05-git-tasks.md                # Git 与任务管理（原 section-03）
├── section-06-docker-compose.md           # 容器与环境管理（原 section-04）
└── section-07-automation.md               # 自动化脚本（原 section-05）
```

**阶段二调整后**：

```
stage-02-language/chapter-12-modularity/
├── readme.md
├── section-01-module-basics.md            # 模块基础
├── section-02-include-require.md          # include 与 require
├── section-03-using-imported.md           # 使用导入的内容
├── section-04-conflicts-circular.md       # 避免冲突和循环导入
├── section-05-paths-returns.md           # 路径处理和返回值
└── section-06-namespaces.md              # 命名空间与模块组织（原 section-09）
```

## 执行步骤

1. **创建新的 Composer 小节**（在阶段一）
   - 创建 `section-01-composer-basics.md`（整合现有内容和阶段二的详细内容）
   - 创建 `section-02-composer-autoload.md`（从阶段二移动并更新章节编号）
   - 创建 `section-03-composer-packages.md`（从阶段二移动并更新章节编号）

2. **重命名现有小节**（在阶段一）
   - `section-02-ide-extensions.md` → `section-04-ide-extensions.md`
   - `section-03-git-tasks.md` → `section-05-git-tasks.md`
   - `section-04-docker-compose.md` → `section-06-docker-compose.md`
   - `section-05-automation.md` → `section-07-automation.md`

3. **删除阶段二的 Composer 小节**
   - 删除 `section-06-composer-basics.md`
   - 删除 `section-07-composer-autoload.md`
   - 删除 `section-08-composer-packages.md`

4. **重命名阶段二的小节**
   - `section-09-namespaces.md` → `section-06-namespaces.md`

5. **更新所有 readme.md 文件**
   - 更新 `stage-01-foundation/chapter-04-toolchain/readme.md`
   - 更新 `stage-02-language/chapter-12-modularity/readme.md`
   - 更新 `stage-01-foundation/readme.md`
   - 更新 `stage-02-language/readme.md`
   - 更新 `README.md`

6. **更新所有交叉引用**
   - 搜索所有引用 `2.12.6`、`2.12.7`、`2.12.8` 的位置
   - 更新为新的章节编号（`1.3.1`、`1.3.2`、`1.3.3`）
   - 搜索所有引用 `2.12.9` 的位置，更新为 `2.12.6`

7. **更新章节编号**
   - 所有文件中的章节编号从 `2.12.x` 更新为 `1.3.x`（Composer 相关）
   - 命名空间章节从 `2.12.9` 更新为 `2.12.6`

## 注意事项

1. **内容整合**：需要仔细整合阶段一和阶段二的 Composer 内容，避免重复，保留最详细和最新的内容
2. **章节编号**：所有文件中的章节编号都需要更新
3. **交叉引用**：需要全面搜索和更新所有交叉引用
4. **学习路径**：调整后，Composer 的学习提前到阶段一，更符合学习逻辑

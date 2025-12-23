# 文档结构规范

## 目录结构

- 按阶段、章、节拆分独立目录
- 格式：`docs/stage-xx/chapter-yy/readme.md`
- 根目录 `readme.md` 保留完整阶段与章节目录，并指向对应文档

## 文件命名

- **所有文件**（包括 readme.md）必须使用小写字母、数字和连字符（-）
- **目录名**同样遵循小写字母、数字和连字符（-）的规范
- **扩展名**统一为 `.md`
- **阶段、章节和小节文档**统一命名为 `readme.md`（小写），不使用 `README.md`

**示例**：
- ✅ 正确：`stage-01-foundation`、`chapter-01-history`、`section-01-history.md`、`readme.md`
- ❌ 错误：`Stage-01-Foundation`、`README.md`、`section_01_history.md`

## 文件完整性检查

- 创建新章节前，必须检查根目录 `readme.md` 中的目录结构
- 确保所有列出的章节文件都已创建
- 如发现缺失文件，必须补全

---

**最后更新**：2025-12-22

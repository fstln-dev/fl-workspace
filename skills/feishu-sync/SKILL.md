---
name: feishu-sync
description: |
  飞书文档同步。在以下场景触发：
  1. 用户说 "同步到飞书"、"推送到飞书"
  2. 用户说 "从飞书拉取"、"同步飞书变更"
  3. 用户说 "检查飞书同步状态"
  4. 用户提到文档需要同步到飞书知识库

  支持双向同步：本地 → 飞书，飞书 → 本地
---
# 飞书文档同步

管理本地文档与飞书知识库的双向同步。

## 前置条件

### MCP 选择

飞书同步涉及两种 MCP：

| MCP | 用途 | 适用操作 |
| --- | --- | --- |
| `feishu-official` | 文档和 Wiki 操作 | create-doc, fetch-doc, search-doc, list-docs, get-comments, add-comments |
| `feishu` | 多维表格及其他操作 | bitable, calendar, task, im 等 |

**文档/Wiki 同步使用 \****`feishu-official`***\*，多维表格操作使用 \****`feishu`**\*\*。**

### 具体前置条件

1. **文档/Wiki 操作** (`feishu-official`):
  - `.mcp.json` 中已配置 `feishu-official`
  - 已获取 UAT Token（通过 Device Flow 授权）

2. **多维表格操作** (`feishu`):
  - `.mcp.json` 中已配置 `feishu`
  - 已通过 `/mcp` → feishu → Authentication 完成授权

3. **项目配置**:
  - Wiki Space ID 已配置（在 `.claude/context/feishu-config.md` 中）

## 同步模式

### 模式 1: 推送到飞书 (Push)

当用户说"同步到飞书"时：

1. **确定同步范围**
```
   📤 将同步以下文档到飞书:

   docs/product/prd-order.md → 飞书 Wiki/产品文档/订单模块PRD
   docs/tech/tech-order.md → 飞书 Wiki/技术文档/订单模块技术方案

   是否继续？[确认] [选择] [取消]
```

2. **检查同步状态**

   读取 `.claude/context/feishu-map.md` 获取文档到飞书的映射：
  - 新文档：创建新飞书文档
  - 已同步文档：更新现有飞书文档
  - 冲突文档：提示用户选择版本

3. **执行同步**

   使用 `feishu_create_doc` 或 `feishu_update_doc` 工具

4. **更新映射**

   同步完成后更新 `feishu-map.md`：
```markdown
   | 本地路径 | 飞书 Token | 最后同步 | 状态 |
   |----------|-----------|----------|------|
   | docs/product/prd-order.md | doxcnXXXX | 2026-03-09 14:00 | synced |
```

### 模式 2: 从飞书拉取 (Pull)

当用户说"从飞书拉取"时：

1. **获取飞书变更**

   使用 `feishu_list_wiki_nodes` 获取 Wiki 节点列表
   对比 `feishu-map.md` 中的 `最后同步` 时间

2. **报告变更**
```
   📥 检测到飞书变更:

   更新的文档:
   - 订单模块PRD (更新于 2026-03-09 13:00)

   新增的文档:
   - 支付模块PRD (创建于 2026-03-09 12:00)

   是否拉取？[全部] [选择] [取消]
```

3. **执行拉取**

   使用 `feishu_fetch_doc` 获取文档内容
   保存到对应的本地路径

4. **更新映射**

   更新 `feishu-map.md` 中的同步时间

### 模式 3: 检查同步状态

当用户说"检查同步状态"时：

```
📊 飞书同步状态:

已同步 (5):
  ✅ docs/product/prd-order.md → 订单模块PRD
  ✅ docs/tech/tech-order.md → 订单模块技术方案

待同步 (2):
  ⏳ docs/product/prd-payment.md (本地更新于 2026-03-09 14:00)
  ⏳ docs/test/test-order.md (新建，未同步)

冲突 (1):
  ⚠️ docs/product/prd-user.md (飞书有更新)

是否同步待同步文档？[是] [否]
```

## 文档路径映射规则

默认映射规则（可在 `feishu-config.md` 中自定义）：

| 本地目录 | 飞书 Wiki 节点 |
| --- | --- |
| `docs/product/` | 产品文档/ |
| `docs/tech/` | 技术文档/ |
| `docs/test/` | 测试文档/ |
| `docs/meetings/` | 会议纪要/ |

## 冲突处理

当检测到冲突时（本地和飞书都有更新）：

```
⚠️ 冲突检测: docs/product/prd-user.md

本地版本: 2026-03-09 14:00 (用户A)
飞书版本: 2026-03-09 13:30 (用户B)

请选择:
[1] 保留本地版本 (覆盖飞书)
[2] 使用飞书版本 (覆盖本地)
[3] 手动合并 (显示 diff)
[4] 跳过
```

选择手动合并时，显示两版本的差异：
```
--- 本地版本
+++ 飞书版本
@@ -10,7 +10,7 @@
-## 功能需求
+## 业务需求
```

## feishu-map.md 格式

```markdown
# 飞书文档映射

> 自动维护，请勿手动编辑

| 本地路径 | 飞书 Token | 飞书标题 | 最后同步 | 状态 | MD5 |
|----------|-----------|----------|----------|------|-----|
| docs/product/prd-order.md | doxcnXXXX | 订单模块PRD | 2026-03-09T14:00:00Z | synced | abc123 |
```

## 错误处理

| 错误 | 处理 |
| --- | --- |
| 飞书 Token 无效 / 过期 | 提示运行 "飞书授权" 或 "检查飞书状态"，引导使用 feishu-connect |
| feishu-official UAT 无效 | 提示运行 "飞书授权" 重新获取 UAT Token |
| feishu Token 无效 | 提示运行 `/mcp` → feishu → Authentication 重新授权 |
| Wiki 权限不足 | 提示检查应用权限配置 |
| 网络超时 | 重试 3 次，失败后提示 |
| 文档过大 | 提示拆分文档 |

### 授权问题处理

当遇到授权相关错误时：

```
⚠️ 飞书授权问题

检测到授权失效或未授权。

请运行以下命令解决:
- "飞书授权" - 引导完成授权流程
- "检查飞书状态" - 查看当前授权状态

或者直接说 "找 feishu-connect" 获取帮助
```

## 批量同步

支持批量操作：

```
用户: "同步所有待同步文档"
用户: "同步 docs/product/ 下的所有文档"
用户: "拉取飞书所有更新"
```

## 自动同步（可选）

在 `feishu-config.md` 中配置：

```yaml
auto_sync:
  on_commit: true      # 提交时自动同步 status=approved 的文档
  on_session_start: true # 会话启动时检查飞书变更
```

### Git Commit Hook

初始化工作空间时会自动配置 `post-commit` hook：

```bash
# .git/hooks/post-commit
# 检测本次提交中 status=approved 的 .md 文件
# 如果有已批准的文档，提示用户可以同步到飞书
```

**工作流程：**
1. 开发者提交包含 `status: approved` 的 .md 文件
2. Git hook 检测到已批准的文档
3. 提示用户可以同步到飞书
4. 用户在 Claude Code 中说 "同步到飞书" 执行同步

**手动配置 Git Hook：**

如果需要手动配置，复制 `scripts/post-commit` 到 `.git/hooks/`：

```bash
cp scripts/post-commit .git/hooks/
chmod +x .git/hooks/post-commit
```

---
name: doc-workflow
description: |
  文档工作流。在以下场景触发：
  1. 用户说 "创建 PRD"、"写技术方案"、"写测试方案"
  2. 用户说 "提交评审"、"发布文档"
  3. 用户说 "检查文档"、"更新文档状态"
  4. 用户提到文档类型关键词（PRD、技术方案、测试用例等）

  支持文档创建、评审、状态流转、模版应用。
---

# 文档工作流

管理项目文档的完整生命周期：创建 → 评审 → 发布。

## 文档类型映射

| 关键词 | 文档类型 | 模版路径 | 目标目录 |
|--------|----------|----------|----------|
| PRD、需求文档 | prd | templates/prd.md | docs/product/ |
| 技术方案、技术设计 | tech-design | templates/tech-design.md | docs/tech/ |
| 测试方案、测试计划 | test-plan | templates/test-plan.md | docs/test/ |
| API 文档 | api-doc | templates/api-doc.md | docs/tech/ |
| 会议纪要 | meeting-notes | templates/meeting-notes.md | docs/meetings/ |

## 状态流转

```
draft → review → approved → released
  ↓        ↓         ↓          ↓
 创建    提交评审   评审通过    正式发布
```

### 状态说明

| 状态 | 含义 | 触发动作 |
|------|------|----------|
| `draft` | 草稿，正在编写 | 无 |
| `review` | 等待评审 | 提醒配置评审人 |
| `approved` | 评审通过 | 建议下一步（根据 SOP） |
| `released` | 已发布 | 可选：同步到飞书 |

## 创建文档流程

当用户说"创建 PRD"或类似指令时：

1. **确认文档类型**
   ```
   📝 将创建文档类型: PRD (产品需求文档)
   目标路径: docs/product/

   请确认或提供更多信息：
   - 文档标题: [等待输入]
   - 所属模块: [等待输入]
   ```

2. **收集必要信息**
   - 文档标题
   - 所属模块（如果有）
   - 关联的需求背景（如果是技术方案）

3. **应用模版**
   - 读取 `templates/{type}.md`
   - 填充 Front Matter:
     ```yaml
     ---
     title: {标题}
     type: {类型}
     status: draft
     author: {当前用户}
     version: "1.0"
     created: {日期}
     updated: {日期}
     module: {模块}
     ---
     ```

4. **生成草稿**
   - 根据上下文生成初步内容
   - 保存到目标路径
   - 更新 `.claude/context/doc-index.md`

5. **确认完成**
   ```
   ✅ 文档已创建: docs/product/prd-{module}.md

   下一步建议:
   - 完善内容后修改 status 为 review
   - 使用 /skill feishu-sync 同步到飞书
   ```

## 状态变更流程

当用户修改文档的 `status` 字段时：

### draft → review
```
📋 文档已提交评审

建议操作:
1. 通知评审人（如果在 team.md 中配置）
2. 在飞书 Bitable 创建评审任务
3. 同步到飞书知识库便于评审

是否执行？[全部] [选择] [稍后]
```

### review → approved
```
✅ 文档评审通过！

根据项目流程，建议下一步:
- PRD 通过 → 创建技术方案
- 技术方案通过 → 拆解开发任务
- 测试方案通过 → 开始测试执行

是否继续？[是] [稍后]
```

### approved → released
```
🎉 文档已正式发布！

可选操作:
- 同步到飞书知识库（公开可见）
- 通知相关团队成员
- 归档评审记录
```

## 模版版本检查

当用户说"检查模版更新"时：

1. 扫描 `templates/` 目录获取模版版本
2. 对比 `docs/` 中各文档的 `template_version`
3. 输出报告：
   ```
   📋 模版更新检查结果:

   templates/prd.md: v1.2.0 (最新)
   - docs/product/prd-order.md 使用 v1.1.0 ← 可更新

   templates/tech-design.md: v2.0.0 (最新)
   - docs/tech/tech-order.md 使用 v2.0.0 ✓

   是否更新过时文档？[选择] [全部] [取消]
   ```

## Front Matter 规范

所有文档必须包含以下 Front Matter：

```yaml
---
title: 文档标题           # 必填
type: prd|tech-design|... # 必填
status: draft|review|...  # 必填
author: 作者              # 必填
version: "1.0"            # 必填
created: 2026-03-09       # 必填
updated: 2026-03-09       # 必填
module: 所属模块          # 可选
template: prd             # 可选
template_version: "1.0"   # 可选
reviewers: [reviewer1]    # 可选
related_docs: [doc1]      # 可选
---
```

## 错误处理

| 错误场景 | 处理方式 |
|----------|----------|
| 模版不存在 | 提示可用的模版类型 |
| 文件已存在 | 询问是否覆盖或创建新版本 |
| Front Matter 缺失 | 自动补充默认值 |
| 状态流转非法 | 提示合法的流转路径 |

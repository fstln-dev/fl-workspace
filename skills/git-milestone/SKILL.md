---
name: git-milestone
description: |
  里程碑提交管理。在以下场景触发：
  1. 完成一个重要功能模块
  2. 文档从 draft 变为 approved
  3. 项目阶段转换（需求 → 设计 → 开发）
  4. 用户说 "创建里程碑"、"提交里程碑"
  5. 用户说 "总结这个阶段"

  帮助用户在关键节点创建有意义的 Git 提交，记录项目演进历史。
---
# Git 里程碑提交

在项目的关键节点创建有意义的提交，记录演进历史。

## 里程碑触发条件

AI 应主动识别以下情况并推荐创建里程碑：

### 自动触发场景

| 场景 | 触发条件 | 里程碑类型 |
| --- | --- | --- |
| 文档批准 | `status: draft` → `status: approved` | `docs:approve` |
| 功能完成 | 新功能代码完成并通过测试 | `feat:complete` |
| 阶段转换 | 项目阶段变更 | `phase:transition` |
| 定期总结 | 同一分支超过 10 个小提交 | `chore:checkpoint` |

### 识别逻辑

```
检测到里程碑条件：

📄 文档状态变更
- docs/prd.md: draft → approved

建议创建里程碑提交：
"需求文档已批准，是否创建里程碑提交？"

[创建里程碑] [稍后]
```

## 里程碑提交流程

### 步骤 1: 确认里程碑内容

```
🏷️ 里程碑提交

触发原因: PRD 文档已批准
影响范围:
  - docs/prd/order-module.md (approved)
  - docs/tech/order-design.md (draft)

建议提交信息:
---
milestone(order): 需求分析完成

- 订单模块 PRD 已批准
- 技术方案初稿完成
- 下一阶段: 技术设计

Co-authored-by: Claude <claude@anthropic.com>
---

[确认提交] [修改信息] [跳过]
```

### 步骤 2: 执行提交

用户确认后，执行：

```bash
# 查看当前状态
git status

# 暂存相关文件
git add docs/prd/order-module.md docs/tech/order-design.md

# 创建里程碑提交
git commit -m "milestone(order): 需求分析完成

- 订单模块 PRD 已批准
- 技术方案初稿完成
- 下一阶段: 技术设计

Co-authored-by: Claude <claude@anthropic.com>"
```

### 步骤 3: 确认结果

```
✅ 里程碑提交完成

commit abc1234
milestone(order): 需求分析完成

提交包含:
  - 2 个文件变更
  - +150 行新增

后续建议:
- 推送到远程: git push
- 创建标签: git tag -a v0.1.0 -m "需求分析完成"
```

## 里程碑提交模板

### 文档批准里程碑

```
milestone({scope}): {document}已批准

- {document} 评审通过
- 评审人: {reviewers}
- 下一阶段: {next_phase}

Co-authored-by: Claude <claude@anthropic.com>
```

### 功能完成里程碑

```
milestone({scope}): {feature}功能完成

- 功能描述: {description}
- 测试覆盖: {coverage}%
- 相关文档: {docs}

Co-authored-by: Claude <claude@anthropic.com>
```

### 阶段转换里程碑

```
milestone({project}): {from_phase} → {to_phase}

{from_phase} 阶段完成:
- {completed_items}

{to_phase} 阶段目标:
- {next_goals}

Co-authored-by: Claude <claude@anthropic.com>
```

### 检查点里程碑

```
chore: 阶段性检查点

本次检查点包含:
- {commit_count} 个提交
- {file_count} 个文件变更
- 主要变更: {summary}

Co-authored-by: Claude <claude@anthropic.com>
```

## 里程碑命名规范

### 提交类型

| 类型 | 说明 | 示例 |
| --- | --- | --- |
| `milestone` | 正式里程碑 | `milestone(auth): 登录模块完成` |
| `checkpoint` | 阶段性检查点 | `checkpoint: 第10次提交` |
| `phase` | 阶段转换 | `phase: 需求 → 设计` |

### Scope 命名

使用模块或功能名称：
- `auth` - 认证模块
- `order` - 订单模块
- `docs` - 文档相关
- `infra` - 基础设施

## 主动推荐时机

AI 应在以下时机主动询问用户：

### 文档状态变更时

```
检测到 docs/prd.md 状态从 draft 变为 approved

📌 这是一个重要里程碑！

建议创建里程碑提交，记录这个节点。
是否现在创建？

[创建里程碑] [稍后提醒]
```

### 功能开发完成时

```
检测到 feature/order 模块代码已完成

📊 代码统计:
  - 新增文件: 5
  - 新增代码: 320 行
  - 测试覆盖: 85%

这是一个功能完成的好时机，是否创建里程碑？

[创建里程碑] [继续开发]
```

### 提交数量累积时

```
检测到当前分支已有 12 个小提交

📦 提交历史较长，建议整理：

选项:
[1] 合并为里程碑提交（推荐）
[2] 保持现状
[3] 查看提交列表

合并后可以保持历史清晰，便于回溯。
```

## 与 Git Hook 配合

当安装了 `post-commit` hook 时：

1. 每次提交后，hook 检查是否有 `status=approved` 的文档
2. 如果有，提示用户考虑创建里程碑
3. 用户可以选择立即创建或稍后处理

## 最佳实践

### 什么时候创建里程碑

✅ **应该创建：**
- 重要文档批准后
- 功能模块完成后
- 项目阶段转换时
- 准备发布前

❌ **不必创建：**
- 小的 bug 修复
- 文档 typo 修正
- 临时的实验性代码

### 里程碑 vs 普通提交

| 特征 | 普通提交 | 里程碑提交 |
| --- | --- | --- |
| 粒度 | 单一变更 | 多个相关变更 |
| 信息 | 简洁描述 | 详细上下文 |
| 标签 | 无 | 可选添加 tag |
| 回溯 | 难以定位 | 易于定位 |

## 错误处理

| 错误 | 处理 |
| --- | --- |
| 没有待提交内容 | 提示用户无变更需要提交 |
| 合并冲突 | 引导用户解决冲突后重试 |
| 未初始化 git | 提示用户先执行 git init |

## 相关 Skills

- `workspace-init` - 初始化时创建第一个里程碑
- `doc-workflow` - 文档状态变更触发里程碑
- `project-status` - 查看里程碑历史

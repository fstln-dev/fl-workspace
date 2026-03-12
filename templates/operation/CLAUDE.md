# {项目名称} - AI 协同工作空间

> 📌 基于 claude-code-workspace v1.0.0

## 项目概况

- **项目类型**: operation（运营项目）
- **当前阶段**: {数据分析 | 策略制定 | SOP执行 | 复盘总结}
- **团队**: 见 .claude/context/team.md

## 文档规范

所有 `docs/` 下的 Markdown 文件必须包含 YAML Front Matter：

```yaml
---
title: 文档标题
type: analysis | strategy | sop | dashboard | review
status: draft | review | approved | archived
author: 作者名
version: "1.0"
created: 2026-03-09
updated: 2026-03-09
period: 2026-W01（可选，周期）
module: 所属模块（可选）
---
```

### 状态流转

```
draft → review → approved → archived
```

### 文件命名

- 使用 kebab-case: `analysis-weekly-sales.md`
- 包含周期前缀: `strategy-q1-growth.md`

## AI 行为指令

### 会话启动检测

每个新会话开始时，执行以下检测：

1. **文档索引同步**
   - 扫描 `docs/` 目录
   - 对比 `.claude/context/doc-index.md`
   - 报告新增/删除的文档

2. **飞书变更检测**（如果配置了飞书）
   - 读取 `.claude/context/feishu-map.md` 获取最后同步时间
   - 使用飞书 MCP 工具检查飞书端更新
   - 报告：有变更 → 提示用户是否拉取

3. **周期性任务提醒**
   - 检查本周期的 SOP 执行情况
   - 报告待执行/已完成的任务

4. **输出检测报告**

### 文档创建

1. 创建文档时自动使用 `templates/` 中的模版
2. 根据用户描述推断文档类型
3. 自动填充 Front Matter 元数据

### 状态变更

1. 修改文档 `status` 时提醒用户触发的后续流程
2. analysis approved → 建议制定策略
3. strategy approved → 建议创建执行 SOP

### 上下文维护

1. 定期更新 `.claude/context/` 下的索引文件
2. 新建文档后更新 `doc-index.md`
3. 同步到飞书后更新 `feishu-map.md`

## 飞书集成

> 通过远程飞书 MCP 服务器实现飞书能力集成
> 服务地址: `https://feishu-mcp.fastgrowth.ai/mcp`

### MCP 配置

在项目 `.mcp.json` 中配置：

```json
{
  "mcpServers": {
    "feishu": {
      "type": "http",
      "url": "https://feishu-mcp.fastgrowth.ai/mcp"
    }
  }
}
```

无需本地安装任何依赖，无需配置 `FEISHU_APP_ID` / `FEISHU_APP_SECRET`。

### 首次授权流程

飞书 MCP 加载后需要完成 OAuth 授权：

```
1. 在 Claude Code 中输入 /mcp
2. 选择 feishu MCP 服务器
3. 按 Enter 进入后，选择 Authentication
4. 浏览器自动打开授权页面
5. 在网页上点击授权，完成后即可使用

授权完成后，所有飞书工具自动可用。
```

### 配置知识库

授权完成后，配置项目关联的知识库：

```
用户: 帮我配置飞书知识库
→ AI 调用相关工具列出可访问的知识库
→ 用户选择知识库
→ AI 自动写入配置
```

### 项目级配置

配置存储在 `.claude/context/feishu-config.md`：

```yaml
wiki:
  space_id: "wikcnXXXX"
  space_name: "运营知识库"

bitable:
  app_token: "appcnXXXX"
  task_table_id: "tblXXXX"
```

### 可用工具

远程 MCP 服务器提供完整的飞书工具集，包括：

- **OAuth**: 用户授权、授权状态检查
- **知识库**: 空间管理、节点管理
- **文档**: 读取/创建/更新文档、评论、媒体
- **多维表格**: 应用/表/记录/字段/视图管理
- **搜索**: 搜索文档和知识库
- **日历**: 日程/参与者/空闲时间
- **任务**: 任务/任务列表/评论/子任务
- **云盘**: 文件管理
- **消息**: 发送/获取/搜索消息
- **用户**: 获取/搜索用户

具体工具名称以 MCP 服务器实际提供的为准，使用时 AI 会自动发现可用工具。

### 常见错误处理

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| 99991663 | 需要用户授权 | 通过 `/mcp` → feishu → Authentication 完成授权 |
| 99991661 | 令牌过期 | 通过 `/mcp` → feishu → Authentication 重新授权 |
| 99991662 | 权限不足 | 检查应用权限配置 |

## 可用 Skills

| Skill | 用途 | 触发方式 |
|-------|------|----------|
| workspace-guide | 使用向导 | "help"、"怎么用" |
| doc-workflow | 文档工作流 | "创建分析报告"、"提交评审" |
| task-management | 任务管理 | "创建任务"、"查看任务" |
| feishu-sync | 飞书同步 | "同步到飞书" |
| project-status | 项目状态 | "生成周报"、"项目进展" |
| operation-sop | 运营流程 | 项目类型为 operation |

## 协作模式

根据任务类型推荐协作模式：

| 任务类型 | 推荐模式 | 说明 |
|----------|----------|------|
| 数据分析 | Mode B | 人主导分析方向，AI 辅助处理数据 |
| 策略制定 | Mode A | 团队讨论为主，AI 记录整理 |
| SOP 编写 | Mode D | AI 根据最佳实践生成 |
| 复盘报告 | Mode D | AI 汇总生成 |
| 周报 | Mode D | AI 汇总生成 |

## 运营特定功能

### 周期性报告

运营项目通常有周期性报告需求：

1. **日报**: 每日数据汇总
2. **周报**: 本周数据 + 进展
3. **月报**: 本月数据 + 趋势分析

使用 `project-status` Skill 自动生成：

```
用户: 生成上周周报
→ AI 读取 docs/analysis/ 目录
→ AI 读取飞书 Bitable 任务数据
→ AI 生成周报模板
→ 用户确认后发布
```

### SOP 管理

SOP（标准操作流程）是运营项目的核心：

1. **SOP 文档**
   - 记录在 `docs/sop/` 目录
   - 包含操作步骤、责任人、检查点

2. **执行追踪**
   - 使用飞书 Bitable 追踪 SOP 执行
   - 自动记录执行时间、执行人、结果

3. **SOP 复盘**
   - 定期检查 SOP 执行效果
   - 根据复盘结果优化 SOP

### 数据看板

飞书多维表格可以用作数据看板：

1. **配置数据源**
   - 连接数据库（可选）
   - 或手动维护数据

2. **自动更新**
   - 定时同步数据
   - 生成趋势图表

3. **告警规则**
   - 设置阈值告警
   - 通过飞书消息通知

## 多人协作

### 冲突检测

当多人同时编辑同一文档时：

1. **飞书端检测**
   - 会话启动时检查飞书文档更新时间
   - 对比 `feishu-map.md` 中的最后同步时间
   - 发现冲突时提示用户选择处理方式

2. **本地检测**
   - 使用 Git 追踪文档变更
   - 检测 `docs/` 目录下的文件冲突

### 协作规范

1. **数据更新规范**
   - 核心数据有明确的更新负责人
   - 更新后记录更新时间

2. **报告审核**
   - 重要报告需要主管审核
   - 审核记录在文档中

3. **知识积累**
   - 定期归档历史文档
   - 提炼可复用的分析框架

## Git 集成

### 提交规范

使用 Conventional Commits 格式：

```
<type>(<scope>): <subject>
```

**类型**:
- `docs`: 文档变更
- `data`: 数据更新
- `sop`: SOP 更新
- `chore`: 构建/工具

### 分支策略

```
main
├── docs/
│   ├── analysis/
│   ├── strategy/
│   └── sop/
└── data/
    └── sources/
```

## 目录结构

```
project-root/
├── CLAUDE.md              # 本文件
├── .mcp.json              # MCP 配置
├── .claude/
│   ├── skills/            # 项目级 Skills
│   └── context/          # AI 上下文
│       ├── team.md       # 团队成员
│       ├── doc-index.md  # 文档索引
│       └── feishu-map.md # 飞书映射
├── docs/
│   ├── analysis/         # 分析报告
│   ├── strategy/         # 策略文档
│   ├── sop/              # SOP 文档
│   ├── dashboard/        # 看板设计
│   └── review/           # 复盘文档
└── templates/            # 文档模版
    ├── analysis.md
    ├── strategy.md
    ├── sop.md
    └── review.md
```

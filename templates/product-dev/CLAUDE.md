# {项目名称} - AI 协同工作空间

> 📌 基于 claude-code-workspace v1.0.0

## 项目概况

- **项目类型**: product-dev（产品研发）
- **当前阶段**: {需求分析 | 技术设计 | 开发 | 测试 | 发布}
- **团队**: 见 .claude/context/team.md

## 文档规范

所有 `docs/` 下的 Markdown 文件必须包含 YAML Front Matter：

```yaml
---
title: 文档标题
type: prd | tech-design | test-plan | api-doc | meeting-notes
status: draft | review | approved | released
author: 作者名
version: "1.0"
created: 2026-03-09
updated: 2026-03-09
module: 所属模块（可选）
---
```

### 状态流转

```
draft → review → approved → released
```

### 文件命名

- 使用 kebab-case: `prd-order-module.md`
- 包含模块名前缀: `tech-order-service.md`

## AI 行为指令

### 会话启动检测

每个新会话开始时，执行以下检测：

1. **文档索引同步**
  - 扫描 `docs/` 目录
  - 对比 `.claude/context/doc-index.md`
  - 报告新增/删除/状态变更的文档

2. **飞书变更检测**（如果配置了飞书）
  - 读取 `.claude/context/feishu-map.md` 获取最后同步时间
  - 使用飞书 MCP 工具检查飞书端更新
  - 报告：有变更 → 提示用户是否拉取

3. **任务状态检查**
  - 检查飞书 Bitable 中分配给当前用户的任务
  - 报告：即将到期/已逾期的任务

4. **输出检测报告**
```
   📋 会话启动检测报告

   文档变更:
   - 新增: docs/product/prd-payment.md
   - 状态变更: prd-order.md (draft → review)

   飞书更新:
   - 订单模块PRD 在飞书有更新 (2026-03-09 13:00)
   → 是否拉取更新？[是] [稍后] [忽略]

   任务提醒:
   - ⚠️ TASK-001 将在 2 天后到期
```

### 文档创建

1. 创建文档时自动使用 `templates/` 中的模版
2. 根据用户描述推断文档类型
3. 自动填充 Front Matter 元数据

### 状态变更

1. 修改文档 `status` 时提醒用户触发的后续流程
2. PRD approved → 建议创建技术方案
3. 技术方案 approved → 建议拆解开发任务

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
  space_name: "项目知识库"

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
| --- | --- | --- |
| 99991663 | 需要用户授权 | 通过 `/mcp` → feishu → Authentication 完成授权 |
| 99991661 | 令牌过期 | 通过 `/mcp` → feishu → Authentication 重新授权 |
| 99991662 | 权限不足 | 检查应用权限配置 |

## 可用 Skills

| Skill | 用途 | 触发方式 |
| --- | --- | --- |
| workspace-guide | 使用向导 | "help"、"怎么用" |
| doc-workflow | 文档工作流 | "创建 PRD"、"提交评审" |
| task-management | 任务管理 | "创建任务"、"查看任务" |
| feishu-sync | 飞书同步 | "同步到飞书" |
| project-status | 项目状态 | "生成周报"、"项目进展" |
| product-dev-sop | 产品研发流程 | 项目类型为 product-dev |

## 协作模式

根据任务类型推荐协作模式：

| 任务类型 | 推荐模式 | 说明 |
| --- | --- | --- |
| 产品方案 | Mode B | 人主导，AI 辅助 |
| 技术方案 | Mode C | AI 生成草稿，人确认 |
| 测试用例 | Mode D | AI 自动生成 |
| 周报 | Mode D | AI 汇总生成 |

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

3. **冲突处理流程**
```
   ⚠️ 检测到冲突: docs/product/prd-order.md

   本地版本: 2026-03-09 14:00 (张三)
   飞书版本: 2026-03-09 13:30 (李四)

   处理方式:
   [1] 保留本地版本 (覆盖飞书)
   [2] 使用飞书版本 (覆盖本地)
   [3] 手动合并 (显示 diff)
   [4] 保留两个版本
```

### 协作规范

1. **文档锁定**（飞书原生支持）
  - 编辑前检查文档是否被锁定
  - 长时间编辑建议锁定文档

2. **变更通知**
  - 重要文档状态变更时通知相关人员
  - 使用飞书群或 IM 推送

3. **评审流程**
  - PRD/技术方案必须经过评审
  - 评审意见记录在文档中

## Git 集成

### 提交规范

使用 Conventional Commits 格式：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型**:
- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档变更
- `refactor`: 重构
- `test`: 测试
- `chore`: 构建/工具

**示例**:
```
feat(order): 添加订单创建接口

- 支持单个商品订单
- 支持批量商品订单
- 添加库存校验

Closes #123
```

### Git Hooks（可选）

在 `.git/hooks/` 中配置：

#### pre-commit

```bash
#!/bin/bash
# 检查文档 Front Matter
# 检查提交信息格式
```

#### post-commit

```bash
#!/bin/bash
# 提醒同步到飞书
# 更新 doc-index.md
```

### 分支策略

```
main
├── develop
│   ├── feature/order-module
│   ├── feature/payment-module
│   └── bugfix/payment-timeout
└── release/v1.0.0
```

**分支命名**:
- `feature/{module}-{feature}` - 新功能
- `bugfix/{module}-{issue}` - Bug 修复
- `release/v{version}` - 发布分支

## 目录结构

```
project-root/
├── CLAUDE.md              # 本文件（AI 自动加载）
├── .mcp.json               # MCP 配置（飞书等）
├── .claude/
│   ├── skills/            # 项目级 Skills
│   └── context/           # AI 上下文
│       ├── team.md        # 团队成员
│       ├── doc-index.md   # 文档索引
│       └── feishu-map.md  # 飞书映射
├── docs/
│   ├── product/           # 产品文档（PRD）
│   ├── tech/              # 技术文档
│   ├── test/              # 测试文档
│   └── meetings/          # 会议纪要
└── templates/             # 文档模版
    ├── prd.md
    ├── tech-design.md
    └── test-plan.md
```

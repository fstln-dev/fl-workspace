# {项目名称} - AI 协同工作空间

> 📌 基于 claude-code-workspace v1.0.0

## 项目概况

- **项目类型**: implementation（实施交付）
- **当前阶段**: {需求调研 | 方案设计 | 实施部署 | UAT验收 | 项目交付}
- **团队**: 见 .claude/context/team.md
- **客户**: {客户名称}

## 文档规范

所有 `docs/` 下的 Markdown 文件必须包含 YAML Front Matter：

```yaml
---
title: 文档标题
type: requirement | solution | design | config | test-report | handover
status: draft | review | approved | released
author: 作者名
version: "1.0"
created: 2026-03-09
updated: 2026-03-09
customer: 客户名称（可选）
module: 所属模块（可选）
---
```

### 状态流转

```
draft → review → approved → released
```

### 文件命名

- 使用 kebab-case: `requirement-customer-name.md`
- 包含客户/模块前缀: `solution-erp-integration.md`

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

### 文档创建

1. 创建文档时自动使用 `templates/` 中的模版
2. 根据用户描述推断文档类型
3. 自动填充 Front Matter 元数据
4. 实施类文档通常需要包含客户背景

### 状态变更

1. 修改文档 `status` 时提醒用户触发的后续流程
2. requirement approved → 建议创建解决方案
3. solution approved → 建议创建实施方案

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
|--------|------|----------|
| 99991663 | 需要用户授权 | 通过 `/mcp` → feishu → Authentication 完成授权 |
| 99991661 | 令牌过期 | 通过 `/mcp` → feishu → Authentication 重新授权 |
| 99991662 | 权限不足 | 检查应用权限配置 |

## 可用 Skills

| Skill | 用途 | 触发方式 |
|-------|------|----------|
| workspace-guide | 使用向导 | "help"、"怎么用" |
| doc-workflow | 文档工作流 | "创建需求文档"、"提交评审" |
| task-management | 任务管理 | "创建任务"、"查看任务" |
| feishu-sync | 飞书同步 | "同步到飞书" |
| project-status | 项目状态 | "生成周报"、"项目进展" |
| implementation-sop | 实施交付流程 | 项目类型为 implementation |

## 协作模式

根据任务类型推荐协作模式：

| 任务类型 | 推荐模式 | 说明 |
|----------|----------|------|
| 客户需求调研 | Mode A | 现场沟通为主，AI 记录整理 |
| 解决方案 | Mode B | 人主导，AI 辅助 |
| 实施方案 | Mode C | AI 生成草稿，人确认 |
| 交付文档 | Mode D | AI 自动生成 |
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

### 协作规范

1. **客户交付规范**
   - 交付文档必须经过内部评审
   - 客户确认签字后标记为 released

2. **变更管理**
   - 需求变更需要客户正式确认
   - 变更记录在文档的 changelog 中

3. **版本控制**
   - 每个交付版本创建 git tag
   - 保持与客户侧的版本对应

## Git 集成

### 提交规范

使用 Conventional Commits 格式：

```
<type>(<scope>): <subject>
```

**类型**:
- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档变更
- `refactor`: 重构
- `test`: 测试
- `chore`: 构建/工具

### 分支策略

```
main
├── develop
│   ├── requirement/
│   ├── solution/
│   └── implementation/
└── release/v1.0
```

## 目录结构

```
project-root/
├── CLAUDE.md              # 本文件
├── .mcp.json              # MCP 配置
├── .claude/
│   ├── skills/            # 项目级 Skills
│   └── context/           # AI 上下文
│       ├── team.md        # 团队成员
│       ├── doc-index.md   # 文档索引
│       └── feishu-map.md  # 飞书映射
├── docs/
│   ├── requirement/       # 需求文档
│   ├── solution/          # 解决方案
│   ├── design/            # 设计文档
│   ├── config/           # 配置文档
│   ├── test-report/       # 测试报告
│   └── handover/          # 交付文档
└── templates/             # 文档模版
    ├── requirement.md
    ├── solution.md
    ├── design.md
    ├── config.md
    └── handover.md
```

---
name: workspace-init
description: |
  工作空间初始化。在以下场景触发：
  1. 用户说 "初始化项目"、"初始化工作空间"
  2. 用户说 "新建项目"、"创建项目"
  3. 用户在一个空目录中说 "开始"
  4. 用户需要快速搭建项目结构

  一键初始化 Claude Code Native 工作空间，支持飞书集成配置。
---
# 工作空间初始化

快速初始化 Claude Code Native 工作空间，创建标准目录结构和配置文件。

## 初始化流程

### 步骤 0: 检查版本更新

首先检查本地工作空间版本：

```bash
# 如果 .claude-workspace 已存在，检查版本
cd .claude-workspace
git fetch origin
git status
```

如果显示有更新，提示用户：
```
🔄 工作空间有新版本可用

当前版本: 1.0.0
最新版本: x.x.x

是否更新？[是] [否]
```

如果用户选择更新：
```bash
git pull origin main
cp -r .claude-workspace/skills/* .claude/
echo "✅ 已更新到最新版本"
```

### 步骤 1: 选择项目类型

```
🚀 初始化 Claude Code Native 工作空间

请选择项目类型:

[1] product-dev    - 产品研发（PRD、技术方案、测试）
[2] implementation - 实施交付（需求、方案、实施、验收）
[3] operation      - 运营活动（分析、策略、SOP、复盘）
[4] it-infra       - IT 信息化（需求、方案、集成、运维）

选择: _
```

### 步骤 2: 基本信息

```
📝 项目基本信息

项目名称: my-project
项目描述: 一个示例项目
负责人: 张三

确认？[是] [修改]
```

### 步骤 3: 飞书集成配置

```
飞书集成配置

是否启用飞书集成？[是] [跳过]

如果选择 [是]：

推荐同时配置两个飞书 MCP，获得完整功能：

┌─────────────────────────────────────────────────────────────┐
│  feishu-official (飞书官方 MCP)                              │
│  → 云文档深度操作                                            │
│  → 需要先在飞书配置平台获取授权后的 MCP URL                   │
│  → 配置后立即可用                                            │
├─────────────────────────────────────────────────────────────┤
│  feishu (第三方 MCP)                                         │
│  → Wiki、多维表格、任务、日历、消息等                         │
│  → 配置后需要通过 /mcp 手动授权                              │
│  → 授权后需要重启 Claude Code                                │
└─────────────────────────────────────────────────────────────┘

[同时配置两个] [仅配置 feishu-official] [仅配置 feishu] [跳过]
```

#### 选项 A: 配置 feishu-official

```
📋 配置 feishu-official (飞书官方 MCP)

请按以下步骤获取 MCP URL：

1. 打开飞书 MCP 配置平台：https://open.feishu.cn/page/mcp
2. 点击「创建 MCP 服务」
3. 在「添加工具」中选择「云文档」工具集
4. 点击「授权」完成用户授权
5. 复制生成的「服务器 URL」

⚠️ 注意：
- 服务器 URL 包含您的授权信息，请勿泄露
- MCP 服务有效期 7 天，过期需重新授权
- 如链接泄露，可在平台点击「重置链接」

请粘贴您的 feishu-official MCP URL: _
```

**AI 收到 URL 后，写入 .mcp.json：**

```json
{
  "mcpServers": {
    "feishu-official": {
      "type": "http",
      "url": "<用户提供的 MCP URL>"
    }
  }
}
```

#### 选项 B: 配置 feishu

```
📋 配置 feishu (第三方 MCP)

feishu MCP 提供多维表格、任务、日历等功能。
配置后需要手动授权。

将在 .mcp.json 中添加：
{
  "feishu": {
    "type": "http",
    "url": "https://feishu-mcp.fastgrowth.ai/mcp"
  }
}

[继续] [跳过]
```

**如果用户选择继续，AI 更新 .mcp.json：**

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

#### 如果选择跳过飞书集成

```
飞书集成未配置

没有飞书集成时，以下功能不可用：
- Wiki 文档同步
- 多维表格操作
- 任务管理
- 日历集成
- 云文档操作

后续可随时通过「配置飞书」重新配置。

[继续] [返回配置]
```

### 步骤 4: feishu 授权（如果配置了 feishu）

```
📱 feishu MCP 授权

feishu MCP 已配置，但需要用户授权才能使用。

授权步骤：
1. 退出 Claude Code
2. 重新打开 Claude Code
3. 输入 /mcp
4. 选择 feishu MCP 服务器，按 Enter
5. 选择 Authentication
6. 浏览器自动打开授权页面，点击授权

⚠️ **重要：授权完成后，必须退出并重新打开 Claude Code！**

[已理解，稍后授权] [跳过]
```

### 步骤 5: 知识库配置（如果已完成授权）

```
知识库配置

是否现在配置项目关联的知识库？

→ 调用飞书 MCP 工具列出可访问的知识库

选择要关联的知识库序号: _

[配置] [跳过]
```

### 步骤 6: 生成结构

```
📁 生成项目结构...

✅ 创建 CLAUDE.md
✅ 创建 .mcp.json (飞书 MCP 配置)
✅ 创建 .claude/settings.json
✅ 创建 .claude/context/team.md
✅ 创建 .claude/context/doc-index.md
✅ 创建 .claude/context/feishu-config.md
✅ 创建 docs/product/.gitkeep
✅ 创建 docs/tech/.gitkeep
✅ 创建 docs/test/.gitkeep
✅ 创建 docs/meetings/.gitkeep
✅ 创建 templates/prd.md
✅ 创建 templates/tech-design.md
✅ 创建 templates/test-plan.md
✅ 创建 scripts/post-commit (Git Hook)

🔧 配置 Git Hooks...
✅ 安装 post-commit hook → .git/hooks/post-commit

🎉 初始化完成！
```

**自动安装 Git Hook：**

初始化时自动执行以下操作：

```bash
# 如果项目已 git init，自动安装 hook
if [ -d ".git/hooks" ]; then
  cp scripts/post-commit .git/hooks/post-commit
  chmod +x .git/hooks/post-commit
  echo "✅ Git post-commit hook 已安装"
fi
```

如果项目尚未初始化 git，hook 脚本会保存在 `scripts/post-commit`，用户可以在后续 `git init` 后手动安装。

## 功能对比

| 功能 | feishu-official | feishu |
| --- | --- | --- |
| 云文档深度操作 | ✅ | ❌ |
| Wiki 文档操作 | ✅ | ✅ |
| 多维表格操作 | ❌ | ✅ |
| 任务管理 | ❌ | ✅ |
| 日历集成 | ❌ | ✅ |
| 消息发送 | ❌ | ✅ |
| 配置方式 | 飞书平台获取 URL | /mcp 手动授权 |
| 有效期 | 7 天需续期 | 长期有效 |

**建议：** 同时配置两个 MCP，获得完整的飞书功能。

## 生成的目录结构

### product-dev 类型

```
project-root/
├── CLAUDE.md                    # AI 配置（自动加载）
├── .mcp.json                    # MCP 配置
├── .claude/
│   ├── settings.json            # Claude Code 设置
│   └── context/
│       ├── team.md              # 团队成员
│       ├── doc-index.md         # 文档索引
│       ├── feishu-config.md     # 飞书配置
│       └── feishu-map.md        # 飞书映射
├── docs/
│   ├── product/                 # 产品文档
│   ├── tech/                    # 技术文档
│   ├── test/                    # 测试文档
│   └── meetings/                # 会议纪要
└── templates/
    ├── prd.md
    ├── tech-design.md
    └── test-plan.md
```

## CLAUDE.md 模板

生成的 `CLAUDE.md` 根据项目类型自动配置：

### product-dev 模板

```markdown
# {项目名称} - AI 协同工作空间

## 项目概况

- **项目类型**: product-dev（产品研发）
- **当前阶段**: 需求分析
- **负责人**: {负责人}
- **团队**: 见 .claude/context/team.md

## 文档规范

[... 标准 product-dev 配置 ...]

## 飞书集成

Wiki Space: {space_id}
Bitable: {app_token}

## 可用 Skills

[... product-dev 可用 Skills 列表 ...]
```

## .mcp.json 配置示例

**同时配置两个 MCP：**

```json
{
  "mcpServers": {
    "feishu-official": {
      "type": "http",
      "url": "https://mcp.feishu.cn/mcp/xxx（用户从飞书平台获取的 URL）"
    },
    "feishu": {
      "type": "http",
      "url": "https://feishu-mcp.fastgrowth.ai/mcp"
    }
  }
}
```

## 团队配置

生成的 `.claude/context/team.md`：

```markdown
# 团队成员

| 姓名 | 角色 | 飞书 ID | 邮箱 |
|------|------|---------|------|
| 张三 | 负责人 | ou_xxx | zhang@example.com |

## 角色说明

- **负责人**: 项目整体负责，审批决策
- **产品**: 负责需求、PRD
- **技术**: 负责技术方案、开发
- **测试**: 负责测试用例、质量

## 评审规则

- PRD: 需要 2 人评审通过
- 技术方案: 需要技术负责人评审通过
- 代码: 需要 1 人评审通过
```

### 步骤 7: Git Hooks 配置

```
🔧 Git Hooks 配置

检测到 Git 仓库，自动安装 post-commit hook...

✅ post-commit hook 已安装到 .git/hooks/
```

**自动安装逻辑：**

| 场景 | 操作 |
| --- | --- |
| 项目已 `git init` | 自动复制 `scripts/post-commit` → `.git/hooks/post-commit` 并添加执行权限 |
| 项目未初始化 git | 保留脚本在 `scripts/post-commit`，等待用户 `git init` 后手动安装 |

**post-commit hook 功能：**
- 检测本次提交中 `status=approved` 的 .md 文件
- 如果有已批准的文档，提示用户可以同步到飞书
- 不自动执行同步，避免意外覆盖

**手动安装（如需要）：**

```bash
cp scripts/post-commit .git/hooks/
chmod +x .git/hooks/post-commit
```

## 快速模式

支持命令行快速初始化：

```
用户: "用 product-dev 模板初始化"

AI: 直接使用 product-dev 默认配置初始化，跳过交互步骤
```

```
用户: "初始化一个研究项目，名字叫 AI-Research"

AI: 使用 research 模板，项目名 AI-Research，快速初始化
```

## 检测已存在的工作空间

如果目录中已有 `CLAUDE.md`：

```
⚠️ 检测到已有工作空间配置

当前项目: {项目名称}
类型: {项目类型}
创建时间: {创建时间}

选项:
[1] 更新配置（保留现有文件）
[2] 重新初始化（覆盖现有文件）
[3] 取消
```

## 初始化后操作

初始化完成后，AI 建议：

```
🎉 工作空间已就绪！

下一步建议:

1. 编辑 .claude/context/team.md 添加团队成员

2. 完成飞书授权（如果配置了 feishu）:
   a. 退出并重新打开 Claude Code
   b. 输入 /mcp → 选择 feishu → Authentication
   c. 在浏览器中点击授权
   d. 再次退出并重新打开 Claude Code

3. 配置知识库（可选）:
   "帮我配置飞书知识库"

4. 创建第一个文档:
   "创建订单模块的 PRD"

开始工作？
```

## feishu-official URL 续期

feishu-official 的 MCP URL 有效期为 7 天。过期后：

1. 打开飞书 MCP 配置平台：https://open.feishu.cn/page/mcp
2. 找到对应的 MCP 服务
3. 点击「重新授权」
4. 复制新的服务器 URL
5. 更新项目中的 .mcp.json 文件
6. 重启 Claude Code

## 从现有项目迁移

如果目录中已有文档：

```
📦 检测到现有文档

发现:
- 5 个 .md 文件
- 2 个 .docx 文件

选项:
[1] 导入现有文档（自动分类）
[2] 仅初始化配置（不导入）
[3] 查看文件列表

导入后文档将被移动到对应目录:
- *.md → docs/product/ 或 docs/tech/
- *.docx → 需要手动转换
```

## 错误处理

| 错误 | 处理 |
| --- | --- |
| 目录非空 | 提示用户确认是否继续 |
| 权限不足 | 提示检查目录权限 |
| 飞书连接失败 | 允许跳过，稍后配置 |
| 模板不存在 | 使用默认模板 |
| MCP URL 无效 | 提示用户重新获取 |

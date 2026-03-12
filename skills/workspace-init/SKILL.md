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

飞书 MCP 使用远程服务器，无需配置本地凭证。

可选的飞书 MCP:
[1] feishu            - 日常交互，全功能（推荐）
[2] feishu-official   - 云文档深度操作（需要 OAuth 授权）
[3] 两者都配置       - 同时启用两个 MCP

将在 .mcp.json 中添加远程 MCP 服务器地址。

[继续] [跳过]
```

**重要：选择飞书集成后，AI 必须立即创建 .mcp.json 文件：**

```bash
# 创建 .mcp.json 文件
cat > .mcp.json << 'EOF'
{
  "mcpServers": {
    "feishu": {
      "type": "http",
      "url": "https://feishu-mcp.fastgrowth.ai/mcp"
    }
  }
}
EOF

echo "✅ 已创建 .mcp.json 配置文件"
```

如果选择 feishu-official 或两者都配置，则创建对应的配置。

如果选择跳过：
```
飞书集成未配置

后续可在 .mcp.json 中添加飞书 MCP 配置：
{
  "mcpServers": {
    "feishu": {
      "type": "http",
      "url": "https://feishu-mcp.fastgrowth.ai/mcp"
    }
  }
}

没有飞书集成时，以下功能不可用：
- Wiki 文档同步
- 多维表格操作
- 任务管理
- 日历集成

[继续] [返回配置]
```

### 步骤 4: OAuth 授权（如果配置了飞书）

```
飞书用户授权

是否现在完成用户授权 (UAT)？

用户授权后可以使用：
- 访问您的 Wiki 知识库
- 操作多维表格
- 管理个人任务
- 访问日历

[立即授权] [稍后授权]

如果选择 [立即授权]：
→ 在 Claude Code 中输入 /mcp
→ 选择 feishu MCP 服务器，按 Enter 进入
→ 选择 Authentication
→ 浏览器自动打开授权页面，用户点击授权即可

⚠️ **重要：授权完成后，必须退出并重新打开 Claude Code 才能加载 MCP！**

如果选择 [稍后授权]：
→ 提示用户随时可以通过 /mcp → feishu → Authentication 完成授权
→ 提醒：授权后需要重启 Claude Code
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
✅ 创建 .mcp.json (飞书远程 MCP 配置)
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

## .mcp.json 配置

生成的 `.mcp.json`：

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
2. 获取飞书授权（如果启用了飞书集成）:
   输入 /mcp → 选择 feishu → Authentication
   ⚠️ 授权后需要退出并重新打开 Claude Code
3. 创建第一个文档:
   "创建订单模块的 PRD"

开始工作？
```

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

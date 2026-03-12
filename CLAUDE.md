# Claude Code Native AI 协同工作空间

> ⚠️ **版本**: 1.0.0 | **最新版本**: https://github.com/fstln-dev/fl-workspace/releases

这是一套用于 Claude Code 的工作空间配置，让 AI 成为项目协作的原生运行时。

---

## 初始化（新项目）

当用户在一个新项目目录启动 Claude Code，并且让你读取这个文件时，你需要自动完成以下初始化步骤：

### 步骤 1：克隆工作空间

```bash
git clone --depth 1 https://github.com/fstln-dev/fl-workspace.git .claude-workspace
```

### 步骤 2：复制 Skills

```bash
cp -r .claude-workspace/skills/* .claude/
```

### 步骤 3：添加 Gitignore

检查项目是否有 `.gitignore` 文件，如果没有则创建。确保以下内容已添加：

```
# Claude Code Workspace (本地克隆，不提交)
.claude-workspace/
```

### 步骤 4：初始化项目

使用 workspace-init Skill 引导用户完成项目配置：

```
请使用 workspace-init 技能初始化项目
```

---

### 初始化检查清单

- [ ] `.claude-workspace/` 目录已克隆
- [ ] skills 已复制到 `.claude/` 目录
- [ ] `.gitignore` 已更新
- [ ] 用户已选择项目类型（product-dev / implementation / operation / it-infra）
- [ ] 对应的 SOP Skill 已生效

## 版本更新

当工作空间有新版本时，AI 应主动检查并提示用户更新：

### 自动检查版本

```bash
cd .claude-workspace
git fetch origin
git status
```

如果显示有更新，提示用户：
```
🔄 工作空间有新版本可用 (当前: 1.0.0 → 最新: x.x.x)

是否更新？回复"更新"我来帮你同步。
```

### 执行更新

```bash
cd .claude-workspace
git pull origin main

# 同步更新 skills
cp -r .claude-workspace/skills/* .claude/

# 重启 Claude Code 以加载新 skills
```

版本号遵循 [Semantic Versioning](https://semver.org/)：
- **主版本**：重大架构变更
- **次版本**：新增功能/模板
- **修订号**：bug 修复/文档更新

## 目录结构

```
fl-workspace/
├── CLAUDE.md                    # 本文件 - 工作空间使用指南
├── README.md                    # 项目介绍
├── LICENSE                      # MIT 许可证
├── templates/                   # 项目模版
│   ├── product-dev/            # 产品研发项目
│   ├── implementation/          # 实施交付项目
│   ├── operation/               # 运营项目
│   └── it-infra/               # IT 信息化项目
│       ├── CLAUDE.md            # 项目级 AI 配置
│       └── templates/           # 文档模版
│           ├── prd.md
│           └── tech-design.md
└── skills/                      # 通用 Skills
    ├── workspace-guide/         # 工作空间使用向导
    ├── workspace-init/          # 项目初始化
    ├── doc-workflow/            # 文档创建、评审、发布
    ├── task-management/         # 任务管理
    ├── project-status/          # 项目状态查询
    ├── feishu-connect/          # 飞书连接配置
    ├── feishu-sync/             # 飞书同步
    ├── product-dev-sop/         # 产品研发流程
    ├── implementation-sop/      # 实施交付流程
    ├── operation-sop/           # 运营流程
    └── it-infra-sop/            # IT 信息化流程
```

## 重要：Gitignore 配置

克隆到本地的 `.claude-workspace` 目录**不应该**提交到你的项目代码库。

在项目的 `.gitignore` 中添加：

```
# Claude Code Workspace (本地克隆，不提交)
.claude-workspace/
```

## 可用功能

### Skills

| Skill | 用途 | 触发方式 |
|-------|------|----------|
| workspace-init | 项目初始化向导 | 说"初始化项目" |
| workspace-guide | 工作空间使用指南 | 说"工作空间帮助" |
| doc-workflow | 文档创建、评审、发布 | 说"创建文档" |
| task-management | 任务管理 | 说"管理任务" |
| project-status | 项目状态查询 | 说"项目进展" |
| feishu-connect | 飞书连接配置 | 说"配置飞书" |
| feishu-sync | 飞书同步 | 说"同步飞书" |
| product-dev-sop | 产品研发流程 | 项目类型为 product-dev |
| implementation-sop | 实施交付流程 | 项目类型为 implementation |
| operation-sop | 运营流程 | 项目类型为 operation |
| it-infra-sop | IT 信息化流程 | 项目类型为 it-infra |

### 飞书集成

在项目的 `.mcp.json` 中配置远程飞书 MCP 服务器：

**feishu** (日常交互，全功能):
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

**feishu-official** (云文档深度操作，需要 OAuth):
```json
{
  "mcpServers": {
    "feishu": {
      "type": "http",
      "url": "https://feishu-mcp.fastgrowth.ai/mcp"
    },
    "feishu-official": {
      "type": "http",
      "url": "https://mcp.feishu.cn/mcp",
      "headers": {
        "X-Lark-MCP-UAT": "${FEISHU_UAT}",
        "X-Lark-MCP-Allowed-Tools": "search-doc,create-doc,fetch-doc,update-doc,list-docs,get-comments,add-comments,search-user,get-user,fetch-file"
      }
    }
  }
}
```

首次使用流程：
```
1. 授权: 在 Claude Code 中输入 /mcp → 选择 feishu → Authentication → 在浏览器点击授权
2. 配置知识库: "帮我配置飞书知识库"
3. 开始使用: 直接操作飞书资源
```

## 项目类型

根据项目性质选择对应的模板：

| 类型 | 适用场景 | 流程 |
|------|----------|------|
| product-dev | 产品研发、功能开发 | 需求 → 设计 → 开发 → 测试 → 发布 |
| implementation | 实施交付、客户项目 | 需求 → 方案 → 实施 → 验收 → 部署 |
| operation | 运营活动、持续运营 | 规划 → 执行 → 监控 → 复盘 |
| it-infra | IT 信息化建设、系统对接 | 需求 → 方案 → 开发 → 测试 → 上线 → 运维 |

选择模板后，对应的 SOP Skill 会自动生效。

## 文档规范

所有项目文档使用 Markdown + YAML Front Matter 格式：

```yaml
---
title: 文档标题
type: prd
status: draft
author: 作者
version: "1.0"
created: 2026-03-12
updated: 2026-03-12
---

# 文档内容
```

## 技术支持

- 问题反馈：https://github.com/fstln-dev/fl-workspace/issues
- 参与贡献：欢迎提交 Pull Request

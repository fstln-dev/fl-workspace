# Claude Code Native AI 协同工作空间

让 AI 成为项目协作的原生运行时。

## 特性

- 🎯 **4 种项目类型**：产品研发、实施交付、运营、IT 信息化
- 📋 **完整 SOP 流程**：每种项目类型都有标准化流程指导
- 🔗 **飞书深度集成**：通过 MCP 连接飞书，实现文档、任务同步
- 🧩 **可组合 Skills**：按需启用所需技能
- 📦 **一键初始化**：快速克隆并初始化到新项目

## 快速开始

### 方式一：直接使用 GitHub RAW URL

在任何新项目目录下，告诉 Claude Code：
```
请读取 https://raw.githubusercontent.com/fstln-dev/fl-workspace/main/CLAUDE.md 并按照指示初始化工作空间。
```

### 方式二：手动克隆

```bash
# 1. 克隆到本地
git clone --depth 1 https://github.com/fstln-dev/fl-workspace.git .claude-workspace

# 2. 复制 skills 到 .claude 目录
cp -r .claude-workspace/skills/* .claude/

# 3. 在 .gitignore 中添加（不要提交到代码库）
echo ".claude-workspace/" >> .gitignore

# 4. 重启 Claude Code，然后说"初始化项目"
```

## 项目类型

| 类型 | 适用场景 | 触发 Skill |
|------|----------|------------|
| product-dev | 产品研发、功能开发 | product-dev-sop |
| implementation | 实施交付、客户项目 | implementation-sop |
| operation | 运营活动、持续运营 | operation-sop |
| it-infra | IT 信息化建设 | it-infra-sop |

## 目录结构

```
fl-workspace/
├── CLAUDE.md              # 工作空间使用指南（本文件）
├── README.md              # 项目介绍
├── LICENSE                # MIT 许可证
├── templates/             # 项目模版
│   ├── product-dev/      # 产品研发
│   ├── implementation/   # 实施交付
│   ├── operation/       # 运营
│   └── it-infra/        # IT 信息化
└── skills/               # Claude Code Skills
    ├── workspace-init/    # 项目初始化
    ├── doc-workflow/     # 文档工作流
    ├── task-management/  # 任务管理
    ├── project-status/   # 项目状态
    ├── feishu-connect/  # 飞书连接
    ├── feishu-sync/     # 飞书同步
    └── *-sop/           # 各类型 SOP 流程
```

## Skills 列表

### 核心技能
- **workspace-init**: 项目初始化向导
- **workspace-guide**: 工作空间使用指南
- **doc-workflow**: 文档创建、评审、发布

### 项目管理
- **task-management**: 任务管理
- **project-status**: 项目状态查询

### 飞书集成
- **feishu-connect**: 飞书连接配置
- **feishu-sync**: 飞书同步

### 流程 SOP
- **product-dev-sop**: 产品研发流程
- **implementation-sop**: 实施交付流程
- **operation-sop**: 运营流程
- **it-infra-sop**: IT 信息化流程

## 飞书 MCP 配置

在项目的 `.mcp.json` 中添加：

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

首次使用：在 Claude Code 中输入 `/mcp` → 选择 feishu → 完成授权

## 版本更新

```bash
cd .claude-workspace
git fetch origin
git pull origin main
cp -r .claude-workspace/skills/* .claude/
```

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

MIT License - 详见 [LICENSE](./LICENSE) 文件

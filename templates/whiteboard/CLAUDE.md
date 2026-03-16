# {项目名称} - 个人工作空间

> 极简配置，自由记录

## 项目概况

- **项目类型**: whiteboard（白板/笔记）
- **创建时间**: {timestamp}

## 使用方式

这是一个极简工作空间，你可以：

1. **自由记录** - 在 `docs/` 目录下创建任意文档
2. **AI 辅助** - 让 AI 帮你整理、总结、扩展内容
3. **飞书同步** - 可选：同步文档到飞书知识库
4. **按需升级** - 随时可以切换到完整的项目模板

## 目录结构

```
project-root/
├── CLAUDE.md          # 本文件
├── docs/              # 你的文档
│   └── .gitkeep
└── templates/
    └── note.md        # 简单笔记模板
```

## 可用 Skills

| Skill | 用途 |
| --- | --- |
| doc-workflow | 文档创建（简化版） |
| feishu-sync | 飞书同步（如已配置） |
| workspace-guide | 工作空间指南 |

## 飞书集成（可选）

如需同步文档到飞书：

```
请配置飞书集成
```

AI 将引导你完成飞书 MCP 配置。

## 升级到完整模板

当你需要更规范的流程时，可以升级：

| 模板 | 适用场景 |
| --- | --- |
| product-dev | 产品研发流程（PRD、技术方案、测试） |
| implementation | 实施交付流程（需求、方案、实施、验收） |
| operation | 运营流程（分析、策略、SOP、复盘） |
| it-infra | IT 信息化流程（需求、方案、集成、运维） |

说 "升级到 {type} 模板" 即可。

## Git 提交规范

本项目使用 Conventional Commits：

```
<type>(<scope>): <subject>

<body>
```

类型：
- `docs`: 文档变更
- `feat`: 新功能
- `fix`: 修复
- `refactor`: 重构
- `chore`: 杂项

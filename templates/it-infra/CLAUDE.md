# {项目名称} - AI 协同工作空间

> 📌 基于 claude-code-workspace v1.0.0

## 项目概况

- **项目类型**: it-infra（IT 信息化项目）
- **当前阶段**: {需求分析 | 技术方案 | 开发实施 | 测试验收 | 上线部署 | 运维支持}
- **团队**: 见 .claude/context/team.md

## 文档规范

所有 `docs/` 下的 Markdown 文件必须包含 YAML Front Matter：

```yaml
---
title: 文档标题
type: requirement | tech-design | integration | migration | test-report | operation
status: draft | review | approved | released | deprecated
author: 作者名
version: "1.0"
created: 2026-03-09
updated: 2026-03-09
system: 涉及系统（可选）
module: 所属模块（可选）
---
```

### 状态流转

```
draft → review → approved → released → deprecated
```

### 文件命名

- 使用 kebab-case: `requirement-erp-integration.md`
- 包含系统前缀: `tech-design-payment-gateway.md`

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

### 状态变更

1. 修改文档 `status` 时提醒用户触发的后续流程
2. requirement approved → 建议创建技术方案
3. tech-design approved → 建议创建实施计划
4. 测试完成 → 建议创建部署文档

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
  space_name: "IT项目知识库"

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
| it-infra-sop | IT 信息化流程 | 项目类型为 it-infra |

## 协作模式

根据任务类型推荐协作模式：

| 任务类型 | 推荐模式 | 说明 |
|----------|----------|------|
| 需求分析 | Mode A | 现场沟通为主，AI 记录整理 |
| 技术方案 | Mode C | AI 生成草稿，人确认 |
| 集成方案 | Mode C | AI 生成草稿，人确认 |
| 迁移方案 | Mode D | AI 根据模板生成 |
| 测试用例 | Mode D | AI 自动生成 |
| 部署文档 | Mode D | AI 自动生成 |
| 周报 | Mode D | AI 汇总生成 |

## IT 项目特定功能

### 系统关联

IT 项目通常涉及多个系统：

1. **系统清单**
   - 记录在 `docs/systems/` 目录
   - 包含系统名称、接口、负责人

2. **接口文档**
   - 记录各系统的 API 接口
   - 使用 OpenAPI 或自研格式

3. **数据流图**
   - 记录系统间数据流向
   - 存放在 `docs/diagrams/` 目录

### 集成项目管理

涉及第三方系统集成的项目：

1. **集成清单**
   - 记录所有第三方系统
   - 包含接口协议、认证方式

2. **联调计划**
   - 记录联调时间、责任人
   - 追踪联调进度

3. **问题跟踪**
   - 使用飞书任务跟踪集成问题
   - 记录解决方案

### 变更管理

IT 项目需要严格的变更管理：

1. **变更申请**
   - 记录变更内容、影响范围
   - 需要审批流程

2. **变更记录**
   - 记录所有变更历史
   - 包含变更原因、实施时间

3. **回滚方案**
   - 每个变更必须有回滚方案
   - 记录在变更文档中

### 运维交接

项目交付后需要运维交接：

1. **运维手册**
   - 系统操作指南
   - 常见问题处理

2. **应急响应**
   - 故障处理流程
   - 联系方式清单

3. **监控配置**
   - 监控指标定义
   - 告警规则

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

1. **评审流程**
   - 技术方案必须经过技术评审
   - 集成方案需要各方确认

2. **变更流程**
   - 重大变更需要变更委员会审批
   - 紧急变更事后补办手续

3. **知识沉淀**
   - 重要决策记录在文档中
   - 经验教训定期复盘

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
- `infra`: 基础设施

**示例**:
```
feat(payment): 集成支付宝支付

- 新增支付接口
- 支持异步回调
- 添加支付状态查询

Closes #123
```

### 分支策略

```
main
├── develop
│   ├── feature/
│   ├── bugfix/
│   └── hotfix/
├── release/
└── docs/
```

### 版本管理

IT 项目版本管理：

```
v1.0.0-rc1  # 候选版本
v1.0.0       # 正式版本
v1.0.1       # Bug 修复版本
v2.0.0       # 大版本升级
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
│   ├── tech/              # 技术方案
│   ├── integration/       # 集成方案
│   ├── migration/        # 迁移方案
│   ├── test/             # 测试文档
│   ├── deployment/       # 部署文档
│   └── operation/        # 运维文档
├── configs/              # 配置文件
├── scripts/              # 脚本文件
├── diagrams/            # 架构图、流程图
└── templates/           # 文档模版
    ├── requirement.md
    ├── tech-design.md
    ├── integration.md
    ├── migration.md
    └── operation.md
```

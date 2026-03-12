---
name: feishu-connect
description: |
  飞书 MCP 连接管理。在以下场景触发：
  1. 用户说 "飞书连接"、"连接飞书"、"飞书配置"
  2. 用户说 "飞书状态"、"检查飞书连接"
  3. 用户说 "飞书授权"、"授权飞书"
  4. 用户说 "飞书 token"、"刷新飞书 token"
  5. 会话启动时自动检测飞书 MCP 状态

  管理两个飞书 MCP 的连接状态、授权流程和 Token 刷新。
---
# 飞书 MCP 连接管理

管理飞书 MCP 的连接、授权和 Token 刷新。

## 两个飞书 MCP

| 名称 | 用途 | 地址 |
| --- | --- | --- |
| `feishu` | 日常交互，全功能 | `https://feishu-mcp.fastgrowth.ai/mcp` |
| `feishu-official` | 云文档深度操作 | `https://mcp.feishu.cn/mcp` |

## 功能列表

| 功能 | 描述 |
| --- | --- |
| 状态检测 | 检测两个 MCP 的配置和授权状态 |
| 配置引导 | 引导用户配置 MCP |
| 授权引导 | 引导用户完成 OAuth 授权 |
| Token 刷新 | 刷新官方 MCP 的 UAT Token |
| 问题诊断 | 检测常见问题并给出解决方案 |

## 检测 MCP 状态

### 步骤 1: 读取 MCP 配置

读取项目根目录的 `.mcp.json`:

```json
{
  "mcpServers": {
    "feishu": { ... },
    "feishu-official": { ... }
  }
}
```

### 步骤 2: 读取 Token 配置

检查官方 MCP 的 UAT Token 状态：

1. 检查环境变量 `FEISHU_UAT` 是否存在
2. 检查本地缓存的 Token（`.claude/feishu/uat.json`）

### 步骤 3: 输出状态

```
📋 飞书 MCP 状态检测

┌─────────────────┬──────────┬────────────────────────────────┐
│ MCP             │ 状态     │ 详情                           │
├─────────────────┼──────────┼────────────────────────────────┤
│ feishu          │ ✅ 已配置 │ 远程 MCP，自动 OAuth           │
│ feishu-official │ ⚠️ 待授权 │ 需手动 OAuth 授权            │
└─────────────────┴──────────┴────────────────────────────────┘

详细说明:
- feishu: 通过 /mcp → feishu → Authentication 完成授权
- feishu-official: 需要先获取 UAT Token

[刷新 Token] [重新检测] [查看帮助]
```

## 配置 MCP

### 场景 1: 添加 feishu MCP

如果 `.mcp.json` 中没有 `feishu` 配置：

```
是否添加 feishu MCP？

feishu MCP 提供完整的飞书能力：
- 文档/知识库操作
- 多维表格操作
- 日历/任务管理
- 消息发送

[添加] [跳过]
```

如果选择添加，生成配置并写入 `.mcp.json`:

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

### 场景 2: 添加 feishu-official MCP

如果 `.mcp.json` 中没有 `feishu-official` 配置：

```
是否添加 feishu-official MCP？

feishu-official MCP 提供云文档深度操作：
- search-doc (仅 UAT 可用)
- create-doc, fetch-doc, update-doc
- list-docs, get-comments, add-comments
- 搜索用户等

[添加] [跳过]
```

如果选择添加，生成配置：

```json
{
  "mcpServers": {
    "feishu": { ... },
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

## 授权流程

### feishu MCP 授权

```
飞书授权流程 (feishu)

1. 在 Claude Code 中输入 /mcp
2. 选择 feishu MCP 服务器
3. 按 Enter 进入后，选择 Authentication
4. 浏览器自动打开授权页面
5. 在网页上点击授权

授权完成后，飞书工具即可使用。
```

### feishu-official MCP 授权

此 MCP 使用 UAT Token，需要通过 OAuth Device Flow 完成授权。

#### 首次授权

**Step 1: 启动授权流程**

调用以下 API 获取授权码：

```bash
curl -X POST 'https://accounts.feishu.cn/oauth/v1/device_authorization' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'client_id=cli_a924f4657d389bd7&client_secret=DJkvBSsQpDIe0cXJ6nki7csoBlxeqvqd&scope=docx:document:readonly docx:document:create docx:document:write_only wiki:node:read wiki:node:create wiki:wiki:readonly search:docs:read contact:user.base:readonly contact:user:search'
```

返回示例：
```json
{
  "device_code": "Ox...",
  "user_code": "XXXX-XXXX",
  "verification_uri": "https://accounts.feishu.cn/oauth/v1/device/verify?flow_id=...",
  "verification_uri_complete": "https://accounts.feishu.cn/oauth/v1/device/verify?flow_id=...&user_code=XXXX-XXXX",
  "expires_in": 180,
  "interval": 5
}
```

**Step 2: 用户完成授权**

告诉用户：

**Step 3: 轮询获取 Token**

用户完成授权后，调用：

```bash
curl -X POST 'https://open.feishu.cn/open-apis/authen/v2/oauth/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=urn:ietf:params:oauth:grant-type:device_code&client_id=cli_a924f4657d389bd7&client_secret=DJkvBSsQpDIe0cXJ6nki7csoBlxeqvqd&device_code={device_code}'
```

成功返回：
```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 7200,
  "refresh_token": "...",
  "scope": "..."
}
```
```
1. 打开浏览器访问:
   {verification_uri_complete}

2. 输入验证码: {user_code}

3. 完成飞书登录和授权
```

**Step 4: 测试连接**

用获取的 access_token 测试 MCP：

```bash
curl -X POST 'https://mcp.feishu.cn/mcp' \
  -H 'Content-Type: application/json' \
  -H 'X-Lark-MCP-UAT: {access_token}' \
  -H 'X-Lark-MCP-Allowed-Tools: search-doc,fetch-doc,create-doc,list-docs,get-user,search-user' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize"}'
```

成功响应：
```json
{"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2025-03-26","serverInfo":{"name":"AI friendly MCP Server","version":"1.0.0"}}}
```

#### 刷新 Token

当 UAT 过期时（有效期约 2 小时），用 refresh_token 续期：

```bash
curl -X POST 'https://open.feishu.cn/open-apis/authen/v1/refresh_access_token' \
  -H 'Content-Type: application/json' \
  -d '{
    "grant_type": "refresh_token",
    "refresh_token": "存储的refresh_token",
    "client_id": "cli_a924f4657d389bd7",
    "client_secret": "DJkvBSsQpDIe0cXJ6nki7csoBlxeqvqd"
  }'
```

## Token 管理

### 存储位置

Token 存储在工作空间本地目录，不提交版本控制（会自动添加到 .gitignore）：

- 路径: `.claude/feishu/uat.json`
- 结构: 同时包含 UAT 和 refresh_token

### 格式

```json
{
  "feishu-official": {
    "uat": "u-xxxxx",
    "refresh_token": "r-xxxxx",
    "expires_at": "2026-03-12T15:00:00Z",
    "created_at": "2026-03-12T13:00:00Z"
  }
}
```

### 自动刷新

启动 Claude Code 时自动检查 UAT 有效性，过期则自动刷新。

## 问题诊断

| 问题 | 可能原因 | 解决方案 |
| --- | --- | --- |
| MCP 连接失败 | 网络问题、URL 错误 | 检查 `.mcp.json` 配置 |
| 授权失败 | 授权码过期 | 重新获取授权码 |
| Token 无效 | UAT 过期 | 刷新 Token |
| 工具调用失败 | 权限不足 | 检查应用权限配置 |

## 常用命令

- "检查飞书状态" - 检测两个 MCP 的状态
- "连接飞书" - 配置飞书 MCP
- "刷新飞书 token" - 刷新 feishu-official 的 UAT
- "飞书授权" - 引导完成 OAuth 授权

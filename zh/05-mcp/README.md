<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../../resources/logos/claude-howto-logo.svg">
</picture>

# MCP (Model Context Protocol)

本文件夹包含关于 MCP server 配置以及在 Claude Code 中使用 MCP 的完整文档和示例。

## 概览

MCP（Model Context Protocol）是一种标准化方式，让 Claude 能够访问外部工具、API 和实时数据源。与 Memory 不同，MCP 提供对变化数据的实时访问。

主要特性：
- 实时访问外部服务
- 实时数据同步
- 可扩展的架构
- 安全的认证
- 基于工具的交互

## MCP 架构

```mermaid
graph TB
    A["Claude"]
    B["MCP Server"]
    C["External Service"]

    A -->|Request: list_issues| B
    B -->|Query| C
    C -->|Data| B
    B -->|Response| A

    A -->|Request: create_issue| B
    B -->|Action| C
    C -->|Result| B
    B -->|Response| A

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
    style C fill:#e8f5e9,stroke:#333,color:#333
```

## MCP 生态

```mermaid
graph TB
    A["Claude"] -->|MCP| B["Filesystem<br/>MCP Server"]
    A -->|MCP| C["GitHub<br/>MCP Server"]
    A -->|MCP| D["Database<br/>MCP Server"]
    A -->|MCP| E["Slack<br/>MCP Server"]
    A -->|MCP| F["Google Docs<br/>MCP Server"]

    B -->|File I/O| G["Local Files"]
    C -->|API| H["GitHub Repos"]
    D -->|Query| I["PostgreSQL/MySQL"]
    E -->|Messages| J["Slack Workspace"]
    F -->|Docs| K["Google Drive"]

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
    style C fill:#f3e5f5,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#f3e5f5,stroke:#333,color:#333
    style F fill:#f3e5f5,stroke:#333,color:#333
    style G fill:#e8f5e9,stroke:#333,color:#333
    style H fill:#e8f5e9,stroke:#333,color:#333
    style I fill:#e8f5e9,stroke:#333,color:#333
    style J fill:#e8f5e9,stroke:#333,color:#333
    style K fill:#e8f5e9,stroke:#333,color:#333
```

## MCP 安装方式

Claude Code 支持多种用于 MCP server 连接的传输协议：

### HTTP 传输（推荐）

```bash
# Basic HTTP connection
claude mcp add --transport http notion https://mcp.notion.com/mcp

# HTTP with authentication header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### Stdio 传输（本地）

适用于在本地运行的 MCP server：

```bash
# Local Node.js server
claude mcp add --transport stdio myserver -- npx @myorg/mcp-server

# With environment variables
claude mcp add --transport stdio myserver --env KEY=value -- npx server
```

#### 用于 stdio server 的 `CLAUDE_PROJECT_DIR`（v2.1.139+）

每个 MCP stdio server 在启动时，其环境中都已经设置了 `CLAUDE_PROJECT_DIR=<absolute path to repo root>` —— 这与 hooks 使用的约定相同。plugin 和项目的 `.mcp.json` 文件可以在 `command`、`args` 和 `env` 的值中引用 `${CLAUDE_PROJECT_DIR}`，替换会在 `execve()` 之前完成：

```json
{
  "mcpServers": {
    "repo-tools": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PROJECT_DIR}/.claude/mcp/repo-tools.js"],
      "env": {
        "REPO_ROOT": "${CLAUDE_PROJECT_DIR}"
      }
    }
  }
}
```

当你的 stdio server 需要相对于项目根目录读取文件、且无论从何处启动 Claude Code 都要保持一致时，使用此方式。

stdio MCP server 还会收到 `CLAUDE_CODE_SESSION_ID`（与传给 hooks 和 Bash 的值一致），包括使用 `--resume` 恢复会话时（v2.1.163+）。

### SSE 传输（已弃用）

Server-Sent Events 传输已被弃用，推荐改用 `http`，但仍受支持：

```bash
claude mcp add --transport sse legacy-server https://example.com/sse
```

### Windows 专用说明

在原生 Windows（非 WSL）上，对 npx 命令使用 `cmd /c`：

```bash
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### OAuth 2.0 认证

Claude Code 支持为需要 OAuth 2.0 的 MCP server 使用该认证方式。连接到启用了 OAuth 的 server 时，Claude Code 会处理整个认证流程：

```bash
# Connect to an OAuth-enabled MCP server (interactive flow)
claude mcp add --transport http my-service https://my-service.example.com/mcp

# Pre-configure OAuth credentials for non-interactive setup
claude mcp add --transport http my-service https://my-service.example.com/mcp \
  --client-id "your-client-id" \
  --client-secret "your-client-secret" \
  --callback-port 8080
```

| 特性 | 说明 |
|---------|-------------|
| **交互式 OAuth** | 使用 `/mcp` 触发基于浏览器的 OAuth 流程 |
| **预配置的 OAuth 客户端** | 为 Notion、Stripe 等常见服务内置的 OAuth 客户端（v2.1.30+） |
| **预配置凭据** | 用于自动化设置的 `--client-id`、`--client-secret`、`--callback-port` 标志 |
| **令牌存储** | 令牌安全地存储在你的系统钥匙串中 |
| **分步提权认证（Step-up auth）** | 支持针对特权操作的分步提权认证 |
| **发现缓存** | OAuth 发现元数据会被缓存，以加快重新连接速度 |
| **元数据覆盖** | `.mcp.json` 中的 `oauth.authServerMetadataUrl` 可覆盖默认的 OAuth 元数据发现 |

#### 覆盖 OAuth 元数据发现

如果你的 MCP server 在标准 OAuth 元数据端点（`/.well-known/oauth-authorization-server`）上返回错误，但暴露了一个可用的 OIDC 端点，你可以让 Claude Code 从指定 URL 获取 OAuth 元数据。在 server 配置的 `oauth` 对象中设置 `authServerMetadataUrl`：

```json
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

该 URL 必须使用 `https://`。此选项需要 Claude Code v2.1.64 或更高版本。

### Claude.ai MCP 连接器

在你的 Claude.ai 账户中配置的 MCP server 会自动在 Claude Code 中可用。这意味着你通过 Claude.ai 网页界面设置的任何 MCP 连接，都无需额外配置即可访问。

Claude.ai MCP 连接器在 `--print` 模式下也可用（v2.1.83+），从而支持非交互式和脚本化使用。

> **启动说明（v2.1.117+）：** 当同时配置了本地和 claude.ai MCP server 时，并发连接成为默认行为（此前为串行），在使用多个 server 时减少启动延迟。

要在 Claude Code 中禁用 Claude.ai MCP server，将 `ENABLE_CLAUDEAI_MCP_SERVERS` 环境变量设置为 `false`：

```bash
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

> **注意：** 此功能仅对使用 Claude.ai 账户登录的用户可用。

## MCP 设置流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Config as Config File
    participant Service as External Service

    User->>Claude: Type /mcp
    Claude->>Claude: List available MCP servers
    Claude->>User: Show options
    User->>Claude: Select GitHub MCP
    Claude->>Config: Update configuration
    Config->>Claude: Activate connection
    Claude->>Service: Test connection
    Service-->>Claude: Authentication successful
    Claude->>User: ✅ MCP connected!
```

### `/mcp` 命令

在会话中输入 `/mcp` 可列出已连接的 server、触发 OAuth 流程并检查连接状态。

- 自 **v2.1.121** 起，MCP 在遇到临时错误时会重试初始连接最多 3 次。
- 自 **v2.1.128** 起，`/mcp` 会显示每个已连接 server 的**工具数量**，并以视觉方式标记报告 **0 个工具**的 server，使配置错误的 server 一目了然。

## MCP 工具搜索

当 MCP 工具描述超过上下文窗口的 10% 时，Claude Code 会自动启用工具搜索，以高效地选择正确的工具，而不会让模型上下文不堪重负。

| 设置 | 取值 | 说明 |
|---------|-------|-------------|
| `ENABLE_TOOL_SEARCH` | `auto`（默认） | 当工具描述超过上下文的 10% 时自动启用 |
| `ENABLE_TOOL_SEARCH` | `auto:<N>` | 在自定义阈值 `N` 个工具时自动启用 |
| `ENABLE_TOOL_SEARCH` | `true` | 无论工具数量多少始终启用 |
| `ENABLE_TOOL_SEARCH` | `false` | 禁用；完整发送所有工具描述 |

> **注意：** 工具搜索需要 Sonnet 4 或更高版本，或 Opus 4 或更高版本。Haiku 模型不支持工具搜索。

### 按 server 绕过工具搜索（v2.1.121+）

如果每一轮都需要某个特定 MCP server 的工具，可在其配置中标记
`"alwaysLoad": true`，以跳过工具搜索的延迟加载，
让其工具始终可用：

```json
{
  "mcpServers": {
    "always-on-tool": {
      "command": "node",
      "args": ["./tools/always.js"],
      "alwaysLoad": true
    }
  }
}
```

请节制使用 —— 每个始终加载的工具都会消耗本可以
用于工具搜索来呈现更相关工具的上下文。

## 动态工具更新

Claude Code 支持 MCP 的 `list_changed` 通知。当某个 MCP server 动态地添加、移除或修改其可用工具时，Claude Code 会接收到更新并自动调整其工具列表 —— 无需重新连接或重启。

## MCP Apps

MCP Apps 是首个官方 MCP 扩展，使 MCP 工具调用能够返回直接在聊天界面中渲染的交互式 UI 组件。MCP server 不再只能返回纯文本响应，而是可以提供丰富的仪表盘、表单、数据可视化和多步骤工作流 —— 全部内联显示，无需离开对话。

## MCP Elicitation

MCP server 可以通过交互式对话框向用户请求结构化输入（v2.1.49+）。这允许 MCP server 在工作流中途请求额外信息 —— 例如，提示确认、从选项列表中选择，或填写必填字段 —— 为 MCP server 交互增加交互性。

## 工具描述与指令上限

自 v2.1.84 起，Claude Code 对每个 MCP server 的工具描述和指令强制施加 **2 KB 上限**。这可以防止单个 server 因过于冗长的工具定义而消耗过多上下文，减少上下文膨胀并保持交互高效。

## 把 MCP Prompts 暴露成 Slash Commands

MCP server 可以暴露出现在 Claude Code 中作为 slash command 的 prompts。这些 prompts 可使用以下命名约定访问：

```
/mcp__<server>__<prompt>
```

例如，如果一个名为 `github` 的 server 暴露了名为 `review` 的 prompt，你可以通过 `/mcp__github__review` 调用它。

## Server 去重

当同一个 MCP server 在多个作用域（local、project、user）中定义时，local 配置优先。这让你能够用 local 自定义覆盖项目级或用户级的 MCP 设置，而不产生冲突。

## 近期生命周期修复（v2.1.136）

v2.1.136 修复了两个长期存在的 MCP 生命周期 bug —— 如果你运行多 server 设置，值得升级：

- **MCP server 在 `/clear` 后保持存在**：通过 `.mcp.json`、plugins 或 claude.ai 连接器配置的 server，在 VS Code、JetBrains 或 Agent SDK 中执行 `/clear` 后不再消失。早期版本会悄悄丢弃它们并要求重启。
- **OAuth 刷新令牌并发刷新修复**：多 server OAuth 设置在多个 server 同时竞争刷新时不再丢失刷新令牌。这消除了影响多个受 OAuth 保护的 MCP server 设置的"每天早上都得重新认证"现象。

## 通过 `@` 提及使用 MCP 资源

你可以使用 `@` 提及语法在 prompt 中直接引用 MCP 资源：

```
@server-name:protocol://resource/path
```

例如，要引用某个特定的数据库资源：

```
@database:postgres://mydb/users
```

这允许 Claude 获取并内联包含 MCP 资源内容，作为对话上下文的一部分。

## MCP 作用域

MCP 配置可以存储在不同的作用域中，具有不同的共享级别：

| 作用域 | 位置 | 说明 | 共享对象 | 是否需要批准 |
|-------|----------|-------------|-------------|------------------|
| **Local**（默认） | `~/.claude.json`（在项目路径下） | 仅对当前用户、当前项目私有（在旧版本中称为 `project`） | 仅你自己 | 否 |
| **Project** | `.mcp.json` | 提交到 git 仓库 | 团队成员 | 是（首次使用） |
| **User** | `~/.claude.json` | 在所有项目中可用（在旧版本中称为 `global`） | 仅你自己 | 否 |

### 使用项目作用域

将项目特定的 MCP 配置存储在 `.mcp.json` 中：

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.github.com/mcp"
    }
  }
}
```

团队成员在首次使用项目 MCP 时会看到批准提示。

## MCP 配置管理

### 添加 MCP server

```bash
# Add HTTP-based server
claude mcp add --transport http github https://api.github.com/mcp

# Add local stdio server
claude mcp add --transport stdio database -- npx @company/db-server

# List all MCP servers
claude mcp list

# Get details on specific server
claude mcp get github

# Remove an MCP server
claude mcp remove github

# Reset project-specific approval choices
claude mcp reset-project-choices

# Authenticate an MCP server from the CLI (v2.1.186+)
claude mcp login github

# Sign out of an MCP server (v2.1.186+)
claude mcp logout github

# Import from Claude Desktop
claude mcp add-from-claude-desktop
```

`claude mcp login <name>` / `claude mcp logout <name>` 是 `/mcp` 菜单中 OAuth 流程的非交互式等价命令 —— 无需打开菜单即可认证或登出。为 `login` 添加 `--no-browser` 可在 SSH 上或无头会话中完成 OAuth（它会通过 stdin 重定向流程）。

## 可用 MCP Server 表

| MCP Server | 用途 | 常用工具 | 认证 | 实时 |
|------------|---------|--------------|------|-----------|
| **Filesystem** | 文件操作 | read, write, delete | OS 权限 | ✅ 是 |
| **GitHub** | 仓库管理 | list_prs, create_issue, push | OAuth | ✅ 是 |
| **Slack** | 团队沟通 | send_message, list_channels | Token | ✅ 是 |
| **Database** | SQL 查询 | query, insert, update | 凭据 | ✅ 是 |
| **Google Docs** | 文档访问 | read, write, share | OAuth | ✅ 是 |
| **Asana** | 项目管理 | create_task, update_status | API Key | ✅ 是 |
| **Stripe** | 支付数据 | list_charges, create_invoice | API Key | ✅ 是 |
| **Memory** | 持久化记忆 | store, retrieve, delete | 本地 | ❌ 否 |

## 实战示例

### 示例 1：GitHub MCP 配置

**文件：** `.mcp.json`（项目根目录）

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**可用的 GitHub MCP 工具：**

#### Pull Request 管理
- `list_prs` - 列出仓库中的所有 PR
- `get_pr` - 获取 PR 详情（包括 diff）
- `create_pr` - 创建新 PR
- `update_pr` - 更新 PR 描述/标题
- `merge_pr` - 将 PR 合并到主分支
- `review_pr` - 添加评审评论

**示例请求：**
```
/mcp__github__get_pr 456

# Returns:
Title: Add dark mode support
Author: @alice
Description: Implements dark theme using CSS variables
Status: OPEN
Reviewers: @bob, @charlie
```

#### Issue 管理
- `list_issues` - 列出所有 issue
- `get_issue` - 获取 issue 详情
- `create_issue` - 创建新 issue
- `close_issue` - 关闭 issue
- `add_comment` - 为 issue 添加评论

#### 仓库信息
- `get_repo_info` - 仓库详情
- `list_files` - 文件树结构
- `get_file_content` - 读取文件内容
- `search_code` - 在代码库中搜索

#### Commit 操作
- `list_commits` - commit 历史
- `get_commit` - 特定 commit 详情
- `create_commit` - 创建新 commit

**设置**：
```bash
export GITHUB_TOKEN="your_github_token"
# Or use the CLI to add directly:
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

### 配置中的环境变量展开

MCP 配置支持带回退默认值的环境变量展开。`${VAR}` 和 `${VAR:-default}` 语法可用于以下字段：`command`、`args`、`env`、`url` 和 `headers`。

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}",
        "X-Custom-Header": "${CUSTOM_HEADER:-default-value}"
      }
    },
    "local-server": {
      "command": "${MCP_BIN_PATH:-npx}",
      "args": ["${MCP_PACKAGE:-@company/mcp-server}"],
      "env": {
        "DB_URL": "${DATABASE_URL:-postgresql://localhost/dev}"
      }
    }
  }
}
```

变量在运行时展开：
- `${VAR}` - 使用环境变量，未设置则报错
- `${VAR:-default}` - 使用环境变量，未设置则回退到默认值

### 示例 2：数据库 MCP 设置

**配置：**

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-database"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost/mydb"
      }
    }
  }
}
```

**示例用法：**

```markdown
User: Fetch all users with more than 10 orders

Claude: I'll query your database to find that information.

# Using MCP database tool:
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id
HAVING COUNT(o.id) > 10
ORDER BY order_count DESC;

# Results:
- Alice: 15 orders
- Bob: 12 orders
- Charlie: 11 orders
```

**设置**：
```bash
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
# Or use the CLI to add directly:
claude mcp add --transport stdio database -- npx @modelcontextprotocol/server-database
```

### 示例 3：多 MCP 工作流

**场景：每日报告生成**

```markdown
# Daily Report Workflow using Multiple MCPs

## Setup
1. GitHub MCP - fetch PR metrics
2. Database MCP - query sales data
3. Slack MCP - post report
4. Filesystem MCP - save report

## Workflow

### Step 1: Fetch GitHub Data
/mcp__github__list_prs completed:true last:7days

Output:
- Total PRs: 42
- Average merge time: 2.3 hours
- Review turnaround: 1.1 hours

### Step 2: Query Database
SELECT COUNT(*) as sales, SUM(amount) as revenue
FROM orders
WHERE created_at > NOW() - INTERVAL '1 day'

Output:
- Sales: 247
- Revenue: $12,450

### Step 3: Generate Report
Combine data into HTML report

### Step 4: Save to Filesystem
Write report.html to /reports/

### Step 5: Post to Slack
Send summary to #daily-reports channel

Final Output:
✅ Report generated and posted
📊 47 PRs merged this week
💰 $12,450 in daily sales
```

**设置**：
```bash
export GITHUB_TOKEN="your_github_token"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="your_slack_token"
# Add each MCP server via the CLI or configure them in .mcp.json
```

### 示例 4：文件系统 MCP 操作

**配置：**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**可用操作：**

| 操作 | 命令 | 用途 |
|-----------|---------|---------|
| 列出文件 | `ls ~/projects` | 显示目录内容 |
| 读取文件 | `cat src/main.ts` | 读取文件内容 |
| 写入文件 | `create docs/api.md` | 创建新文件 |
| 编辑文件 | `edit src/app.ts` | 修改文件 |
| 搜索 | `grep "async function"` | 在文件中搜索 |
| 删除 | `rm old-file.js` | 删除文件 |

**设置**：
```bash
# Use the CLI to add directly:
claude mcp add --transport stdio filesystem -- npx @modelcontextprotocol/server-filesystem /home/user/projects
```

## MCP vs Memory：决策矩阵

```mermaid
graph TD
    A["Need external data?"]
    A -->|No| B["Use Memory"]
    A -->|Yes| C["Does it change frequently?"]
    C -->|No/Rarely| B
    C -->|Yes/Often| D["Use MCP"]

    B -->|Stores| E["Preferences<br/>Context<br/>History"]
    D -->|Accesses| F["Live APIs<br/>Databases<br/>Services"]

    style A fill:#fff3e0,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#fff3e0,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#e8f5e9,stroke:#333,color:#333
    style F fill:#e8f5e9,stroke:#333,color:#333
```

## 请求/响应模式

```mermaid
sequenceDiagram
    participant App as Claude
    participant MCP as MCP Server
    participant DB as Database

    App->>MCP: Request: "SELECT * FROM users WHERE id=1"
    MCP->>DB: Execute query
    DB-->>MCP: Result set
    MCP-->>App: Return parsed data
    App->>App: Process result
    App->>App: Continue task

    Note over MCP,DB: Real-time access<br/>No caching
```

## 环境变量

将敏感凭据存储在环境变量中：

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxx"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="xoxb-xxxxxxxxxxxxx"
```

然后在 MCP 配置中引用它们：

```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}"
  }
}
```

## Claude 作为 MCP Server（`claude mcp serve`）

Claude Code 自身可以充当其他应用程序的 MCP server。这使得外部工具、编辑器和自动化系统能够通过标准 MCP 协议利用 Claude 的能力。

```bash
# Start Claude Code as an MCP server on stdio
claude mcp serve
```

其他应用程序随后可以像连接任何基于 stdio 的 MCP server 一样连接到此 server。例如，要在另一个 Claude Code 实例中将 Claude Code 添加为 MCP server：

```bash
claude mcp add --transport stdio claude-agent -- claude mcp serve
```

这对于构建一个 Claude 实例编排另一个 Claude 实例的多 agent 工作流很有用。

## 受管 MCP 配置（企业）

对于企业部署，IT 管理员可以通过 `managed-mcp.json` 配置文件强制实施 MCP server 策略。该文件提供对组织范围内哪些 MCP server 被允许或被阻止的独占控制。

**位置：**
- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux: `~/.config/ClaudeCode/managed-mcp.json`
- Windows: `%APPDATA%\ClaudeCode\managed-mcp.json`

**特性：**
- `allowedMcpServers` —— 允许的 server 白名单
- `deniedMcpServers` —— 禁止的 server 黑名单
- `allowAllClaudeAiMcps` —— 受管设置，允许在组织范围内加载 claude.ai 云端 MCP 连接器（v2.1.149+）
- 支持按 server 名称、命令和 URL 模式匹配
- 组织范围内的 MCP 策略在用户配置之前强制实施
- 防止未经授权的 server 连接

**示例配置：**

```json
{
  "allowedMcpServers": [
    {
      "serverName": "github",
      "serverUrl": "https://api.github.com/mcp"
    },
    {
      "serverName": "company-internal",
      "serverCommand": "company-mcp-server"
    }
  ],
  "deniedMcpServers": [
    {
      "serverName": "untrusted-*"
    },
    {
      "serverUrl": "http://*"
    }
  ]
}
```

> **注意：** 当 `allowedMcpServers` 和 `deniedMcpServers` 同时匹配某个 server 时，deny 规则优先。

## 插件提供的 MCP Servers

plugin 可以捆绑自己的 MCP server，使其在 plugin 安装时自动可用。plugin 提供的 MCP server 可以用两种方式定义：

1. **独立的 `.mcp.json`** —— 在 plugin 根目录中放置一个 `.mcp.json` 文件
2. **在 `plugin.json` 中内联** —— 直接在 plugin 清单内定义 MCP server

使用 `${CLAUDE_PLUGIN_ROOT}` 变量引用相对于 plugin 安装目录的路径：

```json
{
  "mcpServers": {
    "plugin-tools": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/dist/mcp-server.js"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

## Subagent 作用域 MCP

MCP server 可以使用 `mcpServers:` 键在 agent frontmatter 中内联定义，将它们的作用域限定到特定 subagent，而不是整个项目。当某个 agent 需要访问其他 agent 不需要的特定 MCP server 时，这很有用。

```yaml
---
mcpServers:
  my-tool:
    type: http
    url: https://my-tool.example.com/mcp
---

You are an agent with access to my-tool for specialized operations.
```

Subagent 作用域的 MCP server 仅在该 agent 的执行上下文中可用，不会与父 agent 或兄弟 agent 共享。

## MCP 输出限制

Claude Code 对 MCP 工具输出强制施加限制，以防止上下文溢出：

| 限制 | 阈值 | 行为 |
|-------|-----------|----------|
| **警告** | 10,000 tokens | 显示输出过大的警告 |
| **默认最大值** | 25,000 tokens | 超过此限制后输出被截断 |
| **磁盘持久化** | 50,000 字符 | 超过 50K 字符的工具结果会被持久化到磁盘 |

最大输出限制可通过 `MAX_MCP_OUTPUT_TOKENS` 环境变量配置：

```bash
# Increase the max output to 50,000 tokens
export MAX_MCP_OUTPUT_TOKENS=50000
```

## 用代码执行解决上下文膨胀

随着 MCP 应用规模扩大，连接到几十个 server、拥有成百上千个工具会带来一个重大挑战：**上下文膨胀**。这可以说是 MCP 在大规模场景下最大的问题，而 Anthropic 的工程团队提出了一个优雅的解决方案 —— 用代码执行代替直接的工具调用。

> **来源**：[Code Execution with MCP: Building More Efficient Agents](https://www.anthropic.com/engineering/code-execution-with-mcp) —— Anthropic 工程博客

### 问题：两个 token 浪费来源

**1. 工具定义占满上下文窗口**

大多数 MCP 客户端会预先加载所有工具定义。当连接到数千个工具时，模型必须在读取用户请求之前先处理数十万 token。

**2. 中间结果消耗额外 token**

每个中间工具结果都会经过模型的上下文。考虑将一份会议记录从 Google Drive 传输到 Salesforce —— 完整的记录会**两次**流经上下文：一次是读取它时，另一次是把它写入目标时。一场 2 小时的会议记录可能意味着额外的 50,000+ token。

```mermaid
graph LR
    A["Model"] -->|"Tool Call: getDocument"| B["MCP Server"]
    B -->|"Full transcript (50K tokens)"| A
    A -->|"Tool Call: updateRecord<br/>(re-sends full transcript)"| B
    B -->|"Confirmation"| A

    style A fill:#ffcdd2,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
```

### 方案：把 MCP 工具当作代码 API

与其将工具定义和结果传过上下文窗口，agent 可以**编写代码**，把 MCP 工具当作 API 来调用。代码在沙箱化的执行环境中运行，只有最终结果返回给模型。

```mermaid
graph LR
    A["Model"] -->|"Writes code"| B["Code Execution<br/>Environment"]
    B -->|"Calls tools directly"| C["MCP Servers"]
    C -->|"Data stays in<br/>execution env"| B
    B -->|"Only final result<br/>(minimal tokens)"| A

    style A fill:#c8e6c9,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#f3e5f5,stroke:#333,color:#333
```

#### 工作方式

MCP 工具以带类型的函数构成的文件树形式呈现：

```
servers/
├── google-drive/
│   ├── getDocument.ts
│   └── index.ts
├── salesforce/
│   ├── updateRecord.ts
│   └── index.ts
└── ...
```

每个工具文件包含一个带类型的包装器：

```typescript
// ./servers/google-drive/getDocument.ts
import { callMCPTool } from "../../../client.js";

interface GetDocumentInput {
  documentId: string;
}

interface GetDocumentResponse {
  content: string;
}

export async function getDocument(
  input: GetDocumentInput
): Promise<GetDocumentResponse> {
  return callMCPTool<GetDocumentResponse>(
    'google_drive__get_document', input
  );
}
```

随后 agent 编写代码来编排这些工具：

```typescript
import * as gdrive from './servers/google-drive';
import * as salesforce from './servers/salesforce';

// Data flows directly between tools — never through the model
const transcript = (
  await gdrive.getDocument({ documentId: 'abc123' })
).content;

await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

**结果：token 使用量从约 150,000 降至约 2,000 —— 减少 98.7%。**

### 关键优势

| 优势 | 说明 |
|---------|-------------|
| **渐进式披露（Progressive Disclosure）** | agent 浏览文件系统，只加载它需要的工具定义，而不是预先加载所有工具 |
| **上下文高效的结果** | 数据在执行环境中被过滤/转换后再返回给模型 |
| **强大的控制流** | 循环、条件和错误处理在代码中运行，无需往返模型 |
| **隐私保护** | 中间数据（PII、敏感记录）保留在执行环境中；绝不进入模型上下文 |
| **状态持久化** | agent 可以将中间结果保存到文件，并构建可复用的 skill 函数 |

#### 示例：过滤大型数据集

```typescript
// Without code execution — all 10,000 rows flow through context
// TOOL CALL: gdrive.getSheet(sheetId: 'abc123')
//   -> returns 10,000 rows in context

// With code execution — filter in the execution environment
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(
  row => row["Status"] === 'pending'
);
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5)); // Only 5 rows reach the model
```

#### 示例：无往返的循环

```typescript
// Poll for a deployment notification — runs entirely in code
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({
    channel: 'C123456'
  });
  found = messages.some(
    m => m.text.includes('deployment complete')
  );
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
console.log('Deployment notification received');
```

### 需要权衡的取舍

代码执行引入了自身的复杂性。运行 agent 生成的代码需要：

- 一个带适当资源限制的**安全沙箱化执行环境**
- 对所执行代码的**监控和日志记录**
- 与直接工具调用相比的额外**基础设施开销**

其带来的优势 —— 降低 token 成本、降低延迟、改善工具组合 —— 应与这些实现成本相权衡。对于只有少数几个 MCP server 的 agent，直接工具调用可能更简单。对于大规模 agent（几十个 server、几百个工具），代码执行是一项显著的改进。

### MCPorter：用于 MCP 工具组合的运行时

[MCPorter](https://github.com/steipete/mcporter) 是一个 TypeScript 运行时和 CLI 工具包，让调用 MCP server 无需样板代码即可实用 —— 并通过选择性工具暴露和带类型的包装器帮助减少上下文膨胀。

**它解决了什么：** 与其预先加载所有 MCP server 的所有工具定义，MCPorter 让你能够按需发现、检查和调用特定工具 —— 让你的上下文保持精简。

**关键特性：**

| 特性 | 说明 |
|---------|-------------|
| **零配置发现** | 从 Cursor、Claude、Codex 或本地配置自动发现 MCP server |
| **带类型的工具客户端** | `mcporter emit-ts` 生成 `.d.ts` 接口和可直接运行的包装器 |
| **可组合的 API** | `createServerProxy()` 将工具暴露为 camelCase 方法，并带有 `.text()`、`.json()`、`.markdown()` 辅助函数 |
| **CLI 生成** | `mcporter generate-cli` 将任意 MCP server 转换为独立 CLI，支持 `--include-tools` / `--exclude-tools` 过滤 |
| **参数隐藏** | 可选参数默认保持隐藏，减少 schema 冗余 |

**安装：**

```bash
npx mcporter list          # No install required — discover servers instantly
pnpm add mcporter          # Add to a project
brew install steipete/tap/mcporter  # macOS via Homebrew
```

**示例 —— 在 TypeScript 中组合工具：**

```typescript
import { createRuntime, createServerProxy } from "mcporter";

const runtime = await createRuntime();
const gdrive = createServerProxy(runtime, "google-drive");
const salesforce = createServerProxy(runtime, "salesforce");

// Data flows between tools without passing through the model context
const doc = await gdrive.getDocument({ documentId: "abc123" });
await salesforce.updateRecord({
  objectType: "SalesMeeting",
  recordId: "00Q5f000001abcXYZ",
  data: { Notes: doc.text() }
});
```

**示例 —— CLI 工具调用：**

```bash
# Call a specific tool directly
npx mcporter call linear.create_comment issueId:ENG-123 body:'Looks good!'

# List available servers and tools
npx mcporter list
```

MCPorter 与上文描述的代码执行方法相辅相成，它提供了将 MCP 工具作为带类型 API 调用所需的运行时基础设施 —— 让中间数据保持在模型上下文之外变得简单直接。

## 最佳实践

### 安全注意事项

#### 应该做的 ✅
- 对所有凭据使用环境变量
- 定期轮换令牌和 API 密钥（建议每月一次）
- 尽可能使用只读令牌
- 将 MCP server 的访问范围限制到最低所需
- 监控 MCP server 的使用和访问日志
- 在可用时对外部服务使用 OAuth
- 对 MCP 请求实施速率限制
- 在投入生产使用前测试 MCP 连接
- 记录所有活动的 MCP 连接
- 保持 MCP server 软件包更新

#### 不要做的 ❌
- 不要在配置文件中硬编码凭据
- 不要把令牌或密钥提交到 git
- 不要在团队聊天或邮件中分享令牌
- 不要在团队项目中使用个人令牌
- 不要授予不必要的权限
- 不要忽视认证错误
- 不要公开暴露 MCP 端点
- 不要以 root/admin 权限运行 MCP server
- 不要在日志中缓存敏感数据
- 不要禁用认证机制

### 配置最佳实践

1. **版本控制**：将 `.mcp.json` 保留在 git 中，但用环境变量存放密钥
2. **最小权限**：为每个 MCP server 授予所需的最低权限
3. **隔离**：在可能时让不同的 MCP server 在独立进程中运行
4. **监控**：记录所有 MCP 请求和错误以用于审计追踪
5. **测试**：在部署到生产之前测试所有 MCP 配置

### 性能技巧

- 在应用层缓存频繁访问的数据
- 使用具体的 MCP 查询以减少数据传输
- 监控 MCP 操作的响应时间
- 对外部 API 考虑速率限制
- 在执行多个操作时使用批处理

## 安装说明

### 前置条件
- 已安装 Node.js 和 npm
- 已安装 Claude Code CLI
- 外部服务的 API 令牌/凭据

### 分步设置

1. **使用 CLI 添加你的第一个 MCP server**（示例：GitHub）：
```bash
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

   或者在项目根目录创建一个 `.mcp.json` 文件：
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

2. **设置环境变量：**
```bash
export GITHUB_TOKEN="your_github_personal_access_token"
```

3. **测试连接：**
```bash
claude /mcp
```

4. **使用 MCP 工具：**
```bash
/mcp__github__list_prs
/mcp__github__create_issue "Title" "Description"
```

### 特定服务的安装

**GitHub MCP：**
```bash
npm install -g @modelcontextprotocol/server-github
```

**Database MCP：**
```bash
npm install -g @modelcontextprotocol/server-database
```

**Filesystem MCP：**
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**Slack MCP：**
```bash
npm install -g @modelcontextprotocol/server-slack
```

## 故障排查

### 找不到 MCP Server
```bash
# Verify MCP server is installed
npm list -g @modelcontextprotocol/server-github

# Install if missing
npm install -g @modelcontextprotocol/server-github
```

### 认证失败
```bash
# Verify environment variable is set
echo $GITHUB_TOKEN

# Re-export if needed
export GITHUB_TOKEN="your_token"

# Verify token has correct permissions
# Check GitHub token scopes at: https://github.com/settings/tokens
```

### 连接超时
- 检查网络连通性：`ping api.github.com`
- 验证 API 端点是否可访问
- 检查 API 的速率限制
- 尝试增大配置中的超时时间
- 检查防火墙或代理问题

### MCP Server 崩溃
- 检查 MCP server 日志：`~/.claude/logs/`
- 验证所有环境变量是否已设置
- 确保正确的文件权限
- 尝试重新安装 MCP server 软件包
- 检查同一端口上是否有冲突进程

## 相关概念

### Memory vs MCP
- **Memory**：存储持久、不变的数据（偏好、上下文、历史）
- **MCP**：访问实时、变化的数据（API、数据库、实时服务）

### 何时使用各自
- **使用 Memory** 用于：用户偏好、对话历史、学到的上下文
- **使用 MCP** 用于：当前 GitHub issue、实时数据库查询、实时数据

### 与其他 Claude 功能的集成
- 将 MCP 与 Memory 结合以获得丰富的上下文
- 在 prompt 中使用 MCP 工具以改善推理
- 利用多个 MCP 完成复杂工作流

## 更多资源

- [Official MCP Documentation](https://code.claude.com/docs/en/mcp)
- [MCP Protocol Specification](https://modelcontextprotocol.io/specification)
- [MCP GitHub Repository](https://github.com/modelcontextprotocol/servers)
- [Available MCP Servers](https://github.com/modelcontextprotocol/servers)
- [MCPorter](https://github.com/steipete/mcporter) —— 用于调用 MCP server 而无需样板代码的 TypeScript 运行时与 CLI
- [Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) —— Anthropic 关于解决上下文膨胀的工程博客
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude API Documentation](https://docs.anthropic.com)

---

**最后更新**：2026 年 6 月 24 日
**Claude Code 版本**：2.1.187
**来源**：
- https://code.claude.com/docs/en/mcp
- https://code.claude.com/docs/en/changelog
- https://github.com/anthropics/claude-code/releases/tag/v2.1.117
- https://github.com/anthropics/claude-code/releases/tag/v2.1.139
- https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- https://docs.anthropic.com/en/docs/claude-code/mcp
**兼容模型**：Claude Sonnet 4.6, Claude Opus 4.8, Claude Haiku 4.5

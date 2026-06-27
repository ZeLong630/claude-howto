<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../../resources/logos/claude-howto-logo.svg">
</picture>

# Hooks

Hook 是在 Claude Code 会话期间响应特定事件而执行的自动化脚本。它们可用于自动化、校验、权限管理和自定义工作流。

## 概览

Hook 是在 Claude Code 中特定事件发生时自动执行的自动化动作（shell 命令、HTTP webhook、LLM 提示、MCP 工具调用或 subagent 评估）。它们通过 stdin 接收 JSON 输入，并通过退出码和 JSON 输出传达结果。

**核心特性：**
- 事件驱动的自动化
- 基于 JSON 的输入/输出
- 支持 `command`、`http`、`mcp_tool`、`prompt` 和 `agent` 等 hook 类型
- 针对特定工具的 hook 模式匹配

## 配置

Hook 在 settings 文件中按特定结构配置：

- `~/.claude/settings.json` —— 用户设置（所有项目）
- `.claude/settings.json` —— 项目设置（可共享，纳入提交）
- `.claude/settings.local.json` —— 本地项目设置（不纳入提交）
- 受管策略（Managed policy）—— 组织级设置
- 插件的 `hooks/hooks.json` —— 插件作用域的 hook
- Skill/Agent frontmatter —— 组件生命周期 hook

### 基础配置结构

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

**关键字段：**

| 字段 | 说明 | 示例 |
|-------|-------------|---------|
| `matcher` | 用于匹配工具名的模式（区分大小写） | `"Write"`、`"Edit\|Write"`、`"*"` |
| `hooks` | hook 定义的数组 | `[{ "type": "command", ... }]` |
| `type` | hook 类型：`"command"`（bash）、`"prompt"`（LLM）、`"http"`（webhook）、`"mcp_tool"`（MCP 工具调用，v2.1.118+）或 `"agent"`（subagent） | `"command"` |
| `command` | 要执行的 shell 命令 | `"$CLAUDE_PROJECT_DIR/.claude/hooks/format.sh"` |
| `timeout` | 可选的超时秒数（默认 60） | `30` |
| `once` | 若为 `true`，则该 hook 每个会话只运行一次 | `true` |

### Matcher 模式

| 模式 | 说明 | 示例 |
|---------|-------------|---------|
| 精确字符串 | 匹配特定工具 | `"Write"` |
| 正则模式 | 匹配多个工具 | `"Edit\|Write"` |
| 通配符 | 匹配所有工具 | `"*"` 或 `""` |
| MCP 工具 | server 与 tool 模式 | `"mcp__memory__.*"` |

**InstructionsLoaded 的 matcher 取值：**

| Matcher 取值 | 说明 |
|---------------|-------------|
| `session_start` | 在会话启动时加载的指令 |
| `nested_traversal` | 在嵌套目录遍历期间加载的指令 |
| `path_glob_match` | 通过路径 glob 模式匹配加载的指令 |

### 用 `if` 条件做更细的筛选（工具参数路径）

`matcher` 字段按**工具名**（`"Write"`、`"Edit|Write"`、`"*"`）来选择 hook。若要根据工具的**参数**做更细的筛选——例如仅在某次编辑涉及 `src/` 时才运行 hook，或为读取机密文件设防——可在单个 hook 处理器上添加 `if` 条件。这与工具名 matcher 不同：`matcher` 决定*哪个工具*，`if` 决定*哪次调用*。

`if` 使用 [permission-rule 语法](https://code.claude.com/docs/en/permissions)（`ToolName(pattern)`），针对工具名**和**其参数共同进行求值。对 `Read`/`Edit`/`Write` 而言，路径模式遵循 gitignore 语义，并与权限规则使用相同的锚定方式：像 `.env` 这样的裸名称在任意深度都能匹配，`src/**` 相对于当前目录，`/src/**` 相对于项目根目录，`~/...` 相对于你的主目录，而 `//...` 是文件系统的绝对路径。

`if` 字段位于 **hook 处理器层级**——是 `type` 和 `command` 的同级字段，位于 `hooks` 数组内部——而不是位于 `matcher` 上：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "if": "Edit(src/**)",
            "command": "./hooks/lint-src.sh"
          }
        ]
      },
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "if": "Read(.env)",
            "command": "./hooks/block-secret-read.sh"
          }
        ]
      }
    ]
  }
}
```

有效的 `if` 模式示例：`Edit(src/**)`（`src/` 下的编辑）、`Read(~/.ssh/**)`（读取任意 SSH 密钥）、`Read(.env)`（当前目录及以下的任意 `.env`）、`Bash(git push *)`（仅 `git push` 子命令）。

## Hook 类型

Claude Code 支持五种 hook 类型：

### Command Hook

默认的 hook 类型。执行一条 shell 命令，并通过 JSON 的 stdin/stdout 及退出码进行通信。

```json
{
  "type": "command",
  "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py\"",
  "timeout": 60
}
```

#### Exec 形式（`args`）

> 在 v2.1.139 中加入。

command hook 可以不使用 shell 形式的 `"command": "..."`，而是通过 `execve()` 配合 `args` 数组直接启动一个二进制程序。由于没有 shell 解析，路径占位符永远不需要加引号，且该配置不受 shell 注入 bug 的影响。

```json
{
  "type": "command",
  "args": ["python3", "$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py", "--strict"],
  "timeout": 60
}
```

这两种形式是**互斥的**——同时设置了 `command` 和 `args` 的 hook 会在配置加载时被拒绝。当你需要管道、重定向、`&&` 链式调用或 shell 展开时使用 `command`；当你只是带参数调用单个二进制程序时使用 `args`。

### HTTP Hook

> 在 v2.1.63 中加入。

远程 webhook 端点，接收与 command hook 相同的 JSON 输入。HTTP hook 会把 JSON POST 到 URL 并接收一个 JSON 响应。启用沙箱时，HTTP hook 会经由沙箱路由。出于安全考虑，URL 中的环境变量插值需要一个显式的 `allowedEnvVars` 列表。

```json
{
  "hooks": {
    "PostToolUse": [{
      "type": "http",
      "url": "https://my-webhook.example.com/hook",
      "matcher": "Write"
    }]
  }
}
```

**关键属性：**
- `"type": "http"` —— 标识这是一个 HTTP hook
- `"url"` —— webhook 端点 URL
- 启用沙箱时经由沙箱路由
- URL 中任何环境变量插值都需要显式的 `allowedEnvVars` 列表

### Prompt Hook

由 LLM 评估的提示，其中 hook 内容是一段由 Claude 评估的提示。主要用于 `Stop` 和 `SubagentStop` 事件，以进行智能的任务完成检查。

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude completed all requested tasks.",
  "timeout": 30
}
```

LLM 会评估该提示并返回一个结构化决策（详见 [基于 Prompt 的 Hook](#基于-prompt-的-hook)）。

### MCP Tool Hook

> 在 v2.1.118 中加入。

`mcp_tool` 类型会直接调用一个已配置的 MCP 工具；其配置引用的是 MCP server 和工具名，而不是 shell 命令或 URL。当校验或响应逻辑已经存在于你已配置的某个 MCP server 中时，这很有用。

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "mcp_tool",
    "server": "my-mcp-server",
    "tool": "validate_edit"
  }]
}
```

**关键属性：**
- `"type": "mcp_tool"` —— 标识这是一个 MCP 工具 hook
- `"server"` —— 已配置的 MCP server 的名称
- `"tool"` —— 该 server 上要调用的工具名

hook 输入（工具名、工具输入、会话上下文）会作为该 MCP 工具的参数传入。配置 MCP server 请参见 [MCP server 设置](../05-mcp/README.md)。

### Agent Hook

基于 subagent 的校验 hook，会派生一个专用 agent 来评估条件或执行复杂检查。与 prompt hook（单轮 LLM 评估）不同，agent hook 可以使用工具并进行多步推理。

```json
{
  "type": "agent",
  "prompt": "Verify the code changes follow our architecture guidelines. Check the relevant design docs and compare.",
  "timeout": 120
}
```

**关键属性：**
- `"type": "agent"` —— 标识这是一个 agent hook
- `"prompt"` —— 给该 subagent 的任务描述
- 该 agent 可以使用工具（Read、Grep、Bash 等）来执行评估
- 返回与 prompt hook 类似的结构化决策

## Hook 事件

Claude Code 支持 **30 个 hook 事件**：

| 事件 | 触发时机 | Matcher 输入 | 能否阻止 | 常见用途 |
|-------|---------------|---------------|-----------|------------|
| **SessionStart** | 会话开始/恢复/清空/压缩 | startup/resume/clear/compact | 否 | 环境初始化 |
| **Setup** | 初始环境初始化（每会话一次） | （无） | 否 | 准备工具链、安装依赖 |
| **InstructionsLoaded** | CLAUDE.md 或规则文件加载之后 | （无） | 否 | 修改/过滤指令 |
| **UserPromptSubmit** | 用户提交提示 | （无） | 是 | 校验提示 |
| **UserPromptExpansion** | 用户提示被展开（例如 `@` 提及、slash command 解析完成） | （无） | 是 | 转换或检查展开后的提示 |
| **PreToolUse** | 工具执行之前 | 工具名 | 是（allow/deny/ask） | 校验、修改输入 |
| **PermissionRequest** | 显示权限对话框 | 工具名 | 是 | 自动批准/拒绝 |
| **PermissionDenied** | 用户拒绝某个权限提示 | 工具名 | 否 | 日志、分析、策略执行 |
| **PostToolUse** | 工具成功之后 | 工具名 | 否 | 添加上下文、反馈 |
| **PostToolUseFailure** | 工具执行失败 | 工具名 | 否 | 错误处理、日志 |
| **PostToolBatch** | 一批工具调用完成之后 | （无） | 否 | 汇总报告、批量校验 |
| **Notification** | 发送通知 | 通知类型 | 否 | 自定义通知 |
| **MessageDisplay** | 在显示助手消息文本期间 | （无） | 否 | 转换或隐藏所显示的消息文本（v2.1.152） |
| **SubagentStart** | 派生 subagent | agent 类型名 | 否 | subagent 初始化 |
| **SubagentStop** | subagent 结束 | agent 类型名 | 是 | subagent 校验 |
| **Stop** | Claude 完成响应 | （无） | 是 | 任务完成检查 |
| **StopFailure** | API 错误结束本轮 | （无） | 否 | 错误恢复、日志 |
| **TeammateIdle** | agent 团队中的队友空闲 | （无） | 是 | 队友协调 |
| **TaskCompleted** | 任务被标记为完成 | （无） | 是 | 任务后续动作 |
| **TaskCreated** | 通过 TaskCreate 创建任务 | （无） | 否 | 任务跟踪、日志 |
| **ConfigChange** | 配置文件变更 | （无） | 是（策略除外） | 响应配置更新 |
| **CwdChanged** | 工作目录变更 | （无） | 否 | 目录专属初始化 |
| **FileChanged** | 被监视的文件变更 | （无） | 否 | 文件监控、重新构建 |
| **PreCompact** | 上下文压缩之前 | manual/auto | 否 | 压缩前动作 |
| **PostCompact** | 压缩完成之后 | （无） | 否 | 压缩后动作 |
| **WorktreeCreate** | 正在创建 worktree | （无） | 是（返回路径） | worktree 初始化 |
| **WorktreeRemove** | 正在移除 worktree | （无） | 否 | worktree 清理 |
| **Elicitation** | MCP server 请求用户输入 | （无） | 是 | 输入校验 |
| **ElicitationResult** | 用户响应 elicitation | （无） | 是 | 响应处理 |
| **SessionEnd** | 会话终止 | （无） | 否 | 清理、最终日志 |

> **PostToolUse 时长（v2.1.119）：** `PostToolUse` 和 `PostToolUseFailure` 的 hook 输入现在包含 `duration_ms` —— 详见 [PostToolUse](#posttooluse) 小节。

### PreToolUse

在 Claude 创建好工具参数之后、处理之前运行。用它来校验或修改工具输入。

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py"
          }
        ]
      }
    ]
  }
}
```

**常见 matcher：** `Task`、`Bash`、`Glob`、`Grep`、`Read`、`Edit`、`Write`、`WebFetch`、`WebSearch`

**输出控制：**
- `permissionDecision`：`"allow"`、`"deny"` 或 `"ask"`
- `permissionDecisionReason`：决策的解释说明
- `updatedInput`：修改后的工具输入参数

### PostToolUse

在工具完成后立即运行。用于校验、记录日志，或把上下文反馈给 Claude。

**配置：**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `"block"` 决策会带着反馈提示 Claude
- `additionalContext`：为 Claude 添加的上下文

**额外的输入字段（v2.1.119）：**

| 字段 | 类型 | 说明 |
|-------|------|-------------|
| `duration_ms` | number | 工具执行时间（毫秒）。不含权限提示和 PreToolUse hook 执行所花的时间。在 `PostToolUse` 和 `PostToolUseFailure` hook 上均可用。 |

#### 可恢复的阻止（`continueOnBlock`，v2.1.139）

默认情况下，返回 `"decision": "block"` 的 `PostToolUse` hook 会中止当前这一轮。在 hook 上设置 `"continueOnBlock": true`，则会把拒绝作为 `tool_result` 反馈回 Claude，让模型读到反馈并据此重试或调整。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/policy-check.py",
            "continueOnBlock": true
          }
        ]
      }
    ]
  }
}
```

当 hook 的 `reason` 是 Claude 能据此行动的内容时使用它（例如“此文件只读；请写到别处”）；当某次阻止必须彻底中止本轮时，则不要开启它。

### UserPromptSubmit

在用户提交提示时、Claude 处理它之前运行。

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `decision`：`"block"` 以阻止处理
- `reason`：被阻止时的解释说明
- `additionalContext`：添加到提示中的上下文

### Stop 和 SubagentStop

在 Claude 完成响应（Stop）或 subagent 完成（SubagentStop）时运行。支持基于 prompt 的评估，以进行智能的任务完成检查。

**额外的输入字段：** `Stop` 和 `SubagentStop` hook 都会在其 JSON 输入中收到一个 `last_assistant_message` 字段，包含 Claude 或 subagent 停止前的最后一条消息。这对评估任务完成情况很有用。

**配置：**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude completed all requested tasks.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

> **连续阻止的安全上限（v2.1.143）**：如果一个 `Stop` hook 对同一轮**连续 8 次**返回 `"decision": "block"`（或设置 `continue: false`），Claude Code 会短路该循环并以警告结束会话。可用环境变量 `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP=<integer>` 覆盖该阈值（设为 `0` 可完全禁用上限）。这可以防止一个有 bug 的 Stop hook 让会话永远循环。

**返回字段（v2.1.163）：** `Stop` 或 `SubagentStop` hook 可以返回 `hookSpecificOutput.additionalContext`，以便给 Claude 反馈并**在不显示错误标签的情况下继续本轮**。此前，从 Stop hook 影响模型很别扭；现在 hook 可以干净地注入上下文，从而避免旧反馈路径（例如 `"decision": "block"`）的错误标签行为。

```json
{
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "additionalContext": "Reminder: run the test suite before declaring done."
  }
}
```

### SubagentStart

在 subagent 开始执行时运行。matcher 输入是 agent 类型名，使 hook 能够针对特定的 subagent 类型。

**配置：**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "code-review",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/subagent-init.sh"
          }
        ]
      }
    ]
  }
}
```

### SessionStart

在会话开始或恢复时运行。可以持久化环境变量。

**Matcher：** `startup`、`resume`、`clear`、`compact`

**特殊功能：** 使用 `CLAUDE_ENV_FILE` 来持久化环境变量（在 `CwdChanged` 和 `FileChanged` hook 中也可用）：

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=development' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

**会话作用域的输出（v2.1.152）：** `SessionStart` hook 可以返回 JSON 来重新扫描 skills 并设置会话标题：

```json
{
  "reloadSkills": true,
  "hookSpecificOutput": {
    "sessionTitle": "Payments migration"
  }
}
```

顶层的 `reloadSkills: true` 会在同一会话中触发 skill 重新扫描（与 `/reload-skills` 命令的动作相同），使该 hook 刚安装的 skills 立即可用。`hookSpecificOutput.sessionTitle` 会在启动和恢复时设置会话的显示标题。

### SessionEnd

在会话结束时运行，以执行清理或最终日志记录。无法阻止终止。

**Reason 字段取值：**
- `clear` —— 用户清空了会话
- `logout` —— 用户登出
- `prompt_input_exit` —— 用户通过提示输入退出
- `other` —— 其他原因

**配置：**
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-cleanup.sh\""
          }
        ]
      }
    ]
  }
}
```

### Notification 事件

通知事件更新后的 matcher：
- `permission_prompt` —— 权限请求通知
- `idle_prompt` —— 空闲状态通知
- `auth_success` —— 认证成功
- `elicitation_dialog` —— 向用户显示的对话框

## 组件作用域的 Hook

Hook 可以在其 frontmatter 中附加到特定组件（skills、agents、commands）：

**在 SKILL.md、agent.md 或 command.md 中：**

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check.sh"
          once: true  # Only run once per session
---
```

**组件 hook 支持的事件：** `PreToolUse`、`PostToolUse`、`Stop`

这使你可以直接在使用 hook 的组件中定义 hook，把相关代码放在一起。

### Subagent Frontmatter 中的 Hook

当在某个 subagent 的 frontmatter 中定义了 `Stop` hook 时，它会被自动转换为作用于该 subagent 的 `SubagentStop` hook。这确保该 stop hook 仅在那个特定 subagent 完成时触发，而不是在主会话停止时触发。

```yaml
---
name: code-review-agent
description: Automated code review subagent
hooks:
  Stop:
    - hooks:
        - type: prompt
          prompt: "Verify the code review is thorough and complete."
  # The above Stop hook auto-converts to SubagentStop for this subagent
---
```

## PermissionRequest 事件

以自定义输出格式处理权限请求：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow|deny",
      "updatedInput": {},
      "message": "Custom message",
      "interrupt": false
    }
  }
}
```

## Hook 的输入与输出

### JSON 输入（通过 stdin）

所有 hook 都通过 stdin 接收 JSON 输入：

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  },
  "tool_use_id": "toolu_01ABC123...",
  "agent_id": "agent-abc123",
  "agent_type": "main",
  "worktree": "/path/to/worktree",
  "effort": { "level": "medium" }
}
```

**常见字段：**

| 字段 | 说明 |
|-------|-------------|
| `session_id` | 唯一的会话标识符 |
| `transcript_path` | 对话 transcript 文件的路径 |
| `cwd` | 当前工作目录 |
| `hook_event_name` | 触发该 hook 的事件名 |
| `agent_id` | 运行此 hook 的 agent 的标识符 |
| `agent_type` | agent 类型（`"main"`、subagent 类型名等） |
| `worktree` | 若 agent 运行在某个 git worktree 中，则为该 worktree 的路径 |
| `effort.level` | （v2.1.133+）当前生效的 effort 级别：`low`、`medium`、`high`、`xhigh` 或 `max` |

### 退出码

| 退出码 | 含义 | 行为 |
|-----------|---------|----------|
| **0** | 成功 | 继续，解析 JSON stdout |
| **2** | 阻止性错误 | 阻止操作，stderr 作为错误显示 |
| **其他** | 非阻止性错误 | 继续，stderr 在 verbose 模式下显示 |

### JSON 输出（stdout，退出码 0）

```json
{
  "continue": true,
  "stopReason": "Optional message if stopping",
  "suppressOutput": false,
  "systemMessage": "Optional warning message",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "File is in allowed directory",
    "updatedInput": {
      "file_path": "/modified/path.js"
    }
  }
}
```

> **作用范围（v2.1.121+）：** `hookSpecificOutput.updatedToolOutput` 现在对**所有**工具都生效，而不仅仅是 MCP 工具。作用于 `Bash`、`Edit`、`Read` 等工具上的 `PostToolUse` hook 可以在 Claude 看到工具输出之前重写它——可用于脱敏机密、规范化 diff，或过滤嘈杂的命令输出。示例（从 `Bash` 输出中去除 ANSI 颜色码）：
>
> ```json
> {
>   "hookSpecificOutput": {
>     "hookEventName": "PostToolUse",
>     "updatedToolOutput": "<plain-text output with ANSI escapes removed>"
>   }
> }
> ```

#### `terminalSequence`（v2.1.141）

hook 可以通过在 JSON 输出中设置 `terminalSequence` 来发出原始的 OSC（operating system command）转义序列。当 hook 返回时，宿主会把该序列写入其控制终端——这对桌面通知、窗口标题更新和终端响铃很有用，且无需你自己拥有 TTY。

| 字段 | 类型 | 说明 |
|-------|------|-------------|
| `terminalSequence` | string | 原始转义序列（通常是 OSC 9 / OSC 0 / OSC 777）。会原样写入宿主终端。 |

示例——当一个长任务完成时，触发一条 OSC 9 桌面通知：

```json
{
  "terminalSequence": "]9;Task complete"
}
```

把它配置在 `Stop` hook 上，这样当 Claude 完成一轮时通知就会触发。序列支持与终端相关；Kitty/iTerm2/Windows Terminal 支持 OSC 9。

## 环境变量

| 变量 | 可用范围 | 说明 |
|----------|-------------|-------------|
| `CLAUDE_PROJECT_DIR` | 所有 hook | 项目根目录的绝对路径 |
| `CLAUDE_ENV_FILE` | SessionStart、CwdChanged、FileChanged | 用于持久化环境变量的文件路径 |
| `CLAUDE_CODE_REMOTE` | 所有 hook | 在远程环境中运行时为 `"true"` |
| `${CLAUDE_PLUGIN_ROOT}` | 插件 hook | 插件目录的路径 |
| `${CLAUDE_PLUGIN_DATA}` | 插件 hook | 插件数据目录的路径 |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hook | SessionEnd hook 的可配置超时（毫秒，覆盖默认值） |
| `CLAUDE_CODE_SESSION_ID` | Bash 工具的子进程（v2.1.132+） | 会话 UUID；与 hook 输入 JSON 中的 `session_id` 字段相匹配。用于把 bash 日志与 hook 遥测关联起来。 |
| `CLAUDE_EFFORT` | Bash 工具的子进程（v2.1.133+） | 当前生效的 effort 级别（`low`/`medium`/`high`/`xhigh`/`max`）；与 hook 输入 JSON 中的 `effort.level` 相匹配。 |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | 进程范围（v2.1.143+） | 会话以警告结束前允许的最大连续 Stop-hook 阻止次数（默认 `8`）。设为 `0` 可禁用上限。 |

## 基于 Prompt 的 Hook

对于 `Stop` 和 `SubagentStop` 事件，你可以使用基于 LLM 的评估：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if all tasks are complete. Return your decision.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**LLM 响应 Schema：**
```json
{
  "decision": "approve",
  "reason": "All tasks completed successfully",
  "continue": false,
  "stopReason": "Task complete"
}
```

## 示例

### 示例 1：Bash 命令校验器（PreToolUse）

**文件：** `.claude/hooks/validate-bash.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"\brm\s+-rf\s+/", "Blocking dangerous rm -rf / command"),
    (r"\bsudo\s+rm", "Blocking sudo rm command"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name != "Bash":
        sys.exit(0)

    command = input_data.get("tool_input", {}).get("command", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, command):
            print(message, file=sys.stderr)
            sys.exit(2)  # Exit 2 = blocking error

    sys.exit(0)

if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\""
          }
        ]
      }
    ]
  }
}
```

### 示例 2：安全扫描器（PostToolUse）

**文件：** `.claude/hooks/security-scan.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

SECRET_PATTERNS = [
    (r"password\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded password"),
    (r"api[_-]?key\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded API key"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name not in ["Write", "Edit"]:
        sys.exit(0)

    tool_input = input_data.get("tool_input", {})
    content = tool_input.get("content", "") or tool_input.get("new_string", "")
    file_path = tool_input.get("file_path", "")

    warnings = []
    for pattern, message in SECRET_PATTERNS:
        if re.search(pattern, content, re.IGNORECASE):
            warnings.append(message)

    if warnings:
        output = {
            "hookSpecificOutput": {
                "hookEventName": "PostToolUse",
                "additionalContext": f"Security warnings for {file_path}: " + "; ".join(warnings)
            }
        }
        print(json.dumps(output))

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 3：自动格式化代码（PostToolUse）

**文件：** `.claude/hooks/format-code.sh`

```bash
#!/bin/bash

# Read JSON from stdin
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_name', ''))")
FILE_PATH=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_input', {}).get('file_path', ''))")

if [ "$TOOL_NAME" != "Write" ] && [ "$TOOL_NAME" != "Edit" ]; then
    exit 0
fi

# Format based on file extension
case "$FILE_PATH" in
    *.js|*.jsx|*.ts|*.tsx|*.json)
        command -v prettier &>/dev/null && prettier --write "$FILE_PATH" 2>/dev/null
        ;;
    *.py)
        command -v black &>/dev/null && black "$FILE_PATH" 2>/dev/null
        ;;
    *.go)
        command -v gofmt &>/dev/null && gofmt -w "$FILE_PATH" 2>/dev/null
        ;;
esac

exit 0
```

### 示例 4：提示校验器（UserPromptSubmit）

**文件：** `.claude/hooks/validate-prompt.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"delete\s+(all\s+)?database", "Dangerous: database deletion"),
    (r"rm\s+-rf\s+/", "Dangerous: root deletion"),
]

def main():
    input_data = json.load(sys.stdin)
    prompt = input_data.get("user_prompt", "") or input_data.get("prompt", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, prompt, re.IGNORECASE):
            output = {
                "decision": "block",
                "reason": f"Blocked: {message}"
            }
            print(json.dumps(output))
            sys.exit(0)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 5：智能 Stop Hook（基于 Prompt）

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if Claude completed all requested tasks. Check: 1) Were all files created/modified? 2) Were there unresolved errors? If incomplete, explain what's missing.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 示例 6：上下文用量跟踪器（Hook 成对使用）

通过把 `UserPromptSubmit`（消息前）和 `Stop`（响应后）hook 配合使用，跟踪每个请求的 token 消耗。

**文件：** `.claude/hooks/context-tracker.py`

```python
#!/usr/bin/env python3
"""
Context Usage Tracker - Tracks token consumption per request.

Uses UserPromptSubmit as "pre-message" hook and Stop as "post-response" hook
to calculate the delta in token usage for each request.

Token Counting Methods:
1. Character estimation (default): ~4 chars per token, no dependencies
2. tiktoken (optional): More accurate (~90-95%), requires: pip install tiktoken
"""
import json
import os
import sys
import tempfile

# Configuration
CONTEXT_LIMIT = 128000  # Claude's context window (adjust for your model)
USE_TIKTOKEN = False    # Set True if tiktoken is installed for better accuracy


def get_state_file(session_id: str) -> str:
    """Get temp file path for storing pre-message token count, isolated by session."""
    return os.path.join(tempfile.gettempdir(), f"claude-context-{session_id}.json")


def count_tokens(text: str) -> int:
    """
    Count tokens in text.

    Uses tiktoken with p50k_base encoding if available (~90-95% accuracy),
    otherwise falls back to character estimation (~80-90% accuracy).
    """
    if USE_TIKTOKEN:
        try:
            import tiktoken
            enc = tiktoken.get_encoding("p50k_base")
            return len(enc.encode(text))
        except ImportError:
            pass  # Fall back to estimation

    # Character-based estimation: ~4 characters per token for English
    return len(text) // 4


def read_transcript(transcript_path: str) -> str:
    """Read and concatenate all content from transcript file."""
    if not transcript_path or not os.path.exists(transcript_path):
        return ""

    content = []
    with open(transcript_path, "r") as f:
        for line in f:
            try:
                entry = json.loads(line.strip())
                # Extract text content from various message formats
                if "message" in entry:
                    msg = entry["message"]
                    if isinstance(msg.get("content"), str):
                        content.append(msg["content"])
                    elif isinstance(msg.get("content"), list):
                        for block in msg["content"]:
                            if isinstance(block, dict) and block.get("type") == "text":
                                content.append(block.get("text", ""))
            except json.JSONDecodeError:
                continue

    return "\n".join(content)


def handle_user_prompt_submit(data: dict) -> None:
    """Pre-message hook: Save current token count before request."""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # Save to temp file for later comparison
    state_file = get_state_file(session_id)
    with open(state_file, "w") as f:
        json.dump({"pre_tokens": current_tokens}, f)


def handle_stop(data: dict) -> None:
    """Post-response hook: Calculate and report token delta."""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # Load pre-message count
    state_file = get_state_file(session_id)
    pre_tokens = 0
    if os.path.exists(state_file):
        try:
            with open(state_file, "r") as f:
                state = json.load(f)
                pre_tokens = state.get("pre_tokens", 0)
        except (json.JSONDecodeError, IOError):
            pass

    # Calculate delta
    delta_tokens = current_tokens - pre_tokens
    remaining = CONTEXT_LIMIT - current_tokens
    percentage = (current_tokens / CONTEXT_LIMIT) * 100

    # Report usage
    method = "tiktoken" if USE_TIKTOKEN else "estimated"
    print(f"Context ({method}): ~{current_tokens:,} tokens ({percentage:.1f}% used, ~{remaining:,} remaining)", file=sys.stderr)
    if delta_tokens > 0:
        print(f"This request: ~{delta_tokens:,} tokens", file=sys.stderr)


def main():
    data = json.load(sys.stdin)
    event = data.get("hook_event_name", "")

    if event == "UserPromptSubmit":
        handle_user_prompt_submit(data)
    elif event == "Stop":
        handle_stop(data)

    sys.exit(0)


if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ]
  }
}
```

**工作原理：**
1. `UserPromptSubmit` 在你的提示被处理之前触发 —— 保存当前 token 计数
2. `Stop` 在 Claude 响应之后触发 —— 计算增量并报告用量
3. 每个会话通过临时文件名中的 `session_id` 进行隔离

**Token 计数方法：**

| 方法 | 准确度 | 依赖 | 速度 |
|--------|----------|--------------|-------|
| 字符估算 | ~80-90% | 无 | <1ms |
| tiktoken (p50k_base) | ~90-95% | `pip install tiktoken` | <10ms |

> **注意：** Anthropic 尚未发布官方的离线 tokenizer。两种方法都是近似值。transcript 包含用户提示、Claude 的响应和工具输出，但**不**包含系统提示或内部上下文。

### 示例 7：预置 Auto-Mode 权限（一次性安装脚本）

一个一次性安装脚本，向 `~/.claude/settings.json` 预置约 67 条安全的权限规则，等价于 Claude Code 的 auto-mode 基线——无需任何 hook，也不会记住未来的选择。运行一次即可；可安全重复运行（会跳过已存在的规则）。

**文件：** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply
python3 09-advanced-features/setup-auto-mode-permissions.py
```

**会添加哪些内容：**

| 类别 | 示例 |
|----------|---------|
| 内置工具 | `Read(*)`、`Edit(*)`、`Write(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)` |
| Git 读取 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)` |
| Git 写入（本地） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建与测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常用 shell | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

**有意排除的内容**（此脚本绝不会添加）：
- `rm -rf`、`sudo`、强制 push、`git reset --hard`
- `DROP TABLE`、`kubectl delete`、`terraform destroy`
- `npm publish`、`curl | bash`、生产环境部署

### 示例 8：学习进度记录器（SessionEnd）

在每次 Claude Code 会话结束时记录你学习了哪些模块。进度存储在
`~/.claude-howto-progress.json` 中——位于仓库之外，因此它能在 `git pull` 后保留而
不会被覆盖。

**为什么用 `SessionEnd` 而不是 `Stop`？**
`Stop` 在*每一次* Claude 响应之后触发。`SessionEnd` 在会话终止时触发一次——
这正是会话结束时记一笔日记所需要的。

**为什么用 `/dev/tty` 做输入？**
hook 脚本通过 `stdin` 接收 hook 的 JSON 负载，因此交互式 `read` 必须直接使用
`/dev/tty` 才能连到终端。

**文件：** `06-hooks/session-end.sh`

```bash
#!/usr/bin/env bash
# SessionEnd hook: prompts for modules worked on, then appends a session record
# to ~/.claude-howto-progress.json for persistent learning progress tracking.

PROGRESS_FILE="$HOME/.claude-howto-progress.json"

# Guard: only run inside this repo
if [[ "$CLAUDE_PROJECT_DIR" != *"claude-howto"* ]] && [[ "$PWD" != *"claude-howto"* ]]; then
  exit 0
fi

if [ ! -f "$PROGRESS_FILE" ]; then
  echo '{"sessions":[]}' > "$PROGRESS_FILE"
fi

DATE=$(date +"%Y-%m-%d")
TIME=$(date +"%H:%M")

echo ""
echo " Which modules did you work on? (e.g. 06,07 or press Enter to skip)"
echo " 01=Slash  02=Memory  03=Skills  04=Subagents  05=MCP"
echo " 06=Hooks  07=Plugins 08=Checkpoints 09=Advanced 10=CLI"
printf " > "
read -r INPUT </dev/tty

if [ -z "$INPUT" ] || [ "$INPUT" = "skip" ]; then
  exit 0
fi

MODULES_JSON=$(echo "$INPUT" | tr ',' '\n' | tr -d ' ' | while read -r m; do
  case "$m" in
    01) echo '"01-slash-commands"' ;;
    02) echo '"02-memory"' ;;
    03) echo '"03-skills"' ;;
    04) echo '"04-subagents"' ;;
    05) echo '"05-mcp"' ;;
    06) echo '"06-hooks"' ;;
    07) echo '"07-plugins"' ;;
    08) echo '"08-checkpoints"' ;;
    09) echo '"09-advanced-features"' ;;
    10) echo '"10-cli"' ;;
    *)  echo "\"$m\"" ;;
  esac
done | paste -sd ',' -)

printf " Notes? (optional, press Enter to skip): "
read -r NOTES </dev/tty

# Pass NOTES as a separate argument so Python handles JSON escaping —
# avoids broken JSON when notes contain quotes or backslashes.
python3 - "$PROGRESS_FILE" "$DATE" "$TIME" "$MODULES_JSON" "$NOTES" <<'PYEOF'
import sys, json

path, date, time_str, modules_raw, notes = sys.argv[1], sys.argv[2], sys.argv[3], sys.argv[4], sys.argv[5]

new_session = {
    "date": date,
    "time": time_str,
    "modules": json.loads(f"[{modules_raw}]") if modules_raw else [],
    "notes": notes,
}

with open(path, 'r') as f:
    data = json.load(f)

data.setdefault('sessions', []).append(new_session)

with open(path, 'w') as f:
    json.dump(data, f, indent=2)
PYEOF

echo " Saved to $PROGRESS_FILE"
```

**安装** —— 把该脚本复制到项目的 hook 目录，使 `settings.json` 中的路径能够解析：

```bash
mkdir -p .claude/hooks
cp 06-hooks/session-end.sh .claude/hooks/
chmod +x .claude/hooks/session-end.sh
```

**配置**（在 `.claude/settings.json` 中）：

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-end.sh\""
          }
        ]
      }
    ]
  }
}
```

**输出 —— `~/.claude-howto-progress.json`：**

```json
{
  "sessions": [
    {
      "date": "2026-04-18",
      "time": "14:32",
      "modules": ["06-hooks", "07-plugins"],
      "notes": "Installed first hook, tried pre-commit example"
    }
  ]
}
```

**所展示的关键模式：**

| 模式 | 为何重要 |
|---------|----------------|
| `SessionEnd` 事件 | 退出时仅触发一次 —— 不像 `Stop` 那样每次响应都触发 |
| `read -r INPUT </dev/tty` | hook 占用 `stdin`（JSON 负载）；用 `/dev/tty` 获取用户输入 |
| `$CLAUDE_PROJECT_DIR` | 可移植路径 —— 切勿硬编码 `/Users/yourname/...` |
| 顶部的 guard 子句 | 若被全局安装，可防止 hook 在不相关的项目中运行 |
| 存储在仓库之外 | `~/` 路径可在 `git pull` 后保留而不会覆盖你的数据 |

**配套：可视化进度跟踪器**

如需覆盖全部 10 个模块、基于复选框的完整 UI，请在浏览器中打开内置的跟踪器：

```bash
open local-progress/index.html
```

进度存储在浏览器的 `localStorage` 中（绝不会写入仓库内的磁盘）。
使用 **Export** 按钮把快照保存为 JSON，使用 **Import** 恢复它。

## 插件 Hook

插件可以在其 `hooks/hooks.json` 文件中包含 hook：

**文件：** `plugins/hooks/hooks.json`

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

**插件 Hook 中的环境变量：**
- `${CLAUDE_PLUGIN_ROOT}` —— 插件目录的路径
- `${CLAUDE_PLUGIN_DATA}` —— 插件数据目录的路径

这使插件可以包含自定义的校验和自动化 hook。

## MCP Tool Hook

MCP 工具遵循 `mcp__<server>__<tool>` 模式：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"systemMessage\": \"Memory operation logged\"}'"
          }
        ]
      }
    ]
  }
}
```

## 安全注意事项

### 免责声明

**风险自担（USE AT YOUR OWN RISK）**：Hook 会执行任意 shell 命令。你需独自负责：
- 你所配置的命令
- 文件访问/修改权限
- 潜在的数据丢失或系统损坏
- 在投入生产前于安全环境中测试 hook

### 安全提示

- **需要工作区信任：** `statusLine` 和 `fileSuggestion` 这两类 hook 输出命令现在需要先获得工作区信任的接受，才会生效。
- **状态行终端尺寸（v2.1.153）：** 状态行命令脚本现在会收到 `COLUMNS` 和 `LINES` 环境变量，因此脚本可以根据终端的宽/高来调整其输出（例如 `[ "$COLUMNS" -lt 80 ] && short_output`）。
- **HTTP hook 与环境变量：** HTTP hook 要在 URL 中使用环境变量插值，需要一个显式的 `allowedEnvVars` 列表。这可防止把敏感环境变量意外泄露给远程端点。
- **受管设置层级：** `disableAllHooks` 设置现在会遵循受管设置的层级，这意味着组织级设置可以强制禁用 hook，且个体用户无法覆盖。
- **PowerShell 自动批准（v2.1.119）：** PowerShell 工具命令可以在权限模式下被自动批准，与 Bash 一致。这为在 Claude Code 中使用 PowerShell 后端 shell 工具的 Windows 用户带来了对等能力。
- **修复 Bash 裸环境变量自动批准（v2.1.145）：** 在 v2.1.145 之前，形如 `FOO=bar somecommand` 的 Bash 命令（一个裸变量赋值与一个未列入白名单的命令同行）在白名单上仅有 `FOO=bar` 本身时也可能被自动批准。v2.1.145 修复了这一点——此类命令现在会触发权限提示。依赖隐式放行的脚本将开始出现提示；请通过覆盖完整命令（而不仅是变量赋值）的 `Bash(...)` 权限规则，显式地重新放行它们。

### 最佳实践

| 应当 | 不应当 |
|-----|-------|
| 校验并清洗所有输入 | 盲目信任输入数据 |
| 给 shell 变量加引号：`"$VAR"` | 使用未加引号的：`$VAR` |
| 阻止路径穿越（`..`） | 允许任意路径 |
| 用 `$CLAUDE_PROJECT_DIR` 构造绝对路径 | 硬编码路径 |
| 跳过敏感文件（`.env`、`.git/`、密钥） | 处理所有文件 |
| 先隔离测试 hook | 部署未经测试的 hook |
| 为 HTTP hook 使用显式的 `allowedEnvVars` | 把所有环境变量暴露给 webhook |

## 调试

### 启用调试模式

带调试标志运行 Claude，以获取详细的 hook 日志：

```bash
claude --debug
```

### Verbose 模式

在 Claude Code 中使用 `Ctrl+O` 启用 verbose 模式，查看 hook 的执行进度。

### 独立测试 Hook

```bash
# Test with sample JSON input
echo '{"tool_name": "Bash", "tool_input": {"command": "ls -la"}}' | python3 .claude/hooks/validate-bash.py

# Check exit code
echo $?
```

## 完整配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/format-code.sh\"",
            "timeout": 30
          },
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-init.sh\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify all tasks are complete before stopping.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## Hook 执行细节

| 方面 | 行为 |
|--------|----------|
| **超时** | 默认 60 秒，可按命令配置 |
| **并行化** | 所有匹配的 hook 并行运行 |
| **去重** | 相同的 hook 命令会被去重 |
| **环境** | 在当前目录中运行，使用 Claude Code 的环境 |

## 故障排查

### Hook 未执行
- 确认 JSON 配置语法正确
- 检查 matcher 模式是否匹配工具名
- 确保脚本存在且可执行：`chmod +x script.sh`
- 运行 `claude --debug` 查看 hook 执行日志
- 确认 hook 从 stdin 读取 JSON（而非命令参数）

### Hook 意外阻止
- 用示例 JSON 测试 hook：`echo '{"tool_name": "Write", ...}' | ./hook.py`
- 检查退出码：allow 应为 0，block 应为 2
- 检查 stderr 输出（在退出码 2 时显示）

### JSON 解析错误
- 始终从 stdin 读取，而非命令参数
- 使用恰当的 JSON 解析（而非字符串操作）
- 优雅地处理缺失字段

## 安装说明

### 第 1 步：创建 Hooks 目录
```bash
mkdir -p ~/.claude/hooks
```

### 第 2 步：复制示例 Hook
```bash
cp 06-hooks/*.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/*.sh
```

### 第 3 步：在 Settings 中配置
编辑 `~/.claude/settings.json` 或 `.claude/settings.json`，写入上文所示的 hook 配置。

## 相关概念

- **[Checkpoints and Rewind](../08-checkpoints/README.md)** —— 保存并恢复对话状态
- **[Slash Commands](../01-slash-commands/README.md)** —— 创建自定义 slash command
- **[Skills](../03-skills/README.md)** —— 可复用的自主能力
- **[Subagents](../04-subagents/README.md)** —— 委派的任务执行
- **[Plugins](../07-plugins/README.md)** —— 打包的扩展包
- **[Advanced Features](../09-advanced-features/README.md)** —— 探索 Claude Code 的高级能力

## 更多资源

- **[Official Hooks Documentation](https://code.claude.com/docs/en/hooks)** —— 完整的 hooks 参考
- **[CLI Reference](https://code.claude.com/docs/en/cli-reference)** —— 命令行接口文档
- **[Memory Guide](../02-memory/README.md)** —— 持久化上下文配置

---

**最后更新**：2026 年 6 月 15 日
**Claude Code 版本**：2.1.176
**来源**：
- https://code.claude.com/docs/en/hooks
- https://code.claude.com/docs/en/permissions
- https://code.claude.com/docs/en/changelog
- https://code.claude.com/docs/en/changelog#2-1-176
- https://github.com/anthropics/claude-code/releases/tag/v2.1.139
- https://github.com/anthropics/claude-code/releases/tag/v2.1.145
- https://github.com/anthropics/claude-code/releases/tag/v2.1.152
- https://github.com/anthropics/claude-code/releases/tag/v2.1.153
**兼容模型**：Claude Sonnet 4.6、Claude Opus 4.8、Claude Haiku 4.5

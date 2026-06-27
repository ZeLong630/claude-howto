<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../../resources/logos/claude-howto-logo.svg">
</picture>

# 高级功能

这是一份关于 Claude Code 高级能力的完整指南，涵盖 planning mode、extended thinking、auto mode、后台任务、Monitor 工具、动态工作流、定时任务、权限模式、打印模式（非交互）、会话管理、交互功能、TUI 模式、语音输入、消息通道（Channels）、Chrome 集成、远程控制、网页会话、桌面应用、任务列表、提示词建议、Git 工作树、沙盒、企业受管设置和配置。

## 目录

1. [概览](#概览)
2. [规划模式](#规划模式)
3. [Ultraplan（云端计划起草）](#ultraplan云端计划起草)
4. [扩展思考](#扩展思考)
5. [自动模式](#自动模式)
6. [后台任务](#后台任务)
7. [Monitor 工具（事件驱动流）](#monitor-工具事件驱动流)
8. [动态工作流](#动态工作流)
9. [定时任务](#定时任务)
10. [权限模式](#权限模式)
11. [无头模式](#无头模式)
12. [会话管理](#会话管理)
13. [交互功能](#交互功能)
14. [TUI 模式（全屏）](#tui-模式全屏)
15. [语音输入](#语音输入)
16. [消息通道（Channels）](#消息通道channels)
17. [Chrome 集成](#chrome-集成)
18. [远程控制](#远程控制)
19. [网页会话](#网页会话)
20. [桌面应用](#桌面应用)
21. [任务列表](#任务列表)
22. [提示词建议](#提示词建议)
23. [Git 工作树（Git Worktrees）](#git-工作树git-worktrees)
24. [沙盒](#沙盒)
25. [企业受管设置](#企业受管设置)
26. [配置与设置](#配置与设置)
27. [Agent Teams](#agent-teams)
28. [最佳实践](#最佳实践)
29. [更多资源](#更多资源)

---

## 概览

Claude Code 的高级功能把基础能力扩展到了规划、推理、自动化和控制层面。它们能支持复杂开发任务、代码审查、自动化以及多会话管理等更复杂的工作流。

**核心高级功能包括：**
- **Planning Mode**：先写详细实现计划，再开始编码
- **Extended Thinking**：对复杂问题进行深入推理
- **Auto Mode**：由后台安全分类器在每个动作执行前进行审查（Research Preview）
- **Background Tasks**：长时间操作不阻塞对话
- **Permission Modes**：控制 Claude 可以做什么（`default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`）
- **Print Mode**：非交互式运行 Claude Code，适合自动化和 CI/CD（`claude -p`）
- **Session Management**：管理多个工作会话
- **Interactive Features**：快捷键、多行输入、命令历史
- **Voice Dictation**：按住说话的语音输入，支持 20 种语言的 STT
- **Channels**：MCP server 向运行中的会话推送消息（Research Preview）
- **Remote Control**：从 Claude.ai 或 Claude app 控制 Claude Code
- **Web Sessions**：在 claude.ai/code 的浏览器中运行 Claude Code
- **Desktop App**：用于可视化 diff 审查和多会话的独立应用
- **Task List**：跨 context compaction 持久跟踪任务
- **Prompt Suggestions**：根据上下文智能推荐命令
- **Git Worktrees**：隔离的 worktree 分支，适合并行工作
- **Sandboxing**：操作系统级文件系统和网络隔离
- **Managed Settings**：通过 plist、Registry 或受管文件进行企业部署
- **Configuration**：用 JSON 配置文件定制行为

---

## 规划模式

Planning mode 允许 Claude 在真正实现复杂任务前先进行思考，生成一份你可以审阅并批准的详细计划。

### 什么是 Planning Mode？

Planning mode 是一个两阶段流程：
1. **规划阶段**：Claude 分析任务并生成详细实现计划
2. **实现阶段**：在你批准后，Claude 执行计划

### 什么时候使用 Planning Mode

✅ 适合在以下情况使用 planning mode：
- 复杂的多文件重构
- 新功能实现
- 架构调整
- 数据库迁移
- 大型 API 重设计

❌ 不要在以下情况使用 planning mode：
- 简单 bug 修复
- 格式化修改
- 单文件编辑
- 快速查询

### 如何启动 Planning Mode

**Slash command**：
```bash
/plan Implement user authentication system
```

**CLI 参数**：
```bash
claude --permission-mode plan
```

**设为默认**：
```json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

**快捷键**：
- `Shift + Tab` - 切换权限模式（包括 plan）
- `Alt + M` - 切换权限模式（Windows/Linux）

### Planning Mode 的好处

- **结构化的实施方式**：把实现步骤拆解得更清楚
- **审阅和批准**：在执行前批准或调整计划
- **风险识别**：在实现前发现潜在问题
- **阶段清晰**：有组织的实现阶段和里程碑

### 示例：功能实现

```
User: /plan Add a real-time notification system

Claude: I'll create a plan for implementing real-time notifications.

## Implementation Plan

### Phase 1: Infrastructure Setup
1. Install WebSocket libraries (socket.io)
2. Set up WebSocket server
3. Configure connection management

### Phase 2: Backend Implementation
4. Create notification service
5. Implement notification queue
6. Add database schema for notifications
7. Create API endpoints

### Phase 3: Frontend Integration
8. Set up WebSocket client
9. Create notification UI components
10. Implement notification state management
11. Add notification permissions

### Phase 4: Testing & Deployment
12. Write unit tests
13. Write integration tests
14. Update documentation
15. Deploy to staging

Estimated time: 3-4 hours
Files to modify: 12
New files to create: 8

Ready to proceed? (yes/no/modify plan)

User: yes

Claude: [Begins implementation following the plan]
```

### 计划修改

```
User: Modify the plan - skip the queue for now, we'll add it later

Claude: Updated plan:
[Shows modified plan with queue removed]

User: Looks good, proceed

Claude: [Implements modified plan]
```

### Planning Mode 配置

Planning mode 通过 CLI 参数或 slash command 启用：

```bash
# Activate plan mode via CLI
claude --permission-mode plan

# Or use the /plan slash command inside the REPL
/plan Implement user authentication system
```

**规划专用模型别名**：使用 `opusplan` 作为模型别名，规划用 Opus、执行用 Sonnet：

```bash
claude --model opusplan "design and implement the new API"
```

**外部编辑计划**：按 `Ctrl+G` 可以把当前计划打开到外部编辑器中进行更详细的修改。

> **v2.1.112 更新**：计划文件现在以产生它的提示词命名（而不是随机词语），更便于浏览和重用。

> **v2.1.136 更新 —— plan 模式的写入阻止是无条件的**：Plan 模式现在阻止所有文件写入，包括 `permissions.allow` 中存在匹配 `Edit(...)` 规则的情况。此前一条宽松的 `Edit(...)` 规则可能让写入在 plan 模式下放行；该绕过路径已被关闭。如果某个工作流依赖旧行为，请先退出 plan 模式（`Shift+Tab`）再编辑。

---

## Ultraplan（云端计划起草）

> **v2.1.101 新增**：Ultraplan 现在会在你首次调用它时自动创建一个 Claude Code on the web 云端环境 —— 无需手动设置，也不必在起草开始前等待容器预热。

> **注意**：Ultraplan 是 research preview，需要 Claude Code v2.1.91 或更高版本。

`/ultraplan` 把一个规划任务从你的本地 CLI 交给一个运行在 plan 模式下的 Claude Code on the web 会话。Claude 在云端起草计划，同时你的终端保持空闲以处理其他工作；随后你在浏览器中审阅草稿，并选择在哪里执行 —— 在同一个云端会话中，或者 teleport 回你的终端。

### 什么时候使用 Ultraplan

- 你想要比终端更丰富的审阅界面：行内评论、emoji 反应、大纲侧边栏，以及持久的历史记录。
- 你想在本地继续编码的同时进行无人值守的起草 —— 云端会话会研究仓库并撰写计划，而不阻塞你的 CLI。
- 计划在执行前需要相关方审阅 —— 一个可分享的 web URL 胜过粘贴终端回滚内容。

### 要求

- 一个 Claude Code on the web 账户。
- 一个 GitHub 仓库（云端会话会克隆你的仓库，针对真实代码起草）。
- 在 Amazon Bedrock、Google Cloud Vertex AI 或 Microsoft Foundry 上**不可用**。

### 三种启动方式

- **命令**：`/ultraplan <prompt>` —— 显式调用。
- **关键词**：在普通提示词中任意位置包含单词 `ultraplan`，Claude 就会把请求路由到云端。
- **从本地计划起步**：在 Claude 本地完成一份计划后，在批准对话框中选择 "No, refine with Ultraplan on Claude Code on the web"，把草稿交出去做更深入的研究。

### 用法示例

```bash
/ultraplan migrate the auth service from sessions to JWTs
```

Claude 会确认、启动云端环境（在 v2.1.101+ 首次运行时自动创建），并返回一个你可以在浏览器中打开的会话链接。

### 状态指示符

| 状态 | 含义 |
|---|---|
| `◇ ultraplan` | Claude 正在研究你的代码库并起草计划 |
| `◇ ultraplan needs your input` | Claude 有一个澄清性问题；打开会话链接进行回复 |
| `◆ ultraplan ready` | 计划已就绪，可在浏览器中审阅 |

### 执行选项

计划就绪后，你有两条执行路径。在浏览器中批准计划即可在同一个云端会话中执行 —— Claude 远程实现改动并从 web UI 打开一个 pull request。或者选择 "Approve plan and teleport back to terminal" 在本地实现。终端 teleport 对话框提供三个选择：

- **Implement here** —— 在你当前的终端会话中运行已批准的计划。
- **Start new session** —— 在同一个工作目录中打开一个全新会话并在那里实现。
- **Cancel** —— 把计划保存到文件，便于你稍后继续。

> **警告**：Ultraplan 启动时 Remote Control 会断开连接。两个功能共享 claude.ai/code 界面，所以同一时间只能激活一个。

---

## 扩展思考

Extended thinking 让 Claude 在给出解决方案前，花更多时间推理复杂问题。

### 什么是 Extended Thinking？

Extended thinking 是一个有意识的、分步骤的推理过程，Claude 会：
- 拆解复杂问题
- 考虑多种方案
- 评估权衡
- 推导边界情况

### 如何启用 Extended Thinking

**快捷键**：
- `Option + T`（macOS）/ `Alt + T`（Windows/Linux）- 切换 extended thinking

**自动启用**：
- 对所有模型默认开启（Opus 4.8、Opus 4.7、Sonnet 4.6、Haiku 4.5）
- Opus 4.8：自适应推理，effort 等级为 `low`（○）、`medium`（◐）、`high`（●）、`xhigh`、`max`。默认在 Opus 4.8（v2.1.154）、Opus 4.6 和 Sonnet 4.6 上为 `high`，在 Opus 4.7 上为 `xhigh`。`xhigh` 在 Opus 4.8 和 Opus 4.7 上可用（在 Opus 4.6 / Sonnet 4.6 上回退到 `high`）。`max` 在 Opus 4.8/4.7/4.6 和 Sonnet 4.6 上可用（仅限会话）。Haiku 4.5 没有 effort 等级。Opus 4.8 和 Opus 4.7 拥有 1M token 的原生上下文窗口（1M 上下文修复在 v2.1.117 落地 —— 在此之前 `/context` 会按 200K 窗口错误计算 Opus 4.7，并触发过早的 autocompact）。从 v2.1.129 起，`/context` 仅在 UI 内显示其可视化；ASCII 可视化不再泄漏到对话上下文中（每次调用节省约 1.6k token），因此可以放心随意调用 `/context`。
- Opus 4.6 / Sonnet 4.6 上的 Pro/Max 订阅者：默认 effort 在 v2.1.117 中从 `medium` 提升到 `high`。
- 其他模型：固定预算最多 31,999 token

**配置方法**：
- 切换：`Alt+T` / `Option+T`，或通过 `/config`
- 查看推理：`Ctrl+O`（verbose 模式）
- 设置 effort：`/effort` 命令或 `--effort` 参数

**自定义预算**：
```bash
export MAX_THINKING_TOKENS=1024
```

**Effort 等级**（在 Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6 上支持 —— 不支持 Haiku 4.5）：
```bash
export CLAUDE_CODE_EFFORT_LEVEL=high   # low (○), medium (◐), high (●), xhigh (Opus 4.8/4.7), or max — default is high on Opus 4.8
```

**CLI 参数**：
```bash
claude --effort high "complex architectural review"
```

**Slash command**：
```
/effort high
```

> **注意：** 提示词中的关键词 "ultrathink" 会激活深度推理模式。Effort 等级 `low`、`medium`、`high` 和 `max` 在 Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6 上支持（Haiku 4.5 没有）。`xhigh` 在 Opus 4.8 和 Opus 4.7 上可用。默认 effort 在 Opus 4.8（以及 Opus 4.6 / Sonnet 4.6）上为 `high`，在 Opus 4.7 上为 `xhigh`。`/effort` 菜单还提供 `ultracode`，它**不是**模型 effort 等级 —— 它发送 `xhigh` 并让 Claude 编排动态工作流（仅限会话）。

### Extended Thinking 的好处

- **深度推理**：花更多时间思考复杂问题
- **更好的架构决策**：通过全面评估做出有依据的选择
- **考虑边界情况**：推演潜在问题和场景
- **彻底的分析**：用结构化方式解决复杂问题

### 示例：架构决策

```
User: ultrathink Should we use microservices or a monolith for our e-commerce platform?

Claude: Let me think through this architectural decision carefully...

<extended_thinking>
## Context Analysis
- Team size: 5 developers
- Expected traffic: 10k daily users initially
- Need to scale to 100k+ users
- Limited DevOps resources
- 6-month timeline to MVP

## Approach 1: Microservices
Pros:
+ Independent scaling
+ Technology flexibility
+ Team independence
+ Fault isolation

Cons:
- Operational complexity
- Network overhead
- Distributed tracing needed
- Requires DevOps expertise
- Slower initial development

## Approach 2: Monolith
Pros:
+ Simpler deployment
+ Faster initial development
+ Easier debugging
+ Lower operational overhead
+ Single codebase

Cons:
- Scaling challenges later
- All-or-nothing deployments
- Technology lock-in
- Potential for tight coupling

## Decision Factors
1. Team size (5 devs) → Favors monolith
2. DevOps resources (limited) → Favors monolith
3. Timeline (6 months) → Favors monolith
4. Future scale (100k users) → Achievable with monolith + caching
5. Complexity budget → Limited

## Recommendation
Start with a **modular monolith**:
- Single deployment unit
- Clear module boundaries
- Clean interfaces between modules
- Prepare for future extraction to microservices if needed

This gives you:
✓ Fast initial development
✓ Low operational complexity
✓ Option to extract services later
✓ Clear architecture

Migration path: When you reach 50k+ users or specific modules need independent scaling, extract those modules into microservices.
</extended_thinking>

**Recommendation**: Implement a modular monolith with clean module boundaries.

This approach balances your current constraints (team size, timeline, DevOps resources) with future scalability needs. You can migrate specific modules to microservices later when there's a clear business need.
```

### Extended Thinking 配置

Extended thinking 通过环境变量、快捷键和 CLI 参数控制：

```bash
# Set thinking token budget
export MAX_THINKING_TOKENS=16000

# Set effort level (Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6): low (○), medium (◐), high (●), xhigh (Opus 4.8/4.7), or max — default is high on Opus 4.8
export CLAUDE_CODE_EFFORT_LEVEL=high
```

在会话中用 `Alt+T` / `Option+T` 切换，用 `/effort` 设置 effort，或通过 `/config` 配置。

> **精简系统提示词（v2.1.154）：** 精简系统提示词现在是除 Haiku、Sonnet 和 Opus 4.7 及更早版本之外**所有模型的默认值**，降低了 Opus 4.8 上的基线 token 开销。

---

## 自动模式

Auto Mode 是一个 research preview 权限模式（2026 年 3 月），它使用后台安全分类器在每个动作执行前进行审查。它允许 Claude 自主工作，同时阻止危险操作。

### 要求

- **套餐**：Team、Enterprise 或 API（Pro 或 Max 套餐不可用）
- **模型**：Claude Sonnet 4.6 或 Opus 4.8
- **提供商**：仅限 Anthropic API（不支持 Bedrock、Vertex 或 Foundry）
- **分类器**：运行在 Claude Sonnet 4.6 上（会增加额外的 token 成本）

### 启用 Auto Mode

```bash
# Unlock auto mode with CLI flag (no longer required for Max subscribers on Opus 4.7 — access it directly)
claude --enable-auto-mode

# Then cycle to it with Shift+Tab in the REPL
```

> **v2.1.112 更新**：Auto mode 不再需要 `--enable-auto-mode` 参数。Max 订阅者在 Opus 4.7 上可直接访问。

> **v2.1.158 更新**：Auto mode 现在在 Bedrock、Vertex 和 Foundry 上对 Opus 4.7/4.8 可用 —— 通过设置 `CLAUDE_CODE_ENABLE_AUTO_MODE=1` **选择启用**。

或者把它设为默认权限模式：

```bash
claude --permission-mode auto
```

通过配置设置：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 分类器如何工作

后台分类器按以下决策顺序评估每个动作：

1. **允许/拒绝规则** —— 首先检查显式权限规则
2. **只读/编辑自动批准** —— 文件读取和编辑自动通过
3. **分类器** —— 后台分类器审查该动作
4. **回退** —— 在连续 3 次或累计 20 次阻止后回退到提示

### 默认阻止的动作

Auto mode 默认阻止以下动作：

| 阻止的动作 | 示例 |
|----------------|---------|
| 管道到 shell 的安装 | `curl \| bash` |
| 向外部发送敏感数据 | 通过网络发送 API 密钥、凭据 |
| 生产环境部署 | 针对生产环境的部署命令 |
| 大规模删除 | 在大目录上执行 `rm -rf` |
| IAM 变更 | 权限和角色修改 |
| 强制推送到 main | `git push --force origin main` |

### 默认允许的动作

| 允许的动作 | 示例 |
|----------------|---------|
| 本地文件操作 | 读取、写入、编辑项目文件 |
| 声明依赖的安装 | 从清单执行 `npm install`、`pip install` |
| 只读 HTTP | 用 `curl` 获取文档 |
| 推送到当前分支 | `git push origin feature-branch` |

### 配置 Auto Mode

**以 JSON 打印默认规则**：
```bash
claude auto-mode defaults
```

**配置可信基础设施**：通过 `autoMode.environment` 受管设置进行企业部署。这允许管理员定义可信的 CI/CD 环境、部署目标和基础设施模式。

#### 用 `"$defaults"` 扩展默认值（v2.1.118）

从 v2.1.118 起，`autoMode.allow`、`autoMode.soft_deny` 和 `autoMode.environment` 接受一个 `"$defaults"` 令牌，它会把你的规则**追加**到内置列表中，而不是替换它。在 v2.1.118 之前，任何用户定义的数组都会静默覆盖内置值。

#### 用 `autoMode.hard_deny` 无条件阻止（v2.1.136）

`autoMode.hard_deny`（v2.1.136+）是一个分类器规则数组，它会**不考虑推断出的用户意图**地阻止一类动作。把它用于在 auto mode 下绝不应运行的动作 —— 例如在根路径上执行 `rm -rf`，或在受保护分支上执行 `git push --force`。与 `soft_deny` 不同，hard-deny 规则无法被分类器协商。

```json
{
  "autoMode": {
    "hard_deny": ["Bash(rm -rf /:*)", "Bash(git push --force*)"]
  }
}
```

**之前（替换内置值 —— v2.1.118 之前的行为）：**

```json
{
  "autoMode": {
    "allow": ["Bash(gh pr list:*)"]
  }
}
```

**之后（扩展内置值 —— v2.1.118+）：**

```json
{
  "autoMode": {
    "allow": ["$defaults", "Bash(gh pr list:*)"],
    "soft_deny": ["$defaults", "Bash(kubectl delete:*)"],
    "environment": ["$defaults", "trusted-ci.internal"]
  }
}
```

使用 `"$defaults"` 可以保留出厂的基线规则，同时在其上叠加组织或项目专用的补充规则。

#### 基于意图的内置保护（v2.1.183）

与用户配置的 `hard_deny` 分开，auto mode 默认阻止以下破坏性命令，除非你在本次会话中显式要求过它们：

- `git reset --hard`、`git checkout -- .`、`git clean -fd`、`git stash drop`
- `git commit --amend`（当该 commit 不是 agent 在本次会话中提交的）
- `terraform destroy`、`pulumi destroy`、`cdk destroy`（除非你要求过特定的 stack）

这是由推断意图驱动的内置默认保护 —— 你不需要自己把这些加进 `hard_deny`。

### 回退行为

当分类器不确定时，auto mode 会回退到提示用户：
- 在**连续 3 次**分类器阻止之后
- 在一次会话中**累计 20 次**分类器阻止之后

这确保了在分类器无法自信地批准某个动作时，用户始终保有控制权。

### 预置等效于 Auto-Mode 的权限（无需 Team 套餐）

如果你没有 Team 套餐，或者想要一种不带后台分类器的更简单方式，你可以用一套保守的安全权限规则基线来预置你的 `~/.claude/settings.json`。脚本从只读和本地检查规则开始，然后只在你想要时让你选择性地加入编辑、测试、本地 git 写入、包安装和 GitHub 写操作。

**文件：** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added (no changes written)
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply the conservative baseline
python3 09-advanced-features/setup-auto-mode-permissions.py

# Add more capability only when you need it
python3 09-advanced-features/setup-auto-mode-permissions.py --include-edits --include-tests
python3 09-advanced-features/setup-auto-mode-permissions.py --include-git-write --include-packages
```

脚本会跨以下类别添加规则：

| 类别 | 示例 |
|----------|---------|
| 核心只读工具 | `Read(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)`、`WebFetch(*)` |
| 本地检查 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)`、`Bash(cat:*)` |
| 可选编辑 | `Edit(*)`、`Write(*)`、`NotebookEdit(*)` |
| 可选测试/构建 | `Bash(pytest:*)`、`Bash(python3 -m pytest:*)`、`Bash(cargo test:*)` |
| 可选 git 写入 | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git stash:*)` |
| Git（本地写入） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建与测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常见 shell | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

危险操作（`rm -rf`、`sudo`、强制推送、`DROP TABLE`、`terraform destroy` 等）被有意排除。脚本是幂等的 —— 运行两次不会重复添加规则。

---

## 后台任务

后台任务允许长时间运行的操作在不阻塞对话的情况下执行。

### 什么是后台任务？

后台任务在你继续工作时异步运行：
- 长时间的测试套件
- 构建流程
- 数据库迁移
- 部署脚本
- 分析工具

**基本用法：**
```bash
User: Run tests in background

Claude: Started task bg-1234

/task list           # Show all tasks
/task status bg-1234 # Check progress
/task show bg-1234   # View output
/task cancel bg-1234 # Cancel task
```

### 启动后台任务

```
User: Run the full test suite in the background

Claude: Starting tests in background (task-id: bg-1234)
You can continue working while tests run.

[You can continue the conversation]

User: Meanwhile, let's refactor the auth module

Claude: [Works on auth module while tests run in background]

[Later, when tests complete]

Claude: 📢 Background task bg-1234 completed:
✅ 245 tests passed
❌ 3 tests failed
View results: /task show bg-1234
```

### 管理后台任务

**列出活动任务**：
```
User: /task list

Active background tasks:
1. [bg-1234] Running tests (50% complete, 2min remaining)
2. [bg-1235] Building Docker image (25% complete, 8min remaining)
3. [bg-1236] Deploying to staging (90% complete, 30sec remaining)
```

**检查任务状态**：
```
User: /task status bg-1234

Task bg-1234: Running tests
Status: In progress
Progress: 120/245 tests (49%)
Started: 2025-11-08 10:30:15
Estimated completion: 2025-11-08 10:34:22
```

**查看任务输出**：
```
User: /task show bg-1234

[Shows live output from the test run]
```

**取消任务**：
```
User: /task cancel bg-1234

Cancelled background task bg-1234
```

### 示例：并行开发

```
User: Run the build in the background

Claude: Starting build... (task-id: bg-5001)

User: Also run the linter in background

Claude: Starting linter... (task-id: bg-5002)

User: While those run, let's implement the new API endpoint

Claude: [Implements API endpoint while build and linter run]

[10 minutes later]

Claude: 📢 Build completed successfully (bg-5001)
📢 Linter found 12 issues (bg-5002)

User: Show me the linter issues

Claude: [Shows linter output from bg-5002]
```

### 配置

```json
{
  "backgroundTasks": {
    "enabled": true,
    "maxConcurrentTasks": 5,
    "notifyOnCompletion": true,
    "autoCleanup": true,
    "logOutput": true
  }
}
```

---

## Monitor 工具（事件驱动流）

> **v2.1.98 新增**：Monitor 工具让 Claude 监视后台命令的 stdout，并在匹配事件出现的那一刻做出反应 —— 取代了为等待长时间运行进程而使用的轮询循环和 `sleep`。

Monitor 可以附加到任何向 stdout 写入的 shell 命令上。命令的每一行 stdout 都会成为一个唤醒会话的通知。Claude 指定命令；harness 流式传输输出并在事件触发时投递它们。启动底层进程请参见相关的 [后台任务](#后台任务) 小节。

### 为什么它重要

用 `/loop` 或 `sleep` 轮询会在每个周期消耗一整次 API 往返，无论是否有任何变化。Monitor 在事件触发前保持静默，命令安静时消耗**零 token**。当事件确实发生时，Claude 会立即做出反应 —— 不必等到下一个轮询节拍才延迟发现。对于任何运行超过几分钟的任务，这都比轮询循环更便宜也更快。

### 两种常见模式

**流过滤器**监视来自长时间运行源的连续输出。命令永远运行；每一行匹配的内容都是一个事件。

```bash
tail -f /var/log/app.log | grep --line-buffered "ERROR"
```

**轮询并发出过滤器**周期性地检查一个源，只在有变化时才发出。把它用于 API、数据库或任何没有原生流的源。

```bash
last=$(date -u +%Y-%m-%dT%H:%M:%SZ)
while true; do
  gh api "repos/owner/repo/issues/123/comments?since=$last" || true
  last=$(date -u +%Y-%m-%dT%H:%M:%SZ)
  sleep 30
done
```

### 具体示例

"启动我的开发服务器并监控它的错误。" Claude 把服务器作为后台任务启动，附加一个 Monitor 过滤器（`tail -F server.log | grep --line-buffered -E "ERROR|FATAL"`），然后会话安静下来。一旦日志中出现错误行，Claude 就会被唤醒、读取错误，并能做出反应 —— 重启服务器、修复 bug，或将其呈现给你 —— 你无需主动检查。

> **警告**：当管道到 `grep` 时，**务必**使用 `grep --line-buffered`。如果不用，grep 会以 4KB 块缓冲 stdout，在低流量的流上可能把事件延迟几分钟。这是实践中 Monitor 失效的头号原因 —— 如果你的过滤器在本不该静默时却静默，请先检查 `--line-buffered` 参数。

---

## 动态工作流

> **v2.1.154 新增**

动态工作流让 Claude **确定性地**编排数十到数百个后台 [subagent](../04-subagents/README.md) —— 把 fan-out、流水线和并行阶段编码进一个脚本里，而不是交给模型即兴发挥。一个单一 agent 持有一个上下文窗口，而一个工作流则把任务分解到许多 agent 上并重新组合它们的结果。

### 什么时候使用它们

- **全面覆盖** —— 并行地跨许多文件/维度进行审计或审查。
- **置信度** —— 生成独立的视角，然后在提交前对抗性地验证发现。
- **超越单个上下文的规模** —— 大型迁移、广泛扫荡，或任何单个上下文无法容纳的研究。

对于你已经理解的一次性任务，单个 agent（或直接编辑）仍然是合适的工具 —— 工作流在工作 fan out 时才会带来回报。

### 启动与查看

- **启动**：让 Claude 为任务创建一个工作流（例如 "run a workflow to review every file in `src/`"）。Claude 撰写编排脚本并在后台运行它。
- **查看**：`/workflows` 命令展示正在运行和已完成的工作流运行，并带有实时进度。
- **`ultracode`**：在 `/effort` 菜单中选择 `ultracode` 会为该会话开启此功能 —— 它向模型发送 `xhigh`，*并且*让 Claude 默认编排动态工作流。它仅限会话，且不被设置文件接受。（截至 v2.1.160，触发关键词是 `ultracode`；裸词 "workflow" 不再触发运行。）

工作流构建于 subagent 模型之上 —— 关于单个 agent 如何定义和限定范围，请参见 [Subagents](../04-subagents/README.md)。

---

## 定时任务

Scheduled Tasks 让你按重复计划自动运行提示词，或作为一次性提醒。任务的作用域为会话级 —— 它们在 Claude Code 活动期间运行，会话结束时被清除。自 v2.1.72+ 起可用。

> **在 claude.com 上以 "Routines" 营销（2026-05-14）**：Anthropic 的产品博客把这个界面介绍为 **Routines**。CLI 命令仍是 `/schedule`；本指南为保持连续性而使用最初的 "Scheduled Tasks" 命名。如果你在 claude.com 文档或桌面应用中看到 "Routines"，它指的是同一个功能。

### `/loop` 命令

```bash
# Explicit interval
/loop 5m check if the deployment finished

# Natural language
/loop check build status every 30 minutes
```

也支持标准的 5 字段 cron 表达式以进行精确调度。

### 一次性提醒

设置在特定时间触发一次的提醒：

```
remind me at 3pm to push the release branch
in 45 minutes, run the integration tests
```

### 管理定时任务

| 工具 | 说明 |
|------|-------------|
| `CronCreate` | 创建一个新的定时任务 |
| `CronList` | 列出所有活动的定时任务。自 v2.1.136 起，输出还包括限定符和已调度的提示词正文，因此你无需打开就能审计每个 cron 将运行什么。 |
| `CronDelete` | 移除一个定时任务 |

**限制与行为**：
- 每个会话最多 **50 个定时任务**
- 会话级作用域 —— 会话结束时被清除
- 重复任务在 **3 天**后自动过期
- 任务只在 Claude Code 运行时触发 —— 对错过的触发不补偿

### 行为细节

| 方面 | 细节 |
|--------|--------|
| **重复任务抖动** | 最多为间隔的 10%（最大 15 分钟） |
| **一次性任务抖动** | 在 :00/:30 边界上最多 90 秒 |
| **错过的触发** | 不补偿 —— 如果 Claude Code 未运行则跳过 |
| **持久化** | 不跨重启持久化 |

### 云端定时任务

使用 `/schedule` 创建运行在 Anthropic 基础设施上的云端定时任务：

```
/schedule daily at 9am run the test suite and report failures
```

云端定时任务跨重启持久化，且不要求 Claude Code 在本地运行。

### 禁用定时任务

```bash
export CLAUDE_CODE_DISABLE_CRON=1
```

> **`/schedule` 被 API-key 层级自动禁用（v2.1.139）**：当 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN` 或 `apiKeyHelper` 中的任何一个被设置时，云端 `/schedule` 会静默不可用 —— 即使你同时用 claude.ai 登录也是如此。相同条件会禁用 [Remote Control](#禁用-remote-controldisableremotecontrolv21128)、claude.ai MCP connector 和通知偏好。取消设置 API key（或运行在 Pro/Max OAuth 层级上）以使用 `/schedule`。本地 `CronCreate` 不受影响。

### 示例：监控部署

```
/loop 5m check the deployment status of the staging environment.
        If the deploy succeeded, notify me and stop looping.
        If it failed, show the error logs.
```

> **提示**：定时任务的作用域为会话级。对于需要在重启后仍存活的持久自动化，请改用 CI/CD 流水线、GitHub Actions 或桌面应用的定时任务。

---

## 权限模式

权限模式控制 Claude 无需显式批准就能采取哪些动作。

### 可用的权限模式

| 模式 | 行为 |
|---|---|
| `default` | 仅读取文件；对所有其他动作提示 |
| `acceptEdits` | 读取和编辑文件；对命令提示 |
| `plan` | 仅读取文件（研究模式，无编辑） |
| `auto` | 所有动作均经后台安全分类器检查（Research Preview） |
| `bypassPermissions` | 所有动作，无权限检查（危险） |
| `dontAsk` | 只有预先批准的工具会执行；其余全部拒绝 |

在 CLI 中用 `Shift+Tab` 循环切换模式。用 `--permission-mode` 参数或 `permissions.defaultMode` 设置一个默认值。

截至 v2.1.160，即使 `acceptEdits` 也会在写入 shell 启动文件（`.zshenv`、`.zlogin`、`.bash_login`、`~/.config/git/`）和执行代码的构建配置（`.npmrc`、`.yarnrc*`、`bunfig.toml`、`.bazelrc`、`.pre-commit-config.yaml`、`.devcontainer/` 等）之前提示，否则它们可能导致意外的命令执行。

> **`--dangerously-skip-permissions` 扩展的路径覆盖（v2.1.121、v2.1.126）**：`--dangerously-skip-permissions` CLI 参数（及等效的 `bypassPermissions` 模式）现在会对一个大得多的允许列表的写入绕过提示 —— `.claude/skills/`、`.claude/agents/`、`.claude/commands/`、`.claude/`、`.git/`、`.vscode/` 和 shell 配置文件。灾难性的删除命令（`rm -rf /` 等）无论何种模式仍会提示。把这个参数当作比以前更锋利的工具；只在用完即弃的沙盒中使用它。

> **Windows shell 检测（v2.1.120、v2.1.126）**：不再需要 Git for Windows / Git Bash。当 Git Bash 缺失时，Claude Code 使用 PowerShell 作为 shell 工具。从 v2.1.126 起，当 PowerShell 工具启用时 PowerShell 是*主*shell，且检测涵盖通过 Microsoft Store 安装的 PowerShell 7、不带 PATH 的 MSI，或作为 `.NET global tool` 安装的版本。

> **PowerShell 工具在 Windows 上对 Bedrock/Vertex/Foundry 默认启用（v2.1.143）**：截至 v2.1.143，PowerShell 工具在 Windows 上对 Bedrock、Vertex 和 Foundry 用户**默认启用**。Claude Code 以 `-ExecutionPolicy Bypass` 调用 PowerShell，使脚本即便系统策略为 `Restricted` 也能运行。要让 Claude Code 遵守系统执行策略，请设置 `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY=1`。要完全禁用 PowerShell 工具，请设置 `CLAUDE_CODE_USE_POWERSHELL_TOOL=0`。

### 启用方式

**快捷键**：
```bash
Shift + Tab  # Cycle through all 6 modes
```

**Slash command**：
```bash
/plan                  # Enter plan mode
```

**CLI 参数**：
```bash
claude --permission-mode plan
claude --permission-mode auto
```

**设置**：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 权限模式示例

#### Default Mode
Claude 在重要动作上请求确认：

```
User: Fix the bug in auth.ts

Claude: I need to modify src/auth.ts to fix the bug.
The change will update the password validation logic.

Approve this change? (yes/no/show)
```

#### Plan Mode
在执行前审阅实现计划：

```
User: /plan Implement user authentication system

Claude: I'll create a plan for implementing authentication.

## Implementation Plan
[Detailed plan with phases and steps]

Ready to proceed? (yes/no/modify)
```

#### Accept Edits Mode
自动接受文件修改：

```
User: acceptEdits
User: Fix the bug in auth.ts

Claude: [Makes changes without asking]
```

### 使用场景

**代码审查**：
```
User: claude --permission-mode plan
User: Review this PR and suggest improvements

Claude: [Reads code, provides feedback, but cannot modify]
```

**结对编程**：
```
User: claude --permission-mode default
User: Let's implement the feature together

Claude: [Asks for approval before each change]
```

**自动化任务**：
```
User: claude --permission-mode acceptEdits
User: Fix all linting issues in the codebase

Claude: [Auto-accepts file edits without asking]
```

---

## 无头模式

打印模式（`claude -p`）允许 Claude Code 在没有交互式输入的情况下运行，非常适合自动化和 CI/CD。这是非交互式模式，取代了较旧的 `--headless` 参数。

### 什么是 Print Mode？

Print mode 启用：
- 自动化脚本执行
- CI/CD 集成
- 批处理
- 定时任务

### 在 Print Mode 中运行（非交互）

```bash
# Run specific task
claude -p "Run all tests"

# Process piped content
cat error.log | claude -p "Analyze these errors"

# CI/CD integration (GitHub Actions)
- name: AI Code Review
  run: claude -p "Review PR"
```

### 其他 Print Mode 用法示例

```bash
# Run a specific task with output capture
claude -p "Run all tests and generate coverage report"

# With structured output
claude -p --output-format json "Analyze code quality"

# With input from stdin
echo "Analyze code quality" | claude -p "explain this"
```

### 示例：CI/CD 集成

**GitHub Actions**：
```yaml
# .github/workflows/code-review.yml
name: AI Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p --output-format json \
            --max-turns 3 \
            "Review this PR for:
            - Code quality issues
            - Security vulnerabilities
            - Performance concerns
            - Test coverage
            Output results as JSON" > review.json

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = JSON.parse(fs.readFileSync('review.json', 'utf8'));
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: JSON.stringify(review, null, 2)
            });
```

### Print Mode 配置

Print mode（`claude -p`）支持若干用于自动化的参数：

```bash
# Limit autonomous turns
claude -p --max-turns 5 "refactor this module"

# Structured JSON output
claude -p --output-format json "analyze this codebase"

# With schema validation
claude -p --json-schema '{"type":"object","properties":{"issues":{"type":"array"}}}' \
  "find bugs in this code"

# Disable session persistence
claude -p --no-session-persistence "one-off analysis"
```

### Safe Mode（故障排查）

`--safe-mode`（以及 `CLAUDE_CODE_SAFE_MODE` 环境变量，例如 `CLAUDE_CODE_SAFE_MODE=1`）会以**所有定制全部禁用**的方式启动 Claude Code —— CLAUDE.md、plugin、skill、hook 和 MCP server 全部关闭。

```bash
# Launch with every customization disabled
claude --safe-mode

# Equivalent via environment variable
CLAUDE_CODE_SAFE_MODE=1 claude
```

它是一个故障排查工具：当某个自定义配置引发问题时，以 safe mode 启动以隔离问题出在你的配置里还是 Claude Code 本身。

---

## 会话管理

有效地管理多个 Claude Code 会话。

### 会话管理命令

| 命令 | 说明 |
|---------|-------------|
| `/resume` | 按 ID 或名称恢复对话 |
| `/rename` | 命名当前会话 |
| `/fork` | 把当前会话分叉为一个新分支 |
| `claude -c` | 继续最近的对话 |
| `claude -r "session"` | 按名称或 ID 恢复会话 |

### 恢复会话

**继续上一次对话**：
```bash
claude -c
```

**恢复一个命名会话**：
```bash
claude -r "auth-refactor" "finish this PR"
```

**重命名当前会话**（在 REPL 内）：
```
/rename auth-refactor
```

### 分叉会话

分叉一个会话以尝试替代方案，同时不丢失原始会话：

```
/fork
```

或从 CLI：
```bash
claude --resume auth-refactor --fork-session "try OAuth instead"
```

### 会话持久化

会话会自动保存并可以恢复：

```bash
# Continue last conversation
claude -c

# Resume specific session by name or ID
claude -r "auth-refactor"

# Resume and fork for experimentation
claude --resume auth-refactor --fork-session "alternative approach"
```

### 会话回顾（v2.1.108）

当你离开一段时间后回到会话时，Claude 可以展示一段简短的回顾，说明此前完成了什么。这对于禁用了遥测的用户（Bedrock、Vertex、Foundry 用户）默认启用。

> **OTEL 遥测 —— 重新启用反馈调查（v2.1.136+）**：捕获 OpenTelemetry 数据的组织可以通过设置 `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL=1` 重新启用 Anthropic 的会话质量调查。该调查在 OTEL 部署中默认关闭，因为它此前被从遥测管道中重定向走了。

**控制回顾行为：**

```bash
/recap                                 # manually trigger a recap
/config                                # toggle auto-recap on/off
```

或通过环境变量：
```bash
CLAUDE_CODE_ENABLE_AWAY_SUMMARY=0 claude   # disable recap
CLAUDE_CODE_ENABLE_AWAY_SUMMARY=1 claude   # force enable recap
```

---

## 交互功能

### 快捷键

Claude Code 支持快捷键以提升效率。以下是官方文档中的完整参考：

| 快捷键 | 说明 |
|----------|-------------|
| `Ctrl+C` | 取消当前输入/生成 |
| `Ctrl+D` | 退出 Claude Code |
| `Ctrl+G` | 在外部编辑器中编辑计划 |
| `Ctrl+L` | 清空终端屏幕 |
| `Ctrl+O` | 切换 verbose 输出（查看推理） |
| `Ctrl+R` | 反向搜索历史。默认为**所有项目的全部提示词**（v2.1.129+）；在选择器内按 `Ctrl+S` 缩小到当前项目。更早版本默认仅当前项目。 |
| `Ctrl+T` | 切换任务列表视图 |
| `Ctrl+B` | 后台运行任务 |
| `Esc+Esc` | 回退代码/对话 |
| `Shift+Tab` / `Alt+M` | 切换权限模式 |
| `Option+P` / `Alt+P` | 切换模型 |
| `Option+T` / `Alt+T` | 切换 extended thinking |

**行编辑（标准 readline 快捷键）：**

| 快捷键 | 动作 |
|----------|--------|
| `Ctrl + A` | 移到行首 |
| `Ctrl + E` | 移到行尾 |
| `Ctrl + K` | 剪切到行尾 |
| `Ctrl + U` | 剪切到行首 |
| `Ctrl + W` | 向后删除一个单词 |
| `Ctrl + Y` | 粘贴（yank） |
| `Tab` | 自动补全 |
| `↑ / ↓` | 命令历史 |

### 自定义快捷键

通过运行 `/keybindings` 创建自定义快捷键，它会打开 `~/.claude/keybindings.json` 供编辑（v2.1.18+）。

**配置格式**：

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null,
        "ctrl+k ctrl+s": "chat:stash"
      }
    },
    {
      "context": "Confirmation",
      "bindings": {
        "ctrl+a": "confirmation:yes"
      }
    }
  ]
}
```

把一个绑定设为 `null` 以解绑默认快捷键。

### 可用上下文

快捷键的作用域限定在特定的 UI 上下文中：

| 上下文 | 关键动作 |
|---------|-------------|
| **Chat** | `submit`、`cancel`、`cycleMode`、`modelPicker`、`thinkingToggle`、`undo`、`externalEditor`、`stash`、`imagePaste` |
| **Confirmation** | `yes`、`no`、`previous`、`next`、`nextField`、`cycleMode`、`toggleExplanation` |
| **Global** | `interrupt`、`exit`、`toggleTodos`、`toggleTranscript` |
| **Autocomplete** | `accept`、`dismiss`、`next`、`previous` |
| **HistorySearch** | `search`、`previous`、`next` |
| **Settings** | 上下文相关的设置导航 |
| **Tabs** | 标签切换与管理 |
| **Help** | 帮助面板导航 |

总共有 18 个上下文，包括 `Transcript`、`Task`、`ThemePicker`、`Attachments`、`Footer`、`MessageSelector`、`DiffDialog`、`ModelPicker` 和 `Select`。

### Chord 支持

快捷键支持 chord 序列（多键组合）：

```
"ctrl+k ctrl+s"   → Two-key sequence: press ctrl+k, then ctrl+s
"ctrl+shift+p"    → Simultaneous modifier keys
```

**按键语法**：
- **修饰键**：`ctrl`、`alt`（或 `opt`）、`shift`、`meta`（或 `cmd`）
- **大写隐含 Shift**：`K` 等价于 `shift+k`
- **特殊键**：`escape`、`enter`、`return`、`tab`、`space`、`backspace`、`delete`、方向键

### 保留键与冲突键

| 键 | 状态 | 备注 |
|-----|--------|-------|
| `Ctrl+C` | 保留 | 不能重新绑定（中断） |
| `Ctrl+D` | 保留 | 不能重新绑定（退出） |
| `Ctrl+B` | 终端冲突 | tmux 前缀键 |
| `Ctrl+A` | 终端冲突 | GNU Screen 前缀键 |
| `Ctrl+Z` | 终端冲突 | 进程挂起 |

> **提示**：如果某个快捷键不起作用，请检查它是否与你的终端模拟器或多路复用器冲突。

### Tab 补全

Claude Code 提供智能的 tab 补全：

```
User: /rew<TAB>
→ /rewind

User: /plu<TAB>
→ /plugin

User: /plugin <TAB>
→ /plugin install
→ /plugin enable
→ /plugin disable
```

### 命令历史

访问之前的命令：

```
User: <↑>  # Previous command
User: <↓>  # Next command
User: Ctrl+R  # Search history

(reverse-i-search)`test': run all tests
```

### 多行输入

对于复杂查询，使用多行模式：

```bash
User: \
> Long complex prompt
> spanning multiple lines
> \end
```

**示例：**

```
User: \
> Implement a user authentication system
> with the following requirements:
> - JWT tokens
> - Email verification
> - Password reset
> - 2FA support
> \end

Claude: [Processes the multi-line request]
```

### 行内编辑

在发送前编辑命令：

```
User: Deploy to prodcution<Backspace><Backspace>uction

[Edit in-place before sending]
```

### Vim 模式

启用 Vi/Vim 快捷键进行文本编辑：

**启用**：
- 通过 `/config` 启用（切换 "Editor / Vim mode"），或在 `~/.claude/settings.json` 中通过 `editorMode: "vim"`。独立的 `/vim` slash command 已被移除（见 [issue #43370](https://github.com/anthropics/claude-code/issues/43370)）；vim 模式现在由配置驱动。
- 用 `Esc` 切换到 NORMAL，`i/a/o` 进入 INSERT，`v` 进入 VISUAL，`V` 进入 VISUAL-LINE（v2.1.118+）

**导航键**：
- `h` / `l` - 左移/右移
- `j` / `k` - 下移/上移
- `w` / `b` / `e` - 按单词移动
- `0` / `$` - 移到行首/行尾
- `gg` / `G` - 跳到文本开头/结尾

**文本对象**：
- `iw` / `aw` - 单词内/单词周围
- `i"` / `a"` - 引号字符串内/周围
- `i(` / `a(` - 括号内/周围

**Visual 模式（v2.1.118+）**：

| 键 | 模式 | 行为 |
|-----|------|----------|
| `v` | Visual | 带可视反馈的字符级选择；用 motion 键扩展 |
| `V` | Visual-line | 行级选择；始终选择整行 |
| `y` | Yank | 复制当前 visual 选区 |
| `d` / `x` | Delete | 删除当前 visual 选区 |
| `c` | Change | 删除选区并进入 INSERT 模式 |
| `Esc` | Exit | 返回 NORMAL 模式 |

Visual 选区会在输入框中高亮显示，让你在提交操作符之前清楚看到将要 yank、删除或修改的内容。

### Bash 模式

用 `!` 前缀直接执行 shell 命令：

```bash
! npm test
! git status
! cat src/index.js
```

用它在不切换上下文的情况下快速执行命令。

**自 v2.1.186 起：** `!` 命令的输出现在会自动发送给 Claude，由 Claude 对其做出回应。要保留之前那种只把输出加入上下文而不回应的行为，请在 `settings.json` 中设置 `"respondToBashCommands": false`。

---

## TUI 模式（全屏）

> **v2.1.110 新增**

TUI（Text User Interface，文本用户界面）模式以全屏、无闪烁的输出渲染 Claude Code —— 非常适合 tmux 或 iTerm2 分屏窗格等终端多路复用器。

### 启用 TUI 模式

用 `/tui` 命令切换 TUI 模式，或用 `--tui` 参数启动：

```bash
/tui          # toggle from within a session
claude --tui  # start directly in TUI mode
```

### 配置

| 设置 | 说明 | 默认 |
|---------|-------------|---------|
| `autoScrollEnabled` | 自动滚动到最新消息 | `true` |

通过 `/config` 或 `settings.json` 禁用自动滚动：

```json
{
  "autoScrollEnabled": false
}
```

### Focus 视图

`/focus` 命令切换 focus 视图 —— 一个只显示最相关输出的无干扰显示。`Ctrl+O` 现在只在普通和 verbose 文字记录之间切换（focus 视图是 `/focus`）。

---

## 语音输入

语音输入为 Claude Code 提供按住说话的语音输入，让你可以说出提示词而不是打字。

### 启用语音输入

```
/voice
```

### 特性

| 特性 | 说明 |
|---------|-------------|
| **按住说话** | 按住一个键录音，松开即发送 |
| **20 种语言** | 语音转文本支持 20 种语言 |
| **自定义快捷键** | 通过 `/keybindings` 配置按住说话的按键 |
| **账户要求** | 需要 Claude.ai 账户进行 STT 处理 |

### 配置

在你的快捷键文件（`/keybindings`）中自定义按住说话的快捷键。语音输入使用你的 Claude.ai 账户进行语音转文本处理。

---

## 消息通道（Channels）

Channels 是一个 research preview 功能，它通过 MCP server 把来自外部服务的事件推送进运行中的 Claude Code 会话。来源包括 Telegram、Discord、iMessage 和任意 webhook，使 Claude 能够对实时通知做出反应而无需轮询。

> **认证（v2.1.128+）**：`--channels` 现在同时支持 Pro/Max OAuth **和** API-key（console）认证。更早的版本需要 OAuth。

### 订阅 Channels

```bash
# Subscribe to channel plugins at startup
claude --channels discord,telegram

# Subscribe to multiple sources
claude --channels discord,telegram,imessage,webhooks
```

### 支持的集成

| 集成 | 说明 |
|-------------|-------------|
| **Discord** | 在你的会话中接收并回应 Discord 消息 |
| **Telegram** | 在你的会话中接收并回应 Telegram 消息 |
| **iMessage** | 在你的会话中接收 iMessage 通知 |
| **Webhooks** | 接收来自任意 webhook 源的事件 |

### 配置

在启动时用 `--channels` 参数配置 channels。对于企业部署，使用受管设置来控制允许哪些 channel 插件：

```json
{
  "allowedChannelPlugins": ["discord", "telegram"]
}
```

`allowedChannelPlugins` 受管设置控制整个组织中允许哪些 channel 插件。

### 工作方式

1. MCP server 作为连接到外部服务的 channel 插件
2. 传入的消息和事件被推送进活动的 Claude Code 会话
3. Claude 可以在会话上下文中读取并回应消息
4. Channel 插件必须通过 `allowedChannelPlugins` 受管设置批准
5. 无需轮询 —— 事件被实时推送

---

## Chrome 集成

Chrome 集成把 Claude Code 连接到你的 Chrome 或 Microsoft Edge 浏览器，以进行实时的 web 自动化和调试。这是一个 beta 功能，自 v2.0.73+ 起可用（Edge 支持自 v1.0.36+ 起添加）。

### 启用 Chrome 集成

**在启动时**：

```bash
claude --chrome      # Enable Chrome connection
claude --no-chrome   # Disable Chrome connection
```

**在会话内**：

```
/chrome
```

选择 "Enabled by default" 以为所有未来的会话激活 Chrome 集成。Claude Code 共享你浏览器的登录状态，因此它可以与已认证的 web 应用交互。

### 能力

| 能力 | 说明 |
|------------|-------------|
| **实时调试** | 读取 console 日志、检查 DOM 元素、实时调试 JavaScript |
| **设计验证** | 把渲染的页面与设计稿对比 |
| **表单验证** | 测试表单提交、输入验证和错误处理 |
| **Web 应用测试** | 与已认证的应用交互（Gmail、Google Docs、Notion 等） |
| **数据提取** | 抓取并处理 web 页面的内容 |
| **会话录制** | 把浏览器交互录制为 GIF 文件 |

### 站点级权限

Chrome 扩展管理逐站点的访问权限。随时通过扩展弹窗为特定站点授予或撤销访问权限。Claude Code 只与你显式允许的站点交互。

### 工作方式

Claude Code 在一个可见窗口中控制浏览器 —— 你可以实时观看动作发生。当浏览器遇到登录页面或 CAPTCHA 时，Claude 会暂停并等待你手动处理，然后再继续。

### 已知限制

- **浏览器支持**：仅限 Chrome 和 Edge —— 不支持 Brave、Arc 和其他 Chromium 浏览器
- **WSL**：在 Windows Subsystem for Linux 中不可用
- **第三方提供商**：不支持 Bedrock、Vertex 或 Foundry API 提供商
- **Service worker 空闲**：Chrome 扩展的 service worker 在长时间会话中可能进入空闲

> **提示**：Chrome 集成是一个 beta 功能。浏览器支持可能在未来版本中扩展。

---

## 远程控制

Remote Control 让你从手机、平板或任何浏览器继续一个本地运行的 Claude Code 会话。你的本地会话仍在你的机器上运行 —— 没有任何东西移到云端。在 Pro、Max、Team 和 Enterprise 套餐上可用（v2.1.51+）。

### 启动 Remote Control

**从 CLI**：

```bash
# Start with default session name
claude remote-control

# Start with a custom name
claude remote-control --name "Auth Refactor"
```

**在会话内**：

```
/remote-control
/remote-control "Auth Refactor"
```

**可用参数**：

| 参数 | 说明 |
|------|-------------|
| `--name "title"` | 用于便于识别的自定义会话标题 |
| `--verbose` | 显示详细的连接日志 |
| `--sandbox` | 启用文件系统和网络隔离 |
| `--no-sandbox` | 禁用沙盒（默认） |

### 连接到会话

从另一台设备连接的三种方式：

1. **会话 URL** —— 会话启动时打印到终端；在任何浏览器中打开
2. **二维码** —— 启动后按 `spacebar` 显示一个可扫描的二维码
3. **按名称查找** —— 在 claude.ai/code 或 Claude 手机 app（iOS/Android）中浏览你的会话

### 安全性

- 机器上**不开放入站端口**
- **仅出站 HTTPS**，经 TLS
- **限定范围的凭据** —— 多个短期、范围狭窄的 token
- **会话隔离** —— 每个远程会话相互独立

### Remote Control vs Claude Code on the web

| 方面 | Remote Control | Claude Code on Web |
|--------|---------------|-------------------|
| **执行** | 在你的机器上运行 | 在 Anthropic 云上运行 |
| **本地工具** | 完全访问本地 MCP server、文件和 CLI | 无本地依赖 |
| **使用场景** | 从另一台设备继续本地工作 | 从任何浏览器全新开始 |

### 局限

- 每个 Claude Code 实例只能有一个远程会话
- 终端必须在宿主机上保持打开
- 如果网络不可达，会话约 10 分钟后超时

### 使用场景

- 离开座位时从移动设备或平板控制 Claude Code
- 在保持本地工具执行的同时使用更丰富的 claude.ai UI
- 用你完整的本地开发环境在路上做快速代码审查

### 推送通知（v2.1.110）

当 Remote Control 处于活动状态且 `/config` 中启用了 "Push when Claude decides" 时，Claude 可以向你的手机发送移动推送通知 —— 例如当一个长任务完成或需要你的输入时。

要启用：
1. 激活 Remote Control：`/remote-control` 或 `claude --rc`
2. 打开 `/config` 并启用 **Push when Claude decides**

推送通知需要 Claude 订阅和 Claude 手机 app。

### 禁用 Remote Control（`disableRemoteControl`，v2.1.128+）

Team 或 Enterprise 套餐上的管理员可以用 `disableRemoteControl` 设置完全屏蔽 Remote Control。当为 `true` 时，`claude remote-control` 和 `/remote-control` 都会拒绝启动。

```json
{
  "disableRemoteControl": true
}
```

该设置在**受管/策略**作用域（例如 macOS 上的 `/Library/Application Support/ClaudeCode/managed-settings.json`）被遵守，因此无法被单个用户覆盖。当必须在组织范围内强制仅本地执行时很有用。

> **当 Remote Control 被 API-key 层级自动禁用（v2.1.139）**：无论何时设置了以下任何一项，Remote Control 都会**静默禁用**，即使你同时用 claude.ai 登录：
>
> - `ANTHROPIC_API_KEY`
> - `ANTHROPIC_AUTH_TOKEN`
> - `apiKeyHelper`（settings.json）
>
> 相同条件会禁用 [`/schedule`](#定时任务)、claude.ai MCP connector 和通知偏好 —— 所有四个 claude.ai 桥接界面都以 OAuth 登录为活动凭据为前提。取消设置 API key（或运行在 Pro/Max OAuth 层级上）以使用这些功能。

---

## 网页会话

Web Sessions 允许你直接在 claude.ai/code 的浏览器中运行 Claude Code，或从 CLI 创建 web 会话。

### 创建 Web Session

```bash
# Create a new web session from the CLI
claude --remote "implement the new API endpoints"
```

这会在 claude.ai 上启动一个你可以从任何浏览器访问的 Claude Code 会话。

### 在本地恢复 Web Session

如果你在 web 上启动了一个会话并想在本地继续它：

```bash
# Resume a web session in the local terminal
claude --teleport
```

或从交互式 REPL 内：
```
/teleport
```

### 使用场景

- 在一台机器上开始工作并在另一台继续
- 与团队成员分享会话 URL
- 用 web UI 做可视化 diff 审查，然后切换到终端执行

---

## 桌面应用

Claude Code 桌面应用提供一个带可视化 diff 审查、并行会话和集成 connector 的独立应用。在 macOS 和 Windows 上可用（Pro、Max、Team 和 Enterprise 套餐）。

### 安装

从 [claude.ai](https://claude.ai) 为你的平台下载：
- **macOS**：通用构建（Apple Silicon 和 Intel）
- **Windows**：提供 x64 和 ARM64 安装程序

设置说明请参见 [Desktop Quickstart](https://code.claude.com/docs/en/desktop-quickstart)。

### 从 CLI 接力

把你当前的 CLI 会话转移到桌面应用：

```
/desktop
```

### 核心功能

| 功能 | 说明 |
|---------|-------------|
| **Diff 视图** | 逐文件的可视化审查，带行内评论；Claude 读取评论并修订 |
| **App 预览** | 自动启动开发服务器，带内嵌浏览器以进行实时验证 |
| **PR 监控** | GitHub CLI 集成，自动修复 CI 失败并在检查通过时自动合并 |
| **并行会话** | 侧边栏中的多个会话，带自动 Git worktree 隔离 |
| **定时任务** | 在应用打开时运行的重复任务（每小时、每天、工作日、每周） |
| **丰富渲染** | 代码、markdown 和图表渲染，带语法高亮；GitHub-Flavored-Markdown 任务列表复选框（`- [ ]` / `- [x]`）渲染为复选框（v2.1.149+） |

### App 预览配置

在 `.claude/launch.json` 中配置开发服务器行为：

```json
{
  "command": "npm run dev",
  "port": 3000,
  "readyPattern": "ready on",
  "persistCookies": true
}
```

### Connectors

连接外部服务以获得更丰富的上下文：

| Connector | 能力 |
|-----------|------------|
| **GitHub** | PR 监控、issue 跟踪、代码审查 |
| **Slack** | 通知、频道上下文 |
| **Linear** | issue 跟踪、sprint 管理 |
| **Notion** | 文档、知识库访问 |
| **Asana** | 任务管理、项目跟踪 |
| **Calendar** | 日程感知、会议上下文 |

> **注意**：Connector 不适用于远程（云端）会话。

### 远程和 SSH 会话

- **远程会话**：运行在 Anthropic 云基础设施上；即使应用关闭也继续运行。可从 claude.ai/code 或 Claude 手机 app 访问
- **SSH 会话**：通过 SSH 连接到远程机器，完全访问远程文件系统和工具。Claude Code 必须安装在远程机器上

### Desktop 中的权限模式

桌面应用支持与 CLI 相同的 4 种权限模式：

| 模式 | 行为 |
|------|----------|
| **Ask permissions**（默认） | 审阅并批准每一次编辑和命令 |
| **Auto accept edits** | 文件编辑自动批准；命令需要手动批准 |
| **Plan mode** | 在做出任何改动前审阅方案 |
| **Bypass permissions** | 自动执行（仅限沙盒，由管理员控制） |

### 企业功能

- **管理控制台**：为组织控制 Code 标签访问和权限设置
- **MDM 部署**：在 macOS 上通过 MDM 或在 Windows 上通过 MSIX 部署
- **SSO 集成**：要求组织成员单点登录
- **受管设置**：集中管理团队配置和模型可用性

---

## 任务列表

Task List 功能提供跨 context compaction（当对话历史被裁剪以适配上下文窗口时）持久存在的任务跟踪。

### 切换任务列表

在会话期间按 `Ctrl+T` 切换任务列表视图的开关。

### 持久任务

任务跨 context compaction 持久存在，确保在对话上下文被裁剪时长期运行的工作项不会丢失。这对复杂的多步骤实现尤其有用。

### 命名任务目录

使用 `CLAUDE_CODE_TASK_LIST_ID` 环境变量创建跨会话共享的命名任务目录：

```bash
export CLAUDE_CODE_TASK_LIST_ID=my-project-sprint-3
```

这允许多个会话共享同一个任务列表，对团队工作流或多会话项目很有用。

---

## 提示词建议

Prompt Suggestions 根据你的 git 历史和当前对话上下文显示灰色的示例命令。

### 工作方式

- 建议以灰色文本出现在你的输入提示下方
- 按 `Tab` 接受建议
- 按 `Enter` 接受并立即提交
- 建议是上下文感知的，取材于 git 历史和对话状态

### 禁用提示词建议

```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
```

---

## Git 工作树（Git Worktrees）

Git Worktrees 允许你在一个隔离的 worktree 中启动 Claude Code，从而在不同分支上并行工作而无需 stash 或切换。

### 在 Worktree 中启动

```bash
# Start Claude Code in an isolated worktree
claude --worktree
# or
claude -w
```

### Worktree 位置

Worktree 创建于：
```
<repo>/.claude/worktrees/<name>
```

### Monorepo 的 Sparse Checkout

使用 `worktree.sparsePaths` 设置在 monorepo 中执行 sparse-checkout，减少磁盘使用和克隆时间：

```json
{
  "worktree": {
    "sparsePaths": ["packages/my-package", "shared/"]
  }
}
```

### 基分支引用（`worktree.baseRef`）

**`worktree.baseRef`**（v2.1.133 添加）—— 控制 `claude --worktree` 是从 `origin/<default>` 还是从本地 `HEAD` 分支。

- `"fresh"`（默认）—— 从 `origin/<default-branch>` 分支，忽略本地未推送的 commit。**这恢复了 v2.1.128 引入的行为**，因此在 v2.1.128 之后依赖本地 HEAD 分支的用户必须重新选择启用。
- `"head"` —— 从本地 `HEAD` 分支，保留未推送的 commit。

在 `~/.claude/settings.json` 中设置：

```json
{ "worktree": { "baseRef": "head" } }
```

### 后台会话隔离（`worktree.bgIsolation`）

**`worktree.bgIsolation`**（v2.1.143 添加）—— 控制后台会话（例如来自 `/bg`、`claude --bg` 或 Agent View）是获得自己的 worktree 还是直接编辑前台工作副本。

- *（默认）* —— 后台会话在 `<repo>/.claude/worktrees/` 下创建一个隔离的 worktree，方式与 `--worktree` 相同。
- `"none"` —— 后台会话直接编辑当前工作副本。当 worktree 不切实际时（例如沉重的原生构建产物），或当后台 agent 必须与前台会话协调编辑时使用此项。

```json
{ "worktree": { "bgIsolation": "none" } }
```

权衡：`"none"` 移除了 worktree 隔离的安全网 —— 后台和前台会话的并发编辑可能在实时工作副本中产生合并冲突。

### Worktree 工具和 Hooks

| 项目 | 说明 |
|------|-------------|
| `EnterWorktree` | 进入 worktree 的工具；截至 v2.1.157 它可以在会话中途在 Claude 托管的 worktree 之间切换 |
| `ExitWorktree` | 退出并清理当前 worktree 的工具 |
| `WorktreeCreate` | 创建 worktree 时触发的 hook 事件 |
| `WorktreeRemove` | 移除 worktree 时触发的 hook 事件 |

截至 v2.1.157，Claude 托管的 worktree 在 agent 完成时保持解锁状态，因此 `git worktree remove`/`prune` 可以清理它们。

### 自动清理

如果 worktree 中没有做任何改动，它会在会话结束时被自动清理。

### 使用场景

- 在某个功能分支上工作而保持 main 分支不动
- 在隔离环境中运行测试而不影响工作目录
- 在一个用完即弃的环境中尝试实验性改动
- 在 monorepo 中对特定包做 sparse-checkout 以加快启动

---

## 沙盒

沙盒为 Claude Code 执行的 Bash 命令提供操作系统级的文件系统和网络隔离。这与权限规则互补，提供了额外的安全层。

### 启用沙盒

**Slash command**：
```
/sandbox
```

**CLI 参数**：
```bash
claude --sandbox       # Enable sandboxing
claude --no-sandbox    # Disable sandboxing
```

### 配置设置

| 设置 | 说明 |
|---------|-------------|
| `sandbox.enabled` | 启用或禁用沙盒 |
| `sandbox.failIfUnavailable` | 如果沙盒无法激活则失败 |
| `sandbox.filesystem.allowWrite` | 允许写入访问的路径 |
| `sandbox.filesystem.allowRead` | 允许读取访问的路径 |
| `sandbox.filesystem.denyRead` | 拒绝读取访问的路径 |
| `sandbox.network.allowedDomains` | 允许 Bash 启动的进程访问的域（支持 `*.` 通配符） |
| `sandbox.network.deniedDomains` | 即使 `allowedDomains` 通配符本会放行也要阻止的域（v2.1.113+） |
| `sandbox.enableWeakerNetworkIsolation` | 在 macOS 上启用较弱的网络隔离 |
| `sandbox.bwrapPath` | （v2.1.133+，Linux/WSL）`bubblewrap` 二进制的路径。默认：`$PATH` 查找。 |
| `sandbox.socatPath` | （v2.1.133+，Linux/WSL）`socat` 二进制的路径。默认：`$PATH` 查找。 |
| `sandbox.credentials` | （v2.1.187+）阻止沙盒命令读取凭据文件和密钥环境变量。 |
| `sandbox.allowAppleEvents` | （v2.1.181+，macOS）选择启用以让沙盒命令发送 Apple Events。 |

**Linux/WSL 二进制路径**（v2.1.133+）—— 把 Claude Code 指向非标准的安装位置：

```json
{
  "sandbox": {
    "bwrapPath": "/opt/bubblewrap/bin/bwrap",
    "socatPath": "/opt/socat/bin/socat"
  }
}
```

`deniedDomains` 覆盖宽泛通配符的示例（v2.1.113+）：

```json
{
  "sandbox": {
    "network": {
      "allowedDomains": ["*.example.com"],
      "deniedDomains": ["evil.example.com"]
    }
  }
}
```

通配符让 `example.com` 上的一切通过，但 `deniedDomains` 仍然阻止那个被特别命名的主机。

### 示例配置

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "filesystem": {
      "allowWrite": ["/Users/me/project"],
      "allowRead": ["/Users/me/project", "/usr/local/lib"],
      "denyRead": ["/Users/me/.ssh", "/Users/me/.aws"]
    },
    "enableWeakerNetworkIsolation": true
  }
}
```

### 工作方式

- Bash 命令在一个文件系统访问受限的沙盒环境中运行
- 网络访问可以被隔离，以防止意外的外部连接
- 与权限规则协同工作，实现纵深防御
- 在 macOS 上，使用 `sandbox.enableWeakerNetworkIsolation` 进行网络限制（macOS 上不提供完整的网络隔离）

### 使用场景

- 安全地运行不受信任或生成的代码
- 防止意外修改项目之外的文件
- 在自动化任务期间限制网络访问

---

## 企业受管设置

受管设置使企业管理员能够使用平台原生的管理工具在整个组织中部署 Claude Code 配置。

### 部署方式

| 平台 | 方式 | 起始版本 |
|----------|--------|-------|
| macOS | 受管 plist 文件（MDM） | v2.1.51+ |
| Windows | Windows Registry | v2.1.51+ |
| 跨平台 | 受管配置文件 | v2.1.51+ |
| 跨平台 | 受管 drop-in（`managed-settings.d/` 目录） | v2.1.83+ |

### 受管 Drop-in

自 v2.1.83 起，管理员可以把多个受管设置文件部署到一个 `managed-settings.d/` 目录中。文件按字母顺序合并，允许跨团队的模块化配置：

```
~/.claude/managed-settings.d/
  00-org-defaults.json
  10-team-policies.json
  20-project-overrides.json
```

### 可用的受管设置

| 设置 | 说明 |
|---------|-------------|
| `disableBypassPermissionsMode` | 阻止用户启用 bypass permissions |
| `availableModels` | 限制用户可以选择哪些模型 |
| `enforceAvailableModels` | （v2.1.175）当为 `true` 时，`availableModels` 允许列表*也*约束 **Default** 模型 —— 如果配置的默认模型不在列表中，Claude Code 会回退到第一个允许的模型。用户和项目设置不能再放宽受管的 `availableModels` 列表。 |
| `allowedChannelPlugins` | 控制允许哪些 channel 插件 |
| `autoMode.environment` | 为 auto mode 配置可信基础设施 |
| `wslInheritsWindowsSettings` | 仅 Windows/WSL（v2.1.118+）：当为 `true` 时，运行在 WSL 内的 Claude Code 继承 Windows 宿主的受管设置，使通过 Registry/MDM 部署的企业策略在 Windows 和 WSL shell 中一致地生效 |
| `parentSettingsBehavior` | （v2.1.133+，admin 层级）控制 SDK 的 `managedSettings` 如何与父进程设置合并。`"first-wins"` 保持现有优先级（冲突时较早的设置胜出）；`"merge"` 深度合并值。 |
| 自定义策略 | 组织专用的权限和工具策略 |

### 示例：macOS Plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>disableBypassPermissionsMode</key>
  <true/>
  <key>availableModels</key>
  <array>
    <string>claude-sonnet-4-6</string>
    <string>claude-haiku-4-5</string>
  </array>
</dict>
</plist>
```

---

## 配置与设置

### 配置文件位置

1. **全局配置**：`~/.claude/config.json`
2. **项目配置**：`./.claude/config.json`
3. **用户配置**：`~/.config/claude-code/settings.json`

### 完整配置示例

**核心高级功能配置：**

```json
{
  "permissions": {
    "mode": "default"
  },
  "hooks": {
    "PreToolUse:Edit": "eslint --fix ${file_path}",
    "PostToolUse:Write": "~/.claude/hooks/security-scan.sh"
  },
  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"]
      }
    }
  }
}
```

**扩展配置示例：**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Bash(git log:*)", "Read"],
    "disallowedTools": ["Bash(rm -rf:*)"]
  },

  "hooks": {
    "PreToolUse": [{ "matcher": "Edit", "hooks": ["eslint --fix ${file_path}"] }],
    "PostToolUse": [{ "matcher": "Write", "hooks": ["~/.claude/hooks/security-scan.sh"] }],
    "Stop": [{ "hooks": ["~/.claude/hooks/notify.sh"] }]
  },

  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      }
    }
  }
}
```

### 回退模型（`fallbackModel`）

`fallbackModel` 设置让你配置**最多三个**回退模型，当主模型过载或不可用时按顺序尝试。

```json
{
  "fallbackModel": ["claude-opus-4-8", "claude-sonnet-4-6", "claude-haiku-4-5"]
}
```

截至 **v2.1.166**，`--fallback-model` 参数也适用于交互式会话（不仅仅是无头）。发生回退时，Claude Code 会对一个意外的不可重试错误重试一次；认证、速率限制、请求大小和传输错误仍然立即失败。

### 环境变量

用环境变量覆盖配置：

```bash
# Model selection
export ANTHROPIC_MODEL=claude-opus-4-8
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-8
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-6
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5

# API configuration
export ANTHROPIC_API_KEY=sk-ant-...

# Thinking configuration
export MAX_THINKING_TOKENS=16000
export CLAUDE_CODE_EFFORT_LEVEL=high   # low, medium, high, xhigh (Opus 4.8/4.7), or max — default is high on Opus 4.8 (supported on Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6)

# Feature toggles
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=true
export CLAUDE_CODE_DISABLE_CRON=1
export CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS=true
export CLAUDE_CODE_DISABLE_TERMINAL_TITLE=true
export CLAUDE_CODE_DISABLE_1M_CONTEXT=true
export CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK=true
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
export CLAUDE_CODE_ENABLE_TASKS=true
export CLAUDE_CODE_SIMPLE=true              # Set by --bare flag

# MCP configuration
export MAX_MCP_OUTPUT_TOKENS=50000
export ENABLE_TOOL_SEARCH=true

# Prompt caching
export ENABLE_PROMPT_CACHING_1H=1      # Use 1-hour prompt cache TTL (default is 5 min)

# Task management
export CLAUDE_CODE_TASK_LIST_ID=my-project-tasks

# Agent teams (experimental)
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Subagent and plugin configuration
export CLAUDE_CODE_SUBAGENT_MODEL=sonnet
export CLAUDE_CODE_PLUGIN_SEED_DIR=./my-plugins
export CLAUDE_CODE_NEW_INIT=1

# Subprocess and streaming
export CLAUDE_CODE_SUBPROCESS_ENV_SCRUB="SECRET_KEY,DB_PASSWORD"
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=80
export CLAUDE_STREAM_IDLE_TIMEOUT_MS=30000
export ANTHROPIC_CUSTOM_MODEL_OPTION=my-custom-model
export SLASH_COMMAND_TOOL_CHAR_BUDGET=50000

# Output and package manager (v2.1.129+)
export CLAUDE_CODE_FORCE_SYNC_OUTPUT=1                      # Force synchronous output for terminals where auto-detect misses (Emacs eat, etc.)
export CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE=1            # Enable background upgrades for Homebrew/WinGet installs
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1         # Opt in to /v1/models gateway discovery when ANTHROPIC_BASE_URL is set

# Windows PowerShell tool (v2.1.143+) — default-on for Bedrock/Vertex/Foundry on Windows
export CLAUDE_CODE_USE_POWERSHELL_TOOL=0                    # Disable the PowerShell tool entirely
export CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY=1    # Honor system ExecutionPolicy instead of `-ExecutionPolicy Bypass`

# Workload identity federation (v2.1.141+)
export ANTHROPIC_WORKSPACE_ID=ws_abc123                     # Scope the federated token to a specific workspace when the rule covers multiple

# Stop hook safety cap (v2.1.143+)
export CLAUDE_CODE_STOP_HOOK_BLOCK_CAP=8                    # Max consecutive Stop-hook blocks before the session ends with a warning. Set 0 to disable the cap.
```

> **v2.1.108**：`ENABLE_PROMPT_CACHING_1H=1` —— 使用 1 小时的 prompt cache TTL，而不是默认的 5 分钟 TTL。减少长而稳定的会话中的缓存未命中。（v2.1.129 修复了一个 1 小时 TTL 被静默降级为 5 分钟的回归。）

> **v2.1.129**：`CLAUDE_CODE_FORCE_SYNC_OUTPUT=1` 为能力自动检测失败的终端（例如 Emacs `eat`）强制同步输出。`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE=1` 为 Homebrew/WinGet 安装启用后台升级，否则它们永远不会自动更新。

### 配置管理命令

```
User: /config
[Opens interactive configuration menu]
```

`/config` 命令提供一个交互式菜单来切换设置，例如：
- Extended thinking 开/关
- Verbose 输出
- 权限模式
- 模型选择

在交互式菜单中，按 Enter 或 Space 更改选中的设置，按 Esc 保存并关闭（v2.1.183+）。

你也可以不打开菜单，直接从提示词设置某个设置：

```bash
/config thinking=false      # set a single setting inline (v2.1.181+)
/config --help              # list available shorthand keys (v2.1.183+)
```

`key=value` 简写在交互式会话中、配合 `-p` 以及在 Remote Control 中都有效。

### 每个项目的配置

在你的项目中创建 `.claude/config.json`：

```json
{
  "hooks": {
    "PreToolUse": [{ "matcher": "Bash", "hooks": ["npm test && npm run lint"] }]
  },
  "permissions": {
    "mode": "default"
  },
  "mcp": {
    "servers": {
      "project-db": {
        "command": "mcp-postgres",
        "env": {
          "DATABASE_URL": "${PROJECT_DB_URL}"
        }
      }
    }
  }
}
```

---

## Agent Teams

Agent Teams 是一个实验性功能，它使多个 Claude Code 实例能够协作完成一个任务。它默认禁用。

### 启用 Agent Teams

通过环境变量或设置启用：

```bash
# Environment variable
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

或添加到你的 settings JSON：

```json
{
  "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
}
```

### Agent Teams 如何工作

- 一个 **team lead** 协调整体任务并把子任务委派给队友
- **队友** 独立工作，各自拥有自己的上下文窗口
- 一个 **共享任务列表** 使团队成员之间能够自我协调
- 使用 subagent 定义（`.claude/agents/` 或 `--agents` 参数）来定义队友的角色和专长

### 显示模式

Agent Teams 支持两种显示模式，用 `--teammate-mode` 参数配置：

| 模式 | 说明 |
|------|-------------|
| `in-process`（默认） | 队友在同一个终端进程内运行 |
| `tmux` | 每个队友获得一个专用的分屏窗格（需要 tmux 或 iTerm2） |
| `auto` | 自动选择最佳显示模式 |

```bash
# Use tmux split panes for teammate display
claude --teammate-mode tmux

# Explicitly use in-process mode
claude --teammate-mode in-process
```

### 使用场景

- 不同队友处理不同模块的大型重构任务
- 并行的代码审查和实现
- 跨代码库的协调多文件改动

> **注意**：Agent Teams 是实验性的，可能在未来版本中改变。完整参考见 [code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams)。

---

## 最佳实践

### Planning Mode
- ✅ 用于复杂的多步骤任务
- ✅ 在批准前审阅计划
- ✅ 需要时修改计划
- ❌ 不要用于简单任务

### Extended Thinking
- ✅ 用于架构决策
- ✅ 用于复杂问题求解
- ✅ 审阅思考过程
- ❌ 不要用于简单查询

### Background Tasks
- ✅ 用于长时间运行的操作
- ✅ 监控任务进度
- ✅ 优雅地处理任务失败
- ❌ 不要启动过多的并发任务

### Permissions
- ✅ 用 `plan` 做代码审查（只读）
- ✅ 用 `default` 做交互式开发
- ✅ 用 `acceptEdits` 做自动化工作流
- ✅ 用 `auto` 做带安全护栏的自主工作
- ❌ 除非绝对必要，不要用 `bypassPermissions`

### Sessions
- ✅ 为不同任务使用独立的会话
- ✅ 保存重要的会话状态
- ✅ 清理旧会话
- ❌ 不要在一个会话中混入不相关的工作

---

## 更多资源

关于 Claude Code 和相关功能的更多信息：

- [Official Interactive Mode Documentation](https://code.claude.com/docs/en/interactive-mode)
- [Official Headless Mode Documentation](https://code.claude.com/docs/en/headless)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Checkpoints Guide](../08-checkpoints/) - 会话管理与回退
- [Slash Commands](../01-slash-commands/) - 命令参考
- [Memory Guide](../02-memory/) - 持久上下文
- [Skills Guide](../03-skills/) - 自主能力
- [Subagents Guide](../04-subagents/) - 委派任务执行
- [MCP Guide](../05-mcp/) - 外部数据访问
- [Hooks Guide](../06-hooks/) - 事件驱动自动化
- [Plugins Guide](../07-plugins/) - 捆绑扩展
- [Official Scheduled Tasks Documentation](https://code.claude.com/docs/en/scheduled-tasks)
- [Official Chrome Integration Documentation](https://code.claude.com/docs/en/chrome)
- [Official Remote Control Documentation](https://code.claude.com/docs/en/remote-control)
- [Official Keybindings Documentation](https://code.claude.com/docs/en/keybindings)
- [Official Desktop App Documentation](https://code.claude.com/docs/en/desktop)
- [Official Agent Teams Documentation](https://code.claude.com/docs/en/agent-teams)

---

**最后更新**：2026 年 6 月 24 日
**Claude Code 版本**：2.1.187
**来源**：
- https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- https://docs.anthropic.com/en/docs/claude-code/settings
- https://code.claude.com/docs/en/troubleshooting
- https://code.claude.com/docs/en/changelog#2-1-175
- https://code.claude.com/docs/en/permission-modes
- https://code.claude.com/docs/en/interactive-mode
- https://code.claude.com/docs/en/settings
- https://code.claude.com/docs/en/cli-reference
- https://code.claude.com/docs/en/model-config
- https://www.anthropic.com/news/claude-opus-4-8
- https://claude.com/blog/introducing-routines-in-claude-code
- https://github.com/anthropics/claude-code/releases/tag/v2.1.117
- https://github.com/anthropics/claude-code/releases/tag/v2.1.139
- https://github.com/anthropics/claude-code/releases/tag/v2.1.154
- https://code.claude.com/docs/en/overview
- https://code.claude.com/docs/en/sub-agents
**兼容模型**：Claude Sonnet 4.6、Claude Opus 4.8、Claude Haiku 4.5

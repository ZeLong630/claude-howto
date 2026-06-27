<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../../resources/logos/claude-howto-logo.svg">
</picture>

# Checkpoints 与 Rewind

Checkpoints 可以让你保存对话状态，并在 Claude Code 会话中回退到之前的时间点。无论你是在探索不同方案、修复错误，还是比较多个实现路径，这个能力都非常有用。

## 概览

Checkpoints 会保存会话状态，让你可以回退到之前的时间点，从而安全地试验和探索多种方案。它们本质上是当前对话状态的快照，包含：
- 所有已交换的消息
- 已做出的文件修改
- 工具使用历史
- 当前会话上下文

当你在尝试不同实现方式、从错误中恢复，或比较替代方案时，checkpoints 非常有用。

## 核心概念

| 概念 | 说明 |
|---------|-------------|
| **Checkpoint** | 保存消息、文件和上下文的对话快照 |
| **Rewind** | 回到之前的 checkpoint，并丢弃之后的更改 |
| **Branch Point** | 从同一个 checkpoint 出发，探索多个方案 |

## 访问 Checkpoints

你可以通过两种主要方式访问和管理 checkpoints：

### 使用键盘快捷键

按两次 `Esc`（`Esc` + `Esc`）打开 checkpoint 界面并浏览已保存的 checkpoints。

### 使用 Slash Command

使用 `/rewind` 命令（别名：`/checkpoint`）快速进入：

```bash
# Open rewind interface
/rewind

# Or use the alias
/checkpoint
```

## Rewind 选项

回退时，你会看到一个包含五个选项的菜单：

1. **恢复代码和对话**（Restore code and conversation）-- 把文件和消息都恢复到那个 checkpoint
2. **恢复对话**（Restore conversation）-- 只回退消息，保留当前代码不变
3. **恢复代码**（Restore code）-- 只回退文件修改，保留完整对话历史
4. **从这里开始总结**（Summarize from here）-- 把从这里往后的对话压缩成 AI 生成的摘要，释放上下文窗口空间。所选时间点之前的消息保持不变。磁盘上的文件不会被改动。原始消息仍会保留在会话记录（transcript）中。你也可以额外提供指令，让摘要聚焦于特定主题。
5. **算了**（Never mind）-- 取消并返回当前状态

> **注意**：在恢复对话或进行总结之后，所选消息的原始提示词会被恢复到输入框中，便于你重新发送或编辑。

## 自动 Checkpoints

Claude Code 会自动为你创建 checkpoints：

- **每次用户提示** - 每次用户输入都会创建一个新 checkpoint
- **持久保存** - checkpoints 会跨会话保留
- **自动清理** - checkpoints 会在 30 天后自动清理

这意味着你随时都可以回退到对话中之前的任意节点，从几分钟前到几天前都可以。

## 使用场景

| 场景 | 工作流 |
|----------|----------|
| **探索不同方案** | 保存 → 试方案 A → 保存 → Rewind → 试方案 B → 比较 |
| **安全重构** | 保存 → 重构 → 测试 → 如果失败：Rewind |
| **A/B 测试** | 保存 → 设计 A → 保存 → Rewind → 设计 B → 比较 |
| **错误恢复** | 发现问题 → Rewind 到最后一个正常状态 |

## 如何使用 Checkpoints

### 查看和回退

按两次 `Esc`，或者使用 `/rewind` 打开 checkpoint 浏览器。你会看到一个带时间戳的可用 checkpoint 列表。选择任意一个 checkpoint，即可回退到该状态。

### Checkpoint 详情

每个 checkpoint 会显示：
- 创建时间戳
- 被修改的文件
- 对话中的消息数量
- 使用过的工具

## 实战示例

### 示例 1：探索不同方案

```
User: Let's add a caching layer to the API

Claude: I'll add Redis caching to your API endpoints...
[Makes changes at checkpoint A]

User: Actually, let's try in-memory caching instead

Claude: I'll rewind to explore a different approach...
[User presses Esc+Esc and rewinds to checkpoint A]
[Implements in-memory caching at checkpoint B]

User: Now I can compare both approaches
```

### 示例 2：从错误中恢复

```
User: Refactor the authentication module to use JWT

Claude: I'll refactor the authentication module...
[Makes extensive changes]

User: Wait, that broke the OAuth integration. Let's go back.

Claude: I'll help you rewind to before the refactoring...
[User presses Esc+Esc and selects the checkpoint before the refactor]

User: Let's try a more conservative approach this time
```

### 示例 3：安全试验

```
User: Let's try rewriting this in a functional style
[Creates checkpoint before experiment]

Claude: [Makes experimental changes]

User: The tests are failing. Let's rewind.
[User presses Esc+Esc and rewinds to the checkpoint]

Claude: I've rewound the changes. Let's try a different approach.
```

### 示例 4：分支式探索

```
User: I want to compare two database designs
[Takes note of checkpoint - call it "Start"]

Claude: I'll create the first design...
[Implements Schema A]

User: Now let me go back and try the second approach
[User presses Esc+Esc and rewinds to "Start"]

Claude: Now I'll implement Schema B...
[Implements Schema B]

User: Great! Now I have both schemas to choose from
```

## Checkpoint 保留策略

Claude Code 会自动管理你的 checkpoints：

- 每次用户提示都会自动创建 checkpoint
- 旧 checkpoint 最多保留 30 天
- 系统会自动清理 checkpoints，避免存储无限增长

## 工作流模式

### 探索时的分支策略

当你在探索多个方案时：

```
1. Start with initial implementation → Checkpoint A
2. Try Approach 1 → Checkpoint B
3. Rewind to Checkpoint A
4. Try Approach 2 → Checkpoint C
5. Compare results from B and C
6. Choose best approach and continue
```

### 安全重构模式

当你在做重大改动时：

```
1. Current state → Checkpoint (auto)
2. Start refactoring
3. Run tests
4. If tests pass → Continue working
5. If tests fail → Rewind and try different approach
```

## 最佳实践

因为 checkpoints 是自动创建的，所以你可以专注于工作，而不用担心手动保存状态。不过，下面这些习惯仍然值得记住：

### 如何高效使用 Checkpoints

✅ **应该做：**
- 回退前先查看可用的 checkpoints
- 当你想探索不同方向时使用 rewind
- 保留 checkpoints 以便比较不同方案
- 理解每个 rewind 选项的作用（恢复代码和对话、恢复对话、恢复代码，或总结）

❌ **不要做：**
- 仅依赖 checkpoints 来保存代码
- 指望 checkpoints 跟踪外部文件系统的变化
- 把 checkpoints 当作 git commit 的替代品

## 配置

Checkpoints 是 Claude Code 的内置默认行为，无需任何配置即可启用。每次用户提示都会自动创建一个 checkpoint。

唯一与 checkpoint 相关的设置是 `cleanupPeriodDays`，它控制会话和 checkpoints 的保留时长：

```json
{
  "cleanupPeriodDays": 30
}
```

- `cleanupPeriodDays`：保留会话历史和 checkpoints 的天数（默认：`30`）

> **v2.1.117 更新**：`cleanupPeriodDays` 现在管理四个磁盘缓存的保留时长，而不仅仅是 checkpoints：
>
> - 会话 checkpoints
> - `~/.claude/tasks/` — 持久化的任务列表
> - `~/.claude/shell-snapshots/` — 捕获的 shell 环境快照
> - `~/.claude/backups/` — 滚动保存的设置 / CLAUDE.md 备份
>
> 现在单个设置即可在相同天数后统一清理这全部四个目录。

## 局限性

Checkpoints 存在以下局限：

- **不跟踪 Bash 命令产生的变化** - 文件系统上的 `rm`、`mv`、`cp` 等操作不会被 checkpoints 捕获
- **不跟踪外部变化** - 在 Claude Code 之外（在你的编辑器、终端等中）做出的更改不会被捕获
- **不能替代版本控制** - 对代码库进行永久、可审计的改动时，请使用 git

## 故障排查

### 找不到 Checkpoints

**问题**：找不到预期的 checkpoint

**解决方案**：
- 检查 checkpoints 是否已被清除
- 检查磁盘空间
- 确保 `cleanupPeriodDays` 设置得足够大（默认：30 天）

### Rewind 失败

**问题**：无法回退到某个 checkpoint

**解决方案**：
- 确保没有未提交的更改产生冲突
- 检查 checkpoint 是否已损坏
- 尝试回退到另一个 checkpoint

## 与 Git 的集成

Checkpoints 是对 git 的补充（但不能替代 git）：

| 特性 | Git | Checkpoints |
|---------|-----|-------------|
| 作用范围 | 文件系统 | 对话 + 文件 |
| 持久性 | 永久 | 基于会话 |
| 粒度 | Commits | 任意时间点 |
| 速度 | 较慢 | 即时 |
| 可共享 | 是 | 有限 |

两者结合使用：
1. 用 checkpoints 进行快速试验
2. 用 git commits 保存最终的更改
3. 在执行 git 操作前创建 checkpoint
4. 把成功的 checkpoint 状态提交到 git

## 快速开始

### 基本工作流

1. **正常工作** - Claude Code 会自动创建 checkpoints
2. **想回退？** - 按两次 `Esc` 或使用 `/rewind`
3. **选择 checkpoint** - 从列表中选择要回退到的 checkpoint
4. **选择要恢复的内容** - 在“恢复代码和对话”“恢复对话”“恢复代码”“从这里开始总结”或取消之间选择
5. **继续工作** - 你已回到那个时间点

### 键盘快捷键

- **`Esc` + `Esc`** - 打开 checkpoint 浏览器
- **`/rewind`** - 访问 checkpoints 的另一种方式
- **`/checkpoint`** - `/rewind` 的别名

## 什么时候该 rewind：上下文监控

Checkpoints 可以让你回退——但你怎么知道*什么时候*该回退？随着对话越来越长，Claude 的上下文窗口逐渐被填满，模型质量会悄然下降。你可能在不知不觉中，正在让一个“半盲”的模型给你产出代码。

**[cc-context-stats](https://github.com/luongnv89/cc-context-stats)** 通过在 Claude Code 状态栏中加入实时的**上下文区域（context zones）**来解决这个问题。它会跟踪你在上下文窗口中所处的位置——从 **Plan**（绿色，适合规划和编码）经过 **Code**（黄色，避免开始新的规划）到 **Dump**（橙色，收尾并回退）。当你看到区域发生变化时，就知道该创建 checkpoint 并重新开始，而不是带着已经退化的输出硬撑下去。

## 相关概念

- **[高级功能](../09-advanced-features/)** - Planning 模式及其他高级能力
- **[内存管理](../02-memory/)** - 管理对话历史和上下文
- **[Slash Commands](../01-slash-commands/)** - 用户主动调用的快捷命令
- **[Hooks](../06-hooks/)** - 事件驱动的自动化
- **[Plugins](../07-plugins/)** - 打包的扩展包

## 更多资源

- [官方 Checkpointing 文档](https://code.claude.com/docs/en/checkpointing)
- [高级功能指南](../09-advanced-features/) - 扩展思考及其他能力

## 总结

Checkpoints 是 Claude Code 的一项自动功能，让你可以安全地探索不同方案，而不必担心丢失工作成果。每次用户提示都会自动创建一个新 checkpoint，因此你可以回退到会话中之前的任意时间点。

主要优势：
- 无所顾忌地试验多种方案
- 快速从错误中恢复
- 并排比较不同的解决方案
- 安全地与版本控制系统集成

请记住：checkpoints 不能替代 git。用 checkpoints 进行快速试验，用 git 保存永久的代码更改。

---

**最后更新**：2026 年 6 月 2 日
**Claude Code 版本**：2.1.160
**来源**：
- https://code.claude.com/docs/en/checkpointing
- https://code.claude.com/docs/en/settings
- https://github.com/anthropics/claude-code/releases/tag/v2.1.117
**兼容模型**：Claude Sonnet 4.6、Claude Opus 4.8、Claude Haiku 4.5

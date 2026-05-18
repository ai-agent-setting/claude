<!-- last-reviewed: 2026-05-19 -->
# Subagents

A subagent is a separate Claude instance running in its own context window.
Use subagents to delegate complex or lengthy investigations without polluting the main conversation.

> Reference: https://code.claude.com/docs/en/sub-agents

---

## Why Use Subagents?

| Problem | How Subagents Help |
|---|---|
| Full-codebase analysis consumes main context | Analyze in a separate context, return only the summary |
| Multiple independent tasks run sequentially | Run subagents in parallel |
| Reviewer is biased toward code they just wrote | A fresh context gives an unbiased review |

---

## How to Invoke Subagents

Ask Claude in natural language:

```
Investigate how the auth system handles token refresh using a subagent.
Return only a summary report to the main conversation.
```

Or use the `context: fork` option in a skill to fork automatically:

```yaml
---
context: fork
agent: Explore   # optional: specify a built-in agent type
---
```

### CLI `--agents` Flag

Define session-scoped subagents via the CLI without creating files:

```bash
claude --agents '{"name": "reviewer", "description": "Code reviewer", "tools": ["Read", "Bash"]}'
```

### `/agents` GUI Command

The `/agents` slash command opens a GUI with:
- **Running** tab: currently active subagents
- **Library** tab: saved custom subagents
- **Generate with Claude**: auto-generate a subagent definition from a description
- Model, tool, and memory configuration per agent

### Built-in Agent Types

| Type | Description |
|---|---|
| `general-purpose` | General agent. Default. |
| `Explore` | Optimized for fast file and pattern search in a codebase |
| `Plan` | Specialized for software architecture and implementation planning |
| `statusline-setup` | Configures the status line (auto-invoked via `/statusline`) |
| `claude-code-guide` | Answers Claude Code feature questions (auto-invoked on related queries) |

---

## Custom Subagents (`.claude/agents/`)

Define reusable subagents as files.

### File Format (Single-file, Recommended)

```
.claude/
└── agents/
    ├── code-reviewer.md       # single-file format (recommended)
    └── security-auditor.md
```

Each file is YAML frontmatter + instruction body:

```markdown
---
name: code-reviewer
description: Performs an independent code review from a fresh perspective
tools:
  - Read
  - Bash
disallowedTools:             # denylist — applied before tools allowlist
  - Write
model: sonnet                # Short form: opus / sonnet / haiku
skills:                      # Skills to preload at startup
  - review
  - summarize
permissionMode: auto         # default / acceptEdits / auto / dontAsk / bypassPermissions / plan
maxTurns: 20                 # Maximum number of turns before stopping
mcpServers:                  # MCP servers to connect
  - name: github
    command: npx mcp-github
hooks:                       # Hooks scoped to this subagent
  PostToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "npx prettier --write $CLAUDE_FILE_PATH"
memory: user                 # Auto memory scope: user / project / local
background: false            # Run as background agent
effort: medium               # low / medium / high / xhigh / max
isolation: worktree          # Run in isolated git worktree
color: blue                  # Display color in the UI
initialPrompt: |             # Injected at the start of the session
  You are a strict code reviewer. Focus on correctness first.
---

Independently review $ARGUMENTS.

Check for:
- Bugs and logic errors
- Security vulnerabilities
- Performance issues
- Compliance with CLAUDE.md conventions

Return a structured report.
```

> Note: `.claude/agents/<name>/SKILL.md` format is backward-compatible but the single-file format (`.claude/agents/<name>.md`) is recommended.

### Frontmatter Fields Reference

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Agent identifier (lowercase, hyphens) |
| `description` | Yes | Used by Claude to decide when to invoke the agent |
| `tools` | | Allowlist of tools the agent can use |
| `disallowedTools` | | Denylist — applied before `tools` allowlist |
| `model` | | Model to use: `opus` / `sonnet` / `haiku` or full model ID |
| `skills` | | Skills preloaded at startup (full content injected, not just description) |
| `permissionMode` | | `default` / `acceptEdits` / `auto` / `dontAsk` / `bypassPermissions` / `plan` |
| `maxTurns` | | Maximum turns before the agent stops |
| `mcpServers` | | MCP servers available to this agent (not supported in plugin subagents) |
| `hooks` | | Hooks scoped only to this agent's session (not supported in plugin subagents) |
| `memory` | | Auto memory scope: `user` / `project` / `local` |
| `background` | | Run as background agent |
| `effort` | | Effort level: `low` / `medium` / `high` / `xhigh` / `max` |
| `isolation` | | `worktree` = run in isolated git worktree; cleaned up if no changes |
| `color` | | Display color in the UI |
| `initialPrompt` | | Text injected at the start of the agent session |

### `isolation: worktree`

When set to `worktree`, the subagent runs in a temporary git worktree isolated from the main working tree. If the agent makes no changes, the worktree is automatically cleaned up. The worktree path and branch name are returned in the result if changes were made.

> Note: A subagent starts in the main conversation's current working directory. `cd` commands do not persist between Bash calls within a subagent.

### `disallowedTools` vs `tools`

Both can be specified simultaneously. Resolution order: `disallowedTools` is checked first, then `tools` allowlist.

### Preloaded Skills Behavior

Subagents with `skills` in their frontmatter work differently from normal sessions: the **full skill content is injected at startup**, not just the description. This means the skill's instructions are immediately available without an explicit invocation.

For memory persistence, subagents can maintain their own auto memory. See [memory-and-context.md](memory-and-context.md) for details.

---

## Agent Teams

Complex tasks can be split across multiple subagents:

```
1. Explorer agent:  identify relevant files
2. Analyzer agent:  analyze dependencies in each file
3. Planner agent:   draft a change plan
→ Main agent:       review the plan and implement
```

Prompt Claude:

```
Split this task across multiple subagents:
1. First agent: list authentication-related files
2. Second agent: summarize the current implementation in each file
3. Combine the results and write a migration plan
```

---

## Notes

- Subagents do not share the main conversation's context — pass all needed context explicitly.
- `context: fork` skills run automatically as subagents.
- Subagent results return to the main conversation as a summary.
- Subagents can maintain their own auto memory for persistent learning across sessions.

<!-- last-reviewed: 2026-05-03 -->
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

### Built-in Agent Types

| Type | Description |
|---|---|
| `general-purpose` | General agent. Default. |
| `Explore` | Optimized for fast file and pattern search in a codebase |
| `Plan` | Specialized for software architecture and implementation planning |

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
model: sonnet              # Short form: opus / sonnet / haiku
skills:                    # Skills to preload at startup
  - review
  - summarize
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

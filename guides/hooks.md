<!-- last-reviewed: 2026-05-12 -->
# Hooks (Lifecycle Hooks)

Hooks let you run shell commands automatically in response to specific Claude Code events.

> Reference: https://code.claude.com/docs/en/hooks

---

## What Are Hooks?

Shell commands that fire automatically at defined lifecycle points.

Common uses:
- Run a formatter after every file edit
- Check environment at session start
- Send notifications when Claude finishes a response
- Block context compaction and prompt the user to checkpoint

---

## Configuration in settings.json

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH",
            "timeout": 15000
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Response complete\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

---

## Supported Events

| Event | Fires When | Blockable |
|---|---|---|
| `PreToolUse` | Before any tool call | Yes (exit 2) |
| `PostToolUse` | After any tool call completes | No |
| `PostToolUseFailure` | After a tool call fails | No |
| `PostToolBatch` | After a batch of parallel tool calls completes | No |
| `Notification` | Claude is waiting for input or permission | No |
| `Stop` | Claude finishes a response turn | No |
| `StopFailure` | Claude fails to complete a response | No |
| `PreCompact` | Before context compaction is triggered | Yes (exit 2) |
| `PostCompact` | After context compaction completes | No |
| `SessionStart` | A new Claude Code session begins | No |
| `SessionEnd` | A session ends | No |
| `InstructionsLoaded` | A CLAUDE.md or rules file is loaded | No |
| `UserPromptSubmit` | User submits a prompt | Yes (exit 2) |
| `UserPromptExpansion` | Prompt is expanded (e.g., after skill injection) | No |
| `PermissionRequest` | Claude requests permission for an action | Yes |
| `PermissionDenied` | A permission request is denied | No |
| `SubagentStart` | A subagent session begins | No |
| `SubagentStop` | A subagent session ends | No |
| `TaskCreated` | A task is created | No |
| `TaskCompleted` | A task completes | No |
| `WorktreeCreate` | A git worktree is created | No |
| `WorktreeRemove` | A git worktree is removed | No |
| `CwdChanged` | Working directory changes | No |
| `FileChanged` | A file is modified outside of Claude | No |

### Matcher Field

`PreToolUse` and `PostToolUse` support a `matcher` to filter by tool name:

```json
{
  "matcher": "Edit|Write",   // pipe-separated tool names
  "hooks": [{ "type": "command", "command": "..." }]
}
```

Leave `matcher` empty (`""`) to match all tools.

### `InstructionsLoaded` Event

Fires each time Claude loads a CLAUDE.md or `.claude/rules/*.md` file. Cannot be blocked — monitoring/logging only.

Available input fields: `file_path`, `memory_type`, `load_reason`, `globs`, `trigger_file_path`.

```json
{
  "hooks": {
    "InstructionsLoaded": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Loaded: $CLAUDE_FILE_PATH\" >> ~/.claude/logs/instructions.log"
          }
        ]
      }
    ]
  }
}
```

---

## Hook Fields

| Field | Type | Description |
|---|---|---|
| `type` | string | `"command"` / `"http"` / `"mcp_tool"` / `"prompt"` / `"agent"` |
| `command` | string | Shell command to run (for type `"command"`) |
| `args` | array | Argument array for Exec Form — see below |
| `url` | string | Webhook URL (for type `"http"`) |
| `timeout` | number | Timeout in milliseconds (default: 10000) |
| `if` | string | Permission-rule syntax filter — hook runs only when condition matches |

### Hook Types

| Type | Description |
|---|---|
| `command` | Run a local shell command |
| `http` | POST to a webhook URL |
| `mcp_tool` | Call an MCP tool |
| `prompt` | Inject a prompt into the conversation |
| `agent` | Spawn a subagent |

### Exec Form vs Shell Form

- **Shell Form** (no `args`): command string is passed to `sh -c`. Supports pipes, globs, and shell features.
- **Exec Form** (`args` array present): command is executed directly without a shell. No pipe or glob expansion.

```json
// Shell Form — supports pipes
{ "type": "command", "command": "cat $CLAUDE_FILE_PATH | grep TODO" }

// Exec Form — direct execution, no shell expansion
{ "type": "command", "command": "npx", "args": ["prettier", "--write", "$CLAUDE_FILE_PATH"] }
```

Use Exec Form when you don't need shell features and want predictable quoting.

### `if` Field (Inline Filter)

Filter hook execution using permission-rule syntax:

```json
{
  "type": "command",
  "command": "echo 'git tool used'",
  "if": "Bash(git *)"
}
```

The hook runs only when the matched event satisfies the `if` condition.

---

## JSON Output from Hooks

Hooks can write JSON to stdout to control Claude's behavior:

```json
{
  "decision": "block",          // "block" | "continue"
  "suppressOutput": true,       // hide this hook's output from the user
  "systemMessage": "Blocked.",  // message shown to Claude when blocked
  "hookSpecificOutput": {
    "permissionDecision": "deny"   // for PermissionRequest hooks
  }
}
```

| Field | Description |
|---|---|
| `decision` | `"block"` stops the action; `"continue"` proceeds normally |
| `suppressOutput` | Hide hook stdout from the user |
| `systemMessage` | Text injected as a system message when blocked |
| `hookSpecificOutput.permissionDecision` | For `PermissionRequest`: `"allow"` or `"deny"` |

---

## HTTP Hooks

Send a POST request to a webhook URL instead of running a local command.

```json
{
  "PostToolUse": [
    {
      "matcher": "",
      "hooks": [
        {
          "type": "http",
          "url": "https://hooks.example.com/claude-events"
        }
      ]
    }
  ]
}
```

Allowed URLs and exposed environment variables must be whitelisted in settings:

```json
{
  "allowedHttpHookUrls": ["https://hooks.example.com/*"],
  "httpHookAllowedEnvVars": ["MY_TOKEN"]
}
```

---

## Exit Codes

| Exit Code | Meaning |
|---|---|
| `0` | Success — proceed normally |
| Non-zero | Warning shown to Claude, but execution continues |
| `2` | Block the operation and pass the hook's stdout as feedback to Claude |

Use `exit 2` in `PreCompact` to block auto-compaction and prompt the user.

---

## Environment Variables

| Variable | Available In | Description |
|---|---|---|
| `$CLAUDE_FILE_PATH` | `PostToolUse` with Edit/Write | Path of the file being edited |
| `$CLAUDE_CODE_REMOTE` | All | Remote URL if running remotely |
| `$CLAUDE_EFFORT` | All | Current effort level |
| `$CLAUDE_PROJECT_DIR` | All | Project root directory |
| `$CLAUDE_PLUGIN_ROOT` | All | Plugin installation directory |
| `$CLAUDE_PLUGIN_DATA` | All | Plugin persistent data directory |
| `$CLAUDE_ENV_FILE` | `SessionStart`, `CwdChanged` | Path to env file; write `KEY=VALUE` lines to persist env vars |

---

## Skill-level Hooks

Hooks defined in SKILL.md frontmatter run only when that skill executes:

```markdown
---
name: deploy
description: Deploy the application
hooks:
  before: "echo 'Deploy starting' | slack-notify"
  after: "echo 'Deploy complete' | slack-notify"
---

Run the deploy script.
```

Skill hooks vs settings.json hooks:
- Skill hooks: fire only for that specific skill
- settings.json hooks: apply globally to all sessions

---

## Disabling Hooks

Disable all hooks at once (useful for debugging):

```json
{
  "disableAllHooks": true
}
```

In managed/enterprise environments, allow only approved hooks:

```json
{
  "allowManagedHooksOnly": true
}
```

---

## Practical Examples

### Auto-formatter (TypeScript)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH",
            "timeout": 15000
          }
        ]
      }
    ]
  }
}
```

### Auto-linter (Python)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix $CLAUDE_FILE_PATH",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

### Completion Notification (macOS)

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Response complete\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Block Auto-compaction (Prompt to Checkpoint)

```json
{
  "hooks": {
    "PreCompact": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Context 70% — run /checkpoint\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

---

## Notes

- Hook failures (non-zero exit) show as warnings to Claude but do not stop execution.
- Long-running hooks degrade responsiveness — set timeouts appropriately.
- Hooks can be set in `.claude/settings.json` (project) or `~/.claude/settings.json` (global).
- **No LLM calls inside hooks** — hooks run on every event and directly affect token cost.

<!-- last-reviewed: 2026-05-03 -->
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

| Event | Fires When |
|---|---|
| `PreToolUse` | Before any tool call |
| `PostToolUse` | After any tool call completes |
| `Notification` | Claude is waiting for input or permission |
| `Stop` | Claude finishes a response turn |
| `PreCompact` | Before context compaction is triggered |
| `SessionStart` | A new Claude Code session begins |

### Matcher Field

`PreToolUse` and `PostToolUse` support a `matcher` to filter by tool name:

```json
{
  "matcher": "Edit|Write",   // pipe-separated tool names
  "hooks": [{ "type": "command", "command": "..." }]
}
```

Leave `matcher` empty (`""`) to match all tools.

---

## Hook Fields

| Field | Type | Description |
|---|---|---|
| `type` | string | `"command"` for shell commands; `"http"` for webhooks |
| `command` | string | Shell command to run (for type `"command"`) |
| `url` | string | Webhook URL (for type `"http"`) |
| `timeout` | number | Timeout in milliseconds (default: 10000) |

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
- `$CLAUDE_FILE_PATH` contains the path of the file being edited (PostToolUse with Edit/Write).
- Hooks can be set in `.claude/settings.json` (project) or `~/.claude/settings.json` (global).
- **No LLM calls inside hooks** — hooks run on every event and directly affect token cost.

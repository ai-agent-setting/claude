<!-- last-reviewed: 2026-05-19 -->
# Context Window and Memory Management

Claude Code has two memory systems:
- **CLAUDE.md**: Persistent instruction files you write manually
- **Auto Memory**: Notes Claude writes automatically based on session experience

> Reference: https://code.claude.com/docs/en/memory

---

## CLAUDE.md vs Auto Memory

| | CLAUDE.md | Auto Memory |
|---|---|---|
| Author | Human | Claude |
| Content | Rules and instructions | Learned patterns and preferences |
| Load method | Root CLAUDE.md: fully loaded every session (no size limit). Subdirectory CLAUDE.md: loaded on demand | MEMORY.md index: top 200 lines or 25 KB per session. CLAUDE.md loads in full regardless of length. |
| Delivery | As a user message after the system prompt (not part of the system prompt itself) | — |
| Use cases | Coding standards, workflows, architecture | Build commands, debugging insights |

CLAUDE.md content is delivered as a user message, so strict compliance is not guaranteed — Claude treats it as strong guidance, not a hard constraint.

---

## Auto Memory

Claude automatically stores corrections, preferences, build commands, and debugging patterns. It saves only what it judges useful for future conversations.

**Requires Claude Code v2.1.59 or later.** Check your version: `claude --version`.

### Storage Location

```
~/.claude/projects/<project>/memory/
├── MEMORY.md              # Index file (top 200 lines or 25 KB loaded at session start)
├── debugging.md           # Debugging patterns
└── api-conventions.md     # API design decisions
```

`<project>` is derived from the git repo root.
- Outside a git repo, the project root path is used.
- All worktrees and subdirectories within the same git repository share one auto memory directory.
- Auto memory is **machine-local** — not shared across cloud environments or other machines.

Only `MEMORY.md` (up to 200 lines or 25 KB) is auto-loaded each session. Other files are read by Claude on demand.

### Managing Auto Memory

```
/memory    # View loaded files + toggle auto memory + open memory folder
```

`/memory` lets you:
- See which CLAUDE.md, CLAUDE.local.md, and rules files are loaded
- Toggle auto memory on/off
- Open the memory folder
- Click a file to open it in your editor

Auto memory files are plain markdown — edit or delete them at any time.

### Custom Storage Location

Change the storage path with `autoMemoryDirectory`. Accepted from policy settings, user settings, and the `--settings` flag. **Not accepted in project settings** (`.claude/settings.json`) to prevent a shared project from redirecting auto memory writes to sensitive locations.

The value must be an absolute path or start with `~/`.

```json
{
  "autoMemoryDirectory": "~/custom/memory"
}
```

### Disabling Auto Memory

```json
{
  "autoMemoryEnabled": false
}
```

Or via environment variable: `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

---

## CLAUDE.md Size Guidelines

Official recommendation: keep each file under 200 lines.

| Project size | Strategy |
|---|---|
| Small (docs, learning) | Root CLAUDE.md under 100 lines |
| Medium (typical web app) | Root 150 lines + split into `.claude/rules/` |
| Large (complex systems) | Concise root CLAUDE.md + distribute via path-specific rules |

### HTML Comments in CLAUDE.md

Block-level HTML comments (`<!-- -->`) are stripped before content is injected into context. Use them for metadata that Claude should not see.

```markdown
<!-- last-reviewed: 2026-05-03 -->
<!-- This section is for the backend team only. Ignore if working on frontend. -->
# Project Instructions
```

> Note: comments inside code blocks are preserved as-is.

---

## Context Window Management

### `/clear` — Most Powerful Reset

```
/clear    # Fully reset context (CLAUDE.md is reloaded)
```

When to use:
- Switching to an unrelated task
- Claude repeats the same mistake after two corrections
- Responses are becoming increasingly vague or generic

### `/compact` — Compress While Keeping Essentials

```
/compact                          # Auto-compress
/compact focus on API changes     # Compress with a hint
```

Auto-compaction triggers when context reaches 95% capacity (default). Override the threshold with the `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` environment variable (e.g., `50` = 50%).

Enable or disable auto-compaction in settings:

```json
{
  "autoCompactEnabled": true
}
```

> Note: Compaction uses the session's active model and `effortLevel`. There is no separate setting to override the model or effort level specifically for compaction.

After `/compact`, Claude re-reads root CLAUDE.md from disk and re-injects it into the session. Nested subdirectory CLAUDE.md files are **not** re-injected automatically — they reload the next time Claude reads a file in that subdirectory. If an instruction disappeared after compaction, it was either given only in conversation or lives in a nested CLAUDE.md that hasn't reloaded yet. If subdirectory rules must survive compaction, include the key content in root CLAUDE.md.

To control what is preserved during compaction, add a compaction directive to CLAUDE.md:

```markdown
<!-- compaction instructions -->
When compacting, always preserve:
- The full list of modified files
- Current task progress and next steps
```

### Partial Compaction

To compact only part of the conversation: press `Esc + Esc` (or run `/rewind`), select a message checkpoint, and choose **Summarize from here**. Only the conversation from that point forward is compressed.

### `/rewind` — Restore to Checkpoint

Claude creates a checkpoint automatically on every change. Open the checkpoint menu with `Esc + Esc` or `/rewind`.

Restore options:
1. **Conversation only** — rewind history, keep file changes
2. **Files only** — revert files, keep conversation
3. **Both** — revert files and conversation to the checkpoint
4. **Summarize from here** — compress everything from the selected message forward

> Checkpoints persist after the session ends.

---

## Loading Additional Directory CLAUDE.md

CLAUDE.md files in directories added with `--add-dir` are not loaded by default. To load them:

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir /other/project
```

---

## System-prompt Level Instructions

For instructions that must be delivered at the system prompt level (not as a user message), use `--append-system-prompt`. This is suited for scripts and automation that run Claude non-interactively:

```bash
claude --append-system-prompt "Always respond in JSON format."
```

---

## Session Resume

```bash
claude --continue    # Resume the most recent conversation
claude --resume      # Pick from a list of conversations
```

---

## Signals to Reset Context

- Claude starts breaking rules established earlier in the session
- Responses become increasingly vague or generic
- The same error has been corrected twice without success

<!-- last-reviewed: 2026-05-19 -->
# Skills and Commands

Reusable `/slash` workflows for Claude Code.

> Reference: https://code.claude.com/docs/en/skills
>
> Claude Code skills follow the [Agent Skills (agentskills.io)](https://agentskills.io) open standard, which works across multiple AI tools.

---

## Skills vs Commands

| | Commands (legacy) | Skills (current) |
|---|---|---|
| Path | `.claude/commands/*.md` | `.claude/skills/<name>/SKILL.md` or `.claude/skills/<name>.md` |
| Status | Still works | Recommended |
| Features | Basic | frontmatter, fork context, subagent support |

`.claude/commands/*.md` remains backward-compatible — migration is optional.

---

## SKILL.md File Structure

```
.claude/
└── skills/
    └── my-skill/
        ├── SKILL.md        # Entry point (required)
        ├── context.md      # Supporting files (referenced from SKILL.md)
        └── template.txt
```

### SKILL.md Frontmatter Fields

```markdown
---
name: my-skill          # Optional. Lowercase, digits, hyphens only, max 64 chars.
                        # If omitted, the directory name is used.
description: What this skill does (used by Claude to decide when to invoke it)
                        # Recommended — Claude uses this to auto-trigger the skill.
when_to_use: Example requests that trigger this skill automatically
                        # Combined with description, capped at 1,536 chars total.
arguments:
  - name: filepath
    description: Path to the file to review
    required: true
context: fork           # fork = run in separate context (protects main conversation)
disable-model-invocation: false  # true = only runs when explicitly called
allowed-tools:
  - Read
  - Bash
model: opus             # Short form (opus/sonnet/haiku) or full model ID.
                        # Override applies for the current turn only; session model resumes on next prompt.
user-invocable: true    # false = callable only from other skills
argument-hint: "[filepath] [options]"
effort: medium          # low / medium / high / xhigh / max
agent: general-purpose  # Subagent type when context: fork
hooks:
  before: "echo 'starting'"
  after: "echo 'done'"
paths:
  - "src/**/*.{ts,tsx}" # Activate only for matching files. Omit to always activate.
shell: bash
---

Skill instructions here.
$ARGUMENTS is replaced with text the user typed after /skill-name.
```

Only `description` is required for auto-invocation. `name` defaults to the directory name if omitted.

> Note: `paths` limits when the skill is **auto-activated** by Claude. Manually calling `/skill-name` always works regardless of the `paths` setting.

> Note: `allowed-tools` accepts either a YAML list or a space-separated string (e.g., `allowed-tools: Read Grep`).

---

## Built-in Skills

| Skill | Description |
|---|---|
| `/batch` | Orchestrate large-scale changes in parallel using git worktrees |
| `/claude-api` | Load Claude API reference (auto-activates when `anthropic` is imported) |
| `/debug` | Start a debugging workflow for the current error |
| `/loop [interval] <prompt>` | Run a prompt on a recurring interval (e.g. `/loop 5m check deploy`) |
| `/simplify [focus]` | Run three parallel review agents then apply fixes |

---

## $ARGUMENTS Usage

`$ARGUMENTS` is replaced with everything the user typed after `/skill-name`.

```
/review src/auth.ts
→ $ARGUMENTS = "src/auth.ts"

/review src/auth.ts from a security angle
→ $ARGUMENTS = "src/auth.ts from a security angle"
```

### Positional Arguments

Access by index with `$ARGUMENTS[N]` or the shorthand `$N`:

```
/deploy main production
→ $ARGUMENTS[0] = "main"       ($0 shorthand)
→ $ARGUMENTS[1] = "production" ($1 shorthand)
```

### Named Arguments

Declare named arguments in frontmatter for positional binding:

```markdown
---
arguments: [issue, branch]
---

Fix issue $issue on branch $branch.
```

Called as `/fix-issue 123 feature/auth` → `$issue = "123"`, `$branch = "feature/auth"`.

---

## Built-in Variables

| Variable | Description |
|---|---|
| `$ARGUMENTS` | Full text after `/skill-name` |
| `$ARGUMENTS[N]` / `$N` | N-th argument (0-indexed) |
| `${CLAUDE_SESSION_ID}` | Current session ID |
| `${CLAUDE_SKILL_DIR}` | Skill directory path |
| `${CLAUDE_EFFORT}` | Current effort level: `low` / `medium` / `high` / `xhigh` / `max` |

---

## Skill Context Lifecycle

When a skill is invoked, its rendered SKILL.md content enters the conversation as a single message and stays there for the rest of the session. It is not re-injected on subsequent turns.

After compaction, skills are re-attached from the most recently invoked skill first, up to a budget of:
- **5,000 tokens** per skill
- **25,000 tokens** total across all skills

Keep SKILL.md concise. Move lengthy reference material to companion files and reference them from SKILL.md.

---

## Dynamic Context Injection (Shell Injection)

Use `` !`<command>` `` inside SKILL.md to inject shell output at pre-processing time, before Claude runs:

```markdown
Current branch: !`git branch --show-current`
Recent commits: !`git log --oneline -5`

Review $ARGUMENTS based on the above context.
```

For multi-line commands, use a fenced code block with `!`:

````markdown
```!
git log --oneline -10
git status --short
```
````

### Disabling Shell Injection

```json
{
  "disableSkillShellExecution": true
}
```

When disabled, `` !`command` `` syntax is replaced with `[shell command execution disabled by policy]` rather than being executed. Bundled and managed skills are not affected.

---

## Skill Access Control

Control which skills are allowed or denied via `/permissions`:

```
Skill               # deny all skills
Skill(commit)       # deny the commit skill specifically
Skill(deploy *)     # deny all skills with names starting with "deploy"
```

These rules can be set in `.claude/settings.json` or `~/.claude/settings.json`.

### `skillOverrides` — Visibility Control

Override skill visibility without modifying SKILL.md:

```json
{
  "skillOverrides": {
    "my-skill": "on",               // fully visible (default)
    "internal-helper": "name-only", // listed by name but description hidden
    "draft-skill": "user-invocable-only", // shown only when user explicitly invokes
    "deprecated-skill": "off"       // hidden from all listings
  }
}
```

Four visibility levels:
| Level | Description |
|---|---|
| `on` | Fully visible — name and description shown |
| `name-only` | Listed by name; description hidden from Claude |
| `user-invocable-only` | Visible only when user explicitly invokes `/skill-name` |
| `off` | Hidden from all skill listings |

Toggle visibility interactively via `/skills` menu (press Space).

---

## Skill Priority and Locations

When the same skill name exists in multiple locations, priority is:

```
enterprise > personal > project
```

Plugin skills use the `plugin-name:skill-name` namespace.

### Search Locations

1. `.claude/skills/` — project level
2. `~/.claude/skills/` — personal global
3. Plugin skills — `plugin-name:skill-name` format

**Monorepo auto-discovery**: Project skills load from `.claude/skills/` in your starting directory and in every parent directory up to the repository root. When editing files in a subdirectory, Claude also discovers skills in that subdirectory's `.claude/skills/`.

> Skill files are watched for changes. Adding, editing, or removing a skill takes effect within the current session without restarting. (Exception: creating a brand-new top-level skill directory requires a restart.)

---

## Skill Examples

### Basic Review

```markdown
---
description: Review code for bugs, security issues, and style
argument-hint: "[filepath]"
---

Review $ARGUMENTS.

Check in order:
1. Bugs and logic errors
2. Security vulnerabilities
3. Performance issues
4. Code style and readability

Provide specific improvement suggestions for each.
```

### Fork-based Investigation (Protects Main Context)

```markdown
---
description: Investigate the codebase and return a summary
context: fork
agent: Explore
allowed-tools:
  - Read
  - Bash
---

Investigate $ARGUMENTS.

Return a summary with:
- Findings
- Related file list
- Recommended next steps
```

### Deployment (Disable Auto-invocation)

```markdown
---
description: Deploy the application to production
disable-model-invocation: true
---

Before deploying, verify:
1. All tests pass
2. Changelog updated
3. Version tag added

Then run: npm run deploy
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Skill descriptions are cut short | Too many skills exceed the context budget | Reduce `description` length or set `skillListingBudgetFraction` / `maxSkillDescriptionChars` in settings. Run `/doctor` to check overflow. |
| Skill not auto-invoked | `description` missing or too vague | Add a clear `description` and `when_to_use` field |
| Skill runs when it shouldn't | `disable-model-invocation: false` on a side-effect skill | Set `disable-model-invocation: true` |

### Skill Description Budget

When many skills are loaded, descriptions may be truncated to fit the context window.

```json
{
  "skillListingBudgetFraction": 0.02,  // % of context window for skill listings (default: 0.01)
  "maxSkillDescriptionChars": 2048     // per-skill description cap (default: 1536)
}
```

Run `/doctor` to see if skill descriptions are being cut.

### `disable-model-invocation` and Subagent Preload

When `disable-model-invocation: true` is set, the skill is also excluded from subagent preload. It will not be injected at startup even if listed in a subagent's `skills` frontmatter.

---

## Global Skills

Skills installed in `~/.claude/skills/` are available across all projects.

| Skill | Description |
|---|---|
| `/review` | Code review |
| `/summarize` | Summarize files or changes |
| `/explain` | Explain code |
| `/update-guides` | Fetch official docs and propose guide updates |
| `/check-freshness` | Audit freshness of guides/ files |
| `/review-claude-config` | Quality review of Claude Code config files |

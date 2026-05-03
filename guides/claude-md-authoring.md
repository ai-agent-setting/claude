<!-- last-reviewed: 2026-05-03 -->
# Writing Effective CLAUDE.md Files

CLAUDE.md is the instruction file Claude Code reads automatically when opening a project.
A well-written CLAUDE.md eliminates the need to repeat the same context in every session.

> Reference: https://code.claude.com/docs/en/memory

---

## Core Principles

1. **Be specific**: "Write good code" is meaningless. "Keep functions under 20 lines" is measurable.
2. **Stay under 200 lines**: Official recommendation. Compliance drops as length grows. Split into `.claude/rules/` or use `@import` when over the limit.
3. **Project-specific only**: Exclude anything Claude can infer from reading the code or standard conventions.
4. **Use imperative form**: "Do X" is clearer than "Please do X."
5. **Commit it**: The whole team benefits.

> Tip: Run `/init` to auto-generate a CLAUDE.md draft from the codebase.
> Set `CLAUDE_CODE_NEW_INIT=1` to activate an interactive multi-phase flow that sets up CLAUDE.md + skills + hooks in one go.

---

## What to Include vs Exclude

| Include | Exclude |
|---|---|
| Bash commands Claude cannot guess | Things Claude can infer from reading the code |
| Code style rules that differ from defaults | Standard language conventions |
| How to run tests and which test runner | Frequently changing information |
| Team conventions (branches, PR rules) | Long explanations or tutorials |
| Architecture decisions | Self-evident rules like "write clean code" |

---

## Recommended Minimum Structure

```markdown
# Project Name

## Overview
One or two sentences describing the purpose and tech stack.

## Key Commands
```bash
npm run dev    # dev server
npm test       # run tests
npm run build  # build
```

## Code Conventions
Project-specific rules for indentation, naming, etc.
```

---

## File Imports (`@` Syntax)

Import other files into CLAUDE.md using `@path/to/file`:

```markdown
@README.md
@package.json
- API design: @docs/api-design.md
```

> External imports show an approval dialog on first use.

### Import Path Resolution

- Paths are resolved **relative to the importing file's location**, not the working directory.
- Recursive imports are allowed up to **5 levels** deep.
- `@import` is a structural tool only — it does not reduce context size. Use path-scoped rules to reduce context.

### AGENTS.md Compatibility

If another agent tool uses `AGENTS.md`, import it from CLAUDE.md to avoid duplication:

```markdown
# CLAUDE.md
@AGENTS.md
```

---

## `.claude/rules/` — Modular Rules

When rules grow large, split them by topic into `.claude/rules/`:

```
.claude/
├── CLAUDE.md           # Core instructions (keep concise)
└── rules/
    ├── code-style.md   # Code style
    ├── testing.md      # Testing rules
    └── security.md     # Security requirements
```

### CLAUDE.md vs Rules vs Skills

| | CLAUDE.md / rules/ | Skills |
|---|---|---|
| Load time | At session start (always in context) | Only when invoked or auto-triggered |
| Best for | Persistent standards, conventions, constraints | Task-specific workflows, on-demand procedures |
| Token cost | Fixed cost every session | Zero cost when not invoked |

For instructions that don't need to be in context all the time, use skills instead.

### User-level Rules (Personal Global Rules)

`~/.claude/rules/` holds personal global rules that apply to all projects.

User-level rules are **loaded before project rules**, which means project rules take higher priority — they can override user-level rules.

```
~/.claude/
└── rules/
    ├── personal-style.md
    └── workflow.md
```

Rules without `paths` frontmatter are loaded at launch with the same priority as `.claude/CLAUDE.md`.

Use symlinks to share rule files across projects:

```bash
ln -s ~/shared-rules/security.md .claude/rules/security.md
```

### Path-specific Rules

Use the `paths` frontmatter field to activate a rule only for matching files.
Brace expansion is supported:

```markdown
---
paths:
  - "src/api/**/*.{ts,tsx}"
  - "src/api/**/*.test.ts"
---

# API Development Rules
- Validate all endpoint inputs
- Use the standard error response format
```

Rules without `paths` are always loaded at session start.

---

## CLAUDE.md Placement

| Location | Path | Scope |
|---|---|---|
| Project (shared) | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Shared via git |
| Personal (all projects) | `~/.claude/CLAUDE.md` | All your projects |
| Personal (this project, private) | `./CLAUDE.local.md` | Excluded from git, auto-gitignored |
| Org policy (macOS) | `/Library/Application Support/ClaudeCode/CLAUDE.md` | Enterprise managed |
| Org policy (Linux/WSL) | `/etc/claude-code/CLAUDE.md` | Enterprise managed |
| Org policy (Windows) | `C:\Program Files\ClaudeCode\CLAUDE.md` | Enterprise managed |

---

## Large Monorepo Configuration

Exclude irrelevant team CLAUDE.md files with `claudeMdExcludes` in `.claude/settings.local.json`:

```json
{
  "claudeMdExcludes": [
    "teams/frontend/**",
    "teams/data-science/**"
  ]
}
```

Patterns are matched against **absolute file paths** using glob syntax. Arrays merge across config layers. Managed policy CLAUDE.md files cannot be excluded.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Claude repeatedly breaks rules | CLAUDE.md too long — rules get buried | Trim ruthlessly. Use "IMPORTANT"/"YOU MUST" for critical rules |
| Instructions disappear after `/compact` | Instruction was only said in conversation, not in CLAUDE.md | Add it to CLAUDE.md |
| Rule conflicts | Conflicting instructions in multiple files | Run `/memory` to see load order, then resolve |
| Unclear which rules file loaded | Path-specific rules debugging | Use `InstructionsLoaded` hook to log file loads |

---

## References

- Skills (advanced extension): [skills-and-commands.md](skills-and-commands.md)

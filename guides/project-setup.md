<!-- last-reviewed: 2026-05-12 -->
# Setting Up Claude Code for a New Project

> Reference: https://code.claude.com/docs/en/best-practices#configure-your-environment

---

## Quick Setup

```bash
cd your-project
claude
/init   # Analyzes codebase and generates CLAUDE.md
```

---

## Step-by-step

### Step 1: Generate with `/init` (Recommended)

```
claude
/init
```

Claude analyzes the build system, test framework, and code patterns to generate a draft.
If CLAUDE.md already exists, `/init` proposes improvements instead.

> Tip: Set `CLAUDE_CODE_NEW_INIT=1` to activate an interactive multi-phase flow that sets up CLAUDE.md + skills + hooks in one go:
>
> ```bash
> CLAUDE_CODE_NEW_INIT=1 claude
> /init
> ```

### Step 2: Edit CLAUDE.md

- [ ] Project name and overview (one or two sentences)
- [ ] Actual build / run / test / lint commands
- [ ] Project-specific coding rules
- [ ] Key directory and file structure

Authoring tips → [claude-md-authoring.md](claude-md-authoring.md)

### Step 3: Configure `.claude/skills/` (Optional)

```
.claude/
├── skills/
│   ├── review/SKILL.md
│   └── summarize/SKILL.md
└── settings.json
```

Details → [skills-and-commands.md](skills-and-commands.md)

### Step 4: Set Permissions (Optional)

Three approaches — choose based on your trust level:

**Allowlist (Recommended)**: explicitly allow specific commands.

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(npm run lint)", "Bash(npm test *)"],
    "ask": ["Bash(git push *)", "Bash(npm publish *)"],
    "deny": ["Read(./.env)", "Bash(curl *)"]
  }
}
```

Three rule types:
- `allow` — auto-approve without prompting
- `ask` — always prompt even in auto-permission mode
- `deny` — block outright

**Sandbox mode**: OS-level isolation.

```
/sandbox    # Enable sandbox mode
```

**Auto mode**: approve all permission requests automatically (trusted environments only).

Useful permission commands:
- `/permissions` — manage allowlist interactively
- `/sandbox` — toggle sandbox mode

### Step 5: Install Plugins (Optional)

Browse and install plugins from the marketplace:

```
/plugin     # Browse marketplace
```

Plugins extend Claude Code with additional skills and integrations (e.g., Codex, security scanners).

### Step 6: Connect MCP Servers (Optional)

Add external tools (databases, APIs, services) via MCP:

```bash
claude mcp add <server-name> <command>
```

MCP servers appear as additional tools Claude can call during your session.

### Step 7: Verify

- [ ] Ask "What is this project?" → Does Claude correctly describe the project from CLAUDE.md?
- [ ] Run build/test commands — does Claude execute them correctly?
- [ ] Run `/memory` to confirm which instruction files are loaded

---

## CLAUDE.local.md — Private Personal Config

```markdown
# CLAUDE.local.md (excluded from git)
- My local dev server: http://localhost:3001
```

---

## References

- [Skills guide](skills-and-commands.md)
- [CLAUDE.md authoring](claude-md-authoring.md)

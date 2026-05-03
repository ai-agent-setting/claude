<!-- last-reviewed: 2026-05-03 -->
# Official Sources and Update References

Official documentation URLs and how to keep guides current.

---

## Official Documentation URLs

| Document | URL | Description |
|---|---|---|
| Overview | https://code.claude.com/docs/en/overview | Claude Code introduction |
| Memory & CLAUDE.md | https://code.claude.com/docs/en/memory | CLAUDE.md, Auto Memory, Rules |
| Skills | https://code.claude.com/docs/en/skills | Skills system, SKILL.md format |
| Best Practices | https://code.claude.com/docs/en/best-practices | Official recommended workflows |
| Settings | https://code.claude.com/docs/en/settings | Full settings.json field reference |
| Release Notes | https://code.claude.com/docs/en/release-notes | Changelog by version |
| Hooks | https://code.claude.com/docs/en/hooks | Lifecycle hooks reference |
| Sub-agents | https://code.claude.com/docs/en/sub-agents | Subagent config and usage |
| Permissions | https://code.claude.com/docs/en/permissions | Allowlist, sandbox, auto mode |
| Sessions | https://code.claude.com/docs/en/sessions | Session resume, /rewind, /compact |
| Common Workflows | https://code.claude.com/docs/en/common-workflows | Frequently used workflow patterns |
| Commands | https://code.claude.com/docs/en/commands | Built-in command reference |
| Plugins | https://code.claude.com/docs/en/plugins | Plugin development and usage |
| Features Overview | https://code.claude.com/docs/en/features-overview | When to use skills vs hooks vs subagents |
| How Claude Code Works | https://code.claude.com/docs/en/how-claude-code-works | Internal architecture |
| Context Window | https://code.claude.com/docs/en/context-window | Context window visualization |
| Permission Modes | https://code.claude.com/docs/en/permission-modes | auto / plan / acceptEdits modes |
| Debug Your Config | https://code.claude.com/docs/en/debug-your-config | Config debugging guide |
| Headless Mode | https://code.claude.com/docs/en/headless | Non-interactive / scripted usage |
| Agent Teams | https://code.claude.com/docs/en/agent-teams | Multi-agent orchestration |
| Routines | https://code.claude.com/docs/en/routines | Scheduled recurring tasks |
| Checkpointing | https://code.claude.com/docs/en/checkpointing | Checkpoint and rewind details |
| Costs | https://code.claude.com/docs/en/costs | Token costs and reduction strategies |
| Status Line | https://code.claude.com/docs/en/statusline | Custom status line configuration |

> Deprecated URL pattern: `https://docs.anthropic.com/.../claude-code/...` — no longer used.

---

## Checking Guide Freshness

Each guide file has a `<!-- last-reviewed: YYYY-MM-DD -->` tag at the top.

Run the freshness audit skill:
```
/check-freshness
```

Manual check:
1. Visit https://code.claude.com/docs/en/release-notes for recent changes
2. Check the `last-reviewed` date on affected guides
3. Run `/update-guides [topic]` if a guide is stale

---

## Guide Update Process

### Using the `/update-guides` Skill

```
/update-guides memory
/update-guides skills
/update-guides all
```

This skill:
1. Fetches official docs via WebFetch
2. Compares against the current guide
3. Proposes a diff of needed changes
4. Does not edit files directly — review and apply manually

### Manual Update

1. Read the relevant official doc URL
2. Edit the guide file
3. Update `<!-- last-reviewed: YYYY-MM-DD -->`
4. Commit the change

---

## High-churn Areas (Check First)

- **Skills**: Active development (frontmatter fields, built-in skills)
- **Settings**: New fields added frequently
- **Sub-agents**: Agent types and file format may change
- **Release Notes**: New features announced per version

Stable areas (rarely changes):
- CLAUDE.md authoring principles
- Fundamental prompt engineering principles

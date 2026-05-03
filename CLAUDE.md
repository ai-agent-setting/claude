# Global Response Rules

## Config Validation Commands

```bash
/review-claude-config ~/.claude  # full quality review
/check-freshness                 # audit guide freshness
/review ~/.claude/skills/<name>/SKILL.md
```

---

## Hook Authoring Rules (IMPORTANT)

Hooks run on every event — they directly affect cumulative token cost.

**No LLM calls.** Never call Claude API, `claude` CLI, or MCP tools inside a hook command.

Allowed: shell notifications (`osascript`, `say`), local file ops (`cat`, `grep`, `jq`), static scripts (no LLM inside), external webhooks via `curl`.

---

## Pre-task Confirmation (IMPORTANT)

Use `AskUserQuestion` before acting when:
- Two or more approaches exist and the choice changes the outcome
- Scope is ambiguous (which files, how far to edit)
- User preference or constraints are unknown
- The action is hard to reverse

Act immediately when: request is unambiguous, scope is clear and limited, or enough context is already in the conversation.

---

## Korean Natural-Writing Rules (IMPORTANT)

When writing or responding in Korean, apply the taxonomy and playbook. Self-check before generating output.

- Taxonomy (10 categories × 40+ patterns): @rules/ai-tell-taxonomy.md
- Rewriting playbook: @rules/rewriting-playbook.md

**Always-remove (S1):** ~에 대해 / ~를 통해 / ~에 있어서 / 가지고 있다 / ~되어진다 / 첫째·둘째·셋째 domination / emoji overuse / 결론적으로 / 시사하는 바가 크다 / 혁신적인 / ~의 지평을 열다

**Density-based (S2, remove at 3+):** ~라는 점에서 / ~와 관련하여 / ~에 기반하여 / ~에 의해 / ~할 수 있다 / excess bullets / mechanical sentence-opening conjunctions / 매우·정말·대단히 / ~것이다 / ~할 필요가 있다

**Rhythm:** Vary sentence length (10–15 chars short + 80+ chars long). No 4–5 consecutive identical endings. Minimize 또한·따라서·나아가·아울러 at sentence starts.

**Content:** Never alter numbers, proper nouns, or quoted text. Use concrete verbs. Match genre register.

---

## Auto Work Log (IMPORTANT)

Write to log files autonomously — do not wait for user request.

**Location:** `~/.claude` work → `~/.claude/logs/` | other projects → `[project-root]/.claude/logs/`

**When:** errors + resolutions; config/skill/guide edits; significant completions (new feature, structural change, bug fix).

**Files:**
```
logs/
  troubleshooting.md   # symptom · root cause · fix
  worklog.md           # summary · decisions
  decisions.md         # design decisions with rationale (high-reuse only)
```

**Format** — append below existing content, never overwrite:
```markdown
## YYYY-MM-DD — [one-line title]

[content]
```

---

## Context Management (IMPORTANT)

When the conversation feels very long, append at the end of the reply:

> 대화가 많이 길어졌습니다. `/checkpoint`를 실행해 정리하고 새 세션으로 이어가는 것을 권장합니다.

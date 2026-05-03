<!-- last-reviewed: 2026-05-03 -->
# Common Mistakes and Anti-patterns

> Reference: https://code.claude.com/docs/en/best-practices#avoid-common-failure-patterns

---

## CLAUDE.md

### Kitchen Sink Session

Symptom: Working on one task, asking about something unrelated, then returning to the first task.
Fix: Run `/clear` between unrelated tasks.

### Over-specified CLAUDE.md

Symptom: Wrote hundreds of lines of CLAUDE.md but Claude ignores content in the middle.
Cause: Official recommendation is under 200 lines.
Fix: Trim ruthlessly. Split into `.claude/rules/` or use `@import` for detail. Use "IMPORTANT"/"YOU MUST" for critical rules.

If Claude already follows a rule correctly without an explicit instruction — delete the instruction, or convert it to a hook that enforces the behavior automatically.

### Using `@import` to Reduce Context

Misconception: Splitting files with `@import` reduces context size.
Reality: `@import` is a structural tool only — the full imported file is loaded regardless.
Fix: Use path-scoped rules to reduce context. Rules with a `paths` field load only when Claude edits a matching file.

### Instructions Disappear After `/compact`

Cause: The instruction was said in conversation only, not added to CLAUDE.md.
Fix: Add it to CLAUDE.md. It survives compaction completely.

Add a compaction directive to CLAUDE.md to control what gets preserved:

```markdown
When compacting, always preserve:
- The full list of modified files
- Current task progress and next steps
- Active decisions and their rationale
```

### Missing Build/Test Commands

Symptom: Claude asks how to test the code after every change.
Fix: Always include a commands code block in CLAUDE.md.

---

## Prompting

### Correcting the Same Error More Than Twice

Cause: Context becomes polluted with failed attempts.
Fix: After two failed corrections, run `/clear` and restart with a more specific prompt. A clean context avoids carrying forward the confusion.

### Trust-then-Verify Gap

Symptom: Claude produces a plausible-looking implementation that misses edge cases.
Fix: Always provide verification (tests, screenshots, scripts). Do not ship without verifying.

### Infinite Exploration

Symptom: Claude reads hundreds of files and consumes the whole context.
Fix: Narrow the investigation scope explicitly, or delegate to a subagent.

### "Why doesn't it work?" Without Error Output

Fix: Provide the full error message + the command that triggered it + the relevant code.

---

## Session Management

### No Parallel Sessions for Complex Work

Fix: Separate implementation sessions from review sessions. The review Claude has no bias toward code it did not write.

---

## Skills

### SKILL.md Too Large

Official recommendation: keep SKILL.md under 500 lines. Move detail to companion files.

### Missing `disable-model-invocation` for Side-effecting Actions

Symptom: Claude decides "it seems ready" and auto-runs `/deploy`.
Fix: Set `disable-model-invocation: true` on any skill that deploys, commits, sends notifications, or causes other side effects.

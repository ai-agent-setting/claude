<!-- last-reviewed: 2026-05-03 -->
# Prompt Engineering Best Practices for Claude Code

> Reference: https://code.claude.com/docs/en/best-practices

---

## Core Principle: Provide Verification Criteria

Result quality improves dramatically when Claude can validate its own work.

| Situation | Weak | Strong |
|---|---|---|
| Implement a function | "Write an email validator" | "Implement validateEmail. user@example.com → true. Run tests after implementing." |
| Fix a bug | "The build is broken" | "[Full error output]. Fix it, then confirm the build passes. Address the root cause." |
| UI change | "Make it look better" | "[Screenshot attached] Implement this design, then take a screenshot to compare." |

---

## Explore → Plan → Implement → Commit Workflow

### Using Plan Mode

```
Step 1 (Plan Mode): "Read src/auth and understand how sessions and login are handled."
Step 2 (Plan Mode): "Add Google OAuth. Which files need to change? Write a plan."
Step 3 (Normal Mode): "Implement the OAuth flow following the plan. Write and run tests."
Step 4 (Normal Mode): "Commit with a descriptive message and open a PR."
```

How to enter Plan Mode:
- `Shift+Tab` to toggle
- Select from the menu at the bottom of the input area
- `Ctrl+G` to open the current plan in your text editor for direct editing before Claude proceeds

Skip the plan when: you can describe the full diff in one sentence.

---

## Providing Context

```
@src/auth/login.ts — explain how session handling works
```

- **Images**: paste screenshots or design mockups directly
- **URLs**: provide documentation or API reference URLs directly
- **Pipe input**: `cat error.log | claude`

---

## Effective Patterns

### Assign a Role

```
You are a senior backend engineer. Review the code below from a security perspective.
```

### Step-by-step Instructions

```
Work in this order:
1. Identify problems in the existing code
2. Propose three solutions
3. Choose the best one and explain why
4. Implement it
```

### Claude Interview Mode (Spec-first)

```
I want to build [feature description].
Use the AskUserQuestion tool to ask me about key technical decisions, UI/UX, edge cases, and difficult parts.
When done, write the full spec to SPEC.md.
```

Explicitly naming `AskUserQuestion` makes Claude collect requirements in a structured way.
After the spec is complete, implement in a new session (`claude --continue`).

### `/btw` — Quick Questions Without Context Impact

```
/btw What design pattern is used in this file?
```

The answer appears in a dismissible overlay and never enters conversation history, so you can check details without growing context.

---

## Model Selection

Use `/model` to see currently available models and switch between them.

| Model tier | Best for |
|---|---|
| **Haiku** | Simple questions, fast code generation, straightforward transformations |
| **Sonnet** | General coding tasks — balanced speed and quality (default) |
| **Opus** | Complex reasoning, architecture design, tasks requiring deep thought |

Pin a model for a specific skill in SKILL.md:

```yaml
model: opus   # short form: opus / sonnet / haiku
```

The override applies for the duration of the current turn only. The session model resumes on your next prompt.

> For most tasks, omit the model field and use the session default.

---

## Anti-patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| "Do your best" | No success criteria | Provide specific criteria + verification conditions |
| Vague terms ("good", "clean") | Interpreted differently | Use measurable standards |
| Request result without verification | Plausible-but-wrong outputs slip through | Provide tests / screenshots / scripts to verify |
| "Investigate" without scope | Reads hundreds of files, consuming context | Narrow the scope or delegate to a subagent |
| Correcting the same error twice | Context polluted with failed attempts | After two failed corrections, run `/clear` then restart with a more specific prompt |

---

## Conversation Management

- Switch to unrelated task: `/clear`
- Preserve key context: `/compact [hint]`
- Undo a wrong direction: `Esc` to stop → `/rewind` to restore checkpoint
  - Restore options: conversation only / files only / both / summarize from checkpoint
  - Checkpoints persist after the session ends
- Long investigations: delegate to a subagent ([subagents.md](subagents.md))
- Quick question without context impact: `/btw`

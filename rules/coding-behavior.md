## Surgical Changes

Touch only what the request requires.

- Do not "improve" adjacent code, comments, or formatting.
- Do not refactor code that isn't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- Remove only imports/variables/functions that YOUR changes made unused.

Every changed line should trace directly to the user's request.

## Goal-Driven Execution

Before implementing, define what done looks like.

- "Fix the bug" → reproduce it first, then make it not reproduce.
- "Add validation" → write the failing test first, then make it pass.
- For multi-step tasks, state a brief plan with a verify step for each:
  1. [Step] → verify: [check]
  2. [Step] → verify: [check]

Weak success criteria ("make it work") require constant clarification. Strong criteria let you loop independently.

## Tradeoffs and Ambiguity

When multiple approaches exist, surface them — don't pick silently.

- State assumptions before implementing.
- If something is unclear, name what's confusing and ask.
- If a simpler approach exists, say so and push back.

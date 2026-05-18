<!-- last-reviewed: 2026-05-03 -->
# Frontend Design Best Practices with Claude Code

> References:
> - https://code.claude.com/docs/en/best-practices
> - https://code.claude.com/docs/en/common-workflows

---

## Core Principle: Visual Verification Loop

The highest-leverage practice for UI work is giving Claude a way to verify its own output visually.

**Pattern**: implement → screenshot → compare to target → fix differences

Paste mockups or design references directly into Claude Code (copy-paste or drag-and-drop). Then close the loop:

```
[paste screenshot of target design]
Implement this design. Take a screenshot of the result and compare it to the original.
List the differences and fix them.
```

Without visual verification, Claude produces plausible-looking code that may miss layout, spacing, or color details. With it, Claude can iterate until the result matches.

---

## Playwright MCP — Browser Automation

Install Playwright MCP to give Claude direct browser control:

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

This provides 34+ browser tools: navigation, click, type, screenshot, accessibility snapshot, and more.

Key advantages:
- Uses the accessibility tree for inspection — faster and more token-efficient than screenshots alone
- Enables exploratory QA: Claude navigates the app and finds visual issues on its own
- Supports test generation from live browser sessions

**Auth workflow**: Show Claude the login page, authenticate manually, then direct Claude on what to do next.

Use Playwright for:
- Visual regression checks after UI changes
- Verifying responsive behavior across viewport sizes
- Generating interaction tests from real browser sessions

---

## Design Direction: Avoiding Generic Output

Claude defaults to safe, generic UI. Guide it along specific dimensions rather than asking for "better design":

| Dimension | Vague | Specific |
|---|---|---|
| Typography | "improve the fonts" | "Use a display font for headings paired with a readable body font. Avoid Arial and Inter." |
| Color | "make it more colorful" | "Use CSS variables. One dominant neutral, one sharp accent. Consistent across all interactive states." |
| Motion | "add some animation" | "Subtle entrance animations on scroll. Ease-out curves. No looping animations." |
| Layout | "make it look better" | "Increase vertical rhythm. Use 8px spacing grid. Card padding should be 24px." |

Choose a clear design intent — bold maximalism or refined minimalism — and tell Claude which direction to follow. Intentionality produces better output than intensity.

---

## CLAUDE.md for Frontend Projects

Store your design system constraints in CLAUDE.md so Claude reads them at session start:

```markdown
## Design System
- Color tokens: `--color-primary: #1a1a2e`, `--color-accent: #e94560`
- Spacing scale: 4px base, multiples of 4
- Typography: Heading → "Clash Display", Body → "Inter"
- Component library: shadcn/ui (do not introduce new libraries without asking)
- CSS approach: Tailwind utility classes; no inline styles

## Component Conventions
- All components in `src/components/`, PascalCase filenames
- Co-locate tests: `Button.test.tsx` next to `Button.tsx`
- Use `cn()` helper from `lib/utils` for conditional classes
```

With this in CLAUDE.md, every session starts with your conventions already loaded.

---

## Skills for Reusable UI Workflows

Package proven UI instructions into skills so you get consistent results on every invocation:

**`/baseline-ui`** — strips generic output, improves spacing, typography, and visual states:
```markdown
---
description: Elevate generic UI to production quality — spacing, typography, visual states
---
Review $ARGUMENTS for baseline UI quality issues:
- Inconsistent spacing (not on grid)
- Default browser fonts or common system fonts
- Missing hover/focus/active/disabled states
- Low-contrast text or borders
- Lack of visual hierarchy

Fix all issues found. Do not change functionality.
```

**`/frontend-design`** — full design-process workflow:
```markdown
---
description: Build UI following a real design process: research → direction → implement → verify
---
For $ARGUMENTS, follow this process:
1. Read existing components to understand patterns in use
2. Propose a design direction (typography, color, spacing intent)
3. Implement with that direction
4. Take a screenshot and compare to any reference provided
5. Fix visual differences
```

**`/fixing-accessibility`** — a11y audit and fix:
```markdown
---
description: Audit and fix accessibility issues in a component or page
---
Audit $ARGUMENTS for WCAG 2.1 AA compliance:
- Color contrast ratios
- Missing ARIA labels and roles
- Keyboard navigation (focus order, focus visible)
- Screen reader announcements for dynamic content

Fix all issues. Run axe or similar if available.
```

---

## Figma MCP Integration (Optional)

Connect Figma MCP to pull design tokens and generate components directly from design files:

```bash
claude mcp add figma <figma-mcp-command>
```

With Figma MCP, Claude can:
- Read design tokens (colors, typography, spacing) directly from Figma
- Generate production-ready React components that match the design file
- Keep component output in sync with design updates

---

## Recommended Workflow

```
1. Plan Mode (Shift+Tab)
   → Read component tree, understand existing patterns
   → Propose multi-file strategy before writing any code

2. Implement
   → Reference existing patterns: "@src/components/Button.tsx — follow this pattern for the new Card component"
   → Use design system from CLAUDE.md

3. Verify
   → Screenshot the result and compare to the design target
   → Use Playwright MCP for interactive/responsive checks
   → Do not consider the task done until visual output is confirmed

4. Polish
   → Run /baseline-ui on the component
   → Check all interactive states (hover, focus, disabled, loading)
```

Always provide verification criteria — tests or screenshots — before calling a UI task complete.

---

## Context Management for UI Work

- Screenshots consume moderate context. Use `/clear` between unrelated design tasks.
- For iterative single-component refinement, stay in one session.
- For multiple distinct features, use separate sessions.
- Delegate broad design research to a subagent so it does not pollute the main context.

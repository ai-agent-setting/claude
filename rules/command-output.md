## Command Output Reduction

Suppress verbose output to avoid wasting context tokens on installation logs and progress bars.

**Package managers — always add quiet flags:**
- npm: `--silent` or `--loglevel=error`
- yarn: `--silent`
- pnpm: `--reporter=silent`
- pip: `-q`
- brew: `--quiet`

**Long-running commands — limit visible output:**
- Append `2>/dev/null` to suppress stderr noise
- Use `| tail -5` when only the final result matters
- Combine: `npm install --silent 2>/dev/null`

**Never suppress:**
- Error output when a command fails
- Build output with actionable warnings
- Test results

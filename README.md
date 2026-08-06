# audit-accessibility

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) that performs a static accessibility audit of React components (JSX/TSX, plus `.js` files containing JSX) and their associated CSS/SCSS, against WCAG 2.2 (Levels A, AA, AAA).

## What it does

- Reads your code and reports WCAG 2.2 violations — it does not touch any files until you explicitly approve fixes.
- Groups findings by **severity** (Critical / Serious / Moderate), based on the actual barrier to users, not just the WCAG level.
- Supports two review modes:
  - **File mode** — audit whole files or directories you point it at.
  - **Branch mode** — audit only the code changed on your current branch (diffed against the merge base), tagging each finding as `Introduced` or `Pre-existing (affects new code)`.
- Optionally incorporates a [Level Access](https://www.levelaccess.com/) browser-extension screenshot as enrichment for runtime issues (e.g. color contrast at scale) that static analysis can't catch. This is never required to run an audit.
- Ends every report with a one-line severity summary, a merge-readiness verdict (PASS/FAIL), and a question asking which fixes (if any) to apply.

## Installation

Clone or copy this repo into your Claude Code skills directory, e.g.:

```bash
git clone git@github.com:amandamayfield/audit-accessibility-skill.git ~/.claude/skills/audit-accessibility
```

Claude Code discovers skills by the `SKILL.md` file (with `name`/`description` frontmatter) at the skill root — see [SKILL.md](SKILL.md).

## Usage

Once installed, invoke it naturally in a Claude Code conversation, e.g.:

- "Audit accessibility on `src/components/Modal.tsx`"
- "Check my branch for a11y issues"
- "Run a WCAG compliance audit on the checkout flow"

Claude will:

1. Determine review mode (file vs. branch) from context, or ask if it's unclear.
2. Run a **read-only** audit (Read/Grep/Glob/`git` only — no edits).
3. Present a full report grouped by severity, with a merge-readiness verdict.
4. Wait for your explicit approval (`all` / `by severity` / `specific items` / `none`) before applying any fixes.

## Repository structure

```
SKILL.md                       Entry point: overview, workflow, output format
references/
  POUR.md                      WCAG 2.2 criteria + React-specific patterns (read before auditing)
  WCAG.md                      Authoritative criterion numbers, levels, and testing tools
  A11Y.md                      Accessibility code patterns (React/JSX-first examples)
  BRANCH-OR-MAIN.md            Mode-selection flowchart + branch-mode diffing rules
  READ-ONLY.md                 Enforcement flowchart for the read-only audit phase
```

Reference files are loaded progressively — only when the skill's workflow reaches the section that needs them — to keep unrelated conversations token-efficient.

## Design principles

- **Read-only by default.** The audit phase never edits files; fixes are only applied after the user picks an option from the report's closing question.
- **WCAG level ≠ severity.** Conformance level (A/AA/AAA) is a factual lookup in `WCAG.md`; severity (Critical/Serious/Moderate) is rated by actual user impact. The two are reported separately and never inferred from one another.
- **References over training memory.** WCAG 2.2 is recent enough that model training data can be stale on exact criterion numbers and additions (2.4.11, 2.5.7, 2.5.8, 3.2.6, 3.3.7, 3.3.8). The skill treats `references/*.md` as the authoritative source, not a formality.

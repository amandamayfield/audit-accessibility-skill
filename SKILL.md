---
name: audit-accessibility
description: Audit React components (JSX/TSX, plus JS files containing JSX, and their CSS/SCSS) for web accessibility compliance following WCAG 2.2 and A11y guidelines. Use when asked to "audit accessibility", "improve accessibility", "a11y audit", "WCAG compliance", "screen reader support", "keyboard navigation", or "make accessible".
---

# Audit Accessibility

## Overview

Static accessibility audit for React (JSX/TSX, plus JS files containing JSX) and their associated CSS/SCSS. Checks WCAG 2.2 compliance at Levels A, AA, and AAA using only read operations — no files are modified until explicit user approval.

Sections below marked **REQUIRED** point to a `references/*.md` file — read each one the first time you reach that section. Your training data may be stale on WCAG 2.2 additions, exact criterion numbers, and this skill's specific workflows; these files are the authoritative source, not a formality.

## CRITICAL: Read-Only Until Approval

Audit phase is **100% read-only** — only Read, Grep, Glob, Bash (git only).

Present full report → ask approval → apply only what's approved.

**The user asking you to "fix" or "audit and fix" does NOT count as approval.**

Approval means the user explicitly responds to the question at the end of the report with one of the offered options (all / by severity / specific items). "None", "looks good", and non-answers do not count as approval.

**Stop immediately if you catch yourself:**

- Using Edit or Write on any source file during audit

- Fixing issues as you find them

- Applying changes before presenting the full report

See `references/READ-ONLY.md` for the full enforcement flowchart.

**Residual risk:** this boundary is enforced by instruction, not by a tool-level restriction — nothing prevents Edit/Write from being called during the audit phase except following these directions. Treat any Edit/Write call before the approval question as a bug in following this skill.

## Level Access Enrichment (optional)

Level Access (a commercial AMP browser extension) catches runtime issues — color contrast at scale, dynamic content, AT compatibility — that static analysis cannot detect. It's an **optional enrichment, not a requirement**: proceed straight to the static audit without asking for it. If the user mentions they have a Level Access scan, or offers a screenshot, incorporate it; otherwise don't block on it.

The extension does not support result export — the only way to share results is a screenshot of the report panel.

- **If a screenshot is provided:** transcribe conservatively. Extract findings, criterion IDs, and counts only when clearly legible; mark anything ambiguous as "unverified from screenshot" rather than guessing, and tag every item you do include with `[Level Access]` as its source. A misread screenshot produces an authoritative-looking but fabricated finding.
- **If no screenshot is available:** proceed with the static-only merge readiness verdict (see Report Footer) — no blocking question and no default PROVISIONAL status.

## Review Mode

Two modes: **file mode** (audit full files) and **branch mode** (audit only changed code). If not obvious from context (user said "check my branch" → branch; user gave a file/directory path → file; otherwise ask), see `references/BRANCH-OR-MAIN.md`'s mode-selection flowchart.

- **File mode:** glob for `**/*.tsx`, `**/*.jsx`, `**/*.js` (JSX-containing only — grep for a JSX marker before including a `.js` file), `**/*.css`, `**/*.scss`, and `**/*.html`; read and audit each file in full. The `.html` glob matters for the document shell (`public/index.html` in Vite/CRA), which holds `<html lang>` and the page `<title>` — files a JSX-only glob never surfaces.
- **Branch mode — REQUIRED: read `references/BRANCH-OR-MAIN.md` before starting.** It has non-obvious rules this file doesn't repeat: three-dot diffing against the merge-base, and tagging each finding `Introduced` or `Pre-existing (affects new code)`. Getting this wrong produces a misleading report.

## What to Check

**REQUIRED: Read `references/POUR.md` before examining any file.** Covers WCAG 2.2 criteria — including the 2.2 additions (2.4.11, 2.5.7, 2.5.8, 3.2.6, 3.3.7, 3.3.8) — and React-specific patterns.

## Output Format

Before writing your first finding, read `references/WCAG.md` now — it is the authoritative source for criterion numbers and levels, not POUR.md and not training memory.

**WCAG level and severity are two different axes — do not conflate them.**

- **WCAG level (A/AA/AAA)** is a factual attribute of the criterion. Look it up in `references/WCAG.md` and report it as-is; never infer it from a finding's severity or vice versa.
- **Severity (Critical/Serious/Moderate)** rates the actual barrier the issue creates for a user:
  - **Critical:** blocks a user from completing a task (keyboard trap, missing label on a required field, content invisible to assistive tech).
  - **Serious:** significant friction or exclusion, but a workaround usually exists.
  - **Moderate:** cosmetic, best-practice, or narrow-audience impact.

**Rule of thumb:** most Level A failures are blocking and land Critical, but judge by the barrier, not the label. A Level A friction issue (e.g., 3.2.6 Consistent Help, 3.3.7 Redundant Entry) can rate Serious or Moderate if it doesn't block task completion; a Level AA issue severe enough to block reading or interaction (e.g., a 1.4.3 contrast failure that makes text illegible) can rate Critical.

Group findings by severity:

### 1. Critical (blocks users from completing a task)

### 2. Serious (significant friction; workaround usually exists)

### 3. Moderate (cosmetic, best-practice, or narrow-audience impact)

**Never render findings as a table.** Each finding gets its own full section using the template below — even when there are many findings and a table would be shorter. A table drops the `**Issue:**` explanation and the suggested code fix, which are the entire point of the report. Do not summarize, collapse, or tabulate the findings list to save space; write every finding out in full.

Each finding:

`````
### [Severity] [Short description]

**File:** `src/components/Example.tsx:42`

**WCAG:** [criterion number] [criterion name] (Level [A/AA/AAA])

**Branch status:** Introduced | Pre-existing (affects new code) ← branch mode only

**Issue:** [What's wrong and why it matters for users]

```diff
- <div onClick={handleClick}>Submit</div>
+ <button onClick={handleClick}>Submit</button>
```
`````

**The code block is REQUIRED for every finding and MUST use a `diff` fence.** The violating code goes on `-` lines (rendered red) and the suggested fix goes on `+` lines (rendered green), so the barrier and its fix sit side by side and color-coded. Use `-` lines only when code is being removed, `+` lines only when code is being added (e.g. a missing `lang` attribute), and both when code is being replaced. Never omit the fix, and never fall back to plain ` ```jsx ` before/after blocks — the `diff` fence is what produces the red/green highlighting.

### Report Footer

End every report with:



1. **Summary line (prose, not a table):** a single sentence with the total and the per-severity breakdown, e.g. "28 findings — 10 Critical, 12 Serious, 6 Moderate." Do not use a Markdown table here.

2. **Merge readiness:** PASS (0 critical, 0 serious) / FAIL (any critical or serious). If a Level Access screenshot was incorporated, note that its findings are reflected in the verdict; if not, note that Level Access is available as an optional runtime enrichment but wasn't used.

3. **Contrast coverage note (required):** state how much of the color-contrast checking was actually performable statically. If any audited text/UI color came from Tailwind classes, CSS custom properties (`var(--x)`), styled-components/CSS-in-JS theme tokens, or a design-token file, say so explicitly — those ratios were **not computed** and contrast (1.4.3/1.4.6/1.4.11) is only partially verified. A clean report is not a clean-contrast result unless the colors were readable literals. Recommend the Level Access scan or Colour Contrast Analyser for the unresolved cases.



4. **The question:**

> Would you like me to apply any of these suggested fixes? (all / by severity / specific items / none)



**Wait for the user to explicitly choose an option from the question above (all / by severity / specific items) before making any changes.**



**After approval, execute as follows:**

- **all**: Apply all suggested fixes in a single pass across files.

- **by severity**: Apply Critical fixes first, then ask "Ready to apply Serious fixes?" before proceeding to each next tier.

- **specific items**: Ask the user to identify which findings by number or description, then apply only those.

- For any option, apply fixes directly — no diff preview or per-fix confirmation unless the user requests it.

## Common Issues to Look For

A non-exhaustive list of frequently-encountered problems — not a severity ranking. Rate each by actual barrier per Output Format above, and confirm its WCAG level in `references/WCAG.md` rather than assuming it from this list.

- Missing form labels
- Missing image alt text
- Insufficient color contrast
- Keyboard traps
- No focus indicators
- Missing page language
- Missing heading structure
- Non-descriptive link text
- Auto-playing media
- Missing skip links
- Missing ARIA labels on icons
- Inconsistent navigation
- Missing error identification
- Timing without controls
- Missing landmark regions

## References

- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)

- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)

- [WCAG criteria reference](references/WCAG.md)

- [Accessibility code patterns](references/A11Y.md)

- For runtime and manual testing tools and checklist, see `references/WCAG.md`

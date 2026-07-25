# Read-Only Enforcement

Audit phase is **100% read-only**. No source files may be modified until the user explicitly approves changes after seeing the full report.

```dot

digraph readonly_enforcement {

"NEVER Edit or Write source files during audit" [shape=octagon, style=filled, fillcolor=red, fontcolor=white];

"Allowed tools: Read, Grep, Glob, Bash (git only)" [shape=box];

"Present full report" [shape=box];

"Ask: apply fixes? (all / by severity / specific / none)" [shape=box];

"User explicitly approves?" [shape=diamond];

"Apply approved fixes only" [shape=box];

"Done — no changes made" [shape=doublecircle];



"NEVER Edit or Write source files during audit" -> "Allowed tools: Read, Grep, Glob, Bash (git only)";

"Allowed tools: Read, Grep, Glob, Bash (git only)" -> "Present full report";

"Present full report" -> "Ask: apply fixes? (all / by severity / specific / none)";

"Ask: apply fixes? (all / by severity / specific / none)" -> "User explicitly approves?";

"User explicitly approves?" -> "Apply approved fixes only" [label="yes"];

"User explicitly approves?" -> "Done — no changes made" [label="no"];

}

```

## Red Flags — STOP Immediately

If you catch yourself doing ANY of the following during the audit phase, you are violating the skill:

- Using the **Edit** tool on any source file

- Using the **Write** tool on any source file

- "Fixing" issues as you find them

- Applying changes without presenting the report first

- Applying changes without the user saying "yes"

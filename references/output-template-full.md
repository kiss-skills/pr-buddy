# Output Template — Full (walkthrough + automated checks)

Use when Phase 2 ran at least one agent (menu options [1] or [2]).

Use `<placeholder>` tokens — replace with actual walkthrough content when synthesizing.

```markdown
# Understanding `[walkthrough + automated checks]`
- **Goal:** <one-sentence goal agreed in Station 1>
- **Key decisions:** <1–2 notable choices surfaced in Station 3, with their trade-offs>
- **Main risk identified:** <sharpest risk from Station 5 — reviewer's or Claude's, whichever is more specific>

# Review Details

## [MANUAL REVIEW]

<Narrative synthesis of the full walkthrough. This is the primary review voice — write 4–7
sentences as an informed co-reviewer, not as a summary of the conversation. Include:>

- What this PR is actually doing and why (beyond the title)
- The code path traced: entry point → call flow → key interactions
- Each notable decision from Station 3, with the trade-off and its implications
- The risk(s) from Station 5 — what you and the reviewer named, and how serious they are
- The reviewer's confidence level and anything that was still uncertain at Station 6

## [AUTOMATED]

<Intercept raw agent output. For each finding: assign severity, write one colloquial line,
prepend emoji. Do NOT paste agent output verbatim.

Severity:
- 🔴 critical — must fix before merge
- 🟠 important — should fix, not a hard blocker
- 🟡 nice to have — low priority or follow-up

One line per finding. Agent attribution optional (e.g. "*(silent-failure-hunter)*" at end).
If no issues: write "No findings above the reporting threshold.">

# Summary

## Blocking 🔴:
<List items that must be addressed before this PR can merge. One per line, starting with
a concrete action: "Fix ...", "Add test for ...", "Remove ...". Empty if none.>

## Requests 🟠:
<Things the reviewer would like to see but aren't hard blockers. Same format. Empty if none.>

## Nice to Have 🟡:
<Low-priority suggestions, style improvements, or "consider for a follow-up" notes. Empty if none.>
```

## Synthesis rules

**`## [MANUAL REVIEW]` is always primary.** Write it as a co-reviewer who traced the code and
engaged with the decisions — not as a recap of the conversation.

**`## [AUTOMATED]` is compressed.** Intercept raw agent output and rewrite each finding as
one colloquial line with a severity emoji (🔴/🟠/🟡). No verbatim agent output. No sub-headings
by agent. Anchor findings to walkthrough observations where natural.

**`# Summary` is written from the walkthrough lens.** Automated findings contribute to
Blocking/Requests/Nice to Have only when they relate to code paths, decisions, or risks
already identified in the walkthrough. A finding entirely disconnected from the walkthrough
is still worth listing under Nice to Have.

**Verdict** (optional — add to end of Summary if useful):
```
**Verdict:** Approve / Request changes / Comment
```

For option [2] (code-reviewer only): run only `pr-review-toolkit:code-reviewer`. The
`## [AUTOMATED]` section includes only those findings; format is unchanged.

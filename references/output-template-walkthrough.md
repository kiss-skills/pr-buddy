# Output Template — Walkthrough only

Use when the reviewer selected option [3] (skip agents) from the Phase 2 menu.

Use `<placeholder>` tokens — replace with actual walkthrough content when synthesizing.

```markdown
# Understanding `[walkthrough]`
- **Goal:** <one-sentence goal agreed in Station 1>
- **Key decisions:** <1–2 notable choices surfaced in Station 3, with their trade-offs>
- **Main risk identified:** <sharpest risk from Station 4>

# Review Details

## 1. `[reviewer]`

<Narrative synthesis of the full walkthrough — 4–7 sentences as an informed co-reviewer.
Do not truncate or thin out this section just because no agents ran. Include:>

- What this PR is actually doing and why (beyond the title)
- The code path traced: entry point → call flow → key interactions
- Each notable decision from Station 3, with the trade-off and its implications
- The risk(s) from Station 4 — what you and the reviewer named, and how serious they are
- The reviewer's confidence level and anything that was still uncertain at Station 5

# Summary

## Blocking 🔴:
<Issues identified through the walkthrough that must be addressed before merge. One per
line: "Fix ...", "Add test for ...", "Remove ...". Empty if none.>

## Requests 🟠:
<Things the reviewer wants but aren't hard blockers. Empty if none.>

## Nice to Have 🟡:
<Low-priority suggestions. Empty if none.>
```

## Synthesis rule

**`## 1. [reviewer]` is the entire review voice here.** Without automated agents, all
findings come from the walkthrough conversation. Write it with the full weight of what
was discussed — goal, architecture, decisions, risks, confidence — as a co-reviewer
who traced the code, not as a summary of the chat.

**Verdict** (optional — add to end of Summary if useful):
```
**Verdict:** Approve / Request changes / Comment
```

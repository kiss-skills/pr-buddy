# Output Template — Full (walkthrough + automated checks)

Use when Phase 2 ran at least one agent (menu options [1] or [2]).

Use `<placeholder>` tokens — replace with actual walkthrough content when synthesizing.

```markdown
# Understanding `[walkthrough + automated checks]`
- **Goal:** <one-sentence goal agreed in Station 1>
- **Key decisions:** <1–2 notable choices surfaced in Station 3, with their trade-offs>
- **Main risk identified:** <sharpest risk from Station 4 — reviewer's or Claude's, whichever is more specific>

# Review Details

## 1. `[reviewer]`

<Narrative synthesis of the full walkthrough. This is the primary review voice — write 4–7
sentences as an informed co-reviewer, not as a summary of the conversation. Include:>

- What this PR is actually doing and why (beyond the title)
- The code path traced: entry point → call flow → key interactions
- Each notable decision from Station 3, with the trade-off and its implications
- The risk(s) from Station 4 — what you and the reviewer named, and how serious they are
- The reviewer's confidence level and anything that was still uncertain at Station 5

## 2. Automated findings:

### 2.1. `[code-reviewer]`

<Findings from pr-review-toolkit:code-reviewer. Anchor to walkthrough where relevant:
e.g. "This touches the entry point we traced in Station 2 — ...". If no issues, write:
"No issues found above the reporting threshold.">

### 2.2. `[silent-failure-hunter]`

<Findings from pr-review-toolkit:silent-failure-hunter. Connect to Station 4 risks
where applicable. If no issues, write: "No silent failure patterns identified.">

### 2.3. `[pr-test-analyzer]`

<Findings from pr-review-toolkit:pr-test-analyzer. Note whether the risk scenarios
from Station 4 are covered. If no gaps, write: "Test coverage adequate for the
risk scenarios we identified.">

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

**`## 1. [reviewer]` is always primary.** Write it as a co-reviewer who traced the code and
engaged with the decisions — not as a recap of the conversation. If automated findings
contradict or reinforce what came from the walkthrough, say so here.

**`# Summary` is written from the walkthrough lens.** Automated findings contribute to
Blocking/Requests/Nice to Have only when they relate to code paths, decisions, or risks
already identified in the walkthrough. A finding entirely disconnected from the walkthrough
is still worth listing under Nice to Have, with a note that it was flagged outside the
traced path.

**Verdict** (optional — add to end of Summary if useful):
```
**Verdict:** Approve / Request changes / Comment
```

For option [2] (code-reviewer only): omit sections 2.2 and 2.3 — keep only `### 2.1. [code-reviewer]`.

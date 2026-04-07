# pr-review-toolkit Bridge

## Purpose

Phase 2 invokes pr-review-toolkit sub-agents directly — not via the black-box `/review-pr`
command — so the Phase 1 mental model can be injected into each agent's prompt. The result
is a single combined PR comment that reflects both the reviewer's understanding and the
automated findings.

## Why direct sub-agent invocation

Calling `/review-pr <PR>` hands off to the toolkit without context. The agents run on the
diff alone; they don't know:
- Which risk the reviewer already identified
- Which decisions were discussed as intentional trade-offs
- Which code path the reviewer traced

Direct invocation lets us pass this context as part of each agent's prompt, so findings are
focused and the final comment is coherent — not a disconnected checklist.

## Phase 1 context block

Built after Station 5, before any sub-agent is invoked. Never shown to the reviewer.

```
PHASE_1_CONTEXT:
  goal:          <agreed in Station 1>
  entry_point:   <file from Station 2>
  decisions:     [<decision + trade-off from Station 3>, ...]
  reviewer_risk: <risk the reviewer named in Station 4>
  skill_risk:    <risk Claude identified in Station 4>
  confidence:    <reviewer's Station 5 self-report>
```

## Sub-agent invocation prompts

**code-reviewer** — focus + known trade-offs:
> "Review this PR diff for correctness and project guidelines. The reviewer traced the entry
> point as `<entry_point>`. The following were discussed as intentional design decisions —
> treat them as known trade-offs, not issues: `<decisions>`."

**silent-failure-hunter** — focus on the risk area:
> "Hunt for silent failures in this PR diff. The reviewer identified `<reviewer_risk>` as the
> main risk. Pay particular attention to error handling in that area."

**pr-test-analyzer** — check risk scenarios:
> "Analyze test coverage for this PR. Check whether the following risk scenarios have test
> coverage: `<reviewer_risk>`, `<skill_risk>`."

## Escalation rules

Include in the combined comment (with walkthrough anchor):
- Findings that validate or refute a risk from Station 4
- Issues in the code path traced in Station 2
- Consequences of decisions discussed in Station 3

Pass through without anchoring:
- Style violations unrelated to the walkthrough
- Dependency version warnings
- Boilerplate / generated-code findings

## Combined comment format

```markdown
**Review (walkthrough + automated checks)**

**Understanding:**
- **Goal:** <Station 1>
- **Key decision:** <Station 3 — choice + trade-off>
- **Main risk identified:** <Station 4>

**Automated findings:**
- [code-reviewer] <finding>
- [silent-failure-hunter] <finding>
- [pr-test-analyzer] <finding>

**Verdict:** Approve / Request changes / Comment
```

Post via `gh pr comment <PR> --body '...'` — only after reviewer confirms.

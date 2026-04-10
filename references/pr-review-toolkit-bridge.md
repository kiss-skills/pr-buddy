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
- What the reviewer said in their own words at each station

Direct invocation lets us pass this context as part of each agent's prompt, so findings are
focused and the final comment is coherent — not a disconnected checklist.

## Phase 1 context — two artifacts

Built after Station 5, before any sub-agent is invoked. Neither is shown to the reviewer.

### Artifact 1 — Structured block

```
PHASE_1_CONTEXT:
  goal:          <agreed in Station 1>
  entry_point:   <file from Station 2>
  decisions:     [<decision + trade-off from Station 3>, ...]
  reviewer_risk: <risk the reviewer named in Station 4>
  skill_risk:    <risk Claude identified in Station 4>
  confidence:    <reviewer's Station 5 self-report>
```

### Artifact 2 — Reviewer narrative (new in v0.4.0)

A 3–5 sentence prose paragraph that captures the substance of the walkthrough conversation:
what the reviewer said, what surprised them, what concerns they voiced, what Claude observed,
and how confident the reviewer felt by Station 5.

This is also the seed for `## 1. [reviewer]` in the final comment — expand it there to
4–7 sentences as a co-reviewer voice, not a conversation recap.

Example:
> "The reviewer correctly identified the auth middleware as the entry point but initially
> missed that session.ts was a net-new file with no test coverage. In Station 3 they
> engaged thoughtfully with the JWT expiry choice and were comfortable with the trade-off.
> Their Station 4 risk (concurrent token refresh) turned out to be exactly the failure mode
> I spotted in the diff — an unusual alignment that makes it high confidence. They felt
> ready by Station 5 but flagged token storage as something they'd want to revisit."

## Sub-agent invocation prompts

Each agent receives both artifacts in its prompt.

**code-reviewer** — focus + known trade-offs:
> "Review this PR diff for correctness and project guidelines. The reviewer traced the entry
> point as `<entry_point>`. The following were discussed as intentional design decisions —
> treat them as known trade-offs, not issues: `<decisions>`.
>
> Reviewer's walkthrough context: `<narrative paragraph>`"

**silent-failure-hunter** — focus on the risk area:
> "Hunt for silent failures in this PR diff. The reviewer identified `<reviewer_risk>` as the
> main risk. Pay particular attention to error handling in that area.
>
> Reviewer's walkthrough context: `<narrative paragraph>`"

**pr-test-analyzer** — check risk scenarios:
> "Analyze test coverage for this PR. Check whether the following risk scenarios have test
> coverage: `<reviewer_risk>`, `<skill_risk>`.
>
> Reviewer's walkthrough context: `<narrative paragraph>`"

## Escalation rules

Include in the combined comment (with walkthrough anchor):
- Findings that validate or refute a risk from Station 4
- Issues in the code path traced in Station 2
- Consequences of decisions discussed in Station 3

Pass through without anchoring (list under Nice to Have 🟡):
- Style violations unrelated to the walkthrough
- Dependency version warnings
- Boilerplate / generated-code findings

## Combined comment format

Templates:
- With automated findings: [output-template-full.md](output-template-full.md)
- Walkthrough only: [output-template-walkthrough.md](output-template-walkthrough.md)

Key rule: `## 1. [reviewer]` is always the primary voice. Automated findings are supporting
evidence. The `# Summary` section is written from the walkthrough lens — agents contribute
findings, but the verdict is Claude's as an informed co-reviewer.

Post via `gh pr comment <PR> --body '...'` — only after reviewer confirms.

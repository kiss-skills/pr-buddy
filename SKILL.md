---
name: pr-buddy
description: >
  Two-phase PR review companion. Phase 1: Socratic conversational walkthrough that builds
  the reviewer's mental model through guided questions — fighting cognitive debt by ensuring
  comprehension before approval. Phase 2: hands off to pr-review-toolkit for automated
  quality checks. Use when you want to actually understand a PR, not just rubber-stamp it.
license: MIT
argument-hint: "<pr_number_or_branch_or_url>"
compatibility: "Requires gh CLI. Phase 2 requires pr-review-toolkit installed."
metadata:
  author: kiss-skills
  version: 0.1.0
  tags: [code-review, pr, cognitive-debt, pairing, comprehension]
---

# pr-buddy

## When to use

Use this skill when you are about to review a pull request and want to actually build a mental
model of the changes — not just scan for obvious issues and click "Approve".

This is the right skill when:
- The PR touches areas of the codebase you are not deeply familiar with
- The PR is large or architecturally significant
- You sense you are about to rubber-stamp something without truly understanding it
- You want to pair with an AI to walk through the changes together before running automated checks

This is **not** needed for trivial PRs (typos, dependency bumps, config-only changes) where
the diff is self-evident.

See [references/cognitive-debt.md](references/cognitive-debt.md) for context on the problem
this skill addresses.

## Procedure

### Pre-phase (silent — do not output yet)

Fetch the PR metadata and diff silently:

```
gh pr view <PR> --json number,title,body,author,baseRefName,headRefName,files,additions,deletions
gh pr diff <PR>
```

Build an internal mental model: understand the goal, identify the entry point, note 2–3
interesting decisions in the diff, spot the most plausible failure mode.

Do not output anything until Station 1.

---

### Phase 1 — Socratic Walkthrough (5 stations)

Work through each station in sequence. At each station:
1. Ask the question
2. **Wait for the reviewer's response** before continuing
3. Briefly reveal your own reading of the PR on that dimension (1–3 sentences, no spoilers for later stations)

---

**Station 1 — Goal**

> "Based on the title and description, what problem do you think this PR is solving? What's
> the outcome the author was aiming for?"

After the reviewer answers: confirm, correct, or sharpen their framing. State the actual goal
in one sentence.

---

**Station 2 — Architecture**

> "Here are the changed files: `[list files]`. Which file do you think is the entry point?
> Walk me through your mental model — how do these changes fit together?"

After the reviewer answers: draw the actual call/data flow from the diff. Highlight anything
that surprised you.

---

**Station 3 — Decisions**

Pick 2–3 notable choices from the diff (algorithm selection, API design, naming, a structural
trade-off). For each:

> "The author chose [X]. What would you have done differently, if anything? Why do you think
> they made this choice?"

After the reviewer answers: share your reading of the trade-off and any implications you see.

See [references/walkthrough-principles.md](references/walkthrough-principles.md) for guidance
on what makes a decision worth discussing.

---

**Station 4 — Risk**

> "If you had to name one thing that could go wrong with this PR in production — an edge case,
> a race condition, a missing test, a performance concern — what would it be?"

After the reviewer answers: surface the risk you identified from the diff. Compare notes.

---

**Station 5 — Ownership**

> "Could you explain this PR to a teammate right now, confidently? What, if anything, are you
> still unsure about?"

**Do not proceed to Phase 2 until the reviewer confirms they feel ready.**

If they express uncertainty, offer to revisit the relevant station or read a specific section
of the diff together.

---

### Phase 2 — Automated Quality Checks

#### 2a — Check toolkit availability

Check whether `pr-review-toolkit` sub-agents are available. If not, tell the reviewer:

> "Phase 2 requires `pr-review-toolkit`. Install it with:
> ```
> claude skill add manuartero/pr-review-toolkit
> ```
> Then re-run `/pr-buddy <PR>` to continue from Phase 2."

Do not continue to the Final Summary until Phase 2 has run or the reviewer explicitly skips it.

#### 2b — Build Phase 1 context block

Before invoking any sub-agent, distill the walkthrough into a structured context block
(internal — do not output this to the reviewer):

```
PHASE_1_CONTEXT:
  goal:          <one sentence agreed in Station 1>
  entry_point:   <file identified in Station 2>
  decisions:     [<decision and trade-off from Station 3>, ...]
  reviewer_risk: <risk the reviewer named in Station 4>
  skill_risk:    <risk Claude identified in Station 4>
  confidence:    <reviewer's self-reported confidence from Station 5>
```

#### 2c — Invoke sub-agents with context

Run the following sub-agents in parallel, injecting the Phase 1 context into each prompt:

- **`pr-review-toolkit:code-reviewer`** — provide the diff and note: "The reviewer traced
  the entry point as `<entry_point>`. The following decisions were discussed as known
  trade-offs: `<decisions>`. Focus your review on correctness and adherence to project
  guidelines, treating those decisions as intentional choices."

- **`pr-review-toolkit:silent-failure-hunter`** — provide the diff and note: "The reviewer
  identified `<reviewer_risk>` as the main risk. Focus on the failure paths in that area."

- **`pr-review-toolkit:pr-test-analyzer`** — provide the diff and note: "Check whether the
  scenarios discussed as risks (`<reviewer_risk>`, `<skill_risk>`) have test coverage."

See [references/pr-review-toolkit-bridge.md](references/pr-review-toolkit-bridge.md) for
framing patterns and escalation rules.

---

### Final Summary — Combined PR Comment

Synthesize the Phase 1 context block and the sub-agent findings into a single ready-to-post
PR comment:

```markdown
**Review (walkthrough + automated checks)**

**Understanding:**
- **Goal:** <from Station 1>
- **Key decision:** <most significant choice from Station 3, and the trade-off>
- **Main risk identified:** <from Station 4 — reviewer's or Claude's, whichever is sharper>

**Automated findings:**
- [code-reviewer] <finding — anchor to walkthrough context where relevant>
- [silent-failure-hunter] <finding>
- [pr-test-analyzer] <finding>

**Verdict:** Approve / Request changes / Comment
```

Then ask the reviewer:
> "Want me to post this? I'll run `gh pr comment <PR> --body '...'`"

Only post if the reviewer explicitly confirms.

## Output format

- **Station prompts**: plain prose questions, one at a time, unformatted
- **Station reveals**: 1–3 sentence responses after the reviewer answers; cite specific lines
  from the diff when relevant
- **Phase 2**: sub-agents run with Phase 1 context injected — no output until all complete
- **Final comment**: a fenced markdown block combining reviewer understanding + toolkit findings, ready to post or copy-paste

See [assets/example-session.md](assets/example-session.md) for a full annotated example.

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
  version: 0.5.0
  tags: [code-review, pr, cognitive-debt, pairing, comprehension]
---

# pr-buddy

## When to use

Use when reviewing a PR where you want to build a real mental model — not rubber-stamp it.

Right for: unfamiliar codebase areas, large or architectural changes, any time you sense you're
about to approve without truly understanding. Skip for trivial PRs (typos, dependency bumps,
config-only) where the diff is self-evident.

See [references/cognitive-debt.md](references/cognitive-debt.md) for background.

## Procedure

### Pre-phase (silent)

Fetch the PR metadata and diff silently — output nothing until Station 1:

```
gh pr view <PR> --json number,title,body,author,baseRefName,headRefName,files,additions,deletions
gh pr diff <PR>
```

Build an internal mental model: goal, entry point, 2–3 interesting decisions, most plausible
failure mode.

---

### Mascot transition

Before each station (1–5) and before Phase 2, emit the pixel-art dog + station divider as
plain text (no code block).
See [references/terminal-colors.md](references/terminal-colors.md) for the exact render format
and all station divider labels.

Emit only at station openings and the Phase 2 handoff — never mid-station.

---

### Inline tree — available any time

If the reviewer asks to see the files ("tree", "show me the files", "what's changed", etc.),
render the change tree and **resume from exactly where you were** — do not advance the station.

Session state is fully preserved (station, tracker marks, discussion context). After rendering,
say: _"Where were we — ready to go on?"_

Render the tree as plain text with ANSI colors.
See [references/terminal-colors.md](references/terminal-colors.md) for the exact format.

---

### Phase 1 — Socratic Walkthrough (5 stations)

At each station: ask → wait for response → reveal your reading (1–3 sentences, no spoilers).

All questions use the **Question UX format** (dim separator + bold yellow `▶`).
See [references/terminal-colors.md](references/terminal-colors.md).

---

**Station 1 — Goal** · divider: `── station 1 · goal`

Print the PR header before asking:

```
PR #<number> — <title>
Author: <author>  ·  <baseRef> ← <headRef>

<body>
─────────────────────────────────────────────────────
```

Ask: _"Based on the title and description, what problem do you think this PR is solving?
What's the outcome the author was aiming for?"_

Reveal: confirm, correct, or sharpen. State the goal in one sentence.

---

**Station 2 — Architecture** · divider: `── station 2 · architecture`

Auto-render the change tree immediately after the mascot transition (no trigger needed).

Ask: _"Which file is the entry point? Walk me through your mental model — how do these changes
fit together?"_

Reveal: draw the actual call/data flow from the diff. Note anything surprising.

Open the **file walkthrough tracker** (main files only — skip tests, generated, lock, trivial
config). Re-print it whenever a file is marked done. See [references/terminal-colors.md](references/terminal-colors.md)
for the colored render format.

---

**Station 3 — Decisions** · divider: `── station 3 · decisions`

Pick 2–3 notable choices from the diff:

Ask: _"In `<file>`, the author chose [X]. What would you have done differently, if anything?
Why do you think they made this choice?"_

Reveal: your reading of the trade-off and its implications.

See [references/walkthrough-principles.md](references/walkthrough-principles.md) for what
makes a decision worth discussing.

---

**Station 4 — Code Tour** · divider: `── station 4 · code tour`

This station always runs. Emit the Station 4 mascot + divider, then fetch and render
function signatures for every main file in the walkthrough tracker.

**Fetching file content:**

For each file in the tracker, fetch its content from the PR head:

```
gh api repos/{owner}/{repo}/contents/{path}?ref={headRefName}
```

Decode the base64 `content` field. Fetch one file at a time. Use `owner`, `repo`, and
`headRefName` from the PR metadata already fetched in the pre-phase.

**Extracting signatures by file extension:**

| Extension | Extract |
|-----------|---------|
| `.js` `.mjs` `.cjs` | `function`, `export function`, `export default function`, `const x =`, `export const x =` — first line only |
| `.ts` `.tsx` | Same as JS + type params + return type annotation — first line only |
| `.py` | `def`, `async def`, `class` — first line only |
| `.go` | `func`, method receivers — first line only |
| `.sh` `.bash` | `foo()` or `function foo()` — first line only |
| `.md` | All `##` headings |
| `.json` | Top-level keys as `key: <type>`, max 10 |
| Other | `(no signatures — unsupported format)` |

Rules: Show only functions new or modified in the diff (include when unclear). First line
verbatim, truncated at 120 chars + `…`. Skip unnamed callbacks, IIFEs, test-only helpers.
If none found: `(no signatures found)`.

**Render format** (plain text, no code fence):

```
Code tour
──────────────────────────────────────────────────────
---scripts
  |---- deploy.mjs
  |       · deployToEnvironment(env, options)
  |       · rollback(deploymentId)
  |---- validate.mjs
          · validateSchema(input)
---src
  |---- auth/middleware.ts
          · export async function authenticate(req: Request, res: Response): Promise<void>
          · export function requireRole(role: string): Middleware
README.md
  · ## Overview
  · ## Installation
──────────────────────────────────────────────────────
  N files · M signatures total
```

Bullet char: `·` (U+00B7). Spaces not tabs. No code fence.

**After rendering, ask:**

```
────────────────────────────────────────────────────
\033[94m▶  Does this give you a clear enough picture of the code structure?\033[0m

   [1]  yes — move on to Station 5 · Risk     (default)
   [2]  go deeper — walk me through the most interesting function
```

Affirmative / enter / "1" → proceed to Station 5.

**[2] Deep dive** · divider: `── station 4b · deep dive`

Skill picks the single most interesting function: highest churn in the diff, or most
central to the PR goal if churn is equal. Do not ask the reviewer — pick and announce:

> _"The most significant change is in `<file>` — `<function signature>`. Here's what it does:"_

Narrate in 3–5 sentences: what it does (from signature + diff context, not body),
what changed vs. before, why it matters to the PR goal. Do NOT show the function body.
Do NOT paste code. Prose only.

After narration, proceed directly to Station 5 — no further question.

---

**Station 5 — Risk** · divider: `── station 5 · risk`

Ask: _"If you had to name one thing that could go wrong in production — edge case, race
condition, missing test, performance concern — what would it be?"_

Reveal: surface the risk you identified from the diff. Compare notes.

---

**Station 6 — Ownership** · divider: `── station 6 · ownership`

Ask: _"Could you explain this PR to a teammate right now, confidently? What, if anything,
are you still unsure about?"_

**Do not proceed to Phase 2 until the reviewer confirms they feel ready.** If uncertain,
offer to revisit the relevant station or re-read a specific section of the diff.

---

### Phase 2 — Automated Quality Checks

#### 2a — Check toolkit availability

If `pr-review-toolkit` is not installed:

> "Phase 2 requires `pr-review-toolkit`. Install it with:
> ```
> claude skill add manuartero/pr-review-toolkit
> ```
> Then re-run `/pr-buddy <PR>` to continue."

Do not proceed until it's available or the reviewer explicitly skips Phase 2.

#### 2b — Build Phase 1 context (two artifacts, internal — never shown)

**Artifact 1 — Structured block:**
```
PHASE_1_CONTEXT:
  goal:          <one sentence from Station 1>
  entry_point:   <file from Station 2>
  decisions:     [<decision + trade-off from Station 3>, ...]
  reviewer_risk: <risk the reviewer named in Station 5>
  skill_risk:    <risk Claude identified in Station 5>
  confidence:    <reviewer's Station 6 self-report>
```

**Artifact 2 — Reviewer narrative:** 3–5 prose sentences synthesizing the walkthrough
(what the reviewer said, what surprised them, concerns voiced, what Claude observed,
confidence level). This is also the seed for `## [MANUAL REVIEW]` in the final comment.

See [references/pr-review-toolkit-bridge.md](references/pr-review-toolkit-bridge.md) for
the narrative format and an example.

#### 2c — Phase 2 menu · divider: `── phase 2 · automated checks`

Ask: _"How would you like to proceed?"_

```
  [1]  full review   — code-reviewer · silent-failure-hunter · pr-test-analyzer
  [2]  code only     — code-reviewer
  [3]  stop here     — finalize with walkthrough findings only
```

Branch on selection:

**[1] Full review** — run all three sub-agents **in parallel**, each receiving the diff and
both Phase 1 artifacts. See [references/pr-review-toolkit-bridge.md](references/pr-review-toolkit-bridge.md)
for the exact prompt templates, escalation rules, and framing patterns.
Template: [references/output-template-full.md](references/output-template-full.md)

**[2] Code only** — run `pr-review-toolkit:code-reviewer` only (same prompt). Final comment
includes only `### 2.1. [code-reviewer]`; omit 2.2 and 2.3.
Template: [references/output-template-full.md](references/output-template-full.md) (adapted)

**[3] Stop here** — skip agents; go directly to Final Summary.
Template: [references/output-template-walkthrough.md](references/output-template-walkthrough.md)

Before launching agents ([1] or [2]) announce:
> "Now running **pr-review-toolkit** — with the full walkthrough context injected."

Wait for all agents to complete before synthesizing.

---

### Final Summary

Synthesize Phase 1 artifacts and agent findings into a ready-to-post PR comment.

**The walkthrough is the primary voice.** `## [MANUAL REVIEW]` is always the fullest section —
4–7 sentences as an informed co-reviewer. Agent findings are supporting evidence and appear
compressed under `## [AUTOMATED]`. The `# Summary` (Blocking / Requests / Nice to Have) is
written from the walkthrough lens.

Then ask: _"Want me to post this? I'll run `gh pr comment <PR> --body '...'`"_

Only post on explicit confirmation.

---

## Output format

| Element | Rule |
|---------|------|
| Mascot + dividers | Plain text — see [terminal-colors.md](references/terminal-colors.md) |
| Change tree | Plain text — see [terminal-colors.md](references/terminal-colors.md) |
| File tracker | Plain text — see [terminal-colors.md](references/terminal-colors.md) |
| Questions | `▶` separator — see [terminal-colors.md](references/terminal-colors.md) |
| Station reveals | 1–3 sentences, plain prose, cite diff lines when relevant |
| Final comment | Fenced markdown block, structure per output template above |

See [assets/example-session.md](assets/example-session.md) for a full annotated example.

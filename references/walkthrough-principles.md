# Walkthrough Principles

## The Socratic approach

The skill uses a Socratic structure: ask first, reveal second. The goal is not to quiz the
reviewer — it is to activate their thinking before providing answers. A reviewer who has
formed their own opinion (even a wrong one) absorbs the correct reading more deeply than a
reviewer who was simply told.

Each station follows the same pattern:

1. **Ask** — pose the question and wait
2. **Listen** — take the reviewer's answer seriously; it tells you what they already know
3. **Reveal** — confirm, correct, or sharpen; anchor your answer to the diff

## Station design rationale

**Station 1 — Goal**: Grounds everything that follows. A reviewer who misreads the goal will
misread every decision. This station surfaces misalignment early, before the reviewer has
invested time in a wrong model.

**Station 2 — Architecture**: Forces the reviewer to trace the code path themselves. Reading
a list of changed files is passive; identifying the entry point and explaining the flow is
active. This is where "I skimmed it" becomes visible.

**Station 3 — Decisions**: The most intellectually honest station. Good code review isn't just
checking correctness — it's engaging with the choices the author made. Asking "what would
you have done?" creates genuine dialogue rather than approval theatre.

What makes a decision worth discussing:
- It involves a trade-off that isn't obvious from the code alone
- A simpler alternative exists but wasn't taken
- It will constrain future work in a non-trivial way
- It encodes an assumption about the system that could become wrong

Avoid picking stylistic or trivially correct decisions — they produce noise, not insight.

**Station 4 — Risk**: Shifts the reviewer from "does this look correct" to "what could go
wrong". These are different cognitive modes. Most reviewers default to the former; this station
forces the latter. Risk identification is a skill that atrophies without practice.

**Station 5 — Ownership**: The gate. If a reviewer cannot explain the PR in their own words,
they have not understood it well enough to approve it. This station is not punitive — it is
diagnostic. If the answer is uncertain, return to the relevant station rather than proceeding.

## Timing and pacing

Don't rush. The walkthrough is meant to slow down the review enough for understanding to form.
If a reviewer gives a one-word answer to Station 2, probe gently: "Walk me through it — which
file changes first when a request comes in?"

Phase 2 should feel like a second pass, not a separate tool. The framing ("remember what we
discussed in Station 3?") is load-bearing — it's what turns automated output into reviewer
knowledge rather than a to-do list.

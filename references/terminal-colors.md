# Terminal Render Specs

All output is **plain text only** — **except question prompts**, which use ANSI bright blue (`\033[94m`).
Emit every example below as plain text; do not wrap in a code block when outputting.

---

## Skill intro greeting (one-time, at load)

The mascot renders **exactly once** at the start of the session, as a greeting block.
After this, the skill goes silent for the pre-phase; station transitions are divider-only
(see "Station divider" below).

Emit as plain text (not inside a code block):

```
  / \__
 (    @\___
 /         O
/   (_____/
/_____/   U

         pr-buddy · let's walk through this one

─────────────────────────────────────────────────────
```

Position: first visible output of the skill, before the pre-phase `gh pr view` /
`gh pr diff` fetches. Do NOT re-render the mascot at any later point in the session —
not between stations, not at the Phase 2 handoff, not at the review complete block.

---

## Station divider

Emit at the opening of every Station 1–6 and at the Phase 2 handoff. Plain text, no
mascot above it. Immediately follow the divider with the cognitive-debt meter line
(see "Cognitive debt meter" below) — except at the Phase 2 handoff, which does not
carry a meter.

Example for Station 2:

```
── station 2 · architecture ─────────────────────────────────────────
```

Station divider labels: `station 1 · goal`, `station 2 · architecture`,
`station 3 · decisions`, `station 4 · code tour`, `station 4b · deep dive`,
`station 5 · risk`, `station 6 · ownership`, `phase 2 · automated checks`.

---

## Change tree

Emit as plain text (not inside a code block). Fill in real file paths and stats from the diff.

```
Changed files  (+<total additions> / -<total deletions>)
──────────────────────────────────────────────────────
---src
  |---- auth
  |       |---- middleware.ts        +42 / -8    ← entry point
  |       |---- session.ts           +15 / -0
  |---- utils
          |---- token.ts             +6  / -12   ← key decision
---tests
  |---- auth.test.ts           +30 / -2
──────────────────────────────────────────────────────
  3 dirs · 5 files · +93 / -22 total
```

Tree format rules:
- Root directories: `---dirname`
- Files/subdirs under a directory: `  |---- name` (2-space indent per level)
- Deeper nesting: extend the `|` pipe column (see `auth` example above)
- Top-level files (no parent dir): list flat, no `|----`
- Annotations (`←`) stay on the same line: `← entry point`, `← new module`, `← deleted`, `← most changed`
- One annotation per file. Annotate conservatively early in the session.

---

## File walkthrough tracker

Emit as plain text (not inside a code block). Re-print whenever a file is marked done.

```
File walkthrough
  [x] src/auth/middleware.ts    — entry point (discussed)
  [ ] src/auth/session.ts       — new module
  [ ] src/utils/token.ts        — key decision
```

Cover main files only — skip tests, generated files, lock files, trivial config.

---

## Cognitive debt meter

One line, immediately below each station divider. Plain text, no ANSI color. Chars:
`█` (U+2588) for debt remaining, `░` (U+2591) for debt cleared. 12-char bar; two
chars drain per station over 6 stations.

Render template:

```
cognitive debt  <bar>  <pct>%  <phase label>
```

Per-station values (render at the opening of each station, after the divider):

| Station       | Bar              | Percent | Phase label                 |
|---------------|------------------|---------|-----------------------------|
| 1 · goal      | `████████████`   | `100%`  | `unreviewed`                |
| 2 · arch.     | `██████████░░`   |  `83%`  | `goal discussed`            |
| 3 · decisions | `████████░░░░`   |  `66%`  | `architecture mapped`       |
| 4 · tour      | `██████░░░░░░`   |  `50%`  | `decisions probed`          |
| 5 · risk      | `████░░░░░░░░`   |  `33%`  | `code tour done`            |
| 6 · ownership | `██░░░░░░░░░░`   |  `17%`  | `risk identified`           |
| (post S6)     | `░░░░░░░░░░░░`   |   `0%`  | `cleared · ready to review` |

Notes:
- Station 4b (deep dive) does **not** redraw the meter — it's a sub-station.
- The inline tree trigger ("tree", "show me the files") does **not** redraw the meter.
- If the reviewer revisits an earlier station from Station 6, the meter stays at its
  current level — debt isn't re-added by re-reading.
- The Phase 2 handoff divider does not carry a meter; debt was already cleared at
  the review complete block.

---

## Difficulty line

One line, inserted into the Station 1 PR header block, below author/branches and
above the body. Lowercase, `·` separator.

Render template:

```
difficulty: <tier>  ·  +<additions> / -<deletions>  ·  <N> files
```

Tier computation (pre-phase, from `gh pr view --json files,additions,deletions`):

```
score = additions + deletions + (files_count * 30)
```

| Tier     | Score range  |
|----------|--------------|
| `easy`   | ≤ 80         |
| `medium` | 81 – 400     |
| `hard`   | 401 – 1000   |
| `epic`   | > 1000       |

Fallback: if `additions + deletions` is 0 or unreadable (e.g. binary-only changes),
default to `medium`.

---

## Review complete block

Rendered **once**, at the end of Station 6 after the reviewer confirms ownership,
and immediately before the Phase 2 menu.

Emit as plain text (not inside a code block):

```
── review complete ──────────────────────────────────────────────────

cognitive debt  ░░░░░░░░░░░░    0%  cleared

Review Depth  <score>/10  ·  <difficulty> PR · <flavor tags>
```

### Score computation

Phase 1 data only — does **not** incorporate Phase 2 findings.

```
ceiling = difficulty_ceiling        easy 6 · medium 8 · hard 9 · epic 10
base    = ceiling - 2               baseline for a normal walkthrough
+ 1  if reviewer chose deep-dive at Station 4 (option [2])
+ 1  if reviewer-named risk matched Claude's identified risk at Station 5
+ 1  if reviewer answered with substance (multi-sentence, specific — not "sure"/"ok")
− 2  if reviewer said "unsure" or needed to revisit at Station 6
clamp to [1, ceiling]
```

### Flavor tags

Short, `·`-separated strings attached to the score line. Each reflects an earned
adjustment or a notable characteristic of the session. Pick 2–4 tags. Examples:

- `deep-dive` — reviewer chose option [2] at Station 4
- `risk matched` — reviewer named a risk aligned with Claude's finding
- `substantive` — reviewer answered with specifics, not one-word replies
- `tree checked` — reviewer used the inline tree trigger
- `quick pass` — thin engagement, short answers throughout
- `consider deeper dive next time` — gentle suggestion paired with a low score

### Tone rules

- **Reviewer-facing only.** The Review Depth score and this block NEVER appear in
  the final PR comment. It is a private self-motivation signal.
- **Low scores read gentle, not punitive.** Never use words like "poor", "failed",
  "weak". `quick pass · consider deeper dive next time` is the floor.
- **Do not gate Phase 2.** The reviewer proceeds to the Phase 2 menu normally
  regardless of score.

---

## Question UX format

Every reviewer-facing question (station questions, Phase 2 menu, post-confirmation).
Emit as plain text (not blockquote, not code block). Blank line after.

The `▶` bullet and the full question text are wrapped in ANSI bright blue. The separator
line stays plain.

```
────────────────────────────────────────────────────
\033[94m▶  <question text>\033[0m

```

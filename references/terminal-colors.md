# Terminal Render Specs

All output is **plain text only** — **except question prompts**, which use ANSI bright blue (`\033[94m`).
Emit every example below as plain text; do not wrap in a code block when outputting.

---

## Mascot + station divider

Emit as plain text (not inside a code block). Example for Station 2:

```
  / \__
 (    @\___
 /         O
/   (_____/
/_____/   U

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

## Question UX format

Every reviewer-facing question (station questions, Phase 2 menu, post-confirmation).
Emit as plain text (not blockquote, not code block). Blank line after.

The `▶` bullet and the full question text are wrapped in ANSI bright blue. The separator
line stays plain.

```
────────────────────────────────────────────────────
\033[94m▶  <question text>\033[0m

```

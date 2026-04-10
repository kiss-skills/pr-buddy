# Terminal Colors & Render Specs

All plain-text output uses ANSI escape codes. Do **not** apply ANSI inside fenced code blocks.
Emit every example below as **plain text** — do not wrap in a code block when outputting.

> **CRITICAL — emit the actual ESC byte, not the literal `\033`**
> Every `\033` in these examples represents the ESC character (U+001B, hex 0x1B).
> When outputting, your response must contain the **real ESC byte** — not the four characters `\`, `0`, `3`, `3`.
> Outputting `\033[32m` as literal text produces no color. Outputting the actual ESC byte followed by `[32m` produces green.

## Color palette

| Element | ANSI code | Appearance |
|---------|-----------|-----------|
| Mascot | `\033[38;5;214m` … `\033[0m` | orange |
| Station dividers | `\033[36m` … `\033[0m` | cyan |
| Directory names in tree | `\033[1;34m` … `\033[0m` | bold blue |
| File names in tree | `\033[1m` … `\033[0m` | bold |
| Annotations (`← entry point`) | `\033[90m` … `\033[0m` | dark gray |
| `+N` additions | `\033[32m` … `\033[0m` | green |
| `-N` deletions | `\033[31m` … `\033[0m` | red |
| Tree / PR-header separator lines | `\033[90m` … `\033[0m` | dim |
| Tracker `[x]` | `\033[32m[x]\033[0m` | green |
| Tracker `[ ]` | `\033[90m[ ]\033[0m` | dim |
| Question separator | `\033[90m` … `\033[0m` | dim |
| Question arrow `▶` | `\033[1;33m▶\033[0m` | bold yellow |
| Reset (always close every color) | `\033[0m` | — |

---

## Mascot + station divider

Emit as plain text (not inside a code block). Example for Station 2:

```
\033[38;5;214m  / ╲--
 (    @╲---
 /         O
/   (-----/
/-----/   U\033[0m

\033[36m── station 2 · architecture ─────────────────────────────────────────\033[0m
```

Station divider labels: `station 1 · goal`, `station 2 · architecture`,
`station 3 · decisions`, `station 4 · risk`, `station 5 · ownership`,
`phase 2 · automated checks`.

---

## Change tree

Emit as plain text (not inside a code block). Fill in real file paths and stats from the diff.

```
Changed files  (+<total additions> / -<total deletions>)
\033[90m──────────────────────────────────────────────────────\033[0m
\033[1;34msrc/\033[0m
  \033[1;34mauth/\033[0m
    \033[1mmiddleware.ts\033[0m        \033[32m+42\033[0m / \033[31m-8\033[0m    \033[90m← entry point\033[0m
    \033[1msession.ts\033[0m           \033[32m+15\033[0m / \033[31m-0\033[0m
  \033[1;34mutils/\033[0m
    \033[1mtoken.ts\033[0m             \033[32m+6\033[0m  / \033[31m-12\033[0m   \033[90m← key decision (Station 3)\033[0m
\033[1;34mtests/\033[0m
  \033[1mauth.test.ts\033[0m           \033[32m+30\033[0m / \033[31m-2\033[0m
\033[90m──────────────────────────────────────────────────────\033[0m
  3 dirs · 5 files · \033[32m+93\033[0m / \033[31m-22\033[0m total
```

Annotations in dark gray (`\033[90m`): `← entry point`, `← new module`, `← deleted`,
`← most changed`. One phrase per file. Annotate conservatively early in the session.

---

## File walkthrough tracker

Emit as plain text (not inside a code block). Re-print whenever a file is marked done.

```
\033[1mFile walkthrough\033[0m
  \033[90m[ ]\033[0m \033[1msrc/auth/middleware.ts\033[0m    \033[90m— entry point\033[0m
  \033[32m[x]\033[0m \033[1msrc/auth/session.ts\033[0m       \033[90m— new module (discussed)\033[0m
  \033[90m[ ]\033[0m \033[1msrc/utils/token.ts\033[0m        \033[90m— key decision\033[0m
```

Cover main files only — skip tests, generated files, lock files, trivial config.

---

## Question UX format

Every reviewer-facing question (station questions, Phase 2 menu, post-confirmation).
Emit as plain text (not blockquote, not code block). Blank line after.

```
\033[90m────────────────────────────────────────────────────\033[0m
\033[1;33m▶\033[0m  <question text>

```

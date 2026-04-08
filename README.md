# `/pr-buddy`

![pr-buddy mascot](ICON.png)

A Claude Code skill that makes you understand a PR before you approve it.

## The problem

Agentic PR review tools are fast. Too fast. Teams accumulate *cognitive debt* — reviewers
click "OK-APPLY" without ever building a mental model of the changes. Velocity increases.
Comprehension does not.

## How it works

Two phases:

1. **Walkthrough** — Socratic conversation through 5 stations (Goal → Architecture →
   Decisions → Risk → Ownership). You answer first; the skill reveals its reading second.
   You don't proceed until you can explain the PR yourself.

2. **Automated checks** — Hands off to `pr-review-toolkit`. Findings are framed back to
   the walkthrough so they land in context you already own.

## Install

Add this skill to your Claude Code setup:

```bash
claude skill add kiss-skills/pr-buddy
```

**Requirements:**
- `gh` CLI (authenticated)
- `pr-review-toolkit` installed (for Phase 2)

## Usage

```
/pr-buddy <pr_number_or_branch_or_url>
```

Examples:

```
/pr-buddy #47
/pr-buddy feat/rate-limiting
/pr-buddy https://github.com/org/repo/pull/847
```

## The 5 stations

|  #  |   Station    |                          Question                          |
| --- | ------------ | ---------------------------------------------------------- |
| 1   | Goal         | What problem is this PR solving?                           |
| 2   | Architecture | What's the entry point? Walk me through your mental model. |
| 3   | Decisions    | The author chose X. What would you have done?              |
| 4   | Risk         | What's the one thing that could go wrong?                  |
| 5   | Ownership    | Could you explain this to a teammate right now?            |

## Output

Closes with a 3-bullet mental model recap you can paste directly into the PR comment.

See [assets/example-session.md](assets/example-session.md) for a full annotated example.

## License

MIT — see [LICENSE](LICENSE).

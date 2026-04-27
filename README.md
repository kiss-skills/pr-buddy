# `/pr-buddy`

![pr-buddy mascot](ICON.png)

Understand the PR before you approve it.

## The problem

AI review tools make it too easy to approve code you haven't read.
Click through, apply, merge. Later you can't explain it.

## Solution

`/pr-buddy <pr>` walks you through a PR in 6 stations — **you** answer first, then the
skill shows its reading. Once you can explain the PR confidently, it runs
`pr-review-toolkit` **anchored to what you discovered**.

> The output is a combined PR comment **that's yours** to post.

A cognitive debt meter drains as you progress through stations, and a private Review
Depth score closes the session — weighted by PR difficulty so hard PRs reviewed
thoroughly feel earned. The `tree` command is available any time — natural language like
*"show me the files"* works — and never resets your session state.

## Install

### Claude Code

```bash
claude skill add kiss-skills/pr-buddy
```

### GitHub Copilot (or any agent without a plugin marketplace)

Copilot has no plugin/marketplace, so install is manual — clone the skill once, then point
Copilot at it via `AGENTS.md` (or `.github/copilot-instructions.md`):

```bash
git clone https://github.com/kiss-skills/pr-buddy.git ~/.kiss-skills/pr-buddy
```

Then add this snippet to your `AGENTS.md`:

```md
## /pr-buddy
When the user types `/pr-buddy <PR>`, follow the procedure in `~/.kiss-skills/pr-buddy/SKILL.md`.
Phase 1 is a Socratic walkthrough. Phase 2 falls back to inline review when
`pr-review-toolkit` sub-agents are unavailable — see Phase 2 in `SKILL.md`.
```

Repo-vendored install also works: drop the folder under `.github/skills/pr-buddy/` and
reference it from `.github/copilot-instructions.md` the same way.

**Requirements:** `gh` CLI (authenticated). `pr-review-toolkit` is **optional** — when
present (Claude Code), Phase 2 dispatches three sub-agents in parallel; when absent,
Phase 2 falls back to inline review by the host model with feature parity.

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

## The 6 stations

You answer first. Then the skill shows its reading.

|  #  |   Station    |                              Question                               |
| --- | ------------ | ------------------------------------------------------------------- |
| 01  | Goal         | "What problem do you think this PR is solving?"                     |
| 02  | Architecture | "Which file is the entry point? Walk me through what you think is happening." |
| 03  | Decisions    | "The author chose X. What would you have done differently?"         |
| 04  | Code Tour    | Skill fetches function signatures across all changed files. Optionally go deeper. |
| 05  | Risk         | "What's the one thing that could go wrong here?"                    |
| 06  | Ownership    | "Could you explain this PR to a teammate? What are you still unsure about?" |

Phase 2 doesn't start until you say you're ready at Station 6.

## License

MIT — see [LICENSE](LICENSE).

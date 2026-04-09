# `/pr-buddy`

![pr-buddy mascot](ICON.png)

Understand the PR before you approve it.

## The problem

AI review tools make it too easy to approve code you haven't read.
Click through, apply, merge. Later you can't explain it.

## Solution

`/pr-buddy <pr>` walks you through a PR in 5 stations — **you** answer first, then the
skill shows its reading. Once you can explain the PR confidently, it runs
`pr-review-toolkit` **anchored to what you discovered**.

> The output is a combined PR comment **that's yours** to post.

Each station opens with an ASCII dog mascot as a transition signal. The `tree` command is
available any time — natural language like *"show me the files"* works — and never resets
your session state.

## Install

```bash
claude skill add kiss-skills/pr-buddy
```

**Requirements:** `gh` CLI (authenticated) · `pr-review-toolkit` for Phase 2

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

You answer first. Then the skill shows its reading.

|  #  |   Station    |                              Question                               |
| --- | ------------ | ------------------------------------------------------------------- |
| 01  | Goal         | "What problem do you think this PR is solving?"                     |
| 02  | Architecture | "Which file is the entry point? Walk me through what you think is happening." |
| 03  | Decisions    | "The author chose X. What would you have done differently?"         |
| 04  | Risk         | "What's the one thing that could go wrong here?"                    |
| 05  | Ownership    | "Could you explain this PR to a teammate? What are you still unsure about?" |

Phase 2 doesn't start until you say you're ready at Station 5.

See [assets/example-session.md](assets/example-session.md) for a full annotated example.

## License

MIT — see [LICENSE](LICENSE).

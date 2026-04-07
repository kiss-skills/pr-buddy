# Cognitive Debt in PR Review

## What it is

Cognitive debt is the accumulated gap between *having approved* code and *understanding* the
code you approved. It compounds silently: each rubber-stamped PR leaves you less equipped to
reason about the next one that builds on it.

Unlike technical debt, which shows up in metrics and slows builds, cognitive debt is invisible
until it isn't — until a bug lands that "someone should have caught", or an incident drags on
because no one on the team actually knows what the affected code does.

## How it accumulates

Agentic code review tools are powerful, but they make rubber-stamping frictionless. The
pattern looks like this:

1. Automated tool flags issues and suggests fixes
2. Author applies suggested fixes
3. Reviewer sees "all checks passed" and clicks Approve
4. No one on the team built a mental model of the change

Velocity increases. Comprehension does not.

## Why it matters

- **Incident response slows down.** Nobody knows the code well enough to reason under pressure.
- **Onboarding degrades.** New teammates ask questions that the team can't answer from memory.
- **Review quality erodes.** Without a mental model, reviewers can only check surface-level
  issues — style, naming, obvious bugs. Architectural risks go unnoticed.
- **Ownership diffuses.** "Someone reviewed it" stops meaning "someone understood it."

## The intervention

The walkthrough forces comprehension *before* automation. By the time the automated tools run,
the reviewer has:

- Formed an opinion on the PR's goal
- Traced the code path through the changed files
- Thought critically about 2–3 design decisions
- Identified at least one risk
- Confirmed they could explain the PR to a colleague

Automated findings then land in a context the reviewer owns, rather than a context the tool
constructed for them.

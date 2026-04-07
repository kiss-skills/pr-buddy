# Example Session

This is an annotated walkthrough of the skill in action. The PR is fictional.

---

**Setup:** The reviewer runs `/pr-buddy #47` against a PR titled
"feat: add rate limiting to the public API".

---

## Pre-phase (silent)

The skill fetches the PR and diff. It identifies:
- **Goal:** protect the API from abuse by capping requests per client per minute
- **Entry point:** `src/middleware/rate-limit.ts` (new file), wired into `src/app.ts`
- **Notable decisions:** in-memory store (not Redis), per-IP keying (not per-token), 429 response without Retry-After header
- **Risk:** in-memory store doesn't survive restarts and doesn't work across multiple instances

The skill says nothing yet.

---

## Station 1 — Goal

**Skill:**
> Based on the title and description, what problem do you think this PR is solving? What's the outcome the author was aiming for?

**Reviewer:**
> Looks like they want to stop people from hammering the API. Probably had an incident or got a complaint.

**Skill:**
> Exactly right. The PR adds a rate limiter to the public API to cap requests per client. The description mentions a specific incident last week where a misconfigured client sent 10k requests in 30 seconds. The goal is to prevent that from happening again without adding authentication.

---

## Station 2 — Architecture

**Skill:**
> Here are the changed files:
> - `src/middleware/rate-limit.ts` (new, +89 lines)
> - `src/app.ts` (+3 lines)
> - `tests/middleware/rate-limit.test.ts` (new, +67 lines)
>
> Which file do you think is the entry point? Walk me through your mental model — how do these changes fit together?

**Reviewer:**
> `app.ts` wires in the middleware, and `rate-limit.ts` is the actual logic. Tests are separate. Pretty standard.

**Skill:**
> Right. In `app.ts`, the middleware is registered before any route handlers — line 14, `app.use(rateLimit())`. So every request hits the limiter first. The limiter in `rate-limit.ts` uses a `Map` keyed by IP address. If the count exceeds the threshold within the window, it returns 429 and short-circuits. The test file covers the happy path and the limit-exceeded case.

---

## Station 3 — Decisions

**Skill:**
> The author chose to use an in-memory `Map` for the rate limit store rather than Redis or another external store. What would you have done differently, if anything? Why do you think they made this choice?

**Reviewer:**
> I'd probably use Redis so it works across instances. In-memory only works for a single server. Maybe they're still on a single instance?

**Skill:**
> That's the right read. The PR description doesn't mention multiple instances — this looks like a single-server deployment for now. In-memory is simpler to ship, no new infrastructure dependency, and it works for the immediate problem. The trade-off is that it breaks under horizontal scaling and doesn't survive restarts. Worth noting for the review.

**Skill:**
> Second decision: the limiter keys by IP, not by API token or user ID. Why might they have made that choice?

**Reviewer:**
> No auth on this API? Or maybe they wanted to limit even unauthenticated callers.

**Skill:**
> Correct — the public API doesn't require authentication. IP is the only stable identifier available. The downside is that shared IPs (corporate NATs, proxies) can hit the limit even with legitimate traffic.

---

## Station 4 — Risk

**Skill:**
> If you had to name one thing that could go wrong with this PR in production — an edge case, a race condition, a missing test, a performance concern — what would it be?

**Reviewer:**
> The in-memory thing you mentioned. If they scale out or restart, all the counters reset.

**Skill:**
> That's the main one. The counters are stored in a `Map` on the process heap — every restart wipes them. More critically, if they ever run two instances behind a load balancer, each instance has its own counter, so the effective limit doubles. The test suite doesn't cover this because tests run in a single process. It's not a bug today, but it's a scaling cliff.

---

## Station 5 — Ownership

**Skill:**
> Could you explain this PR to a teammate right now, confidently? What, if anything, are you still unsure about?

**Reviewer:**
> Yeah, I think so. It's a per-IP in-memory rate limiter wired in as middleware. Works for now, won't work when they scale. I'd want to know if there's a scale-out plan before approving blindly.

**Skill:**
> Good. You're ready for Phase 2. Your concern about the scaling cliff is valid — worth leaving a comment asking the author whether horizontal scaling is on the roadmap, so this decision is documented.

---

## Phase 2 — Automated Quality Checks

### Phase 1 context block (internal, not shown to reviewer)

```
PHASE_1_CONTEXT:
  goal:          "Add per-IP rate limiting to prevent API abuse after a specific client incident"
  entry_point:   "src/middleware/rate-limit.ts"
  decisions:     ["in-memory Map over Redis (single-instance tradeoff)",
                  "per-IP keying over per-token (no auth available)"]
  reviewer_risk: "in-memory store won't survive restarts or work across multiple instances"
  skill_risk:    "same — scaling cliff under horizontal deployment"
  confidence:    "ready, with a question about scale-out plans"
```

### Sub-agents invoked in parallel

**code-reviewer** is told: *"The reviewer traced the entry point as `src/middleware/rate-limit.ts`.
The following were discussed as intentional trade-offs: in-memory store over Redis (single-instance
deployment), per-IP keying (no auth available). Treat these as known decisions, not issues."*

→ Finding: `Retry-After` header absent from the 429 response — RFC 6585 recommends it.
(The in-memory and per-IP decisions are acknowledged as intentional, not re-flagged.)

**silent-failure-hunter** is told: *"The reviewer identified the in-memory store as the main
risk — focus on error handling in the rate-limit path."*

→ Finding: The 429 response path swallows the error without logging. Clients get a 429 with
no server-side trace.

**pr-test-analyzer** is told: *"Check whether the following risk scenarios have test coverage:
in-memory counter reset on restart, behavior under concurrent requests, window-boundary reset."*

→ Finding: No test for the reset behavior at window boundary. Restart behavior is also uncovered.

---

## Final Summary — Combined PR Comment

**Skill:**

> Here's the combined review comment. Want me to post it?

```markdown
**Review (walkthrough + automated checks)**

**Understanding:**
- **Goal:** Add per-IP rate limiting to the public API to prevent abuse, triggered by a client incident last week.
- **Key decision:** In-memory store over Redis — intentional for a single-instance deployment, but a documented scaling cliff if they move to horizontal scaling. Worth asking the author if that's on the roadmap.
- **Main risk identified:** The limiter won't survive restarts and won't work across multiple instances.

**Automated findings:**
- [code-reviewer] Missing `Retry-After` header on 429 responses — RFC 6585 recommends it; low-effort fix.
- [silent-failure-hunter] The 429 path swallows errors without logging — if this fires in production, there'll be no server-side trace to diagnose it.
- [pr-test-analyzer] No coverage for window-boundary reset or restart behavior — these are the core scenarios for a rate limiter.

**Verdict:** Request changes
```

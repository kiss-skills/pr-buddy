# Example Session

Real walkthrough on PR #74 — `feat(skill): jira-ticket-automator v0.5.0`
Repository: `MegaCorp/some-repo`

---

## Pre-phase (silent)

The skill fetches the PR and diff. It identifies:
- **Goal:** rename `jira-tickets-from-epic` → `jira-ticket-automator` and upgrade from v0.1.0 to v0.5.0 with real-world learnings from ACME-1158
- **Entry point:** `skills/pgm/jira-ticket-automator/SKILL.md`
- **Notable decisions:** Epic link via `jira_create_issue_link` (not `customfield_10091`); issuetype ID `"5"` hardcoded in `additional_fields`; Confluence writeback patches `* US CORP:` in-place
- **Risk:** `catalog.json` and `INDEX.md` were hand-edited — name, path, and version all mismatch the actual new skill directory

The skill says nothing yet.

---

## Station 1 — Goal

**Skill:** *(prints PR title, author, branch, and full description, then asks)*

> Based on the title and description, what problem do you think this PR is solving? What's the outcome the author was aiming for?

**you (reviewer):**
> ok there was a skill named jira-tickets-from-epic and this is renaming jira-ticket-automator and ALSO adding 5 new fixes/iteration/features
>
> internal jira thing about jira projects ids  
> specific naming of some tickets  
> perf improvement (if(xxx){})  
> some internal thing (some internal enhancement)  
> new output back to Confluence (with the jira link)
>
> now something related to tech detail with the MCPs and a fix for windowsOS

**Skill:**
> Exactly right. The core goal is a v0.1→v0.5 field-tested upgrade of an existing skill, where the rename signals it's now scoped specifically to the ACME+Confluence workflow — not just any epic. The 5 improvements are all hardcoded learnings from a real run against ACME-1158: the "happy path" became the spec.

---

## Station 2 — Architecture

**Skill:**
> Here are the changed files — `[list files]`. Which file do you think is the entry point? Walk me through your mental model — how do these changes fit together?
>
> ```
> INDEX.md
> catalog.json
> skills/pgm/jira-ticket-automator/README.md
> skills/pgm/jira-ticket-automator/SKILL.md
> skills/pgm/jira-ticket-automator/references/jira-field-mapping.md
> ```

**you (reviewer):**
> the main thing should be SKILL.md [tree]

*(reviewer uses the `tree` inline trigger mid-answer)*

**Skill:** *(renders the change tree)*

```
Changed files  (+400 / -0)
──────────────────────────────────────────────────────
skills/pgm/jira-ticket-automator/
  SKILL.md                          +246 / -0    ← entry point
  README.md                         +77  / -0    ← human-facing docs
  references/
    jira-field-mapping.md           +55  / -0    ← field reference
INDEX.md                            +1   / -0    ← catalog pointer
catalog.json                        +21  / -0    ← catalog entry
──────────────────────────────────────────────────────
  2 dirs · 5 files · +400 / -0 total
```

> Where were we — ready to go on?

**Skill:** *(continues)*

> You're right, `SKILL.md` is the entry point — it's the agent-executable definition that drives everything. The other files orbit it: `README.md` is for humans installing the skill, `jira-field-mapping.md` is a lookup reference the agent can consult during execution, and `INDEX.md`/`catalog.json` are the registry bookkeeping.
>
> One thing that already jumped out at me from the diff: the call/data flow is strictly linear — read epic → read Confluence pages → approval gate → create issues → write back to Confluence. The Confluence writeback (Phase 3.7) is the most interesting addition, because it's what makes the skill idempotent on re-runs.

**File walkthrough opened:**
```
  [ ] SKILL.md                  — entry point (procedure + MCP workarounds)
  [ ] README.md                 — human-facing docs
  [ ] references/jira-field-mapping.md — field reference
  [ ] INDEX.md / catalog.json   — registry
```

---

## Station 3 — Decisions

**Skill:**
> OP made a few notable choices in the diff. Let me pick three:
>
> 1. The skill uses `jira_create_issue_link` with type `"Epic-Story Link"` to attach the US to the epic — instead of setting `epicKey` or `customfield_10091` at creation time.
> 2. Sub-task creation forces `"issuetype": {"id": "5"}` via `additional_fields` to override the MCP's default resolution, which maps `Sub-task` to `Sub-Bug`.
> 3. The Confluence writeback patches the `* US CORP:` line in-place in markdown, rather than using a Jira macro or structured metadata.
>
> For each: what would you have done differently, if anything? Why do you think OP made these choices?

**you (reviewer):**
> 1. maybe the customfield_10091 is related to an specific project? and OP wanted to keep it customizable (i dont know)
> 2. but point 1 loses strength cause here the id is fixed
> 3. maybe this is a possible bug/issue
> 4. Why the jira-field-mapping.md? i'd suggest to use jira-fields.mapping.json a fixed hard coded thing instead of a LLM-related md

**Skill:**
> 1. Correct instinct — `customfield_10091` is screen-restricted specifically in BIGPROJECT (SKILL.md says so explicitly). So `jira_create_issue_link` isn't a design preference, it's a forced workaround for that project's field visibility config.
>
> 2. Sharp catch. Hardcoding `"id": "5"` assumes Sub-task type ID is `5` in every Jira instance. If this skill is ever reused in a different project/instance, that ID could differ silently. The inconsistency is real: the epic link workaround is flexible, but the sub-task fix is brittle.
>
> 3. Agreed — the `* US CORP:` line patch is fragile. It assumes the Confluence page has that exact string in that exact format. No fallback is documented.
>
> 4. Strong point. The `jira-field-mapping.md` is written as if it's a reliable reference, but it actually has a direct conflict with `SKILL.md` right now: the reference says `issuetype: Story`, but the skill says `User Story`. A JSON file would be more trustworthy as a lookup — the LLM has to parse freetext MD and could pick the wrong value. A JSON file forces structure and is harder to contradict.

*(reviewer then raises an additional point)*

**you (reviewer):**
> Station 3 -- is it necessary the README.md? there are READMEs in the repo, maybe its duplicating some existing info? maybe the file is ok but should be minimal with no duplication with other READMEs in the repo.

**Skill:**
> Valid Station 3 point — the `README.md` here is largely redundant with `SKILL.md`. Both cover: prerequisites, inputs, execution flow, approval gate. The only content that isn't in `SKILL.md` is the "How to invoke" example prompts — everything else is duplicated.
>
> Also — the README has an active bug: the "File structure" section at the bottom still shows the old path `skills/pgm/jira-tickets-from-epic/`, not `jira-ticket-automator/`. The rename wasn't applied there.

**File walkthrough updated:**
```
  [x] SKILL.md                  — entry point (procedure + MCP workarounds)
  [x] README.md                 — duplicates SKILL.md, has stale path in file structure
  [x] references/jira-field-mapping.md — field reference (conflicts with SKILL.md)
  [ ] INDEX.md / catalog.json   — registry
```

---

## Station 4 — Risk

**Skill:**
> If you had to name one thing that could go wrong with this PR in production — an edge case, a race condition, a missing test, a performance concern — what would it be?

**you (reviewer):**
> removing a skill from the catalog MIGHT get nasty to someone in their laptops (but maybe this is safe cause this is an internal repo)

**Skill:**
> Good general instinct about catalog churn — but this PR actually has the opposite problem, and it's sharper than a removal.
>
> The catalog.json entry being added still says **`name: jira-tickets-from-epic`** pointing to **`skills/pgm/jira-tickets-from-epic/SKILL.md`** — a path that doesn't exist anywhere in this PR. The actual files landed under `jira-ticket-automator/`. The SKILL.md frontmatter says `name: jira-ticket-automator`. Version in catalog: `0.1.0`. Version in SKILL.md: `0.5.0`.
>
> So the catalog is broken in three ways simultaneously: wrong name, wrong path, wrong version. Anyone running `claude skill add` or the install tooling against this catalog would either get a 404 or install something called `jira-tickets-from-epic` that points nowhere.
>
> The likely cause: OP manually edited `catalog.json` and `INDEX.md` instead of running `npm --prefix tools run build-index`. The test plan even lists it as a checkbox — but it wasn't done before opening the PR.

**File walkthrough updated:**
```
  [x] SKILL.md                  — entry point (procedure + MCP workarounds)
  [x] README.md                 — duplicates SKILL.md, has stale path in file structure
  [x] references/jira-field-mapping.md — field reference (conflicts with SKILL.md)
  [x] INDEX.md / catalog.json   — broken: wrong name, wrong path, wrong version
```

---

## Station 5 — Ownership

**Skill:**
> Could you explain this PR to a teammate right now, confidently? What, if anything, are you still unsure about?

**you (reviewer):**
> yes. next step

---

## Phase 2 — Automated Quality Checks

### Phase 1 context block (internal, not shown to reviewer)

```
PHASE_1_CONTEXT:
  goal:          "Rename and upgrade jira-tickets-from-epic → jira-ticket-automator v0.5.0 with real-world learnings from ACME-1158"
  entry_point:   "skills/pgm/jira-ticket-automator/SKILL.md"
  decisions:     ["Epic link via jira_create_issue_link (BIGPROJECT screen restriction forces it)",
                  "Issuetype ID 5 hardcoded in additional_fields (brittle across instances)",
                  "Confluence writeback patches * US CORP: line in-place (fragile, no fallback)"]
  reviewer_risk: "catalog.json / INDEX.md manually edited — name, path, version all wrong"
  skill_risk:    "jira-field-mapping.md conflicts with SKILL.md on issuetype name and QA subtask prefix; README has stale path"
  confidence:    "high — reviewer said yes immediately"
```

### Sub-agents invoked in parallel

**code-reviewer** told: *"Entry point is SKILL.md. Discussed decisions are intentional trade-offs. Focus on correctness and adherence to project guidelines."*

→ Critical: catalog.json and INDEX.md point to wrong name (`jira-tickets-from-epic`), wrong version (`0.1.0`), and non-existent paths. Fix: run `build-index`.
→ Important: `jira-field-mapping.md` conflicts with SKILL.md on 3 points (issuetype name, QA prefix, epic link approach).
→ Important: README.md is largely redundant with SKILL.md and has a stale file path.
→ Important: `jira-field-mapping.md` missing Performance and Supervision naming patterns.

**silent-failure-hunter** told: *"Reviewer identified catalog breakage. Focus on the failure paths in the Confluence writeback, the issuetype ID override, and the 'Jira Links' detection logic."*

→ Critical: Phase 3.7 writeback silently does nothing when `* US CORP:` line is absent — idempotency guard broken, duplicate US on re-run.
→ Critical: Phase 2 "already linked" detection matches any `[A-Z]+-[0-9]+` pattern in body prose — silent false-positive skips.
→ Critical: `confluence_update_page` failure unhandled — US reported CREATED even when writeback fails.
→ High: Hardcoded issuetype ID `"5"` applied to non-BIGPROJECT projects without verification.
→ High: `jira_create_issue_link` failure leaves US orphaned from epic with no ERRORS entry.

**pr-test-analyzer** told: *"Check whether risk scenarios (catalog breakage, reference file conflicts) have test coverage."*

→ All four test-plan checkboxes unchecked — validator not run, build-index not run.
→ Gap: no automated test cross-checking committed catalog.json against what build-index would generate.
→ No `skill-structure.test.ts` added (other skill groups have these as a convention).

---

## Final Summary — Combined PR Comment

Posted at: https://github.com/MegaCorp/some-repo/pull/74#issuecomment-0000000000

```markdown
**Review (walkthrough + automated checks)**

**Understanding:**
- **Goal:** Rename and upgrade `jira-tickets-from-epic` → `jira-ticket-automator` v0.5.0, baking in real-world learnings from ACME-1158 (project mapping, ticket naming, Confluence writeback, MCP workarounds).
- **Key decision:** Epic link is set via `jira_create_issue_link` (not `customfield_10091`) because that field is screen-restricted in BIGPROJECT — a forced workaround documented for future runners.
- **Main risk identified:** The Confluence writeback (Phase 3.7) patches a hardcoded `* US CORP:` line in-place with no fallback — if the line is absent or the space is not CORP, the idempotency guard silently breaks and the next run creates a duplicate US.

---

**Automated findings:**

**[code-reviewer] — CRITICAL: catalog.json and INDEX.md are broken**
Both files still say `name: jira-tickets-from-epic`, `version: 0.1.0`, paths under `skills/pgm/jira-tickets-from-epic/` — none of which exist. Fix: run `npm --prefix tools run build-index`.

**[code-reviewer] — IMPORTANT: `jira-field-mapping.md` conflicts with SKILL.md on 3 points**
1. `issuetype: Story` → should be `User Story`
2. QA subtask prefix `[QA] <name>` → should be `[QA] [Test automation] <name>`
3. Epic link documents `customfield_10008` → should document `jira_create_issue_link` workaround

**[code-reviewer] — IMPORTANT: README.md is largely redundant and has a stale path**

**[silent-failure-hunter] — CRITICAL: Three failure paths all converge on silent duplicate US creation**
- Phase 3.7 has no branch for when `* US CORP:` line is absent
- Phase 3.7 has no error handling for `confluence_update_page` failure
- Phase 2 "already linked" detection matches any bare Jira key in prose

**[silent-failure-hunter] — HIGH: Hardcoded issuetype ID `"5"` applied to all projects without verification**

**[silent-failure-hunter] — HIGH: `jira_create_issue_link` failure leaves US orphaned from epic**

**[pr-test-analyzer] — All four test-plan checkboxes are unchecked**

**[pr-test-analyzer] — GAP: No automated test for catalog consistency**

---

**Verdict:** Request changes

Blockers before merge:
1. Run `build-index` to regenerate `catalog.json` and `INDEX.md`
2. Run `npm --prefix tools run validate` and confirm it passes
3. Fix the three conflicts in `jira-field-mapping.md`
4. Add error handling to Phase 3.7 (writeback failure + missing line fallback)
5. Fix `README.md` stale path; consider stripping to invocation-only
```

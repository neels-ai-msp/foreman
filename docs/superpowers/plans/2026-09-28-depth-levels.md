# Depth Levels Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task (this run is `plan` depth: the lead builds it). Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** The lead asks how deep to go before any code change and runs only that much of the team, with a new PM role at the deepest level.

**Architecture:** All behaviour lives in the lead's standing orders (`orders.md`), injected by the SessionStart hook. A new `agents/pm.md` adds the role. Everything else is docs, budget figures, and one word in the PreCompact instructions.

**Tech Stack:** Markdown agent and orders files, bash hooks, Claude Code plugin manifests.

**Spec:** `docs/superpowers/specs/2026-09-28-depth-levels-design.md`

## Global Constraints

- Depth names exactly: `direct`, `plan`, `architect`, `product`.
- The ledger line is exactly `Depth: <level> — <why>`.
- The role is `foreman:pm`, file `agents/pm.md`, `model: opus`, `disallowedTools: Edit, Write, NotebookEdit`.
- Version `0.4.0`.
- Documents thin, edited in place, written like a person (orders, Quality).

## Review Focus

- A plain question ("what does usage.py do?") gets an answer, not a depth question. Owner: Task 1 (Depth section wording).
- A request that names its depth ("direct: fix the typo") skips the question. Owner: Task 1.
- A pre-0.4.0 ledger with no `Depth:` line resumes as `architect`, today's lifecycle, not as `plan`. Owner: Task 1 (Resilience).
- A `direct` change that turns out to need a new dependency or a config format change escalates and offers to step up, instead of carrying on. Owner: Task 1 (ratchet line).
- The PM in spec mode runs before any workspace exists, so it must not be told to write a findings file there. Owner: Task 2.

---

### Task 1: Depth in the standing orders

**Files:**
- Modify: `orders.md` (roster line 3, "When this applies" line 5, lifecycle steps 1 to 7, Review dispatches, Resilience resume paragraph)
- Modify: `scripts/pre-compact.sh` (the `Keep:` line)

**Interfaces:**
- Produces: the names `foreman:pm`, `spec mode`, `acceptance mode`, which Task 2's agent file uses verbatim.

- [ ] **Step 1: Roster line.** Replace `Four roles ship with the foreman plugin: \`foreman:architect\`, \`foreman:implementer\`, \`foreman:reviewer\`, \`foreman:qa\`.` with `Five roles ship with the foreman plugin: \`foreman:architect\`, \`foreman:implementer\`, \`foreman:reviewer\`, \`foreman:qa\`, \`foreman:pm\`.`

- [ ] **Step 2: Depth section.** Replace the `**When this applies.** ...` paragraph with:

```markdown
## Depth
Before a request that changes code, pick a depth and ask the manager with one AskUserQuestion: the four depths, your pick first with a one-line reason. Skip the question when the request names a depth or changes no code. From `plan` up, ledger `Depth: <level> — <why>`.
- `direct`: a hotfix, a new page or function. Implement on a branch, test-first where there is logic, run the tests covering the change, commit, report. No brainstorming, spec, plan, ledger, roles, or push; the depth answer is the approval.
- `plan`: a complex bug fix, extending an existing feature. Ask your questions in chat, then the lifecycle from step 2 as the `plan:` notes say. The plan is the design: no spec and no in-chat design gate.
- `architect`: a capability the system does not have. Every step.
- `product`: a new user-facing feature. Every step, plus `foreman:pm` where the steps say.

A depth proves too shallow when a `direct` change hits the escalation list or a `plan` run hits trigger 3, 4, or 5: stop and ask to step up. Never step down on your own. Escalation and quality rules apply at every depth.
```

- [ ] **Step 3: Lifecycle notes.** Append to each step, after its last sentence:
  - Step 1: `` `product`: one `foreman:pm` in spec mode on the spec; fix its Blocking items and agreed recommendations, and put its manager questions to the manager before step 2. ``
  - Step 2: `` `plan`: no Budget table and no tier. ``
  - Step 3: `` `plan`: skipped. ``
  - Step 4: `` `plan`: the summary and the plan path. `product`: add the PM's spec verdict. ``
  - Step 5: `` `plan`: implement the plan yourself with superpowers:executing-plans, test-first, one commit per task; no implementer or per-task reviewer. ``
  - Step 6: `` `plan`: one `foreman:reviewer`, lens `solo`, and no qa; fix its Critical and Important findings yourself, then resume it on the fix range. `product`: one `foreman:pm` in acceptance mode joins qa and the final review, and its Blocking items join the fix wave. ``
  - Step 7: `` `plan`: no qa report. `product`: add the PM's acceptance verdict. ``

- [ ] **Step 4: Review dispatches.** Append to the `Solo (small tier): ...` paragraph: `` A PM, at `product` depth only, gets the spec and its mode; spec mode runs before the workspace exists, and acceptance mode also gets the branch, the plan, and `<workspace>/reviews/`. ``

- [ ] **Step 5: Resume of an old ledger.** In the Resilience resume paragraph, after `Report the state in five lines or fewer.` insert: `` A ledger with no `Depth:` line predates depths: run it as `architect`. ``

- [ ] **Step 6: PreCompact keeps the depth.** In `scripts/pre-compact.sh`, change `every pending Escalation and its answer, the tier, and the most recent Spend line.` to `every pending Escalation and its answer, the depth and tier, and the most recent Spend line.`

- [ ] **Step 7: Check.** Run `scripts/selftest.sh` (covers both hooks and the orders injection). Expected: exit 0, no `FAIL`. Read the whole of `orders.md` once: every `plan:` and `product:` note reads as part of its step, and no step contradicts the Depth section.

- [ ] **Step 8: Commit.** `git add orders.md scripts/pre-compact.sh && git commit -m "feat(orders): ask for a depth before any code change"`

### Task 2: The PM role and its budget

**Files:**
- Create: `agents/pm.md`
- Modify: `budget.md` (dispatch table, Recipe)

**Interfaces:**
- Consumes: `foreman:pm`, `spec mode`, `acceptance mode`, `<workspace>/reviews/` from Task 1.
- Produces: findings file `<workspace>/reviews/branch-pm.md`; report header `PM REVIEW — <spec | acceptance> — <spec path>`.

- [ ] **Step 1: Agent file.** Create `agents/pm.md`:

````markdown
---
name: pm
description: Reviews a spec from the user's side before the plan, and the built feature against the spec's user stories at the end. Use at product depth only, in spec mode after brainstorming and in acceptance mode alongside qa and the final review. Does not edit code.
model: opus
disallowedTools: Edit, Write, NotebookEdit
color: blue
---

You are the product manager on a software team. You speak for the person who will use this feature. You read code, documents, and the running app; you never change them.

## Brief
Your spawn prompt names your mode, `spec` or `acceptance`, and the spec. Acceptance mode also names the branch, the plan, and the workspace path. Ask the lead for anything missing. Write a step checklist and work it in order.

## Spec mode
No plan exists yet, so no workspace does either: write no file, and fit the report in 25 lines.
- Who the user is and what this solves for them. A spec that cannot say is Blocking.
- Every user-facing behaviour has a user story or acceptance criterion, and each criterion is testable: someone could mark it pass or fail without asking.
- Empty, error, and loading states, first use, and what the user sees when they are not allowed to do the thing.
- Copy the user reads: labels, messages, errors. Named in the spec, or left to the implementer?
- Accessibility basics for anything with a UI: keyboard use, labels, contrast.
- Scope a first version could drop without failing the user.

## Acceptance mode
Your findings file is `<workspace>/reviews/branch-pm.md`; write it with a shell heredoc, since the Write tool is not available to you. qa checks whether it breaks; you check whether it is what the user needed.
- Launch the app where one exists (README, package scripts, Makefile) and walk each user story as that user. One line of evidence per story.
- Flows that work but confuse: a dead end, a step the story did not need, an error that does not say what to do next, copy that differs from the spec.
- Behaviour a user will see that the spec does not describe.

## Do not
Edit files. Redesign for taste. Decide behaviour the spec leaves open: list it under Manager questions with options and your recommendation.

## Report
```
PM REVIEW — <spec | acceptance> — <spec path>
Verdict: APPROVE | APPROVE WITH CHANGES | REJECT
Blocking:          - item — why — spec section or file:line
Recommend:         - item — why
Stories:           - story — MET | NOT MET — evidence      (acceptance mode only)
Manager questions: - question — options — recommendation
```
Blocking means the spec must change before the plan or, in acceptance mode, the feature must change before Gate 2. Return at most 25 lines; anything longer goes in your findings file, which the return names.
````

- [ ] **Step 2: Budget rows.** In `budget.md`, add after the `qa, one pass` row:

```markdown
| pm, spec review | Opus | not yet measured | $2 (estimated) |
| pm, acceptance | Opus | not yet measured | $2 (estimated) |
```

- [ ] **Step 3: Recipe.** In `budget.md`'s Recipe, append ` At \`product\` depth add $2 for the PM's spec review.` to the intake line and ` At \`product\` depth add $2 for the PM's acceptance pass.` to the final-review line, and add a last bullet: ``- `plan` depth has no Budget table; the spend script still prices the run from the ledger.``

- [ ] **Step 4: Check.** Run `claude plugin validate .`. Expected: `Validation passed`. Confirm `agents/pm.md` spec mode never names a findings file and acceptance mode names `branch-pm.md`.

- [ ] **Step 5: Commit.** `git add agents/pm.md budget.md && git commit -m "feat(agents): add the pm role for product depth"`

### Task 3: Docs and release

**Files:**
- Modify: `README.md` (intro paragraphs, a new `## Depth` section before `## The lifecycle`, roles table)
- Modify: `docs/team-design.md` (R3, section 3 "When the team model applies", section 5 intro, new 5.5, section 8.1 ledger list)
- Modify: `docs/design.md` (layout line for `agents/`, verification item 4)
- Modify: `CHANGELOG.md` (new 0.4.0 entry on top)
- Modify: `.claude-plugin/plugin.json` (version, description)

- [ ] **Step 1: README intro.** Change `four specialist roles — architect, implementer, reviewer, qa —` to `five specialist roles — architect, implementer, reviewer, qa, pm —`. Replace the paragraph starting `Foreman applies to feature work` with the Depth section, placed before `## The lifecycle`:

```markdown
## Depth

Before any code change, the lead asks how deep to go and recommends an
answer. Name the depth in the request ("direct: fix the typo") and it skips
the question. Plain questions are answered with no depth question.

| Depth | For | What runs |
|---|---|---|
| `direct` | hotfixes, a new page or function | The lead codes it on a branch, tests it, and commits. No plan, no roles |
| `plan` | a complex bug fix, extending a feature | A plan you approve at Gate 1, built by the lead, one reviewer on the diff, a draft PR at Gate 2 |
| `architect` | something the system cannot do yet | The full lifecycle below |
| `product` | a new feature users will see | The full lifecycle plus a PM, who reviews the spec before the plan and the built feature at the end |

A depth that turns out too shallow stops and asks to step up. It never steps
down on its own.
```

Change the line under `## The lifecycle` so it opens with `At \`architect\` and \`product\` depth:` before `1. **Intake.**`. Add a roles-table row after `qa`:

```markdown
| `pm` | Opus | At `product` depth, reviews the spec from the user's side before the plan, and walks the built feature against its user stories at the end | Edit anything |
```

- [ ] **Step 2: team-design.md.** R3 decision becomes `architect, implementer, reviewer, qa, and pm at product depth. The lead is the session itself. Up to three implementers run in parallel when the plan splits cleanly.` Replace the section 3 `**When the team model applies.** ...` paragraph with: `**How much of the team applies.** The lead asks the manager for a depth before any code change: \`direct\`, \`plan\`, \`architect\`, or \`product\`. Section 4 is the \`architect\` and \`product\` lifecycle; \`docs/superpowers/specs/2026-09-28-depth-levels-design.md\` says what the other two skip. Questions get an answer and no depth question. The escalation list applies at every depth.` Section 5 intro `Four files` → `Five files`. Add after 5.4:

```markdown
### 5.5 pm

| Field | Value |
|---|---|
| description | Reviews a spec from the user's side before the plan, and the built feature against the spec's user stories at the end. Use at product depth only, in spec mode after brainstorming and in acceptance mode alongside qa and the final review. Does not edit code. |
| model | `opus` |
| disallowedTools | `Edit, Write, NotebookEdit` |
| color | `blue` |

**Spec mode** checks who the user is, testable acceptance criteria, empty,
error and loading states, copy, accessibility basics, and scope to cut. No
workspace exists yet, so it writes no file. **Acceptance mode** walks each
user story in the running app and reports flows that work but confuse; qa
covers whether it breaks. Blocking items join the fix wave.
```

In 8.1 add `` `Depth: direct | plan | architect | product — <why>`, `` before `` `Tier: small | standard — <why>` ``.

- [ ] **Step 3: design.md.** Layout line becomes `├── agents/                  architect.md, implementer.md, reviewer.md, qa.md, pm.md`. Verification item 4: after `` `foreman:qa` `` add `` , `foreman:pm` ``.

- [ ] **Step 4: Release.** `plugin.json`: `"version": "0.4.0"`, description `Run Claude Code as a software team: architect, implementer, reviewer, qa, and pm roles under a lead that reports to you.` CHANGELOG, above `## 0.3.4`:

```markdown
## 0.4.0

The lead asks how deep to go before any code change, and recommends an
answer: `direct` codes it and commits, `plan` builds from a plan you approve
with one reviewer at the end, `architect` is the full lifecycle, and
`product` adds a PM. Until now every build went through brainstorming, and
the orders treated anything that went through brainstorming as feature work,
so a typo fix got a spec, an architect, qa, and a draft PR. Name the depth in
the request to skip the question. A depth that proves too shallow stops and
asks to step up. A ledger from an earlier version resumes as `architect`.

New role, `foreman:pm`, at `product` depth only: it reviews the spec from
the user's side before the plan, and walks the built feature against the
spec's user stories alongside qa and the final review. Its two dispatches are
budgeted at $2 each until a measured run corrects them.
```

- [ ] **Step 5: Check.** `claude plugin validate .` passes. `grep -n "four roles\|Four roles\|four specialist" README.md docs/*.md orders.md` returns nothing that means the plugin's roster (the doctor's "four role names" for manual-install leftovers stays).

- [ ] **Step 6: Commit.** `git add README.md docs/team-design.md docs/design.md CHANGELOG.md .claude-plugin/plugin.json && git commit -m "docs: depth levels and the pm role (0.4.0)"`

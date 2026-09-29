# Depth Levels Design

**Date:** 2026-09-28
**Status:** Approved by the manager in chat; spec and plan presented together at Gate 1
**Parent designs:** `docs/team-design.md` (the team), `docs/superpowers/specs/2026-09-12-lean-lifecycle-design.md` (the tier). This document decides how much of the team a request gets, before any of it starts. Roles, ledger, gates, tier and budget stay.

## 1. Purpose

The orders reserve the lifecycle for "anything that goes through brainstorming", and superpowers sends every build request through brainstorming. So a typo fix gets a spec, a plan, an architect, qa and a draft PR. The tier is the only size control, and it is decided after the spec and plan already exist.

## 2. Decisions

| Topic | Decision |
|---|---|
| When | Before invoking any skill for a code-change request, the lead recommends a depth and asks the manager with one AskUserQuestion: the four depths, its pick first with a one-line reason. It does not ask when the request names a depth, is not a code change, or continues a run under way in this session or one the manager said to resume. From `plan` up, `Depth: <level> — <why>` is line 2 of the ledger. |
| `direct` | Trivial and moderate hotfixes, a new page or function. The lead implements on the current branch, or a new one when that is the default branch, test-first where there is logic, runs the tests covering the change, commits, reports. No brainstorming, spec, plan, ledger, roles, or push. The depth answer is the approval. The request stands in for the spec, and an addition it names is not trigger 3. Escalations and rulings go in chat. |
| `plan` | Complex bug fixes, extending an existing feature. Questions in chat, then superpowers:writing-plans, workspace and ledger, Gate 1. The plan is the design: no spec file, no in-chat design gate, no Budget table, no tier, no architect. The lead implements the plan itself with superpowers:executing-plans, one commit per task. One `foreman:reviewer`, lens `solo`, on the whole diff; the lead fixes Critical and Important findings; the same reviewer re-reviews the fix range. Gate 2 as today. No qa. |
| `architect` | A capability the system does not have. Today's lifecycle unchanged; the tier still decides solo or pairs. |
| `product` | A new user-facing feature. `architect` plus one `foreman:pm`: spec review after the spec is written and before the plan, and acceptance alongside qa and the final review. PM findings join the one fix wave. |
| Ratchet | When a `direct` or `plan` run hits trigger 3, 4, or 5, the lead asks to step up through the escalation procedure. On a no, the trigger's own question still stands. On a yes it runs what the old depth skipped before going on: from `direct` it writes the plan and its ledger; it ledgers the new `Depth:` line, the last one counting; at `architect` or `product` it adds the Budget table, ledgers the tier (a trigger 3 to 5 makes it `standard`), and runs plan review on the plan, with the PM in spec mode on the plan for `product`; then Gate 1. No spec unless the manager asks: the plan stands in for it for every role, and tasks already built get the final review, not per-task review. It never steps down on its own. Escalation and quality rules apply always. |
| Old ledgers | A ledger with no `Depth:` line predates depths and resumes as `architect`. |
| PM role | `agents/pm.md`, Opus, no Edit, Write or NotebookEdit; findings through a heredoc. Spec mode: user stories, testable acceptance criteria, empty, error and loading states, copy, accessibility basics, scope to cut; decisions only the manager can make go under Manager questions. Acceptance mode: exercises the built feature against the spec's user stories and asks whether it is what the user needed, where qa asks whether it breaks. At most 25 lines back. |
| Budget | Two PM rows, estimated at $2 each until a measured run corrects them. The recipe adds them to intake and final-review at `product` depth and says `plan` depth has no Budget table; the spend script still prices the run. |

## 3. What the manager sees

- One question at the start of a code-change request, with a recommendation.
- `direct`: a commit and a report. `plan`: Gate 1 with the plan path, Gate 2 with one verdict. `architect`: as today. `product`: Gate 1 also shows the PM's spec verdict and questions; Gate 2 also shows the PM's acceptance verdict.

## 4. Files

- `depth.md`: new, the depth rules and the step changes per depth. `orders.md` points to it where "When this applies" was. Both together are over Claude Code's 10,000-character hook limit, past which the lead sees only a preview, so `hooks/hooks.json` runs `session-start.sh` twice: `orders`, then `depth` with the ledger notice.
- `agents/pm.md`: new.
- `budget.md`: PM rows and recipe lines.
- `scripts/pre-compact.sh`: print the ledger's `Depth:` line and keep it with the tier; with no open run, one line asking the summary to keep a depth chosen before the ledger exists.
- `scripts/usage.py`: `pm` joins `ROLES`, so its spend column has a fixed place.
- `README.md`, `docs/team-design.md`, `docs/design.md`, `CHANGELOG.md`, `.claude-plugin/plugin.json` (0.4.0, five roles).

`scripts/usage.py` already tolerates a skipped phase and a missing Budget table. The setup scripts' role lists detect leftovers from the manual install, which never had a PM.

## 5. Verification

1. `claude plugin validate .`, `scripts/selftest.sh`, `tests/test_usage.sh`, `tests/test_setup.sh` pass.
2. In a fresh session with the plugin loaded from this checkout, asking for a typo fix brings up the depth question with `direct` first.
3. This feature runs under its own rules at `plan` depth, by the manager's choice.

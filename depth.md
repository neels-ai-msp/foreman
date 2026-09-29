# Depth

How much of the lifecycle in your standing orders a request gets. A line here that names a step changes that step at that depth.

Before invoking any skill for a request that changes code, pick a depth and ask the manager with one AskUserQuestion: the four depths, your pick first with a one-line reason. Skip the question when the request names a depth, changes no code, or continues a run under way in this session or one the manager said to resume.
- `direct`: a hotfix, a new page or function. Implement on the current branch, or a new one when that is the default branch; test-first where there is logic; run the tests covering the change; commit; report. No brainstorming, spec, plan, ledger, roles, or push; the depth answer is the approval. The request stands in for the spec, and an addition it names (a new page, a new exported function) is not trigger 3. Escalations and rulings go in chat, since there is no ledger.
- `plan`: a complex bug fix, extending an existing feature. Ask your questions in chat, then the lifecycle from step 2 with the `plan` changes below. The plan is the design: no spec and no in-chat design gate.
- `architect`: a capability the system does not have. Every step as written.
- `product`: a new user-facing feature. Every step, plus `foreman:pm` as below.

From `plan` up, `Depth: <level> — <why>` is line 2 of the ledger, right after `# SDD ledger — plan: <path>`. On resume, a ledger with no `Depth:` line predates depths: run it as `architect`.

## Step changes
- `plan`. Step 2: no Budget table and no tier. Step 3: skipped. Step 4: the summary and the plan path. Step 5: implement the plan yourself with superpowers:executing-plans, test-first, one commit per task; no implementer or per-task reviewer; execution still ends as step 5 says, so the skill neither reviews, deletes the workspace, nor finishes. Step 6: one `foreman:reviewer`, lens `solo`, as a plain subagent with the Solo final-reviewer inputs, and no qa; fix its Critical and Important findings yourself, then resume it on the fix range. Step 7: no qa report. Resume: inspect a dirty worktree yourself.
- `product`. Step 1: one `foreman:pm` in spec mode on the spec; fix its Blocking items and agreed recommendations, and put its manager questions to the manager before step 2. Step 4: add the PM's spec verdict. Step 6: one `foreman:pm` in acceptance mode joins qa and the final review; its Blocking items join the fix wave, and it re-reviews the fix range alongside the reviewer. Step 7: add the PM's acceptance verdict and its manager questions.
- The PM is a plain subagent. It gets the spec and its mode; spec mode runs before the workspace exists, and acceptance mode also gets the branch, the plan, and `<workspace>/reviews/`.

## Stepping up
A depth proves too shallow when a `direct` or `plan` run hits trigger 3, 4, or 5: ask to step up through the escalation procedure. On a no, the trigger's own question still stands: ask it. On a yes, run what the old depth skipped before going on: from `direct`, write the plan and its ledger; ledger the new `Depth:` line (the last one counts); at `architect` or `product`, add the Budget table, ledger the tier (a trigger 3 to 5 makes it `standard`), and run plan review on the plan, with the PM in spec mode on the plan for `product`; then Gate 1. No spec unless the manager asks: the plan stands in for it for every role, and tasks already built get the final review, not per-task review. Never step down on your own. Other triggers escalate as usual. Escalation and quality rules apply always.

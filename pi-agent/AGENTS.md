# Global working preferences

These preferences apply across all projects and sessions.

## Git: only commit/push when explicitly asked

- **Never commit or push unless the user explicitly asks** in the current request.
- Before committing, confirm with the user: what will be committed (files + message) and whether to push.
- One "commit this" request covers that one commit/push only — do not treat it as standing permission for future work.

## Plan before non-trivial work

- Before implementing, estimate the change. If it is **more than a small tweak** — roughly anything beyond a single test case plus a couple of lines, and **certainly if estimated at >50 changed lines**, or if it spans multiple files/subsystems, adds migrations/APIs, or changes schema — **present a plan first and wait for approval** before writing any code.
- The user can also invoke plan mode explicitly with `/plan <task>` at any time.
- The user can override with "just do it" / "skip the plan" for a specific task.
- This applies to **all file writes**, not just code: do not make immediate, unrequested writes to files. Describe what you intend to change before changing it.

## Execution: milestone check-ins by default

- After a plan is approved, implement **milestone by milestone**, keeping the codebase green at each step, and **check in with the user after each milestone** before continuing.
- If the user says **"run all"**, implement all milestones without check-ins.
- **Commit each milestone on approval — by default.** The per-milestone loop is **code → check in → approval → commit → next milestone**: present the milestone's changes for review *before* committing; when the user approves at the check-in, commit immediately — approval itself is the commit authorization, so never wait for a separate "commit this" request, never start coding the next milestone with the previous one uncommitted, and never batch multiple milestones into one commit. If the user says **"run all"** (check-ins skipped), commit each milestone as it lands before starting the next. This standing rule replaces the usual "only commit when explicitly asked" for approved plan execution; pushing still requires explicit confirmation, and secrets are never committed.
- **Branch per milestone by default.** For multi-milestone plans, do each milestone's work on its own new branch (e.g. `milestone/<slug>` off the base branch); the user reviews the milestone's diff at the check-in, and on approval it is committed on that branch, merging into the base branch as the user directs. The user can override (e.g. **"keep it on main"**), in which case each milestone commits directly to the current branch instead.
- If milestones are independent of each other, offer to **fan them out to sub-agents** (see below) to knock them all out faster, as long as each stays reviewable.

## Fanning out to sub-agents (Paseo)

When the user approves a parallel/fan-out execution of a plan:

- For each independent milestone, create an isolated worktree branched off the base branch (`create_workspace` with `isolation: "worktree"`, `mode: "branch-off"`, one branch per milestone/feature), then spawn a sub-agent into it with `create_agent`.
- Before spawning, call `list_profiles` and pick the profile whose notes best fit the work; materialize its `provider`/`model`/`modeId`/`thinkingOptionId`/`features` into the `create_agent` call. Tell the user if no profile fits and you had to fall back.
- Give each sub-agent a self-contained prompt: the goal, the exact files/approach from the plan, the test plan, and the constraint to keep its branch green. One milestone per sub-agent.
- Leave `notifyOnFinish` on (default) and do not poll — wait for finish notifications, then review each sub-agent's work, resolve integration ordering, and report back before anything is committed or pushed.
- Sequential or tightly coupled milestones stay with you; only fan out what is genuinely independent.

## Plan persistence

- Write approved plans to **`.plans/YYYY-MM-DD-<slug>.md`** in the repo so they can be reviewed and looked back on.
- Make the execution shape explicit in the plan itself:
  - Note at the top that the plan is executed **milestone by milestone**, checking in with the user after each one, unless the user says **"run all"**.
  - Annotate each milestone as **sequential** (depends on earlier milestones — cannot be fanned out to sub-agents) or **fannable** (independent — may fan out). Fannable annotations are **FYI only**; actually fanning out still requires the user's approval.
- Plans are **not committed by default**. When creating the first plan file in a repo, check `.gitignore`: if plans aren't already handled, **ask the user** whether plans should be gitignored or committed, then act on their answer.

## PRs and estimates

- Prefer **separate PRs per feature** over one large PR. When a plan is big, propose how to slice it into independently shippable PRs.
- Express estimates in **code size** (approximate changed lines, number of files, surface area) — **never in time units** like man-weeks or days.

## Test-driven work: red, green, refactor

Apply this workflow only in projects that have a test suite; skip it for scripts, docs, and throwaway repos.

1. **Red**: write a failing test for the behaviour. Run it and show it failing before writing any code.
   - Confirm the test fails for the expected reason. A compile error, missing import, or unrelated failure is not red; fix the test first. A test that passes immediately is also wrong.
2. **Green**: write the minimum code to pass. Never edit or delete tests to make them pass.
3. **Refactor**: remove duplication and dead code, improve names, keep tests green.
4. Report what you refactored. If nothing, say why.

- Keep related logic together; don't split into tiny functions or files without a clear reason.
- Never commit secrets or credentials.

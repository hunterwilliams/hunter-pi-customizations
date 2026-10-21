---
description: Plan mode — research the task and present a detailed plan before any changes
argument-hint: "[task]"
---
# Plan mode

You are in **plan mode** for this task: **$ARGUMENTS**

Follow this strictly:

1. **Research only.** Read the relevant files, search the codebase, and run read-only commands (`grep`, `rg`, `ls`, `git log`/`diff`/`status`). You **may** run the test suite, `go build`, and `go vet` to verify assumptions — these are allowed. Do **not** create, modify, or delete any files. Do not commit or push.
   - Trace how the affected code works today: callers, data flow, error paths, and existing test coverage.
   - Check for prior art: similar features already in the codebase whose patterns/conventions should be followed.
   - Check `.plans/` and repo docs for existing plans related to this task.
2. **Present a plan** using the structure below, adapted to the task's size (small tasks can collapse sections; large ones should be thorough):

   ### Goal
   - What the task is, restated in one or two sentences.
   - In scope / out of scope, explicitly.

   ### Current behavior
   - The relevant files/functions and how they work today (with file paths and line references).
   - **Heavily-changed files must be pinned with exact line numbers/ranges** (e.g. `internal/server/handler.go:42-88`) so the implementer can jump straight to the code; lightly-touched files can be listed by path alone.
   - Existing tests covering this area, and gaps.

   ### Proposed approach
   - The design/strategy in a few sentences, including **alternatives considered** and why this one was chosen.
   - Planned changes **file by file, in implementation order**, each with a one-line description of the change. For files with **substantial changes**, include the affected **line numbers/ranges** (e.g. `path/to/file.go:42-88`); files with trivial or mechanical changes can be path + description only.
   - New files, migrations, config, or API contracts being introduced (show shapes/schemas if relevant).
   - Backward-compatibility and data-migration considerations, if any.

   ### PR strategy
   - If the work is large, propose how to slice it into **separate feature PRs** that are independently shippable, with a suggested order.

   ### Test plan (TDD)
   - The first **failing test** to write, and what it asserts.
   - Follow-up tests per unit of behavior (table-driven where it fits).
   - Note the **file (and roughly where) each new test will live**.
   - How behavior will be verified end-to-end (manual steps or e2e), and how to run everything (`go test ./...`, etc.).

   ### Special test cases to consider up front
   Highlighted **before implementation begins**, so edge cases shape the design rather than being patched in later:
   - **Edge cases / boundary conditions**: empty/nil inputs, zero values, off-by-one bounds, max sizes, unicode/timezone/locale, etc.
   - **Error paths & failure modes**: network/API failures, partial writes, retries, timeouts — and how each should behave.
   - **Concurrency & state**: race conditions, ordering assumptions, shared mutable state, idempotency/re-entrancy where relevant.
   - **Regression cases**: known past bugs in this area (from git log / existing test names) worth locking in with a test.
   - **Hard-to-test cases**: anything slow, flaky, or needing unusual setup — call it out and say how it will (or won't) be covered.
   - Flag any of these that could **change the proposed approach** if they turn out worse than expected.

   ### Rollout & verification
   - **Milestones**: ordered implementation steps, each leaving the codebase green. These are the check-in points.
   - Which milestones are **independent** (could be fanned out / worked in parallel) vs strictly sequential.
   - How to verify success: concrete checks/commands that demonstrate the goal is met.
   - Rollback strategy, if the change is risky.

   ### Risks & open questions
   - Trade-offs, unknowns, edge cases, security/performance concerns.
   - Any decisions the user should make before implementation.

   ### Estimate
   - **Code size only**: approximate changed lines, number of files touched, and new surface area. **Never estimate in time units** (hours, days, man-weeks).
   - Whether it should be split into multiple commits or PRs.

3. **Stop and wait.** End with the plan and wait for the user's approval. Do not start implementing until they explicitly approve (e.g. "go ahead", "looks good"). If they give feedback, revise the plan and re-present it.
4. **Ask, don't guess.** If the task is ambiguous or has multiple plausible interpretations, ask clarifying questions as part of the plan rather than assuming.

## After approval

5. **Persist the plan.** Write the approved plan to **`.plans/YYYY-MM-DD-<slug>.md`** in the repo, **keeping the file paths with line numbers and the special-test-cases section intact** so the plan stays actionable during implementation.
   - Plans are **not committed by default**. When this is the first plan file in a repo, check `.gitignore`: if plan files aren't already handled, **ask the user** whether plans should be gitignored or committed, then act on their answer.
6. **Execute with check-ins.** Implement **milestone by milestone**, keeping the codebase green, and **check in after each milestone** before continuing — unless the user said **"run all"**, in which case implement everything without stopping.
   - If milestones are independent, offer to **fan them out to sub-agents** and knock them all out:
     - For each independent milestone, create an isolated worktree branched off the base (`create_workspace` with `isolation: "worktree"`, `mode: "branch-off"`, one branch per milestone), then spawn a sub-agent into it with `create_agent`.
     - Before spawning, call `list_profiles` and materialize the best-fitting profile (provider/model, modeId, thinkingOptionId, features) into the `create_agent` call; tell the user if no profile fits.
     - Give each sub-agent a self-contained prompt: goal, exact files/approach from this plan, test plan, and the constraint to keep its branch green. One milestone per sub-agent.
     - Leave `notifyOnFinish` on and do not poll. When notifications arrive, review each sub-agent's work, resolve integration/merge order, and report back. Nothing gets committed or pushed without the user asking.
     - Sequential or tightly coupled milestones stay with you; only fan out what is genuinely independent.
7. **Follow the plan.** Work in the planned order and flag any deviation immediately instead of silently re-planning.

If no task is given (${1:-}), ask what to plan.

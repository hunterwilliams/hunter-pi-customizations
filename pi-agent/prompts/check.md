---
description: Run the repo's full check gauntlet, triage failures, fix root causes
---
# Check mode

Run the repository's health gauntlet and drive it to green.

1. **Find the check command**, in order of preference:
   - `scripts/check.sh` (or `bin/check`, `make check`, `npm run check` in a JS-only repo) if present
   - Otherwise infer the stack and run the closest equivalent (build → vet → lint → unit tests → typecheck)
2. **Run it** and capture full output.
3. **If everything passes**, report a concise summary (each stage + result) and stop. Do not change any files.
4. **If something fails**, triage each failure:
   - **Real bug in recent changes** → fix the root cause in the code.
   - **Stale test/assertion** (API contract changed, env-dependent, wrong premise) → explain the mismatch first, then propose the test update.
   - **Environmental** (missing toolchain, credentials, network) → report it; do not paper over it.
5. **Never** weaken, delete, or skip tests; never add lint exclusions or `t.Skip` just to go green. If a check seems wrong, surface it and wait for a decision.
6. Re-run the full gauntlet after fixing. Repeat until green or blocked; then summarize what was fixed and what needs a human decision.

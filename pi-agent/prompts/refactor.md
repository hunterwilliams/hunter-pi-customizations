---
description: Refactor this branch's changes without changing behaviour
argument-hint: "[focus]"
---
Refactor the code changed on this branch compared with the repository's default branch. First determine the default branch (`git symbolic-ref refs/remotes/origin/HEAD`, strip the `refs/remotes/origin/` prefix; fall back to `main` if that fails). Focus on ${1:-duplication, dead code and naming}.

Rules:
- Behaviour must not change.
- All tests must stay green; do not modify tests.
- Run the tests before and after.
- List each refactoring you made and why.
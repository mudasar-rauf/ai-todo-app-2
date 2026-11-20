# Spec: #71 Add a short CONTRIBUTING.md

**Issue:** #71   **Plan:** [docs/plans/71-short-contributing-md.md](../plans/71-short-contributing-md.md)

## User story

As a new contributor, I want a short guide at the repository's root that tells me how to run the app and its checks and where to find work, so that I can start contributing without asking anyone.

## Acceptance criteria

| ID | Criterion | Source |
|---|---|---|
| AC1 | `CONTRIBUTING.md` says how to install dependencies, start the dev server, and run the tests | issue |
| AC2 | It links to `docs/backlog/BACKLOG.md` for what to work on | issue |
| AC3 | It is under 40 lines | issue |
| AC4 | `CONTRIBUTING.md` is at the repository's root | derived (from the issue's goal) |
| AC5 | The commands it gives are the npm script names fixed in the shell contract (`npm ci`, `npm run dev`, `npm test`, `npm run test:e2e`), not invented ones | derived |

## Out of scope

- Code style, commit conventions, release process, code of conduct.
- Changing `README.md` (owned by backlog task T1.3) or `package.json`.

## Assumptions

- The app does not exist on `main` yet. The commands come from the shell contract (`docs/contracts/shell.md` section 7, PR #69) and the backlog (T1.3), which fix the script names. The guide documents those names.

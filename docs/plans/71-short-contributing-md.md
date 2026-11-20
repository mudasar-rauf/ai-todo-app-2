# Technical Plan: #71 Add a short CONTRIBUTING.md

**Ticket:** #71   **Spec:** [docs/specs/71-short-contributing-md.md](../specs/71-short-contributing-md.md)   **Status:** Draft

## 1. Summary

Add one documentation file, `CONTRIBUTING.md`, at the repository's root. It tells a new contributor how to install dependencies, start the dev server and run the tests, using the npm script names the shell contract fixes, and points to `docs/backlog/BACKLOG.md` for what to work on. It stays under 40 lines. No code, configuration or dependency changes.

## 2. Requirements

### Acceptance criteria

| ID | Criterion | Source |
|---|---|---|
| AC1 | Says how to install dependencies, start the dev server, and run the tests | issue |
| AC2 | Links to `docs/backlog/BACKLOG.md` for what to work on | issue |
| AC3 | Under 40 lines | issue |
| AC4 | At the repository's root | derived |
| AC5 | Uses the contract's npm script names | derived |

### Out of scope

README changes (T1.3 owns `README.md`), code style, commit conventions, code of conduct.

### Assumptions

- `main` has no app yet (verified: only `.factory/`, `.github/workflows/spike-ci.yml` and `docs/backlog/`). The guide documents the commands the backlog and shell contract fix, so it is correct once T1.3 merges. See risk R1.

## 3. Current state

- No `CONTRIBUTING.md` and no `package.json` on `main`.
- `docs/backlog/BACKLOG.md` (T1.3, lines 244–257) says the project is Vite + React 19 + TypeScript with Vitest and Playwright, and lists the scripts `dev, build, preview, typecheck, lint, test (vitest run), test:e2e (playwright test)` and that the README covers `npx playwright install chromium`.
- `docs/contracts/shell.md` section 7 (on branch `factory/3-shell-contract`, PR #69) fixes those script names as binding.
- No file-ownership entry in the backlog covers `CONTRIBUTING.md` (searched), so this change collides with no lane.

## 4. Proposed design

### Changes by component

| File | Change | New or modified |
|---|---|---|
| `CONTRIBUTING.md` | Contributor guide | New |

### Content outline (the developer writes the prose)

1. Title and one-line purpose.
2. **Prerequisites:** Node.js and npm.
3. **Install:** `npm ci` (a fresh clone, uses the lockfile).
4. **Run the app:** `npm run dev`, then open the URL Vite prints.
5. **Run the checks:** `npm test` (Vitest unit tests), `npm run typecheck`, `npm run lint`, `npm run build`; end-to-end with `npx playwright install chromium` once, then `npm run test:e2e`.
6. **What to work on:** a relative link `[docs/backlog/BACKLOG.md](docs/backlog/BACKLOG.md)`.
7. **Pull requests:** one line: open a PR into `main`; the checks above must pass.

## 5. Key decisions

| Decision | Choice | Rationale | Alternatives rejected |
|---|---|---|---|
| Install command | `npm ci` | Reproducible from the lockfile; the backlog's acceptance checks use it | `npm install` (may rewrite the lockfile) |
| Where commands come from | Shell contract's script names | They are binding for every later task and CI | Waiting for T1.3 to merge: blocks a doc on unrelated work |
| Duplicate the README? | No, keep it short; README (T1.3) holds the full script list | Issue asks for short; avoids two owners of the same detail | Moving script docs out of README (out of scope) |

## 6. Implementation tasks

### T1. Write CONTRIBUTING.md (S)
- Goal: the file exists at the root with the outline in section 4.
- Changes: `CONTRIBUTING.md` (new). Nothing else.
- Done when: AC1–AC5 hold; `wc -l CONTRIBUTING.md` prints less than 40; the backlog link resolves on GitHub.
- Tests: the checks in section 7.
- Depends on: none.

### Sequence
One task, size S.

## 7. Test strategy

There is no test runner on `main` yet, so verification is by shell checks the developer runs and records as evidence:

| AC | Check |
|---|---|
| AC1 | `grep -E 'npm ci' CONTRIBUTING.md`, `grep 'npm run dev'`, `grep 'npm test'` each match |
| AC2 | `grep -F '(docs/backlog/BACKLOG.md)' CONTRIBUTING.md` matches and `test -f docs/backlog/BACKLOG.md` |
| AC3 | `test $(wc -l < CONTRIBUTING.md) -lt 40` |
| AC4 | `test -f CONTRIBUTING.md` from the repository root |
| AC5 | every `npm run <name>` in the file is one of `dev, build, preview, typecheck, lint, test, test:e2e` |

Regression risk: none; no code or config changes. The `spike-ci` check runs on the PR as usual.

## 8. Rollout and rollback

**Feature flag:** none. This is a static Markdown file with no runtime behaviour, so there is nothing for an environment variable or config flag to switch; the project has no flag service. Shipping it is the merge, and rollback is reverting the commit. No migrations, config or infrastructure.

## 9. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Commands are documented before T1.3 lands, so they fail on `main` until then | High (now) | Low | Script names are binding in the shell contract; the guide may add one line saying the app scaffold is in progress if T1.3 has not merged when this merges |
| R2 | T1.3 changes a non-binding command (e.g. `typecheck`) | Low | Low | Refer to scripts by name only, never to their underlying commands |

## 10. Technical debt

None taken on.

## 11. Open questions

| # | Question | Who | Blocks | Assumption |
|---|---|---|---|---|
| 1 | Should the guide also describe the factory's agent workflow (`.factory/workflow.yaml`)? | Repo owner | No | No; out of scope for a short guide |

## 12. Follow-ups

- When the app's real CI replaces `spike-ci`, mention it in `CONTRIBUTING.md`.

# Technical Plan: #3 [T1.1] Architect: shell contract (setup, theme, frame, file boundaries)

**Ticket:** #3   **Spec:** [docs/specs/3-shell-contract.md](../specs/3-shell-contract.md)   **Status:** Draft

## 1. Summary

The deliverable is one Markdown file, `docs/contracts/shell.md`, that wave-0 build agents (T1.3, T1.5, T1.7) and their reviewers follow. The developer runs the Astryx CLI in a scratch project **outside the repository**, copies the facts it prints (import specifiers, props, Theme usage, init and theme commands) into shell.md, copies the owns of T1.3, T1.5 and T1.7 from `docs/backlog/BACKLOG.md` section 4, and pastes the CLI output in the PR body. No code ships.

## 2. Requirements

Acceptance criteria AC1 to AC11 are in the [spec](../specs/3-shell-contract.md). Every task below names the IDs it covers.

### Assumptions

- A1. Astryx CLI version 0.7.16 is current (verified 2026-11-20 by installing it). The developer records the version they actually used.
- A2. The neutral theme's CLI slug is found with `theme list`. The backlog calls it theme-neutral (package `@astryxdesign/theme-neutral`); shell.md records both the package name and the slug the CLI prints.
- A3. The frame is AppShell (page shell, banner and main landmarks) with a top-nav slot holding the app name as a level-1 Heading. The developer confirms from the AppShell, TopNav and Layout docs which of them gives one banner and one main landmark, and picks the smallest set that does. Components that are not needed are listed as "considered, not used" with the reason (AC11).
- A4. The app name is "Todo" unless the backlog says otherwise; shell.md fixes the exact string, because T1.7's test asserts it.

## 3. Current state (verified)

- The repository holds only `docs/backlog/`, `.factory/workflow.yaml` and `.github/workflows/spike-ci.yml`. There is no `package.json` and no `docs/contracts/`.
- `docs/backlog/BACKLOG.md` section 4 (line 2917 onward) is the ownership map. The rows for T1.3 (14 entries, including "CLI-generated: every file `npx @astryxdesign/cli init` or `theme` writes"), T1.5 (`src/main.tsx`, `src/index.css`) and T1.7 (`src/App.tsx`, `src/App.test.tsx`, plus deletes of `src/App.css`, `src/assets/react.svg`, `public/vite.svg`) must be copied exactly.
- The issue bodies of T1.3, T1.5 and T1.7 in the same file already fix the install command, the Vite template command, the script commands (`test` = `vitest run`, `test:e2e` = `playwright test`), and StrictMode > Theme > App. shell.md must agree with them.

### CLI behaviour found while planning (verified 2026-11-20, CLI 0.7.16)

- `npx @astryxdesign/cli --help` failed in this container with `sh: 1: cli: Permission denied`. Running the bin through node works: `npm i @astryxdesign/cli` in a scratch directory, then `node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs <args>`.
- `component <Name>` fails with `Could not find @astryxdesign/core package` until `@astryxdesign/core` is installed in the same scratch project. Install `@astryxdesign/core @astryxdesign/theme-neutral @stylexjs/stylex` alongside it.
- With those installed, all six names resolve, and the CLI printed these import lines:
  - `import {Theme} from '@astryxdesign/core/theme';`
  - `import {AppShell} from '@astryxdesign/core/AppShell';`
  - `import {TopNav} from '@astryxdesign/core/TopNav';`
  - `import {Layout} from '@astryxdesign/core/Layout';`
  - `import {Heading} from '@astryxdesign/core/Heading';`
  - `import {Stack} from '@astryxdesign/core/Stack';`
- `init --help` lists `--features agents,theme,template` and `--all`; the default installs agent docs. `theme --help` lists `list`, `add <slug>`, `use <slug>` and `build`. The Theme doc recommends `astryx theme build` plus the built theme object for production, and `defineTheme()` for runtime themes.

The developer must re-run these and copy from their own output, not from this plan.

## 4. Proposed design

### Contents of docs/contracts/shell.md

| Section | Content | ACs |
|---|---|---|
| 1. Status and sources | CLI version, date, the commands run, and a note that every Astryx fact below came from that output | AC9 |
| 2. Setup commands (T1.3) | Numbered, in order: `npm create vite@latest . -- --template react-ts` (ignore existing files, never remove `docs/`); `npm install @astryxdesign/core @stylexjs/stylex @astryxdesign/theme-neutral @astryxdesign/cli`; dev dependencies from T1.3's done_when; `npx @astryxdesign/cli init` with the flags chosen from `init --help`; the theme command(s) from `theme --help` / `theme list` that make neutral available and the default; `npx @astryxdesign/cli doctor`; `npx playwright install chromium` | AC1 |
| 3. Theme | Decision: theme-neutral, package `@astryxdesign/theme-neutral`, CLI slug as printed. Theme import `import {Theme} from '@astryxdesign/core/theme';` as printed. Theme objects come only from the module the CLI generates (path recorded by T1.3 in README) | AC4 |
| 4. Global CSS (T1.5) | `src/index.css` is exactly the two lines, in this order: `@import '@astryxdesign/core/reset.css';` then `@import '@astryxdesign/core/astryx.css';` | AC2 |
| 5. Root (T1.5) | `src/main.tsx` imports `./index.css` and renders `StrictMode > Theme > App`; the Theme props (name and type of the prop that takes the theme) copied from `component Theme` | AC8, AC9 |
| 6. Frame (T1.7) | Table: component, import specifier, props used and their values, which landmark it produces. App name string. The rule that main renders only `<TaskScreen />`. Components considered and not used, with reason | AC5, AC9, AC11 |
| 7. Scripts | `dev`, `build`, `preview`, `typecheck`, `lint`, `test` (`vitest run`), `test:e2e` (`playwright test`); names fixed, later agents rely on them | AC6 |
| 8. File ownership | Three tables, T1.3, T1.5, T1.7, copied row for row from BACKLOG.md section 4, plus a one-line answer: "`src/main.tsx` is owned by T1.5" | AC1, AC3 |
| 9. Changing this contract | Changes after T1.2 passes need a plan task | — |

### Key decisions

| Decision | Choice | Rationale | Alternatives rejected |
|---|---|---|---|
| Where to run the CLI | Scratch directory outside the repo, bin run through node | The issue forbids running `init` in the repo; the repo has no package.json; `npx` failed with permission denied here | Running in the repo (breaks must_not); relying on the website alone (issue wants printed CLI output) |
| Source of truth for owns | Copy BACKLOG.md section 4 verbatim | AC3 says "exactly"; paraphrase invites drift | Summarising by glob |
| Props listed | Only props used in the frame, each as printed | AC9; a full prop dump goes stale and invites misuse | Copying every prop |

## 6. Implementation tasks

### T1. Gather CLI output (S)
- Goal: a saved transcript of every CLI command shell.md relies on.
- Changes: none in the repo. Scratch dir, e.g. `/tmp/astryx-shell`.
- Details: install `@astryxdesign/cli @astryxdesign/core @astryxdesign/theme-neutral @stylexjs/stylex`; run `--version`, `--help`, `init --help`, `theme --help`, `theme list`, `theme add --help`, `theme use --help`, `component Theme`, `component AppShell`, `component TopNav`, `component Layout`, `component Heading`, `component Stack`. Use full detail for the component reads so props are visible. If a name does not resolve, run `search <term> --type component` and record the real name.
- Done when: the transcript covers every fact the shell.md sections need.
- Depends on: none.

### T2. Write docs/contracts/shell.md (M)
- Goal: the contract, sections 1 to 9 above.
- Changes: new file `docs/contracts/shell.md` only.
- Details: copy specifiers and props character for character from T1; copy owns from BACKLOG.md lines 2923 to 2947 for T1.3, T1.5, T1.7; keep consistent with the T1.3, T1.5 and T1.7 issue bodies. Choose the frame components from the docs (A3) and say why.
- Done when: AC1 to AC6, AC8, AC9 and AC11 can each be pointed at a section.
- Depends on: T1.

### T3. Self-check and open the PR (S)
- Goal: evidence the contract is right and traceable.
- Details: grep shell.md for every `@astryxdesign/` specifier and every prop name and find each in the transcript; diff the ownership tables against BACKLOG.md section 4; `git diff --name-only main` shows only `docs/contracts/shell.md` plus this spec and plan. Push to `factory/3-shell-contract`, update this draft PR (do not open another), and put `Closes #3` and the CLI transcript (in `<details>` blocks) in its body.
- Done when: AC7 and AC10 hold.
- Depends on: T2.

Sequence: T1 → T2 → T3, serial. Total about 1.5 days at most; realistically under one.

## 7. Test strategy

There is no code, so the checks are documentary and are run by the developer and again by the T1.2 reviewer.

| Acceptance criterion | Tasks | Check | What it verifies |
|---|---|---|---|
| AC1 | T2 | Cold read of shell.md sections 2, 3, 8 | Setup commands, theme, owner of src/main.tsx are stated |
| AC2 | T2 | `grep -F` both import lines; order checked | Verbatim and ordered |
| AC3 | T2, T3 | Diff ownership tables against BACKLOG.md section 4 rows | Exact match |
| AC4, AC5, AC8, AC9 | T1, T2, T3 | Every specifier and prop in shell.md found in the pasted CLI output | Nothing invented |
| AC6 | T2 | All seven script names present | Names fixed |
| AC7 | T3 | PR body contains `Closes #3` and the transcript | Traceability |
| AC10 | T3 | `git diff --name-only` | Only allowed paths |
| AC11 | T1, T2 | Six components each have a "used" or "considered, not used" entry | Every named component confirmed |

Regression risk: none; no code changes. Downstream risk is a wrong fact propagating into T1.3, T1.5, T1.7, which the self-check guards.

## 8. Rollout and rollback

Feature flag: none. This change is a Markdown document with no runtime effect, so there is nothing to put behind a flag; the plan gate and the T1.2 review are the release controls. Rollback is reverting the commit. Flagged for the architect to confirm this exception.

## 9. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| CLI output changes between this contract and T1.3 (patch release) | Medium | Medium | Record the CLI version; T1.3 pins it via package-lock |
| `npx` cannot execute the bin in the container | High (seen) | Low | Run it through node as in section 3 |
| `init` or `theme add` writes files T1.5/T1.7 own (for example `src/main.tsx`) | Medium | Medium | shell.md tells T1.3 to keep CLI changes to those files and list them in the PR body, as the T1.3 issue says |
| AppShell renders its own landmarks so Layout or TopNav are redundant or double a landmark | Medium | Medium | Read the AppShell doc for its slots and landmarks; S1 needs exactly one banner and one main |

## 11. Open questions

| # | Question | Who can answer | Blocks | Assumption made |
|---|---|---|---|---|
| 1 | Exact app name shown in the header | Product owner | No | "Todo" (A4); trivial to change before T1.7 |
| 2 | Are the factory's spec and plan files acceptable beside shell.md, given T1.1 owns only shell.md? | Plan gate approver | No | Yes; they are process documents (spec, AC10) |

## 12. Follow-ups

- Replace `.github/workflows/spike-ci.yml` with real CI once T1.3 lands (already noted in that file).

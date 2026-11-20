# Spec: #3 [T1.1] Architect: shell contract (setup, theme, frame, file boundaries)

**Issue:** #3   **Plan:** [docs/plans/3-shell-contract.md](../plans/3-shell-contract.md)

## User story

As the build and review agents of wave 0 (T1.3, T1.5, T1.7 and their reviews), I want one document, `docs/contracts/shell.md`, that settles how the app shell is set up before any code exists: the setup commands in order, which task owns which file, the theme, the global CSS, how `Theme` wraps the root, which Astryx components build the frame, and the npm script names. Then each of us can build or check our part without guessing or asking.

## Scope

- In: writing `docs/contracts/shell.md`, from the Astryx CLI output and the ownership map in `docs/backlog/BACKLOG.md` section 4.
- Out: any application code, `package.json`, running `npx @astryxdesign/cli init` in the repository (T1.3 does that), the Task contract (T2.1), and choosing props beyond what the CLI prints.

## Acceptance criteria

Criteria from the issue keep its words. The source column says where in the issue each comes from.

| ID | Criterion | Source |
|---|---|---|
| AC1 | Reading only shell.md, a person can list the setup commands, the theme, and which task owns src/main.tsx | Acceptance |
| AC2 | shell.md states the two global CSS imports verbatim: `@import '@astryxdesign/core/reset.css';` then `@import '@astryxdesign/core/astryx.css';` | Acceptance |
| AC3 | shell.md lists the owns of T1.3, T1.5 and T1.7 exactly as in the ownership map | done_when |
| AC4 | shell.md records theme-neutral and the Theme import specifier printed by the CLI | done_when |
| AC5 | shell.md names the frame components with their specifiers and props, as the CLI printed them | done_when |
| AC6 | shell.md fixes script names: dev, build, preview, typecheck, lint, test, test:e2e | done_when |
| AC7 | One branch, one PR; body says `Closes #3` and pastes the CLI output it relied on | done_when |
| AC8 | shell.md states how Theme wraps the root, from the CLI docs | Outcome |
| AC9 | shell.md names no Astryx component, import specifier, or prop the CLI did not print | must_not |
| AC10 | The change edits no path outside `docs/contracts/shell.md`, implements no feature behavior, and does not run `npx @astryxdesign/cli init` in the repo | must_not |
| AC11 | Each component listed under the issue's Astryx heading (Theme, AppShell, TopNav, Layout, Heading, Stack) was confirmed with `npx @astryxdesign/cli component <Name>`; any it does not use in the frame is recorded as considered and not used, with the reason | derived from the Astryx section |

AC10 has one exception the factory requires: this spec and its plan live on the same branch (`docs/specs/3-shell-contract.md`, `docs/plans/3-shell-contract.md`). They are process documents, not part of the shell contract.

## Non-functional requirements

- Every Astryx fact in shell.md is traceable to CLI output pasted in the PR body (CLI version recorded).
- Written so a build agent new to the repo can act on it without a question.

# Backlog: local-first Astryx todo app

68 issues: 8 epics, 8 stories, 52 tasks (8 architect, 18 build, 26 review). Machine-readable copy: `docs/backlog/issues.json` (same titles, labels, parents, bodies).

## 0. Conventions

- **Keys and placeholders.** Titles start with a key (`[E1]`, `[S1]`, `[T1.3]`). `parent` is a placeholder such as `{{E1}}`; the orchestrator swaps it for the real issue number. `depends_on` lists full titles.
- **parallel_group = lane.** `serial` items run one at a time in the order listed. In waves 1 and 2, all lanes run at once, and items inside a lane run as soon as their `depends_on` are met; sibling builds own disjoint files.
- **Stubs.** T2.1 (architect) creates every wave-1 file as a stub with no behavior. When T2.2 passes, each stub belongs to exactly one build task (section 4). Architects are not build agents, so no file ever has two build owners.
- **Fix tasks.** Title `[<key>-fix<n>] Fix: <target title>`, same labels, parent, and owns as the target. depends_on is the review that returned changes, and inputs is that review's verdict comment. It commits to the target's existing PR branch (still one branch, one PR), and the same review issue re-runs. After 3 fix tasks the item goes to persona plan with label `blocked`. For an architect target, the fix is done by persona architect.
- **Astryx names** are catalog names from astryx.atmeta.com in PascalCase. If `npx @astryxdesign/cli component <Name>` does not resolve one, run `npx @astryxdesign/cli search <term> --type component` and record the real name in the lane contract. No issue specifies props; the CLI does.
- **Prerequisite.** `main` has no commits, so a PR has no base. Commit `docs/backlog/` as the first commit on `main` before T1.1 starts.
- **Decisions made here, not left open:** completed tasks sort by createdAt (newest first), the same as incomplete ones. Search covers titles only. Priority filter is single-select with Any. Clear completed has no confirm dialog (the count is in the label). Unreadable stored data is backed up, never discarded. Edit and composer fields are duplicated across lanes rather than shared, to keep lanes parallel.

## 1. Labels

| name | color | description |
|---|---|---|
| `type:epic` | `#6F42C1` | Apply to an outcome slice that only tracks child stories. Never on an item that opens a PR. |
| `type:story` | `#0366D6` | Apply to one user-visible outcome under an epic. Closes when its child reviews pass. No PR. |
| `type:task` | `#6A737D` | Apply to one agent-sized unit ending in one PR or one review verdict. Parent is a story. |
| `persona:plan` | `#F9D0C4` | Apply to epics, stories, and escalations (split or rewrite backlog items). Never writes code. |
| `persona:architect` | `#D4C5F9` | Apply when output is a contract: types, stubs, file boundaries, test list. No feature behavior. |
| `persona:build` | `#C2E0C6` | Apply when output is one PR implementing one task inside its owns list. Fix tasks too. |
| `persona:review` | `#FEF2C0` | Apply when output is a pass/changes verdict on one PR. Reviewers never push commits. |
| `wave:0` | `#1D3557` | Apply to serial foundation: shell, Astryx init, theme, frame, task contract. Gates wave 1. |
| `wave:1` | `#457B9D` | Apply to feature lanes that run in parallel once every wave:0 review passes. |
| `wave:2` | `#A8DADC` | Apply to whole-app a11y, responsive, and e2e checks after every wave:1 review passes. |
| `area:shell` | `#BFD4F2` | Apply when it touches tooling, Astryx setup, theme, global CSS, or the app frame. |
| `area:tasks` | `#BFDADC` | Apply when it touches the Task contract, composer, list and rows, or edit/delete dialogs. |
| `area:filters` | `#C5DEF5` | Apply when it touches status/priority filtering, title search, or clear completed. |
| `area:persistence` | `#FAD8C7` | Apply when it touches localStorage, the task store, or loading/empty/save-error states. |
| `area:a11y` | `#D1F2EB` | Apply when the main check is keyboard access, focus, accessible names, or responsive layout. |
| `area:quality` | `#E4E669` | Apply to end-to-end tests or test infrastructure that spans areas. |
| `priority:p0` | `#B60205` | Apply when the foundation or core loop (add, see, complete, persist) needs it. Schedule first. |
| `priority:p1` | `#D93F0B` | Apply when required for release but not for the core loop. Schedule after p0. |
| `blocked` | `#000000` | Apply when it needs a path outside owns, a contract change, or failed 3 reviews. Comment why. |

## 2. Wave plan

| Step | Runs | Items (persona) | Starts when |
|---|---|---|---|
| W0.1 | alone | T1.1 (architect) then T1.2 (review) | `main` has its first commit |
| W0.2 | alone | T1.3 (build) then T1.4 (review) | T1.2 pass |
| W0.3 | alone | T1.5 (build) then T1.6 (review) | T1.4 pass |
| W0.4 | alone | T2.1 (architect) then T2.2 (review) | T1.6 pass |
| W0.5 | alone | T1.7 (build) then T1.8 (review) | T2.2 pass (frame mounts the TaskScreen stub) |
| W0.6 | alone | T2.3 (build) then T2.4 (review) | T1.8 pass |
| W1.a | together, 5 agents | T3.1, T4.1, T5.1, T6.1, T7.1 (architect), each followed by its review T3.2, T4.2, T5.2, T6.2, T7.2 | T1.8 and T2.4 pass (wave 0 gate) |
| W1.b | together, up to 12 agents | capture: T3.3 / list: T4.3, T4.5 / mutate: T5.3, T5.5 / query: T6.3, T6.5, T6.7 / data: T7.3, T7.5, T7.7, T7.9 (build) | each when its lane's architect review passes |
| W1.c | together | T3.4, T4.4, T4.6, T5.4, T5.6, T6.4, T6.6, T6.8, T7.4, T7.6, T7.8, T7.10 (review) | each when its build PR opens |
| W2.a | alone | T8.1 (architect) then T8.2 (review) | all 12 wave-1 build reviews pass (wave 1 gate) |
| W2.b | together, 2 agents | T8.3, T8.5 (build), then T8.4, T8.6 (review) | T8.2 pass |

Story Acceptance (S3 to S7) is checked on `main` once the lanes it names have merged: the app needs the composer, list, and store together to show anything. Task Acceptance is checked per PR. Epics and stories are persona plan and never run as agents; they close when their children do.

## 3. Issues (dependency order)

### [E1] App shell and design system
- labels: `type:epic`, `persona:plan`, `wave:0`, `area:shell`, `priority:p0`
- parent: none

```markdown
## Outcome
The repo runs a Vite + React 19 + TypeScript app on Astryx, with test tooling and a themed frame (header + main) that all later work renders into. Theme decision: @astryxdesign/theme-neutral, used through the themes module the Astryx CLI generates. Changing the theme needs a plan task.

## Agent intent
persona: plan
wave: 0
parallel_group: serial
depends_on: []
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S1 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm run dev` shows the app in theme-neutral styling with a header and a main region
- [ ] Every task under S1 has a review verdict of pass

## Astryx
none
```

### [S1] Running app renders an Astryx-themed frame with a header and a main region
- labels: `type:story`, `persona:plan`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{E1}}

```markdown
## Outcome
Opening the dev server shows an Astryx-styled page: a header with the app name and a main region where the task screen mounts. Typecheck, lint, Vitest, and Playwright are set up, so no feature task touches package.json.

## Agent intent
persona: plan
wave: 0
parallel_group: serial
depends_on: []
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E1
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T1.2, T1.4, T1.6, T1.8 have verdict pass
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm run dev`: header with the app name, main region below, no Vite demo content
- [ ] Text and spacing come from theme-neutral (no default browser body margin, Astryx font)
- [ ] The browser accessibility tree shows one banner landmark and one main landmark
- [ ] `npm ci && npm test && npm run build` succeed on a fresh clone

## Astryx
none
```

### [T1.1] Architect: shell contract (setup, theme, frame, file boundaries)
- labels: `type:task`, `persona:architect`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
docs/contracts/shell.md settles the shell before any code: setup commands in order, the exact owns of T1.3, T1.5 and T1.7, the theme decision (theme-neutral), the two global CSS imports, how Theme wraps the root (from CLI docs), which Astryx components build the frame (with import specifiers and props the CLI printed), and the npm script names later agents rely on.

## Agent intent
persona: architect
wave: 0
parallel_group: serial
depends_on: []
owns: ["docs/contracts/shell.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Run `npx @astryxdesign/cli init` in the repo (T1.3 does)
inputs:
- https://astryx.atmeta.com/ and https://astryx.atmeta.com/docs/cli
- `npx @astryxdesign/cli --help`
- `npx @astryxdesign/cli init --help` and `npx @astryxdesign/cli theme --help`
- `npx @astryxdesign/cli component Theme` plus each frame component under Astryx
- docs/backlog/BACKLOG.md section 4 (ownership map)
handoff_to: review
done_when:
- shell.md lists the owns of T1.3, T1.5 and T1.7 exactly as in the ownership map
- shell.md records theme-neutral and the Theme import specifier printed by the CLI
- shell.md names the frame components with their specifiers and props, as the CLI printed them
- shell.md fixes script names: dev, build, preview, typecheck, lint, test, test:e2e
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Reading only shell.md, a person can list the setup commands, the theme, and which task owns src/main.tsx
- [ ] shell.md states the two global CSS imports verbatim: `@import '@astryxdesign/core/reset.css';` then `@import '@astryxdesign/core/astryx.css';`

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Theme
- AppShell
- TopNav
- Layout
- Heading
- Stack
```

### [T1.2] Review: shell contract (setup, theme, frame, file boundaries)
- labels: `type:task`, `persona:review`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T1.1] Architect: shell contract (setup, theme, frame, file boundaries), based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 0
parallel_group: serial
depends_on: ["[T1.1] Architect: shell contract (setup, theme, frame, file boundaries)"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T1.1] Architect: shell contract (setup, theme, frame, file boundaries)
- Issue [T1.1] Architect: shell contract (setup, theme, frame, file boundaries)
- Every doc listed in T1.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T1.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T1.1] Architect: shell contract (setup, theme, frame, file boundaries) as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T1.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Theme
- AppShell
- TopNav
- Layout
- Heading
- Stack
```

### [T1.3] Build: scaffold Vite + React 19 + TS, Astryx init, test tooling
- labels: `type:task`, `persona:build`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
The repo is a Vite + React 19 + TypeScript project with the four Astryx packages installed, `npx @astryxdesign/cli init` applied, theme-neutral available through the generated themes module, and Vitest + Testing Library + Playwright ready. After this, no task needs to touch package.json.

## Agent intent
persona: build
wave: 0
parallel_group: serial
depends_on: ["[T1.2] Review: shell contract (setup, theme, frame, file boundaries)"]
owns: ["package.json", "package-lock.json", "index.html", "vite.config.ts", "tsconfig.json", "tsconfig.app.json", "tsconfig.node.json", "eslint.config.js", ".gitignore", "README.md", "src/vite-env.d.ts", "src/test/setup.ts", "playwright.config.ts", "CLI-generated: every file `npx @astryxdesign/cli init` or `theme` writes (themes module, astryx.config.*, agent docs), each listed in the PR body"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Hand-edit src/main.tsx, src/index.css, src/App.tsx, src/App.css, src/assets/**, public/** (template output only; T1.5 and T1.7 own them). If the CLI rewrites one, keep its change and list it in the PR body.
- Delete or overwrite docs/**. If the Vite scaffolder reports a non-empty directory, choose to ignore existing files, never to remove them.
- Pick any theme other than @astryxdesign/theme-neutral
- Add packages beyond those named in done_when
inputs:
- docs/contracts/shell.md
- `npm create vite@latest . -- --template react-ts`
- `npm install @astryxdesign/core @stylexjs/stylex @astryxdesign/theme-neutral @astryxdesign/cli`
- `npx @astryxdesign/cli init`; if it does not produce a themes module with theme-neutral, `npx @astryxdesign/cli theme --help`
- `npx @astryxdesign/cli doctor`
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- package.json has react and react-dom 19, the four Astryx packages, and devDependencies vitest, jsdom, @testing-library/react, @testing-library/user-event, @testing-library/jest-dom, @playwright/test
- Scripts exist: dev, build, preview, typecheck, lint, test (`vitest run`), test:e2e (`playwright test`)
- Vitest uses jsdom, setupFiles src/test/setup.ts (loads jest-dom matchers), passWithNoTests true, and excludes e2e/**
- playwright.config.ts: testDir e2e, webServer runs `npm run build && npm run preview`, projects Desktop Chrome and Pixel 7
- `npx @astryxdesign/cli doctor` reports no errors
- README lists every script, the themes module path, and `npx playwright install chromium`
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] On a fresh clone, `npm ci && npm run build && npm test` succeed
- [ ] `npm run dev` serves a page (Vite template content is expected at this point)
- [ ] The PR body lists every file the Astryx CLI wrote

## Astryx
none
```

### [T1.4] Review: scaffold Vite + React 19 + TS, Astryx init, test tooling
- labels: `type:task`, `persona:review`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T1.3] Build: scaffold Vite + React 19 + TS, Astryx init, test tooling, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 0
parallel_group: serial
depends_on: ["[T1.3] Build: scaffold Vite + React 19 + TS, Astryx init, test tooling"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T1.3] Build: scaffold Vite + React 19 + TS, Astryx init, test tooling
- Issue [T1.3] Build: scaffold Vite + React 19 + TS, Astryx init, test tooling
- Every doc listed in T1.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T1.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T1.3] Build: scaffold Vite + React 19 + TS, Astryx init, test tooling as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T1.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
none
```

### [T1.5] Build: wire theme-neutral and global CSS
- labels: `type:task`, `persona:build`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
Every render goes through Astryx: src/index.css loads the reset and astryx.css, and src/main.tsx wraps <App /> in the Astryx Theme built from the generated themes module with theme-neutral.

## Agent intent
persona: build
wave: 0
parallel_group: serial
depends_on: ["[T1.4] Review: scaffold Vite + React 19 + TS, Astryx init, test tooling"]
owns: ["src/main.tsx", "src/index.css"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Put anything in src/index.css except the two @import lines
- Import a theme from anywhere but the generated themes module
- Edit src/App.tsx
inputs:
- docs/contracts/shell.md
- README (themes module path)
- `npx @astryxdesign/cli component Theme`
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- src/index.css is exactly `@import '@astryxdesign/core/reset.css';` then `@import '@astryxdesign/core/astryx.css';`
- src/main.tsx imports ./index.css and renders StrictMode > Theme (theme-neutral) > App
- The dev page logs no console errors or warnings
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] In the dev page, body margin is 0 and text uses the theme-neutral font (computed styles in devtools)
- [ ] Removing the Theme wrapper locally visibly changes the page (proves the theme applies); restore before pushing

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Theme
```

### [T1.6] Review: wire theme-neutral and global CSS
- labels: `type:task`, `persona:review`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T1.5] Build: wire theme-neutral and global CSS, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 0
parallel_group: serial
depends_on: ["[T1.5] Build: wire theme-neutral and global CSS"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T1.5] Build: wire theme-neutral and global CSS
- Issue [T1.5] Build: wire theme-neutral and global CSS
- Every doc listed in T1.5's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T1.5
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T1.5] Build: wire theme-neutral and global CSS as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T1.5's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Theme
```

### [E2] Task contract
- labels: `type:epic`, `persona:plan`, `wave:0`, `area:tasks`, `priority:p0`
- parent: none

```markdown
## Outcome
One Task type, one TaskStorage interface, one TaskStore shape, and a typed stub for every wave-1 file are on main, and TaskScreen wires them together. Each wave-1 build replaces exactly one set of stubs and needs no other file. No feature behavior ships in this epic.

## Agent intent
persona: plan
wave: 0
parallel_group: serial
depends_on: []
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S2 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] src/tasks/types.ts and src/tasks/storage.ts exist on main and `npm run typecheck` passes
- [ ] Every wave-1 build task's owns names a file that already exists on main as a stub

## Astryx
none
```

### [S2] Every feature shares one Task type and one storage interface
- labels: `type:story`, `persona:plan`, `wave:0`, `area:tasks`, `priority:p0`
- parent: {{E2}}

```markdown
## Outcome
Wave-1 agents build against a fixed contract (types, storage interface, store shape, component props, file boundaries) and a composition root that already renders every feature slot, so lanes never edit a shared file.

## Agent intent
persona: plan
wave: 0
parallel_group: serial
depends_on: []
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E2
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T2.2 and T2.4 have verdict pass
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] docs/contracts/tasks.md names, for each wave-1 file, its owner task, its exports, and its test list
- [ ] The running app still renders the frame with no console errors while every feature is a stub
- [ ] tasks.md states that changing a contract type after this story closes needs a plan task

## Astryx
none
```

### [T2.1] Architect: Task type, storage interface, store shape, and stubs
- labels: `type:task`, `persona:architect`, `wave:0`, `area:tasks`, `priority:p0`
- parent: {{S2}}

```markdown
## Outcome
The shared contract exists as code and one doc. types.ts and storage.ts are the only shared modules; every wave-1 file exists as a behavior-free stub with its final exports, so the composition root compiles and each lane replaces only its own stubs.

## Agent intent
persona: architect
wave: 0
parallel_group: serial
depends_on: ["[T1.6] Review: wire theme-neutral and global CSS"]
owns: ["src/tasks/types.ts", "src/tasks/storage.ts", "docs/contracts/tasks.md", "src/tasks/TaskScreen.tsx (stub only; passes to T2.3 when T2.2 is pass)", "src/tasks/TaskComposer.tsx (stub only; passes to T3.3 when T2.2 is pass)", "src/tasks/TaskRow.tsx (stub only; passes to T4.3 when T2.2 is pass)", "src/tasks/TaskList.tsx (stub only; passes to T4.5 when T2.2 is pass)", "src/tasks/sortTasks.ts (stub only; passes to T4.5 when T2.2 is pass)", "src/tasks/EditTaskDialog.tsx (stub only; passes to T5.3 when T2.2 is pass)", "src/tasks/ConfirmDeleteDialog.tsx (stub only; passes to T5.5 when T2.2 is pass)", "src/tasks/filterTasks.ts (stub only; passes to T6.3 when T2.2 is pass)", "src/tasks/FilterBar.tsx (stub only; passes to T6.5 when T2.2 is pass)", "src/tasks/SearchField.tsx (stub only; passes to T6.7 when T2.2 is pass)", "src/tasks/localTaskStorage.ts (stub only; passes to T7.3 when T2.2 is pass)", "src/tasks/useTasks.ts (stub only; passes to T7.5 when T2.2 is pass)", "src/tasks/StatusViews.tsx (stub only; passes to T7.7 when T2.2 is pass)", "src/tasks/SaveErrorBanner.tsx (stub only; passes to T7.9 when T2.2 is pass)"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Import Astryx components in stubs
inputs:
- docs/contracts/shell.md
- docs/backlog/BACKLOG.md sections 2 and 4
- README (scripts)
handoff_to: review
done_when:
- types.ts exports at least: Priority = 'none' | 'low' | 'medium' | 'high'; PRIORITIES; Task { id: string; title: string; notes: string; priority: Priority; dueDate: string | null (YYYY-MM-DD local calendar date); completed: boolean; createdAt: number }; TaskDraft (title, notes, priority, dueDate); TaskPatch (Partial of TaskDraft plus completed); StatusFilter = 'all' | 'active' | 'completed'; TaskQuery { status; priority: Priority | 'any'; search: string }; DEFAULT_QUERY; COMPOSER_TITLE_ID
- types.ts exports TaskStore { status: 'loading' | 'ready'; tasks: Task[]; saveError: string | null; add(draft); update(id, patch); remove(id); clearCompleted() }
- types.ts exports props types: TaskScreenProps { storage?: TaskStorage }; TaskComposerProps { onAdd }; TaskRowProps { task; onToggle(id); onEdit(task); onDelete(task) }; TaskListProps { tasks; onToggle; onEdit; onDelete }; EditTaskDialogProps { task: Task | null; onSave(id, patch); onClose }; ConfirmDeleteDialogProps { task: Task | null; onConfirm(id); onClose }; FilterBarProps { query; onChange(query); completedCount; onClearCompleted }; SearchFieldProps { value; onChange(value) }; EmptyStateProps { variant: 'no-tasks' | 'no-matches'; onClearFilters }; SaveErrorBannerProps { message: string | null }
- storage.ts exports TaskStorage { load(): Task[]; save(tasks: readonly Task[]): void } (save throws on failure) and STORAGE_KEY
- Stub exports: TaskScreen, TaskComposer, TaskRow, TaskList, sortTasks(tasks), EditTaskDialog, ConfirmDeleteDialog, filterTasks(tasks, query), FilterBar, SearchField, createLocalTaskStorage(storage?) and localTaskStorage, useTasks(storage), LoadingState and EmptyState (StatusViews.tsx), SaveErrorBanner
- tasks.md covers: component tree, data flow, store rules (no save before first load completes; a failed save keeps the list and sets saveError; the next successful save clears it), sort rule, filter rule, focus rules, test list per file, ownership table
- The app builds and renders the frame with every stub in place
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Reading tasks.md alone, a person can say which task owns any file under src/tasks/ and what it exports
- [ ] `npm run typecheck` and `npm run build` pass on the PR branch
- [ ] Every stub's body is `return null`, an input copy, or an inert value (reviewer spot-checks all 14)

## Astryx
none
```

### [T2.2] Review: Task type, storage interface, store shape, and stubs
- labels: `type:task`, `persona:review`, `wave:0`, `area:tasks`, `priority:p0`
- parent: {{S2}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T2.1] Architect: Task type, storage interface, store shape, and stubs, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 0
parallel_group: serial
depends_on: ["[T2.1] Architect: Task type, storage interface, store shape, and stubs"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T2.1] Architect: Task type, storage interface, store shape, and stubs
- Issue [T2.1] Architect: Task type, storage interface, store shape, and stubs
- Every doc listed in T2.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T2.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T2.1] Architect: Task type, storage interface, store shape, and stubs as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T2.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
none
```

### [T1.7] Build: app frame with header and main region
- labels: `type:task`, `persona:build`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
The running app shows a header (banner landmark) whose only h1 is the app name and a main landmark that renders <TaskScreen />, laid out with the Astryx components chosen in docs/contracts/shell.md. The Vite demo files are gone.

## Agent intent
persona: build
wave: 0
parallel_group: serial
depends_on: ["[T2.2] Review: Task type, storage interface, store shape, and stubs"]
owns: ["src/App.tsx", "src/App.test.tsx", "src/App.css (delete)", "src/assets/react.svg (delete)", "public/vite.svg (delete)"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Render task UI directly; main only renders <TaskScreen />
- Add CSS files or inline styles
- Edit src/main.tsx
inputs:
- docs/contracts/shell.md
- docs/contracts/tasks.md (from T2.1)
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- App.test.tsx asserts one banner landmark holding a level-1 heading with the app name, and one main landmark
- Nothing references the deleted demo files
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Dev page: header with the app name at the top, main region below, no Vite or React logos
- [ ] At 375px width there is no horizontal scrollbar
- [ ] The browser accessibility tree shows banner and main landmarks

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- AppShell
- TopNav
- Layout
- Heading
- Stack
```

### [T1.8] Review: app frame with header and main region
- labels: `type:task`, `persona:review`, `wave:0`, `area:shell`, `priority:p0`
- parent: {{S1}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T1.7] Build: app frame with header and main region, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 0
parallel_group: serial
depends_on: ["[T1.7] Build: app frame with header and main region"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T1.7] Build: app frame with header and main region
- Issue [T1.7] Build: app frame with header and main region
- Every doc listed in T1.7's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T1.7
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T1.7] Build: app frame with header and main region as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T1.7's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- AppShell
- TopNav
- Layout
- Heading
- Stack
```

### [T2.3] Build: compose TaskScreen from contract stubs
- labels: `type:task`, `persona:build`, `wave:0`, `area:tasks`, `priority:p0`
- parent: {{S2}}

```markdown
## Outcome
TaskScreen holds all screen state and wiring, so wave-1 lanes never touch it. It calls useTasks(storage ?? localTaskStorage) and holds the TaskQuery, the task being edited, and the task pending delete. It renders SaveErrorBanner, TaskComposer, SearchField, FilterBar, then LoadingState, TaskList, or EmptyState, then EditTaskDialog and ConfirmDeleteDialog. As each stub is replaced, its feature appears with no more wiring.

## Agent intent
persona: build
wave: 0
parallel_group: serial
depends_on: ["[T1.8] Review: app frame with header and main region"]
owns: ["src/tasks/TaskScreen.tsx", "src/tasks/TaskScreen.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Implement child behavior (sorting, filtering, persistence, validation)
- Edit any stub file
- Add dependencies or touch src/tasks/types.ts or src/tasks/storage.ts
inputs:
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/shell.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- TaskScreen.test.tsx mocks child modules with vi.mock and passes: passes filterTasks(tasks, query) to TaskList
- Test: shows LoadingState while status is 'loading'
- Test: shows EmptyState 'no-tasks' when there are no tasks and 'no-matches' when tasks exist but none are visible; onClearFilters resets the query to DEFAULT_QUERY
- Test: onToggle(id) calls update(id, { completed: !task.completed })
- Test: onEdit opens EditTaskDialog with that task; onSave calls update and closes it
- Test: onDelete opens ConfirmDeleteDialog; onConfirm calls remove(id), closes it, and focuses the element with id COMPOSER_TITLE_ID
- Test: FilterBar gets completedCount and onClearCompleted calls clearCompleted; SaveErrorBanner gets saveError
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- TaskScreen` lists the wiring tests above as passing
- [ ] The dev page renders the header and an empty main with no console errors (children are still stubs)

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Stack
```

### [T2.4] Review: compose TaskScreen from contract stubs
- labels: `type:task`, `persona:review`, `wave:0`, `area:tasks`, `priority:p0`
- parent: {{S2}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T2.3] Build: compose TaskScreen from contract stubs, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 0
parallel_group: serial
depends_on: ["[T2.3] Build: compose TaskScreen from contract stubs"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T2.3] Build: compose TaskScreen from contract stubs
- Issue [T2.3] Build: compose TaskScreen from contract stubs
- Every doc listed in T2.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T2.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T2.3] Build: compose TaskScreen from contract stubs as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T2.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Stack
```

### [E3] Capture a task
- labels: `type:epic`, `persona:plan`, `wave:1`, `area:tasks`, `priority:p0`
- parent: none

```markdown
## Outcome
A user can add a task with a required title and optional notes, priority, and due date from a form at the top of the screen.

## Agent intent
persona: plan
wave: 1
parallel_group: capture
depends_on: ["[E1] App shell and design system", "[E2] Task contract"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S3 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] S3 Acceptance checked on main

## Astryx
none
```

### [S3] User adds a task with title, notes, priority, and due date, and sees it appear
- labels: `type:story`, `persona:plan`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{E3}}

```markdown
## Outcome
The composer adds valid tasks to the top of the list and refuses empty titles with a visible, associated error.

## Agent intent
persona: plan
wave: 1
parallel_group: capture
depends_on: ["[S1] Running app renders an Astryx-themed frame with a header and a main region", "[S2] Every feature shares one Task type and one storage interface"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E3
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T3.2 and T3.4 have verdict pass
- Acceptance checked on main after lanes list and data merge
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Typing only a title and pressing Enter adds the task at the top of the list with priority none and no due date
- [ ] With all fields filled, the new row shows the notes, priority, and due date as entered
- [ ] An empty or spaces-only title adds nothing and shows the required-title error next to Title
- [ ] After adding, the form is empty and focus is back in Title

## Astryx
none
```

### [T3.1] Architect: capture contract
- labels: `type:task`, `persona:architect`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S3}}

```markdown
## Outcome
docs/contracts/capture.md fixes the composer: field order and labels, defaults (priority none, no due date), submit triggers (Add task button, Enter in Title), trimming, the required-title message and how it is tied to the field, reset and focus after submit, the Astryx components with printed specifiers and props, and the test list for T3.3.

## Agent intent
persona: architect
wave: 1
parallel_group: capture
depends_on: ["[T1.8] Review: app frame with header and main region", "[T2.4] Review: compose TaskScreen from contract stubs"]
owns: ["docs/contracts/capture.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Change src/tasks/types.ts or src/tasks/storage.ts; a contract gap gets label `blocked` and goes to plan
inputs:
- docs/contracts/tasks.md (from T2.1)
- Parent story S3
- `npx @astryxdesign/cli component <Name>` for each component under Astryx
- `npx @astryxdesign/cli search <term> --type component` if a catalog name does not resolve
handoff_to: review
done_when:
- docs/contracts/capture.md has one section per build task in this lane: components (specifier + exact props), behavior rules, UI copy, named test list
- Due date uses the Astryx DateInput and is stored as YYYY-MM-DD or null
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] A build agent can implement each task in this lane from tasks.md plus this doc without asking a question (reviewer reads it cold)
- [ ] Every string the user sees in this lane is written out in the doc

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
```

### [T3.2] Review: capture contract
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S3}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T3.1] Architect: capture contract, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: capture
depends_on: ["[T3.1] Architect: capture contract"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T3.1] Architect: capture contract
- Issue [T3.1] Architect: capture contract
- Every doc listed in T3.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T3.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T3.1] Architect: capture contract as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T3.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
```

### [T3.3] Build: TaskComposer form with title validation
- labels: `type:task`, `persona:build`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S3}}

```markdown
## Outcome
TaskComposer renders a labeled form (Title, Notes, Priority, Due date) and calls onAdd once with a trimmed TaskDraft when Title has text. An empty or whitespace-only title never calls onAdd and shows the contract's error on the Title field.

## Agent intent
persona: build
wave: 1
parallel_group: capture
depends_on: ["[T3.2] Review: capture contract"]
owns: ["src/tasks/TaskComposer.tsx", "src/tasks/TaskComposer.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Generate ids or timestamps (the store does)
- Persist anything
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/capture.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- TaskComposer.test.tsx passes: rejects empty title; rejects whitespace-only title; submits trimmed title with defaults (notes '', priority 'none', dueDate null); submits all four fields; Enter in Title submits
- Test: after a successful add, fields reset and focus returns to Title (id COMPOSER_TITLE_ID)
- Test: the error clears once Title has text; every control has an accessible name
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- TaskComposer` passes every test named in capture.md
- [ ] In `npm run dev`, pressing Add task with an empty Title shows the required-title error under the field
- [ ] Each field has a visible label; Tab order is Title, Notes, Priority, Due date, Add task

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
```

### [T3.4] Review: TaskComposer form with title validation
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S3}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T3.3] Build: TaskComposer form with title validation, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: capture
depends_on: ["[T3.3] Build: TaskComposer form with title validation"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T3.3] Build: TaskComposer form with title validation
- Issue [T3.3] Build: TaskComposer form with title validation
- Every doc listed in T3.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T3.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T3.3] Build: TaskComposer form with title validation as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T3.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
```

### [E4] View and complete tasks
- labels: `type:epic`, `persona:plan`, `wave:1`, `area:tasks`, `priority:p0`
- parent: none

```markdown
## Outcome
A user sees all tasks in one list, incomplete first and newest first within each group, and can mark any task complete or incomplete.

## Agent intent
persona: plan
wave: 1
parallel_group: list
depends_on: ["[E1] App shell and design system", "[E2] Task contract"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S4 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] S4 Acceptance checked on main

## Astryx
none
```

### [S4] User sees incomplete tasks first and can toggle complete
- labels: `type:story`, `persona:plan`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{E4}}

```markdown
## Outcome
The list orders tasks predictably and each row toggles completion from mouse or keyboard. Row and list container are separate files with separate owners.

## Agent intent
persona: plan
wave: 1
parallel_group: list
depends_on: ["[S1] Running app renders an Astryx-themed frame with a header and a main region", "[S2] Every feature shares one Task type and one storage interface"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E4
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T4.2, T4.4, T4.6 have verdict pass
- Acceptance checked on main after lanes capture and data merge
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Incomplete tasks appear above completed ones; within each group the newest is first
- [ ] Checking a task moves it into the completed group; unchecking moves it back
- [ ] Each checkbox is reachable with Tab and toggles with Space
- [ ] Rows show title, notes when present, a priority badge unless priority is none, and the due date when set

## Astryx
none
```

### [T4.1] Architect: list and row contract
- labels: `type:task`, `persona:architect`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S4}}

```markdown
## Outcome
docs/contracts/list.md fixes the row and list: row layout and copy (checkbox named by the title; Edit and Delete names include the title), completed styling using only Astryx props, priority-to-Badge mapping, due-date formatting from YYYY-MM-DD parts as a local date, list semantics (list and listitem roles, list named 'Tasks'), the sort rule, and test lists for T4.3 and T4.5.

## Agent intent
persona: architect
wave: 1
parallel_group: list
depends_on: ["[T1.8] Review: app frame with header and main region", "[T2.4] Review: compose TaskScreen from contract stubs"]
owns: ["docs/contracts/list.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Change src/tasks/types.ts or src/tasks/storage.ts; a contract gap gets label `blocked` and goes to plan
inputs:
- docs/contracts/tasks.md (from T2.1)
- Parent story S4
- `npx @astryxdesign/cli component <Name>` for each component under Astryx
- `npx @astryxdesign/cli search <term> --type component` if a catalog name does not resolve
handoff_to: review
done_when:
- docs/contracts/list.md has one section per build task in this lane: components (specifier + exact props), behavior rules, UI copy, named test list
- States that dates are never parsed with `new Date('YYYY-MM-DD')` (that is UTC and shifts the day)
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] A build agent can implement each task in this lane from tasks.md plus this doc without asking a question (reviewer reads it cold)
- [ ] Every string the user sees in this lane is written out in the doc

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- CheckboxInput
- Text
- Badge
- Button
- IconButton
- Stack
- List
```

### [T4.2] Review: list and row contract
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S4}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T4.1] Architect: list and row contract, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: list
depends_on: ["[T4.1] Architect: list and row contract"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T4.1] Architect: list and row contract
- Issue [T4.1] Architect: list and row contract
- Every doc listed in T4.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T4.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T4.1] Architect: list and row contract as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T4.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- CheckboxInput
- Text
- Badge
- Button
- IconButton
- Stack
- List
```

### [T4.3] Build: TaskRow
- labels: `type:task`, `persona:build`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S4}}

```markdown
## Outcome
TaskRow shows one task: a checkbox named by the title that reflects completed, the title, notes when present, a priority badge unless priority is none, the due date when set, and Edit and Delete buttons whose accessible names include the title. It only reports intent through onToggle, onEdit, and onDelete.

## Agent intent
persona: build
wave: 1
parallel_group: list
depends_on: ["[T4.2] Review: list and row contract"]
owns: ["src/tasks/TaskRow.tsx", "src/tasks/TaskRow.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Open dialogs or change data itself
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/list.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- TaskRow.test.tsx passes: checkbox checked state follows completed; click and Space call onToggle(id); Edit calls onEdit(task); Delete calls onDelete(task)
- Test: no badge for priority none; notes hidden when empty
- Test: with process.env.TZ = 'America/Los_Angeles', dueDate '2026-03-01' renders as March 1
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- TaskRow` passes every test named in list.md
- [ ] On main after lanes capture and data merge, a completed row looks visibly different from an incomplete one

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- CheckboxInput
- Text
- Badge
- Button
- IconButton
- Stack
```

### [T4.4] Review: TaskRow
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S4}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T4.3] Build: TaskRow, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: list
depends_on: ["[T4.3] Build: TaskRow"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T4.3] Build: TaskRow
- Issue [T4.3] Build: TaskRow
- Every doc listed in T4.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T4.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T4.3] Build: TaskRow as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T4.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- CheckboxInput
- Text
- Badge
- Button
- IconButton
- Stack
```

### [T4.5] Build: TaskList container and sort order
- labels: `type:task`, `persona:build`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S4}}

```markdown
## Outcome
TaskList renders the tasks it gets as a list named 'Tasks' of TaskRow items in sortTasks order: incomplete newest first, then completed newest first, by createdAt. sortTasks is pure and never mutates its input.

## Agent intent
persona: build
wave: 1
parallel_group: list
depends_on: ["[T4.2] Review: list and row contract"]
owns: ["src/tasks/TaskList.tsx", "src/tasks/TaskList.test.tsx", "src/tasks/sortTasks.ts", "src/tasks/sortTasks.test.ts"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Filter tasks (T6.3 owns filtering)
- Depend on T4.3's implementation: tests mock ./TaskRow with vi.mock
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/list.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- sortTasks.test.ts passes: incomplete before completed; newest first in each group; equal createdAt keeps input order; input array unchanged
- TaskList.test.tsx passes: one listitem per task in sorted order; onToggle, onEdit, onDelete forwarded to each row; list accessible name 'Tasks'
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- sortTasks TaskList` passes
- [ ] On main after lanes capture and data merge, completing a task moves it below every incomplete task

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- List
- Stack
```

### [T4.6] Review: TaskList container and sort order
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p0`
- parent: {{S4}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T4.5] Build: TaskList container and sort order, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: list
depends_on: ["[T4.5] Build: TaskList container and sort order"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T4.5] Build: TaskList container and sort order
- Issue [T4.5] Build: TaskList container and sort order
- Every doc listed in T4.5's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T4.5
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T4.5] Build: TaskList container and sort order as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T4.5's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- List
- Stack
```

### [E5] Edit and delete
- labels: `type:epic`, `persona:plan`, `wave:1`, `area:tasks`, `priority:p1`
- parent: none

```markdown
## Outcome
A user can change a task's title, notes, priority, and due date in a dialog, and delete a task only after confirming.

## Agent intent
persona: plan
wave: 1
parallel_group: mutate
depends_on: ["[E1] App shell and design system", "[E2] Task contract"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S5 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] S5 Acceptance checked on main

## Astryx
none
```

### [S5] User edits fields and deletes with confirmation
- labels: `type:story`, `persona:plan`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{E5}}

```markdown
## Outcome
Edit and delete live in two dialog files of their own. The row already raises onEdit and onDelete per the contract, so this lane never touches the row and starts from the contract alone.

## Agent intent
persona: plan
wave: 1
parallel_group: mutate
depends_on: ["[S1] Running app renders an Astryx-themed frame with a header and a main region", "[S2] Every feature shares one Task type and one storage interface"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E5
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T5.2, T5.4, T5.6 have verdict pass
- Acceptance checked on main after lanes capture, list and data merge
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Edit opens a dialog prefilled with the task; Save updates the row; Cancel or Escape changes nothing
- [ ] Saving with an empty title is blocked with the required-title error
- [ ] Delete opens a dialog naming the task; Cancel keeps it; Delete removes it
- [ ] Closing the edit dialog returns focus to the row's Edit button; after a delete, focus is in the composer's Title

## Astryx
none
```

### [T5.1] Architect: edit and delete dialogs contract
- labels: `type:task`, `persona:architect`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{S5}}

```markdown
## Outcome
docs/contracts/mutate.md fixes both dialogs: titles, field labels (same copy as the composer), button copy, validation, initial focus (first field for edit; Cancel for delete), Escape behavior, focus return on close, the Astryx components with printed specifiers and props, and test lists for T5.3 and T5.5.

## Agent intent
persona: architect
wave: 1
parallel_group: mutate
depends_on: ["[T1.8] Review: app frame with header and main region", "[T2.4] Review: compose TaskScreen from contract stubs"]
owns: ["docs/contracts/mutate.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Change src/tasks/types.ts or src/tasks/storage.ts; a contract gap gets label `blocked` and goes to plan
inputs:
- docs/contracts/tasks.md (from T2.1)
- Parent story S5
- `npx @astryxdesign/cli component <Name>` for each component under Astryx
- `npx @astryxdesign/cli search <term> --type component` if a catalog name does not resolve
handoff_to: review
done_when:
- docs/contracts/mutate.md has one section per build task in this lane: components (specifier + exact props), behavior rules, UI copy, named test list
- Says whether Astryx Dialog restores focus on close (from CLI docs); if not, the dialog restores focus itself, except after delete confirm, where TaskScreen moves focus to Title
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] A build agent can implement each task in this lane from tasks.md plus this doc without asking a question (reviewer reads it cold)
- [ ] Every string the user sees in this lane is written out in the doc

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Dialog
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
- AlertDialog
```

### [T5.2] Review: edit and delete dialogs contract
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{S5}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T5.1] Architect: edit and delete dialogs contract, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: mutate
depends_on: ["[T5.1] Architect: edit and delete dialogs contract"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T5.1] Architect: edit and delete dialogs contract
- Issue [T5.1] Architect: edit and delete dialogs contract
- Every doc listed in T5.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T5.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T5.1] Architect: edit and delete dialogs contract as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T5.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Dialog
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
- AlertDialog
```

### [T5.3] Build: EditTaskDialog
- labels: `type:task`, `persona:build`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{S5}}

```markdown
## Outcome
When task is non-null, a dialog named 'Edit task' opens with Title, Notes, Priority, and Due date prefilled. Save calls onSave(id, patch) with trimmed values, then onClose. An empty title blocks Save with the contract error. Cancel or Escape calls only onClose.

## Agent intent
persona: build
wave: 1
parallel_group: mutate
depends_on: ["[T5.2] Review: edit and delete dialogs contract"]
owns: ["src/tasks/EditTaskDialog.tsx", "src/tasks/EditTaskDialog.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Edit TaskRow or TaskScreen
- Import from TaskComposer (separate lanes; the duplicated fields are accepted)
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/mutate.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- EditTaskDialog.test.tsx passes: not rendered when task is null; fields prefilled; empty title blocks save and shows the error
- Test: Save calls onSave(id, { title, notes, priority, dueDate }) trimmed, then onClose
- Test: Cancel and Escape each call onClose only; focus moves into the dialog on open
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- EditTaskDialog` passes every test named in mutate.md
- [ ] On main after lane list merges: Escape closes the dialog without changes and focus returns to the Edit button

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Dialog
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
```

### [T5.4] Review: EditTaskDialog
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{S5}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T5.3] Build: EditTaskDialog, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: mutate
depends_on: ["[T5.3] Build: EditTaskDialog"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T5.3] Build: EditTaskDialog
- Issue [T5.3] Build: EditTaskDialog
- Every doc listed in T5.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T5.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T5.3] Build: EditTaskDialog as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T5.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Dialog
- FormLayout
- Field
- TextInput
- TextArea
- Selector
- DateInput
- Button
```

### [T5.5] Build: ConfirmDeleteDialog
- labels: `type:task`, `persona:build`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{S5}}

```markdown
## Outcome
When task is non-null, a confirm dialog names the task and offers Delete and Cancel. Delete calls onConfirm(id) once. Cancel or Escape calls only onClose. Initial focus is on Cancel.

## Agent intent
persona: build
wave: 1
parallel_group: mutate
depends_on: ["[T5.2] Review: edit and delete dialogs contract"]
owns: ["src/tasks/ConfirmDeleteDialog.tsx", "src/tasks/ConfirmDeleteDialog.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Restore focus after Delete (TaskScreen moves focus to Title)
- Edit TaskRow or TaskScreen
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/mutate.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- ConfirmDeleteDialog.test.tsx passes: not rendered when task is null; shows the task title; Delete calls onConfirm(id) exactly once
- Test: Cancel and Escape each call onClose only; initial focus is on Cancel
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- ConfirmDeleteDialog` passes every test named in mutate.md
- [ ] On main after lane list merges: Cancel keeps the task; Delete removes it and focus lands in Title

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- AlertDialog
- Dialog
- Text
- Button
```

### [T5.6] Review: ConfirmDeleteDialog
- labels: `type:task`, `persona:review`, `wave:1`, `area:tasks`, `priority:p1`
- parent: {{S5}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T5.5] Build: ConfirmDeleteDialog, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: mutate
depends_on: ["[T5.5] Build: ConfirmDeleteDialog"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T5.5] Build: ConfirmDeleteDialog
- Issue [T5.5] Build: ConfirmDeleteDialog
- Every doc listed in T5.5's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T5.5
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T5.5] Build: ConfirmDeleteDialog as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T5.5's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- AlertDialog
- Dialog
- Text
- Button
```

### [E6] Filter, search, and clear completed
- labels: `type:epic`, `persona:plan`, `wave:1`, `area:filters`, `priority:p1`
- parent: none

```markdown
## Outcome
A user narrows the list by status, priority, and title search, and removes all completed tasks at once without touching active ones.

## Agent intent
persona: plan
wave: 1
parallel_group: query
depends_on: ["[E1] App shell and design system", "[E2] Task contract"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S6 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] S6 Acceptance checked on main

## Astryx
none
```

### [S6] User filters, searches, and clears completed tasks without losing active ones
- labels: `type:story`, `persona:plan`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{E6}}

```markdown
## Outcome
Filtering is one pure function; the controls are two components; clearing calls the store's clearCompleted, which removes only completed tasks.

## Agent intent
persona: plan
wave: 1
parallel_group: query
depends_on: ["[S1] Running app renders an Astryx-themed frame with a header and a main region", "[S2] Every feature shares one Task type and one storage interface"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E6
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T6.2, T6.4, T6.6, T6.8 have verdict pass
- Acceptance checked on main after every wave-1 lane merges
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Active shows only incomplete tasks, Completed only completed, All both
- [ ] The priority filter combines with the status filter and search
- [ ] Search matches title substrings, ignoring case; notes are not searched
- [ ] 'Clear completed (n)' removes only completed tasks, keeps every active one, and is disabled when n is 0
- [ ] When filters hide every task, a no-matches state with a Clear filters button appears

## Astryx
none
```

### [T6.1] Architect: filter, search, and clear contract
- labels: `type:task`, `persona:architect`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
docs/contracts/query.md fixes the rules (status AND priority AND search; search is trimmed, case-insensitive, title only), control labels and options, the 'Clear completed (n)' copy and disabled rule, the decision of no confirm dialog for clear completed (the count is in the label), the Astryx components with printed specifiers and props, and test lists for T6.3, T6.5 and T6.7.

## Agent intent
persona: architect
wave: 1
parallel_group: query
depends_on: ["[T1.8] Review: app frame with header and main region", "[T2.4] Review: compose TaskScreen from contract stubs"]
owns: ["docs/contracts/query.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Change src/tasks/types.ts or src/tasks/storage.ts; a contract gap gets label `blocked` and goes to plan
inputs:
- docs/contracts/tasks.md (from T2.1)
- Parent story S6
- `npx @astryxdesign/cli component <Name>` for each component under Astryx
- `npx @astryxdesign/cli search <term> --type component` if a catalog name does not resolve
handoff_to: review
done_when:
- docs/contracts/query.md has one section per build task in this lane: components (specifier + exact props), behavior rules, UI copy, named test list
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] A build agent can implement each task in this lane from tasks.md plus this doc without asking a question (reviewer reads it cold)
- [ ] Every string the user sees in this lane is written out in the doc

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- SegmentedControl
- Selector
- Button
- Toolbar
- Stack
- TextInput
- Field
```

### [T6.2] Review: filter, search, and clear contract
- labels: `type:task`, `persona:review`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T6.1] Architect: filter, search, and clear contract, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: query
depends_on: ["[T6.1] Architect: filter, search, and clear contract"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T6.1] Architect: filter, search, and clear contract
- Issue [T6.1] Architect: filter, search, and clear contract
- Every doc listed in T6.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T6.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T6.1] Architect: filter, search, and clear contract as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T6.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- SegmentedControl
- Selector
- Button
- Toolbar
- Stack
- TextInput
- Field
```

### [T6.3] Build: filterTasks pure function
- labels: `type:task`, `persona:build`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
filterTasks(tasks, query) returns the tasks that match status AND priority AND search, in input order, without mutating the input. Search is a trimmed, case-insensitive substring match on title only; empty search matches everything.

## Agent intent
persona: build
wave: 1
parallel_group: query
depends_on: ["[T6.2] Review: filter, search, and clear contract"]
owns: ["src/tasks/filterTasks.ts", "src/tasks/filterTasks.test.ts"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Sort (TaskList does)
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/query.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- filterTasks.test.ts passes: status all/active/completed; priority 'any' and each value; search case-insensitive, trimmed, title only (notes ignored); combined filters; DEFAULT_QUERY returns all; input unchanged
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- filterTasks` passes every case named in query.md
- [ ] On main after every wave-1 lane merges: Active + High + 'milk' shows only incomplete high-priority tasks whose title contains 'milk'

## Astryx
none
```

### [T6.4] Review: filterTasks pure function
- labels: `type:task`, `persona:review`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T6.3] Build: filterTasks pure function, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: query
depends_on: ["[T6.3] Build: filterTasks pure function"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T6.3] Build: filterTasks pure function
- Issue [T6.3] Build: filterTasks pure function
- Every doc listed in T6.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T6.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T6.3] Build: filterTasks pure function as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T6.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
none
```

### [T6.5] Build: FilterBar with status, priority, and clear completed
- labels: `type:task`, `persona:build`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
FilterBar renders a labeled status control (All, Active, Completed), a labeled priority select (Any, None, Low, Medium, High), and a 'Clear completed (n)' button. A change calls onChange with a full TaskQuery that keeps search as it was. Clear is disabled when completedCount is 0 and otherwise calls onClearCompleted.

## Agent intent
persona: build
wave: 1
parallel_group: query
depends_on: ["[T6.2] Review: filter, search, and clear contract"]
owns: ["src/tasks/FilterBar.tsx", "src/tasks/FilterBar.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Render the search input (T6.7 owns it)
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/query.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- FilterBar.test.tsx passes: controls reflect query; status change calls onChange({ ...query, status }); priority likewise; search is preserved
- Test: Clear completed is disabled at 0, shows the count, and calls onClearCompleted; every control has an accessible name
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- FilterBar` passes every test named in query.md
- [ ] Status and priority controls are reachable by Tab and operable with the keys the Astryx component documents

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- SegmentedControl
- Selector
- Button
- Toolbar
- Stack
```

### [T6.6] Review: FilterBar with status, priority, and clear completed
- labels: `type:task`, `persona:review`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T6.5] Build: FilterBar with status, priority, and clear completed, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: query
depends_on: ["[T6.5] Build: FilterBar with status, priority, and clear completed"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T6.5] Build: FilterBar with status, priority, and clear completed
- Issue [T6.5] Build: FilterBar with status, priority, and clear completed
- Every doc listed in T6.5's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T6.5
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T6.5] Build: FilterBar with status, priority, and clear completed as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T6.5's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- SegmentedControl
- Selector
- Button
- Toolbar
- Stack
```

### [T6.7] Build: SearchField
- labels: `type:task`, `persona:build`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
SearchField is one labeled text input ('Search tasks') that shows value and calls onChange on every edit; clearing it calls onChange('').

## Agent intent
persona: build
wave: 1
parallel_group: query
depends_on: ["[T6.2] Review: filter, search, and clear contract"]
owns: ["src/tasks/SearchField.tsx", "src/tasks/SearchField.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Debounce or filter (filterTasks does)
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/query.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- SearchField.test.tsx passes: accessible name 'Search tasks'; shows value; typing calls onChange; clearing calls onChange('')
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- SearchField` passes
- [ ] On main after every wave-1 lane merges: typing in Search tasks narrows the list as you type

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- TextInput
- Field
```

### [T6.8] Review: SearchField
- labels: `type:task`, `persona:review`, `wave:1`, `area:filters`, `priority:p1`
- parent: {{S6}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T6.7] Build: SearchField, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: query
depends_on: ["[T6.7] Build: SearchField"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T6.7] Build: SearchField
- Issue [T6.7] Build: SearchField
- Every doc listed in T6.7's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T6.7
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T6.7] Build: SearchField as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T6.7's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- TextInput
- Field
```

### [E7] Persistence and failure states
- labels: `type:epic`, `persona:plan`, `wave:1`, `area:persistence`, `priority:p0`
- parent: none

```markdown
## Outcome
Tasks survive reloads through localStorage. Loading, empty, no-matches, and save-error states are deliberate, and a failed write never drops what the user sees.

## Agent intent
persona: plan
wave: 1
parallel_group: data
depends_on: ["[E1] App shell and design system", "[E2] Task contract"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S7 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] S7 Acceptance checked on main

## Astryx
none
```

### [S7] Tasks survive reload. A failed write shows an error and does not drop the in-memory list
- labels: `type:story`, `persona:plan`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{E7}}

```markdown
## Outcome
A localStorage adapter, an in-memory store hook, and the status views ship as four separate files, each with its own owner.

## Agent intent
persona: plan
wave: 1
parallel_group: data
depends_on: ["[S1] Running app renders an Astryx-themed frame with a header and a main region", "[S2] Every feature shares one Task type and one storage interface"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E7
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T7.2, T7.4, T7.6, T7.8, T7.10 have verdict pass
- Acceptance checked on main after lanes capture and list merge
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Add 3 tasks, complete 1, reload: the same 3 tasks with the same completion
- [ ] In devtools, save the original then set `Storage.prototype.setItem = () => { throw new DOMException('full', 'QuotaExceededError') }`; adding a task still shows it, plus an error banner saying changes were not saved
- [ ] Restore setItem; the next change saves and the banner disappears
- [ ] With empty storage, first load shows a brief loading state, then 'No tasks yet'
- [ ] Set the storage key to `{` in devtools and reload: the app shows the empty state and the raw value is kept under the backup key

## Astryx
none
```

### [T7.1] Architect: persistence and status-views contract
- labels: `type:task`, `persona:architect`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
docs/contracts/data.md fixes load semantics (missing key gives []; unreadable data is copied to `${STORAGE_KEY}.backup` before returning the valid entries; storage access that throws gives []), save semantics (JSON; errors propagate), store semantics (loading then ready; no save before the first load completes; save after each change; a failed save keeps the list and sets saveError; the next success clears it; ids from crypto.randomUUID), all copy for loading, empty, no-matches and save-error, the Astryx components with printed specifiers and props, and test lists for T7.3, T7.5, T7.7 and T7.9.

## Agent intent
persona: architect
wave: 1
parallel_group: data
depends_on: ["[T1.8] Review: app frame with header and main region", "[T2.4] Review: compose TaskScreen from contract stubs"]
owns: ["docs/contracts/data.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
- Change src/tasks/types.ts or src/tasks/storage.ts; a contract gap gets label `blocked` and goes to plan
inputs:
- docs/contracts/tasks.md (from T2.1)
- Parent story S7
- `npx @astryxdesign/cli component <Name>` for each component under Astryx
- `npx @astryxdesign/cli search <term> --type component` if a catalog name does not resolve
handoff_to: review
done_when:
- docs/contracts/data.md has one section per build task in this lane: components (specifier + exact props), behavior rules, UI copy, named test list
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] A build agent can implement each task in this lane from tasks.md plus this doc without asking a question (reviewer reads it cold)
- [ ] Every string the user sees in this lane is written out in the doc

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- EmptyState
- Spinner
- Button
- Banner
```

### [T7.2] Review: persistence and status-views contract
- labels: `type:task`, `persona:review`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T7.1] Architect: persistence and status-views contract, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: data
depends_on: ["[T7.1] Architect: persistence and status-views contract"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T7.1] Architect: persistence and status-views contract
- Issue [T7.1] Architect: persistence and status-views contract
- Every doc listed in T7.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T7.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T7.1] Architect: persistence and status-views contract as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T7.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- EmptyState
- Spinner
- Button
- Banner
```

### [T7.3] Build: localStorage adapter
- labels: `type:task`, `persona:build`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
createLocalTaskStorage(storage?) implements TaskStorage over Web Storage under STORAGE_KEY, and localTaskStorage is the default instance. Missing data loads as []. Unreadable data is copied to the backup key before load returns the valid entries, so nothing saved is silently destroyed. save writes JSON and lets write errors (quota, security) reach the caller.

## Agent intent
persona: build
wave: 1
parallel_group: data
depends_on: ["[T7.2] Review: persistence and status-views contract"]
owns: ["src/tasks/localTaskStorage.ts", "src/tasks/localTaskStorage.test.ts"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Swallow save errors (the store needs them)
- Touch window.localStorage at module load; access it lazily so blocked storage cannot crash the import
- Add schema migration logic (v1 only)
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/data.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- localTaskStorage.test.ts passes with a fake Storage: missing key gives []; save then load round-trips
- Test: invalid JSON puts the raw value in the backup key and returns []; an array with one bad entry is backed up and returns the valid entries
- Test: getItem that throws gives []; setItem that throws QuotaExceededError makes save throw
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- localTaskStorage` passes every test named in data.md
- [ ] On main after lane capture merges: after adding a task, devtools Local Storage shows STORAGE_KEY holding a JSON array

## Astryx
none
```

### [T7.4] Review: localStorage adapter
- labels: `type:task`, `persona:review`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T7.3] Build: localStorage adapter, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: data
depends_on: ["[T7.3] Build: localStorage adapter"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T7.3] Build: localStorage adapter
- Issue [T7.3] Build: localStorage adapter
- Every doc listed in T7.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T7.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T7.3] Build: localStorage adapter as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T7.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
none
```

### [T7.5] Build: useTasks store hook
- labels: `type:task`, `persona:build`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
useTasks(storage) holds the in-memory list. It starts at status 'loading', reads storage once, then becomes 'ready'. add, update, remove, and clearCompleted change the list and then save. It never saves before the first load completes. A failed save keeps the in-memory list and sets saveError; the next successful save clears it.

## Agent intent
persona: build
wave: 1
parallel_group: data
depends_on: ["[T7.2] Review: persistence and status-views contract"]
owns: ["src/tasks/useTasks.ts", "src/tasks/useTasks.test.ts"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Import localTaskStorage (storage is injected; TaskScreen passes it)
- Sort or filter
- Retry saves on a timer
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/data.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- useTasks.test.ts (renderHook, fake TaskStorage) passes: starts loading then ready with loaded tasks; save is not called before load completes
- Test: add trims the title, ignores an empty title, and sets id, createdAt, completed false, plus notes, priority and dueDate from the draft
- Test: update changes only the matching id; remove deletes one; clearCompleted removes only completed tasks and keeps every active one
- Test: save gets the new list after each change; when save throws, tasks stay in memory and saveError is set; the next successful change clears saveError
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- useTasks` passes every test named in data.md
- [ ] On main after lanes capture and list merge: S7's reload check passes

## Astryx
none
```

### [T7.6] Review: useTasks store hook
- labels: `type:task`, `persona:review`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T7.5] Build: useTasks store hook, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: data
depends_on: ["[T7.5] Build: useTasks store hook"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T7.5] Build: useTasks store hook
- Issue [T7.5] Build: useTasks store hook
- Every doc listed in T7.5's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T7.5
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T7.5] Build: useTasks store hook as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T7.5's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
none
```

### [T7.7] Build: loading and empty views
- labels: `type:task`, `persona:build`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
StatusViews exports LoadingState (an Astryx spinner announced as loading tasks) and EmptyState with two variants: 'no-tasks' (contract copy inviting the first task) and 'no-matches' (contract copy plus a Clear filters button that calls onClearFilters).

## Agent intent
persona: build
wave: 1
parallel_group: data
depends_on: ["[T7.2] Review: persistence and status-views contract"]
owns: ["src/tasks/StatusViews.tsx", "src/tasks/StatusViews.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Decide which state to show (TaskScreen does)
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/data.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- StatusViews.test.tsx passes: LoadingState has role status with the contract name; no-tasks shows its copy and no button; no-matches shows its copy and Clear filters calls onClearFilters
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- StatusViews` passes
- [ ] On main with empty storage: the first load shows 'No tasks yet'; filtering to Completed with nothing completed shows the no-matches state, and Clear filters brings the list back

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- EmptyState
- Spinner
- Button
```

### [T7.8] Review: loading and empty views
- labels: `type:task`, `persona:review`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T7.7] Build: loading and empty views, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: data
depends_on: ["[T7.7] Build: loading and empty views"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T7.7] Build: loading and empty views
- Issue [T7.7] Build: loading and empty views
- Every doc listed in T7.7's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T7.7
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T7.7] Build: loading and empty views as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T7.7's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- EmptyState
- Spinner
- Button
```

### [T7.9] Build: save-error banner
- labels: `type:task`, `persona:build`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
SaveErrorBanner renders nothing when message is null. Otherwise it renders an Astryx banner, announced as an alert, telling the user that changes are kept in this tab but were not saved.

## Agent intent
persona: build
wave: 1
parallel_group: data
depends_on: ["[T7.2] Review: persistence and status-views contract"]
owns: ["src/tasks/SaveErrorBanner.tsx", "src/tasks/SaveErrorBanner.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Offer a retry button or dismiss that hides an unresolved error
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/data.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- SaveErrorBanner.test.tsx passes: null renders nothing; a message renders role alert with the contract copy; rerendering with null removes it
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- SaveErrorBanner` passes
- [ ] On main with setItem forced to throw in devtools: adding a task shows the banner and the task stays in the list

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Banner
```

### [T7.10] Review: save-error banner
- labels: `type:task`, `persona:review`, `wave:1`, `area:persistence`, `priority:p0`
- parent: {{S7}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T7.9] Build: save-error banner, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 1
parallel_group: data
depends_on: ["[T7.9] Build: save-error banner"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T7.9] Build: save-error banner
- Issue [T7.9] Build: save-error banner
- Every doc listed in T7.9's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T7.9
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T7.9] Build: save-error banner as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T7.9's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- Banner
```

### [E8] Accessible, responsive, testable app
- labels: `type:epic`, `persona:plan`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: none

```markdown
## Outcome
The whole app works from the keyboard alone with visible focus and a name on every control, holds up at phone width, and one Playwright path proves add, complete, reload, still complete.

## Agent intent
persona: plan
wave: 2
parallel_group: a11y
depends_on: ["[E3] Capture a task", "[E4] View and complete tasks", "[E5] Edit and delete", "[E6] Filter, search, and clear completed", "[E7] Persistence and failure states"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close before every child story is closed
inputs:
- docs/backlog/BACKLOG.md
- https://astryx.atmeta.com/
handoff_to: done
done_when:
- S8 is closed
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] S8 Acceptance checked on main

## Astryx
none
```

### [S8] Keyboard-only use, labeled controls, focus, empty state, and one Playwright path: add, complete, reload, still complete
- labels: `type:story`, `persona:plan`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{E8}}

```markdown
## Outcome
Wave 2 adds tests only. A defect a test finds is fixed by the owner of the failing file through a fix task, so ownership stays clean.

## Agent intent
persona: plan
wave: 2
parallel_group: a11y
depends_on: ["[S3] User adds a task with title, notes, priority, and due date, and sees it appear", "[S4] User sees incomplete tasks first and can toggle complete", "[S5] User edits fields and deletes with confirmation", "[S6] User filters, searches, and clears completed tasks without losing active ones", "[S7] Tasks survive reload. A failed write shows an error and does not drop the in-memory list"]
owns: ["none"]
must_not:
- Contain code or open a PR
- Close until every child task's review verdict is pass and the Acceptance below is checked on main
inputs:
- Parent epic E8
- docs/backlog/BACKLOG.md
handoff_to: architect
done_when:
- T8.2, T8.4, T8.6 have verdict pass
- Acceptance checked on main
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Without a mouse, add, complete, edit, delete, filter, search, and clear completed all work with Tab, Shift+Tab, Enter, Space, and Escape
- [ ] Every focused control shows a visible focus indicator
- [ ] The devtools accessibility tree shows no unnamed buttons, inputs, checkboxes, or selects
- [ ] First visit with empty storage shows the empty state
- [ ] At 375px width there is no horizontal scroll and every control is usable
- [ ] `npm run test:e2e` passes in the desktop and mobile projects

## Astryx
none
```

### [T8.1] Architect: accessibility, responsive, and e2e contract
- labels: `type:task`, `persona:architect`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{S8}}

```markdown
## Outcome
docs/contracts/a11y.md fixes the keyboard map (tab order across composer, search, filters, clear completed, and rows), the keys each control answers to, focus rules (dialog open, close, after add, after delete), the inventory of required accessible names, the visible-focus rule, the 375px no-horizontal-scroll rule, the test list for T8.3, and the exact steps for T8.5.

## Agent intent
persona: architect
wave: 2
parallel_group: a11y
depends_on: ["[T3.4] Review: TaskComposer form with title validation", "[T4.4] Review: TaskRow", "[T4.6] Review: TaskList container and sort order", "[T5.4] Review: EditTaskDialog", "[T5.6] Review: ConfirmDeleteDialog", "[T6.4] Review: filterTasks pure function", "[T6.6] Review: FilterBar with status, priority, and clear completed", "[T6.8] Review: SearchField", "[T7.4] Review: localStorage adapter", "[T7.6] Review: useTasks store hook", "[T7.8] Review: loading and empty views", "[T7.10] Review: save-error banner"]
owns: ["docs/contracts/a11y.md"]
must_not:
- Implement feature behavior; stubs (if any) only return null, a copy of their input, or inert values
- Edit any path not in owns
- Name an Astryx component, import specifier, or prop the CLI did not print
inputs:
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/capture.md, list.md, mutate.md, query.md, data.md
- The running app on main
- `npx @astryxdesign/cli component <Name>` keyboard notes for each component in use
handoff_to: review
done_when:
- a11y.md lists every interactive control on screen with its expected accessible name and keys
- a11y.md gives T8.3 one named test per flow and T8.5 a step list
- One branch, one PR; body says `Closes #<this issue>` and pastes the CLI output it relied on
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Following a11y.md by hand on main finds no step that contradicts the lane contracts
- [ ] Every control in the inventory can be found on the running app

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- VisuallyHidden
```

### [T8.2] Review: accessibility, responsive, and e2e contract
- labels: `type:task`, `persona:review`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{S8}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T8.1] Architect: accessibility, responsive, and e2e contract, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 2
parallel_group: a11y
depends_on: ["[T8.1] Architect: accessibility, responsive, and e2e contract"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T8.1] Architect: accessibility, responsive, and e2e contract
- Issue [T8.1] Architect: accessibility, responsive, and e2e contract
- Every doc listed in T8.1's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T8.1
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T8.1] Architect: accessibility, responsive, and e2e contract as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T8.1's owns
- [ ] Every component named in the contract was confirmed with the CLI; recorded specifiers and props match its output
- [ ] Stubs (if any) contain no feature behavior and the contract matches docs/backlog/BACKLOG.md section 4

## Astryx
Components to confirm with `npx @astryxdesign/cli component <Name>` before use:
- VisuallyHidden
```

### [T8.3] Build: keyboard-only and label integration tests
- labels: `type:task`, `persona:build`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{S8}}

```markdown
## Outcome
One Vitest suite renders <App /> and drives it with user-event keyboard input only, through every flow in docs/contracts/a11y.md, and asserts every interactive element has an accessible name. When a check fails because of another file, that file's owner gets a fix task.

## Agent intent
persona: build
wave: 2
parallel_group: a11y
depends_on: ["[T8.2] Review: accessibility, responsive, and e2e contract"]
owns: ["src/a11y.test.tsx"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Edit any product file; for a defect, open a fix task for the file's owner (section 4), label this PR `blocked`, and wait
- Use click(), fireEvent, or pointer events
- Skip tests or mark them expected-to-fail to get green
inputs:
- This issue and its parent story
- docs/contracts/tasks.md (from T2.1)
- docs/contracts/a11y.md
- `npx @astryxdesign/cli component <Name>` for every component under Astryx; import only from the entry it prints
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- src/a11y.test.tsx passes on main, starting from cleared localStorage, covering: empty state on first render; add with Tab, typing, and Enter; toggle with Space; edit with Enter, then Escape returns focus to Edit; delete by keyboard leaves focus in Title; filter, search, and clear completed by keyboard
- Test: every button, textbox, checkbox, combobox, and radio has a non-empty accessible name
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm test -- a11y` passes on main
- [ ] Each flow in a11y.md maps to one named test

## Astryx
none
```

### [T8.4] Review: keyboard-only and label integration tests
- labels: `type:task`, `persona:review`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{S8}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T8.3] Build: keyboard-only and label integration tests, based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 2
parallel_group: a11y
depends_on: ["[T8.3] Build: keyboard-only and label integration tests"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T8.3] Build: keyboard-only and label integration tests
- Issue [T8.3] Build: keyboard-only and label integration tests
- Every doc listed in T8.3's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T8.3
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T8.3] Build: keyboard-only and label integration tests as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T8.3's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
none
```

### [T8.5] Build: Playwright happy path (add, complete, reload, still complete)
- labels: `type:task`, `persona:build`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{S8}}

```markdown
## Outcome
One Playwright test runs in both projects from playwright.config.ts: from fresh storage, add a task with the keyboard, complete it, reload, and see it still completed and below incomplete tasks; the page has no horizontal overflow.

## Agent intent
persona: build
wave: 2
parallel_group: a11y
depends_on: ["[T8.2] Review: accessibility, responsive, and e2e contract"]
owns: ["e2e/happy-path.spec.ts"]
must_not:
- Edit, create, or delete any path not in owns. Need one? Add label `blocked`, comment why, hand to plan.
- Use an Astryx component, import path, or prop that `npx @astryxdesign/cli component <Name>` does not print; hard-code colors, spacing, radii, shadows, or fonts.
- Add, remove, or upgrade dependencies (package.json belongs to T1.3).
- Change src/tasks/types.ts or src/tasks/storage.ts, or add barrel/index files.
- Edit playwright.config.ts or package.json (T1.3 owns them)
- Add a second test path
- Use fixed sleeps
inputs:
- This issue and its parent story
- docs/contracts/a11y.md (e2e steps)
- README (`npx playwright install chromium`)
handoff_to: review
done_when:
- One branch, one PR; body says `Closes #<this issue>`; the diff touches only owns
- e2e/happy-path.spec.ts follows the a11y.md steps and asserts the checkbox is still checked after reload
- It asserts `document.documentElement.scrollWidth <= window.innerWidth`
- `npm run test:e2e` passes in Desktop Chrome and Pixel 7
- `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 on the PR branch
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] `npm run test:e2e` output shows one test passing in 2 projects
- [ ] Breaking persistence locally (save as a no-op) makes the test fail; revert before pushing

## Astryx
none
```

### [T8.6] Review: Playwright happy path (add, complete, reload, still complete)
- labels: `type:task`, `persona:review`, `wave:2`, `area:a11y`, `area:quality`, `priority:p0`
- parent: {{S8}}

```markdown
## Outcome
A pass or changes verdict exists on the PR for [T8.5] Build: Playwright happy path (add, complete, reload, still complete), based on its done_when, its Acceptance, and Astryx usage checked against CLI output. On pass the orchestrator merges the PR; on changes one fix task exists.

## Agent intent
persona: review
wave: 2
parallel_group: a11y
depends_on: ["[T8.5] Build: Playwright happy path (add, complete, reload, still complete)"]
owns: ["none"]
must_not:
- Push commits to, or edit files in, the PR under review
- Pass with any failing check or unmet item
- Widen scope: findings outside the target's owns become a comment for plan, not a blocker
inputs:
- The PR that closes [T8.5] Build: Playwright happy path (add, complete, reload, still complete)
- Issue [T8.5] Build: Playwright happy path (add, complete, reload, still complete)
- Every doc listed in T8.5's inputs
- `npx @astryxdesign/cli component <Name>` for each Astryx component the PR imports or names
handoff_to: done
done_when:
- Verdict comment posted as `verdict: pass` or `verdict: changes` with the failing items listed
- On pass: PR approved; orchestrator merges it and closes T8.5
- On changes: one fix task opened per the retry rule and linked here; this review re-runs on the updated PR
retry: if review returns changes, create one fix task for persona build, max 3, then persona plan

## Acceptance
- [ ] Verdict comment on the PR marks every done_when and Acceptance item of [T8.5] Build: Playwright happy path (add, complete, reload, still complete) as met or not met, with evidence (command output or screenshot)
- [ ] The diff touches only paths in T8.5's owns
- [ ] Every Astryx import in the diff matches the entry printed by `npx @astryxdesign/cli component <Name>`, and every prop used appears in that output
- [ ] No hard-coded colors, spacing, radii, shadows, or fonts
- [ ] `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build` exit 0 when the reviewer runs them on the PR branch

## Astryx
none
```

## 4. File-ownership map

Every path has exactly one build owner (checked when this file was generated: no path appears under two build tasks). Architect docs are listed for completeness. `stub` means T2.1 created the file and ownership passes when T2.2 is pass.

| Path | Owner | Wave | Note |
|---|---|---|---|
| `docs/contracts/shell.md` | T1.1 | 0 | architect doc/contract |
| `package.json` | T1.3 | 0 |  |
| `package-lock.json` | T1.3 | 0 |  |
| `index.html` | T1.3 | 0 |  |
| `vite.config.ts` | T1.3 | 0 |  |
| `tsconfig.json` | T1.3 | 0 |  |
| `tsconfig.app.json` | T1.3 | 0 |  |
| `tsconfig.node.json` | T1.3 | 0 |  |
| `eslint.config.js` | T1.3 | 0 |  |
| `.gitignore` | T1.3 | 0 |  |
| `README.md` | T1.3 | 0 |  |
| `src/vite-env.d.ts` | T1.3 | 0 |  |
| `src/test/setup.ts` | T1.3 | 0 |  |
| `playwright.config.ts` | T1.3 | 0 |  |
| CLI-generated: every file `npx @astryxdesign/cli init` or `theme` writes | T1.3 | 0 |  |
| `src/main.tsx` | T1.5 | 0 |  |
| `src/index.css` | T1.5 | 0 |  |
| `src/tasks/types.ts` | T2.1 | 0 | architect doc/contract |
| `src/tasks/storage.ts` | T2.1 | 0 | architect doc/contract |
| `docs/contracts/tasks.md` | T2.1 | 0 | architect doc/contract |
| `src/App.tsx` | T1.7 | 0 |  |
| `src/App.test.tsx` | T1.7 | 0 |  |
| `src/App.css` | T1.7 | 0 | deletes Vite demo file |
| `src/assets/react.svg` | T1.7 | 0 | deletes Vite demo file |
| `public/vite.svg` | T1.7 | 0 | deletes Vite demo file |
| `src/tasks/TaskScreen.tsx` | T2.3 | 0 | stub from T2.1 |
| `src/tasks/TaskScreen.test.tsx` | T2.3 | 0 |  |
| `docs/contracts/capture.md` | T3.1 | 1 | architect doc/contract |
| `src/tasks/TaskComposer.tsx` | T3.3 | 1 | stub from T2.1 |
| `src/tasks/TaskComposer.test.tsx` | T3.3 | 1 |  |
| `docs/contracts/list.md` | T4.1 | 1 | architect doc/contract |
| `src/tasks/TaskRow.tsx` | T4.3 | 1 | stub from T2.1 |
| `src/tasks/TaskRow.test.tsx` | T4.3 | 1 |  |
| `src/tasks/TaskList.tsx` | T4.5 | 1 | stub from T2.1 |
| `src/tasks/TaskList.test.tsx` | T4.5 | 1 |  |
| `src/tasks/sortTasks.ts` | T4.5 | 1 | stub from T2.1 |
| `src/tasks/sortTasks.test.ts` | T4.5 | 1 |  |
| `docs/contracts/mutate.md` | T5.1 | 1 | architect doc/contract |
| `src/tasks/EditTaskDialog.tsx` | T5.3 | 1 | stub from T2.1 |
| `src/tasks/EditTaskDialog.test.tsx` | T5.3 | 1 |  |
| `src/tasks/ConfirmDeleteDialog.tsx` | T5.5 | 1 | stub from T2.1 |
| `src/tasks/ConfirmDeleteDialog.test.tsx` | T5.5 | 1 |  |
| `docs/contracts/query.md` | T6.1 | 1 | architect doc/contract |
| `src/tasks/filterTasks.ts` | T6.3 | 1 | stub from T2.1 |
| `src/tasks/filterTasks.test.ts` | T6.3 | 1 |  |
| `src/tasks/FilterBar.tsx` | T6.5 | 1 | stub from T2.1 |
| `src/tasks/FilterBar.test.tsx` | T6.5 | 1 |  |
| `src/tasks/SearchField.tsx` | T6.7 | 1 | stub from T2.1 |
| `src/tasks/SearchField.test.tsx` | T6.7 | 1 |  |
| `docs/contracts/data.md` | T7.1 | 1 | architect doc/contract |
| `src/tasks/localTaskStorage.ts` | T7.3 | 1 | stub from T2.1 |
| `src/tasks/localTaskStorage.test.ts` | T7.3 | 1 |  |
| `src/tasks/useTasks.ts` | T7.5 | 1 | stub from T2.1 |
| `src/tasks/useTasks.test.ts` | T7.5 | 1 |  |
| `src/tasks/StatusViews.tsx` | T7.7 | 1 | stub from T2.1 |
| `src/tasks/StatusViews.test.tsx` | T7.7 | 1 |  |
| `src/tasks/SaveErrorBanner.tsx` | T7.9 | 1 | stub from T2.1 |
| `src/tasks/SaveErrorBanner.test.tsx` | T7.9 | 1 |  |
| `docs/contracts/a11y.md` | T8.1 | 2 | architect doc/contract |
| `src/a11y.test.tsx` | T8.3 | 2 |  |
| `e2e/happy-path.spec.ts` | T8.5 | 2 |  |

### Wave-1 disjointness proof

26 wave-1 build paths across 12 build tasks; every path appears once.

| Build task | Lane | Owns |
|---|---|---|
| T3.3 | capture | `src/tasks/TaskComposer.tsx`, `src/tasks/TaskComposer.test.tsx` |
| T4.3 | list | `src/tasks/TaskRow.tsx`, `src/tasks/TaskRow.test.tsx` |
| T4.5 | list | `src/tasks/TaskList.tsx`, `src/tasks/TaskList.test.tsx`, `src/tasks/sortTasks.ts`, `src/tasks/sortTasks.test.ts` |
| T5.3 | mutate | `src/tasks/EditTaskDialog.tsx`, `src/tasks/EditTaskDialog.test.tsx` |
| T5.5 | mutate | `src/tasks/ConfirmDeleteDialog.tsx`, `src/tasks/ConfirmDeleteDialog.test.tsx` |
| T6.3 | query | `src/tasks/filterTasks.ts`, `src/tasks/filterTasks.test.ts` |
| T6.5 | query | `src/tasks/FilterBar.tsx`, `src/tasks/FilterBar.test.tsx` |
| T6.7 | query | `src/tasks/SearchField.tsx`, `src/tasks/SearchField.test.tsx` |
| T7.3 | data | `src/tasks/localTaskStorage.ts`, `src/tasks/localTaskStorage.test.ts` |
| T7.5 | data | `src/tasks/useTasks.ts`, `src/tasks/useTasks.test.ts` |
| T7.7 | data | `src/tasks/StatusViews.tsx`, `src/tasks/StatusViews.test.tsx` |
| T7.9 | data | `src/tasks/SaveErrorBanner.tsx`, `src/tasks/SaveErrorBanner.test.tsx` |

Shared modules (`src/tasks/types.ts`, `src/tasks/storage.ts`) belong to T2.1, so every wave-1 build reads them and none edits them. `package.json` belongs to T1.3, and wave 1 adds no dependencies. `src/tasks/TaskScreen.tsx` belongs to T2.3, so wiring is done before wave 1 starts.

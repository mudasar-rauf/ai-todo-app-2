# Shell contract

**Task:** T1.1 (#3). **Followed by:** T1.3 (scaffold), T1.5 (theme and global CSS), T1.7 (app frame), and their reviews (T1.4, T1.6, T1.8).

This document settles the app shell before any code exists. Build agents follow it; reviewers check against it. Where it and an issue body disagree, raise it on the issue rather than picking one in silence.

## 1. Status and sources

- Astryx CLI **0.7.16** (`@astryxdesign/cli`, `@astryxdesign/core`, `@astryxdesign/theme-neutral` all at 0.7.16), read on 2026-11-20.
- Every Astryx component name, import specifier, prop, command and generated path below was copied from CLI output, not from memory or the website. The commands were run in a scratch project outside this repository:
  - `astryx --version`, `--help`, `init --help`, `theme --help`, `theme list`, `theme add --help`, `theme use --help`, `theme build --help`, `component --help`
  - `astryx component Theme`, `AppShell`, `TopNav`, `TopNavHeading`, `Layout`, `Heading`, `Stack` (and `component AppShell --source` to confirm its landmarks)
  - `astryx init`, `astryx theme add neutral`, `astryx theme use neutral`, `astryx doctor` in a scratch project, to see what files they write
- The full transcript is in the pull request that added this file (#69).

### Running the CLI

Run it as `npx @astryxdesign/cli <command>` (or `npx astryx <command>`, its other bin name) **from the project root after `npm install`**, so npx uses the locally installed copy. In this container, once the package is installed locally, both forms worked.

If `npx @astryxdesign/cli` fails with `sh: 1: cli: Permission denied` (seen while planning, before the package was installed locally), run the bin through node instead:

```bash
node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs <command>
```

Same CLI, same output. Say in the PR body which form you used.

## 2. Setup commands (T1.3)

Run in order, at the repository root, on T1.3's branch:

1. `npm create vite@latest . -- --template react-ts`
   If the scaffolder reports a non-empty directory, choose **ignore existing files**. Never remove or overwrite `docs/**`, `.factory/**` or `.github/**`.
2. `npm install`
3. `npm install @astryxdesign/core @stylexjs/stylex @astryxdesign/theme-neutral @astryxdesign/cli`
4. `npm install -D vitest jsdom @testing-library/react @testing-library/user-event @testing-library/jest-dom @playwright/test`
5. `npx @astryxdesign/cli init`
   Plain, no flags (as T1.3's issue says). In the scratch run it wrote only agent docs, `AGENTS.md` and `GEMINI.md`, and printed "Add and wire a theme: … `npx astryx theme add neutral`". It does **not** create a themes module.
6. `npx @astryxdesign/cli theme add neutral`
   This is T1.3's fallback from `theme --help`, needed because step 5 makes no themes module. In the scratch run it wrote `src/astryx-themes.js` and printed `default: neutral`, `modulePath: src/astryx-themes.js`. Neutral is the default once it is the first theme added, so `theme use neutral` is not needed: it printed `changed: false`.
7. `npx @astryxdesign/cli doctor`
   Must report `fail: 0`. Before T1.5 lands, expect the warning `theme-module-import` ("Could not prove that project source imports src/astryx-themes.js"); T1.5 clears it. The `theme-fonts` warning ("External font imports prevent proof for neutral: Figtree") is informational. List any other warning in the PR body.
8. `npx playwright install chromium`

T1.3 lists every file the CLI wrote in its PR body (expected: `AGENTS.md`, `GEMINI.md`, `src/astryx-themes.js`), and records the themes module path `src/astryx-themes.js` in the README.

**TypeScript and the themes module.** The CLI writes the themes module as JavaScript (`src/astryx-themes.js`) with no `.d.ts`. Vite's `react-ts` template turns on strict mode, so `npm run typecheck` may reject `import … from './astryx-themes'` (no declaration file). T1.3 owns `tsconfig.app.json` and must make sure the import typechecks before handing over. The simplest fix is `"allowJs": true` in `tsconfig.app.json`. Never hand-edit the generated module: its header says "do not edit manually".

## 3. Theme

- **Decision: theme-neutral.** No other theme.
  - npm package: `@astryxdesign/theme-neutral`
  - CLI slug: `neutral` (`theme list`: "Restrained warm grays. Minimal and quiet, so the content stays the focus.", `maintained: yes`)
- **Theme import specifier, as printed by `component Theme`:**
  `import {Theme} from '@astryxdesign/core/theme';`
- **The theme object comes only from the generated themes module**, `src/astryx-themes.js`, which exports `themes` and `defaultThemeSlug`. Never import from `@astryxdesign/theme-neutral` directly in app code, and never call `defineTheme()`. The module already imports the built theme, its `theme.css` and its `fonts.css`. As generated in the scratch run:

  ```js
  import {neutralTheme as _astryxTheme0} from "@astryxdesign/theme-neutral/built";
  import "@astryxdesign/theme-neutral/theme.css";
  import "@astryxdesign/theme-neutral/fonts.css";

  export const themes = {
    "neutral": _astryxTheme0,
  };

  export const defaultThemeSlug = "neutral";
  ```

## 4. Global CSS (T1.5)

`src/index.css` contains exactly these two lines, in this order, and nothing else:

```css
@import '@astryxdesign/core/reset.css';
@import '@astryxdesign/core/astryx.css';
```

## 5. Root: how Theme wraps the app (T1.5)

`src/main.tsx` imports `./index.css` and renders **StrictMode > Theme > App**. The Theme wiring is what `theme add` and `init` printed:

```tsx
import {Theme} from '@astryxdesign/core/theme';
import {themes, defaultThemeSlug} from './astryx-themes';

<Theme theme={themes[defaultThemeSlug]}>
  <App />
</Theme>
```

Theme props, from `component Theme`:

| Prop | Type | Default | Used here |
|---|---|---|---|
| `theme` | `DefinedTheme` | (required) | `themes[defaultThemeSlug]` |
| `mode` | `'light' \| 'dark' \| 'system'` | `'system'` | not passed (follows the OS) |
| `children` | `ReactNode` | (required) | `<App />` |

Import `./index.css` before `./astryx-themes`, so the reset comes first and the theme stylesheet comes after it. Keep the rest of the template's `main.tsx` (`createRoot(document.getElementById('root')!).render(...)`).

## 6. Frame (T1.7)

`src/App.tsx` renders the frame below and nothing else. The main landmark renders only `<TaskScreen />` (from `src/tasks/TaskScreen.tsx`, contract in `docs/contracts/tasks.md`).

**App name: `Todo`.** This exact string is the only level-1 heading; T1.7's test asserts it.

```tsx
<AppShell
  height="auto"
  contentPadding={4}
  topNav={<TopNav label="Todo" heading={<Heading level={1}>Todo</Heading>} />}
>
  <TaskScreen />
</AppShell>
```

| Component | Import (as printed) | Props used | Landmark or element |
|---|---|---|---|
| AppShell | `import {AppShell} from '@astryxdesign/core/AppShell';` | `topNav` (ReactNode), `children` (ReactNode), `height="auto"` ("lets the shell grow with content"; the docs say auto is for "pages that grow with content"), `contentPadding={4}` (the docs say 4 for "forms/settings/text") | The **banner** landmark (`role="banner"`, wraps the topNav slot) and the **main** landmark (`role="main"`, wraps `children`), plus the skip link. Exactly one of each. |
| TopNav | `import {TopNav} from '@astryxdesign/core/TopNav';` | `heading` (ReactNode, "logo, brand"), `label` (string, "Accessible label for the navigation landmark") | A `<nav>` landmark named "Todo", inside the banner. No nav items. |
| Heading | `import {Heading} from '@astryxdesign/core/Heading';` | `level={1}` (required), `children` = `Todo` | `<h1>Todo</h1>`, inside the banner |

Rules from the AppShell docs that T1.7 must keep:

- Do not add your own `<main>`, skip link or `<header>`: AppShell renders them, and a second main landmark "makes the first ambiguous".
- Do not nest AppShell, and do not use Layout as the page shell.
- Do not pass `sideNav`, `banner` or `mobileNav`. With no sideNav there is no mobile drawer, and the default below the breakpoint is a compact bar with the heading.
- No CSS files, no inline styles, no `xstyle` in the frame.

### Considered, not used

| Component | Import (as printed) | Why not |
|---|---|---|
| Layout | `import {Layout} from '@astryxdesign/core/Layout';` | Its docs say "Don't: Use Layout as the page shell or for app-wide navigation; use AppShell for that responsibility", and that "Layout adds no role or name". The frame needs no header, start, end or footer regions. It stays available to feature tasks for sub-page layout inside the main landmark. |
| Stack | `import {Stack} from '@astryxdesign/core/Stack';` | The frame has a single child in each slot, so there is nothing to stack. It stays available to TaskScreen and the feature components. |
| TopNavHeading | `import {TopNavHeading} from '@astryxdesign/core/TopNav';` | Its `heading` prop is a `string` displayed as text, and its source renders no `h1` to `h6`. T1.7 needs a real level-1 heading, so the TopNav `heading` slot gets `<Heading level={1}>`. |

## 7. npm scripts

These names are fixed. Later tasks and CI call them by name. T1.3 creates them:

| Script | Command |
|---|---|
| `dev` | `vite` (template default) |
| `build` | template default (`tsc -b && vite build`) |
| `preview` | `vite preview` (template default) |
| `typecheck` | `tsc -b` (or an equivalent no-emit check covering app and node configs) |
| `lint` | `eslint .` (template default) |
| `test` | `vitest run` |
| `test:e2e` | `playwright test` |

Only the names and the `test` and `test:e2e` commands are binding. T1.3 may adjust the other commands if the template differs, and records them in the README.

## 8. File ownership

Copied row for row from `docs/backlog/BACKLOG.md` section 4. **`src/main.tsx` is owned by T1.5.**

### T1.3

| Path | Owner | Wave | Note |
|---|---|---|---|
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

Note: T1.3's issue body words the last row as "CLI-generated: every file `npx @astryxdesign/cli init` or `theme` writes (themes module, astryx.config.*, agent docs), each listed in the PR body". From the scratch run, that means `AGENTS.md`, `GEMINI.md` and `src/astryx-themes.js`. If `init` or `theme` rewrites a file T1.5 or T1.7 owns, T1.3 keeps the CLI's change and lists it in the PR body; it never hand-edits those files.

### T1.5

| Path | Owner | Wave | Note |
|---|---|---|---|
| `src/main.tsx` | T1.5 | 0 |  |
| `src/index.css` | T1.5 | 0 |  |

### T1.7

| Path | Owner | Wave | Note |
|---|---|---|---|
| `src/App.tsx` | T1.7 | 0 |  |
| `src/App.test.tsx` | T1.7 | 0 |  |
| `src/App.css` | T1.7 | 0 | deletes Vite demo file |
| `src/assets/react.svg` | T1.7 | 0 | deletes Vite demo file |
| `public/vite.svg` | T1.7 | 0 | deletes Vite demo file |

## 9. Changing this contract

Once T1.2 passes, this file changes only through a planned task. A build task that finds the contract wrong does not work around it: it adds the label `blocked`, comments why, and hands to plan.

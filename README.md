# Build the accessible frontend shell

PortfolioPilot is a teaching project for a stock portfolio manager. This learning activity asks a coding agent to build the first usable browser interface: a professional dashboard and six navigable pages backed by fixed demo data. The interface is a foundation for later portfolio APIs, market data, and AI features; it does not claim to provide live prices or investment advice.

Students will see how to organize a React application around routes and reusable components, keep sample data separate from the UI, prepare a typed API boundary, and check responsive and keyboard behavior in a real browser.

## What this milestone builds

| Route | Page | What it shows now |
| --- | --- | --- |
| `/` | Dashboard | Portfolio value, return and cash cards, holdings, news, and an assistant preview |
| `/portfolios` | Portfolios | Summary cards and a holdings table for the selected demo portfolio |
| `/news` | News | Fictional demo headlines with sources and UTC timestamps |
| `/assistant` | Assistant | A labeled preview with its message input disabled |
| `/watchlist` | Watchlist | Fixed example companies and prices |
| `/settings` | Settings | Demo environment details and a stale-snapshot notice |

The sidebar becomes a drawer on smaller screens. The portfolio selector in the header switches between two fixed portfolios. A **DTO** (data transfer object) is the agreed shape of data passed between application layers; these DTOs keep money and quantities as decimal strings. A **fixture** is a predictable example record used while real data services are not yet connected.

## Steps performed

1. **Inspected the existing project.** The workspace already had npm workspaces, a minimal React/Vite page, browser-safe contracts, a Next.js API health route, and shared project rules. The requested milestone was 03 in `docs/project-plan.md`.
2. **Added frontend packages.** `react-router` 8.3.0 supplies page navigation, `@tanstack/react-query` 5.102.8 prepares server-state handling, and `@playwright/test` 1.63.0 supports browser checks. Their exact versions are recorded in `apps/web/package.json`, `package-lock.json`, and `docs/versions.md`. Styling uses one CSS file instead of a component library.
3. **Defined demo data shapes and fixtures.** `packages/contracts/src/index.ts` contains browser-safe portfolio, holding, news, and watchlist DTO schemas. `apps/web/src/data/demo.ts` holds deterministic sample records. The portfolio totals were checked against the listed holdings and cash. `apps/web/src/lib/format.ts` formats decimal strings for display without calculating portfolio values inside JSX.
4. **Built the UI.** `apps/web/src/main.tsx` installs React Router and TanStack Query. `apps/web/src/app.tsx` contains the six routes and reusable cards, table, news panel, chat preview, and empty/loading/error/stale state component. `apps/web/src/styles.css` provides the responsive layout, visible keyboard focus, and stronger contrast for small labels.
5. **Created an API boundary.** `apps/web/src/lib/api-client.ts` accepts an `/api/` path and a response parser. It handles shared JSON error envelopes, malformed responses, network failures, timeouts, and caller cancellation. The API health check uses this client through TanStack Query. Portfolio and news fixtures do not pretend to be API responses.
6. **Verified and documented the result.** `apps/web/src/lib/api-client.test.ts` tests the client. `apps/web/e2e/shell.spec.ts` tests desktop and mobile routes and keyboard actions with Chrome. The lesson notes are in `docs/lessons/03-ui-shell.md`, and the completed milestone is recorded in `docs/project-state.md`.

The UI includes a skip link, labeled navigation and portfolio selector, table caption and headers, visible focus, and a mobile drawer that can be closed with Escape. The Assistant input is deliberately disabled until a later milestone connects an agent endpoint.

## Results observed

The final local checks passed:

| Check | Observed result |
| --- | --- |
| `npm run build` | Passed; Vite produced a web bundle and Next.js built the API |
| `npm run typecheck` | Passed |
| `npm run lint` | Passed; this project script currently runs TypeScript checks, not ESLint |
| `npm run test` | Passed: 10 unit tests across contracts, config, and web |
| `npm run check:browser-boundary` | Passed: the browser workspace dependency graph contains only web, config, and contracts |
| `npm run test:browser --workspace=@portfolio-pilot/web` | Passed: 2 Chrome tests at 1440 px and 390 px, with the dev server running |

Desktop and mobile screenshots from the browser tests were visually inspected. The tests observed all six routes, portfolio switching, Tab/Enter navigation, mobile drawer opening and Escape closing, and no page-wide horizontal overflow at 390 px. For example, the Growth Portfolio fixture displays **$87,477.14** as its portfolio value and a fixed **Sep 30, 2026, 2:30 PM UTC** snapshot. Those values are examples, not market observations.

The first browser test run failed because a broad text locator matched several “demo data” labels. The locator was narrowed, and the final run passed. The first unit run included a Playwright file; the web unit command was scoped to `src`, and the final run passed. An experiment that let Playwright start the dev servers passed its tests but hung during Windows teardown. The final browser workflow starts `npm run dev` separately and exits cleanly.

## Run the project

You need Node.js **24.21.0**, npm **11.19.0**, and access to install the locked npm packages. Chrome is needed for the browser test. Docker, a database, and AI or market-data credentials are not needed for this milestone.

From the project root:

```powershell
node --version
npm --version
npm install
npm run dev
```

Keep the dev command running. Open `http://127.0.0.1:5173/`. Vite serves the React app on port 5173 and proxies `/api` to the Next.js API on port 3001. The dashboard should show a **DEMO DATA** label and an API connection status. Use the sidebar to visit each route, switch portfolios in the header, then narrow the browser below 850 px to try the menu. Press Tab to reveal the skip link and move through navigation; press Escape to close the mobile menu.

In a second terminal, from the project root, run:

```powershell
npm run build
npm run typecheck
npm run lint
npm run test
npm run check:browser-boundary
npm run test:browser --workspace=@portfolio-pilot/web
```

The browser test expects `npm run dev` to be running first. Its screenshots are written under `apps/web/test-results/`, an ignored generated directory.

On the Windows host used for this milestone, the normal `node`/`npm` shim did not activate Node reliably. The checks above were actually run by placing the installed Node 24.21.0 directory first in `PATH` and invoking its `node.exe` and adjacent npm CLI directly. On a machine where Node and npm are active normally, the commands in the code blocks are the intended workflow.

## Current limits and next step

- Portfolio figures, prices, watchlist entries, and news are fixed fixtures. No live provider or database is connected.
- The Assistant cannot accept a question yet. Connecting the first mock and Claude Agent SDK flow is milestone 04.
- The API client currently handles the health check; later milestones will connect portfolio and news screens to authenticated APIs.
- Browser checks covered installed Chrome at two viewport widths. They are a smoke test, not a full accessibility audit or cross-browser test.
- The production build emitted non-fatal Vite warnings about third-party `use client` directives. The build and browser tests still passed.
- This workspace has no `.git` directory, so no commit was made. Next.js-generated instruction files in `apps/api` remain from milestone 02 after automatic approval review rejected their removal.

For the full milestone sequence and current project status, read `docs/project-plan.md` and `docs/project-state.md`.

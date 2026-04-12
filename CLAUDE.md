# CLAUDE.md — omni-code

AI assistant context for the `omni-code` repository. Keep this file up to date when conventions, tooling, or structure change.

---

## Repository Overview

**omni-code** is an [Nx](https://nx.dev) monorepo containing an Angular 19 web application with server-side rendering (SSR). The project is in early bootstrapping stage with a minimal feature set ready for expansion.

- **Workspace name:** `@omni-code/source`
- **License:** MIT
- **Default CI base branch:** `master`

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Angular (standalone components) | ~19.2.0 |
| Language | TypeScript | ~5.7.2 |
| SSR | Angular SSR + Express | ~19.2.0 / ^4.21.2 |
| Monorepo | Nx | 20.5.0 |
| Compiler | SWC | ~1.5.7 |
| Reactive | RxJS | ~7.8.0 |
| Unit tests | Jest + jest-preset-angular | ^29.7.0 / ~14.4.0 |
| E2E tests | Cypress | ^13.13.0 |
| Linting | ESLint 9 (flat config) + angular-eslint | ^9.8.0 |
| Formatting | Prettier | ^2.6.2 |

---

## Project Structure

```
omni-code/
├── apps/
│   ├── omni-code/                  # Main Angular application
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── app.component.ts       # Root component
│   │   │   │   ├── app.component.html
│   │   │   │   ├── app.component.scss
│   │   │   │   ├── app.component.spec.ts  # Unit tests (collocated)
│   │   │   │   ├── app.config.ts          # App providers/config
│   │   │   │   ├── app.routes.ts          # Route definitions
│   │   │   │   └── nx-welcome.component.ts
│   │   │   ├── main.ts                    # Client bootstrap
│   │   │   ├── main.server.ts             # SSR entry point
│   │   │   ├── styles.scss                # Global styles
│   │   │   ├── index.html                 # HTML shell
│   │   │   └── test-setup.ts              # Jest zone.js setup
│   │   ├── eslint.config.mjs
│   │   ├── tsconfig.json
│   │   ├── tsconfig.app.json
│   │   └── tsconfig.spec.json
│   └── omni-code-e2e/              # Cypress E2E tests
│       └── src/
│           ├── e2e/                # Test specs (.cy.ts)
│           ├── support/            # Custom commands, page objects
│           └── fixtures/           # Test data
├── .vscode/                        # Recommended extensions
├── .run/                           # IntelliJ run configurations
├── azure-pipelines.yml             # CI/CD pipeline
├── eslint.config.mjs               # Root ESLint flat config
├── jest.config.ts                  # Root Jest config (Nx projects)
├── nx.json                         # Nx workspace config
├── tsconfig.base.json              # Root TypeScript config
├── .editorconfig
└── .prettierrc
```

---

## Common Commands

All tasks run through **Nx**. There are no npm scripts defined at the root.

```bash
# Development
npx nx serve omni-code              # Dev server at http://localhost:4200
npx nx build omni-code              # Production build to dist/

# Testing
npx nx test omni-code               # Unit tests (Jest)
npx nx test omni-code --watch       # Watch mode
npx nx e2e omni-code-e2e            # E2E tests (Cypress headless)
npx nx open-cypress omni-code-e2e   # Open Cypress UI

# Code quality
npx nx lint                         # Lint all projects
npx nx lint omni-code               # Lint specific project

# Nx utilities
npx nx show project omni-code       # List all available targets
npx nx graph                        # Open dependency graph in browser
npx nx affected --base=main lint test build   # Only run tasks on changed code
```

---

## Angular Architecture Conventions

### Standalone Components (Required)
All components **must** use the standalone API. Do **not** create NgModules.

```typescript
@Component({
  imports: [CommonModule, RouterModule, /* other standalone deps */],
  selector: 'app-my-feature',
  templateUrl: './my-feature.component.html',
  styleUrl: './my-feature.component.scss',
})
export class MyFeatureComponent { }
```

### Selector Prefixes
Enforced by ESLint (`angular-eslint`):
- **Components:** `app-` prefix, **kebab-case** (e.g., `app-user-profile`)
- **Directives:** `app` prefix, **camelCase** (e.g., `appHighlight`)

### Application Providers (`app.config.ts`)
The app uses functional providers — add new providers here, not in component decorators:
- `provideClientHydration(withEventReplay())` — SSR hydration
- `provideZoneChangeDetection({ eventCoalescing: true })` — performance optimisation
- `provideRouter(appRoutes)` — routing

### Routing (`app.routes.ts`)
Define all top-level routes in `appRoutes`. Currently empty — add feature routes here.

### SSR
The app has two entry points:
- `main.ts` — client-side bootstrap (`bootstrapApplication`)
- `main.server.ts` — SSR entry; Express is the server (`server.ts` if added)

---

## Code Style

### Formatting (Prettier)
- **Single quotes** for strings (`.prettierrc`: `{ "singleQuote": true }`)
- Run formatter: `npx prettier --write .`

### EditorConfig
- Indent: **2 spaces**
- Charset: **UTF-8**
- Always insert a **final newline**
- Trim trailing whitespace (except Markdown)

### TypeScript
- Target: `ES2022` for app code, `ES2016` for tests
- Strict mode is on via Angular defaults
- No implicit `any`

### ESLint (Flat Config — ESLint 9)
- Root config: `eslint.config.mjs`
- App config: `apps/omni-code/eslint.config.mjs`
- Module boundary enforcement (`@nx/enforce-module-boundaries`) is active — respect library/app boundaries
- `dist/` is globally ignored

---

## Testing Conventions

### Unit Tests (Jest)
- Test files are **collocated** next to the source file: `foo.component.spec.ts`
- Use Angular's `TestBed` for component tests
- jest-preset-angular handles Angular-specific serializers and transforms
- Code coverage collected in CI (`--codeCoverage` flag under `ci` configuration)
- `passWithNoTests: true` — no failure when a project has no tests yet

```bash
npx nx test omni-code                      # Run once
npx nx test omni-code --testFile=foo.spec  # Run a single file
npx nx test omni-code --ci --codeCoverage  # CI mode
```

### E2E Tests (Cypress)
- Specs in `apps/omni-code-e2e/src/e2e/` with `.cy.ts` extension
- Base URL: `http://localhost:4200`
- Use page objects in `src/support/` for reusable selectors/actions
- Custom commands go in `src/support/commands.ts`
- Fixtures in `src/fixtures/`

---

## CI/CD — Azure Pipelines

Pipeline file: `azure-pipelines.yml`

- **Triggers:** pushes and PRs to `master`
- **Agent:** `ubuntu-latest`
- **Install:** `npm ci --legacy-peer-deps`
- **Key step:** `npx nx affected --base=$(BASE_SHA) --head=$(HEAD_SHA) lint test build e2e`
  - Uses `affected` to only run tasks for changed projects — keep code changes focused to speed up CI
- **BASE_SHA logic:**
  - PRs: merge-base of target branch and HEAD
  - Pushes: last successful pipeline commit (falls back to `HEAD~1`)
- **Nx Cloud:** configured (`nxCloudId` in `nx.json`) but task distribution is currently commented out

Install Cypress binary separately (`npx cypress install`) — this is required in CI and already in the pipeline.

---

## Nx Workspace Notes

- **`nx.json` `defaultBase`** is `master` — Nx affected calculations use `master` as the comparison branch
- **Caching** is enabled for `lint`, `jest`, and `build` tasks; cache is stored in `.nx/cache` (gitignored)
- **`sharedGlobals`** input includes `azure-pipelines.yml` — changing the pipeline invalidates all cached tasks
- **Generators default** to: Angular application, Cypress E2E, ESLint, SCSS, Jest
- When adding a new app or library: `npx nx g @nx/angular:application my-app` or `npx nx g @nx/angular:library my-lib`

---

## IDE Setup

**VS Code** (recommended extensions in `.vscode/extensions.json`):
- `nrwl.angular-console` — Nx Console GUI
- `esbenp.prettier-vscode` — auto-format on save
- `dbaeumer.vscode-eslint` — inline lint errors
- `firsttris.vscode-jest-runner` — run/debug individual tests

**JetBrains / IntelliJ** — run configurations in `.run/`:
- `omni-code_serve_production` — production serve
- `omni-code_test` — Jest tests

---

## Key Files Quick Reference

| File | Purpose |
|---|---|
| `apps/omni-code/src/app/app.config.ts` | App-wide providers (router, hydration, zone) |
| `apps/omni-code/src/app/app.routes.ts` | Route definitions |
| `apps/omni-code/src/main.ts` | Client bootstrap entry point |
| `apps/omni-code/src/main.server.ts` | SSR entry point |
| `nx.json` | Nx workspace settings, cache config, plugins |
| `tsconfig.base.json` | Root TypeScript paths and compiler options |
| `eslint.config.mjs` | Root ESLint flat config |
| `azure-pipelines.yml` | CI pipeline definition |
| `.prettierrc` | Prettier config (`singleQuote: true`) |
| `.editorconfig` | Editor whitespace/indent rules |

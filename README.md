# Fidelis PPM — QA Agentic Solution

Automated regression, functional, and visual test suite for [getfidelis.com](http://getfidelis.com/), built with **Playwright + TypeScript** using the **Page Object Model (POM)** and **Object-Oriented Programming (OOP)** design patterns. The suite is instrumented for agentic execution by **Claude Code**.

---

## Table of Contents

- [Project Purpose](#project-purpose)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Running Tests](#running-tests)
- [Project Architecture](#project-architecture)
- [Claude Code Integration](#claude-code-integration)
- [Contributor Guide](#contributor-guide)

---

## Project Purpose

This repo tests the public-facing website for **Fidelis PPM** — a maintenance program customer loyalty platform. The goal is comprehensive, headless coverage of every discoverable feature without requiring account creation, login, or form submission.

Coverage includes:
- **Smoke** — site loads, HTTP 200, no JS errors, core content visible
- **Functional** — business features: pricing, hero CTAs, feature sections, FAQ/accordions, carousels
- **Regression** — visual snapshots and layout invariants across viewports
- **Navigation** — all nav links reachable, mobile menu, footer links
- **Forms** — field presence, required-field validation, accessibility labels (no submission)
- **Responsive** — layout correctness at mobile (390 px), tablet (768 px), and desktop (1280 px)

---

## Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| [Playwright](https://playwright.dev/) | ^1.44 | Browser automation & assertions |
| TypeScript | ^5.4 | Type-safe test code |
| ESLint | ^8.57 | Linting |
| Claude Code | latest | Agentic test generation & maintenance |
| GitHub Actions | — | CI/CD pipeline |

---

## Prerequisites

- **Node.js** 18 or later
- **npm** 9 or later
- **Claude Code CLI** (`claude`) — [install instructions](https://code.claude.com/docs/en/overview)
- A terminal (PowerShell on Windows, bash/zsh on Mac/Linux)

---

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/<your-org>/fidelis_ppm_QA_Agentic_Solution.git
cd fidelis_ppm_QA_Agentic_Solution

# 2. Install dependencies
npm install

# 3. Install Playwright browsers
npx playwright install --with-deps

# 4. Verify TypeScript compiles
npm run typecheck

# 5. Run smoke tests to confirm setup
npm run test:smoke
```

> **Windows note:** Run the above commands in **PowerShell** or **Git Bash**. WSL is also supported.

### Environment variables

Copy `.env.example` to `.env.local` and fill in any required values:

```bash
cp .env.example .env.local
```

The `.env.local` file is gitignored. Never commit credentials.

---

## Running Tests

| Command | What it runs |
|---------|-------------|
| `npm test` | All tests (all viewports) |
| `npm run test:smoke` | `@smoke` — availability checks |
| `npm run test:navigation` | `@navigation` — nav links and routing |
| `npm run test:forms` | `@forms` — form fields and validation |
| `npm run test:visual` | `@visual` — screenshot regression |
| `npm run test:responsive` | `@responsive` — layout across viewports |
| `npm run test:headed` | All tests with browser visible |
| `npm run report` | Open the last HTML report |
| `npm run baseline` | Refresh visual regression snapshots |
| `npm run lint` | ESLint |
| `npm run typecheck` | TypeScript check |

### Run a single file

```bash
npx playwright test tests/smoke/site-availability.spec.ts
```

### Run with debug tracing

```bash
npx playwright test --trace on
npx playwright show-trace test-results/<run>/trace.zip
```

---

## Project Architecture

### Directory layout

```
site.config.json           # Site URL, flags, expected nav items
playwright.config.ts       # Projects: desktop, mobile, tablet
global-setup.ts            # Pre-test site reachability check
src/
  pages/                   # Page Object Model classes (one per page/section)
    base.page.ts           #   Abstract base class — shared helpers
    home.page.ts           #   HomePage
    navigation.page.ts     #   NavigationPage
    contact.page.ts        #   ContactFormPage
  fixtures/
    site.fixture.ts        # Custom Playwright fixtures (auto-wires page objects)
  utils/
    link-checker.ts        # URL reachability helpers
    visual-helper.ts       # Screenshot diff helpers
  types/
    site-config.types.ts   # TypeScript interface for site.config.json
tests/
  smoke/                   # @smoke
  navigation/              # @navigation
  forms/                   # @forms
  functional/              # @functional (one file per feature area)
  visual/                  # @visual
  responsive/              # @responsive
.claude/
  agents/                  # Claude Code subagent definitions
  commands/                # Slash command scripts (/generate-full-suite, etc.)
  rules/                   # Path-scoped rules loaded into Claude's context
  settings.json            # Claude Code permissions & settings
.github/
  workflows/               # GitHub Actions CI pipelines
  CONTRIBUTING.md          # Contributor guidelines
  PULL_REQUEST_TEMPLATE.md # PR checklist
  copilot-instructions.md  # GitHub Copilot context
```

### Page Object Model

Every page or major section has a dedicated TypeScript class in `src/pages/`. Classes:
- Extend `BasePage` from `./base.page`
- Declare all key elements as `readonly Locator` properties
- Expose methods that represent **user actions** (never assertions)
- Never call `expect()` — assertions belong in test files

```typescript
// Example page object
export class ServicesPage extends BasePage {
  readonly heroHeading: Locator;
  readonly serviceCards: Locator;

  constructor(page: Page) {
    super(page);
    this.heroHeading = page.locator('h1');
    this.serviceCards = page.locator('[class*="service-card"]');
  }

  async clickFirstCard(): Promise<void> {
    await this.serviceCards.first().click();
    await this.waitForLoad();
  }
}
```

### Test tagging

Every test must carry at least one tag:

| Tag | When to use |
|-----|-------------|
| `@smoke` | Site loads, title present, no JS errors |
| `@navigation` | Nav links, routing, menus, breadcrumbs |
| `@forms` | Form fields, validation, accessibility |
| `@functional` | Business features: pricing, CTAs, accordions, video |
| `@visual` | `toHaveScreenshot()` regression |
| `@responsive` | Viewport-specific layout checks |
| `@custom` | Site-specific scenarios not in the shared suite |

### OOP conventions

- All test logic is encapsulated in page objects (OOP encapsulation)
- `BasePage` provides shared behavior via inheritance
- Fixtures use dependency injection to supply page objects to tests
- Interfaces in `src/types/` enforce typed config objects

---

## Claude Code Integration

This repo is instrumented for Claude Code agentic execution. See [`AGENTS.md`](./AGENTS.md) and [`Skills.md`](./Skills.md) for full documentation.

### Quick reference

| Slash command | What it does |
|--------------|-------------|
| `/generate-full-suite` | Crawl the site, generate all POM classes + tests |
| `/analyze-site` | Inspect site structure; report pages, forms, nav |
| `/run-smoke` | Run smoke tests and display results |
| `/update-baseline` | Refresh visual regression baselines |
| `/generate-report` | Summarize the last test run |

### CLAUDE.md

Project-level Claude instructions live in [`CLAUDE.md`](./CLAUDE.md). It is read at the start of every Claude Code session and defines architecture rules, test conventions, and "do not" guardrails.

---

## Contributor Guide

See [`.github/CONTRIBUTING.md`](./.github/CONTRIBUTING.md) for the full guide. Quick rules:

1. **Read `CLAUDE.md` first** — it defines all architecture and test conventions.
2. **One page object per page** — create or update `src/pages/<name>.page.ts`.
3. **No assertions in page objects** — `expect()` belongs in test files only.
4. **No hardcoded URLs** — always use `baseURL` from Playwright config.
5. **Never submit forms** — validate field behavior only.
6. **Run `npm run typecheck` before every PR** — strict TypeScript must pass.
7. **Tag every test** — use the `@tag` system defined above.
8. **No `page.waitForTimeout()`** — use `waitForSelector` or Playwright auto-waiting.

### Branch naming

```
feat/<short-description>
fix/<short-description>
test/<page-or-feature-name>
chore/<short-description>
```

### PR process

1. Create a branch from `main`
2. Write or update page objects and tests
3. Run `npm run typecheck && npm run lint`
4. Run `npm run test:smoke` to confirm nothing is broken
5. Open a PR against `main` using the PR template
6. CI will run the full test suite automatically

---

*Maintained as part of the Phoenix Startup QA Agentic Solutions project.*

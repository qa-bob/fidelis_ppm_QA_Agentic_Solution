# Contributing to Fidelis PPM QA

Thank you for contributing to this test suite. This guide covers everything you need to know to add tests, fix issues, and keep the suite healthy.

---

## Table of Contents

- [Before You Start](#before-you-start)
- [Branch Naming](#branch-naming)
- [Adding Tests](#adding-tests)
- [Page Object Model](#page-object-model)
- [Test Tagging](#test-tagging)
- [TypeScript Requirements](#typescript-requirements)
- [Pre-PR Checklist](#pre-pr-checklist)
- [Pull Request Process](#pull-request-process)
- [Using Claude Code](#using-claude-code)

---

## Before You Start

1. Read [`CLAUDE.md`](../CLAUDE.md) — it is the authoritative reference for all architecture rules, naming conventions, and guardrails.
2. Read [`README.md`](../README.md) — setup instructions and project overview.
3. Run the smoke tests to confirm your local setup is working:
   ```bash
   npm install
   npx playwright install --with-deps
   npm run test:smoke
   ```

---

## Branch Naming

Use the following convention:

```
feat/<short-description>      # New test coverage or feature
fix/<short-description>        # Bug fix in a test or page object
test/<page-or-feature-name>    # Tests for a specific page or feature
chore/<short-description>      # Housekeeping: dependencies, config, docs
```

Examples:
```
feat/pricing-page-tests
fix/mobile-menu-locator
test/contact-form-validation
chore/update-playwright-version
```

---

## Adding Tests

### Step 1 — Create or update the page object

Every page interaction goes through a page object in `src/pages/`. If one already exists for the page you are testing, add to it. If not, create a new file.

Page objects must:
- Extend `BasePage` from `./base.page`
- Declare all locators as `readonly Locator` class properties
- Only expose methods that represent user actions (click, fill, navigate)
- Never call `expect()` inside a page object

```typescript
// src/pages/features.page.ts
import { Page, Locator } from '@playwright/test';
import { BasePage } from './base.page';

export class FeaturesPage extends BasePage {
  readonly heroHeading: Locator;
  readonly featureCards: Locator;

  constructor(page: Page) {
    super(page);
    this.heroHeading = page.locator('h1');
    this.featureCards = page.locator('[class*="feature-card"]');
  }

  async navigateToFeatures(): Promise<void> {
    await this.page.goto('/features');
    await this.waitForLoad();
  }
}
```

### Step 2 — Register the fixture (if it's a new page object)

Add the new page object to `src/fixtures/site.fixture.ts` so tests can import it:

```typescript
featuresPage: async ({ page }, use) => {
  const featuresPage = new FeaturesPage(page);
  await use(featuresPage);
},
```

### Step 3 — Write the test file

Put it in the correct category directory. Tag every test.

```typescript
// tests/functional/features.spec.ts
import { test, expect } from '@fixtures/site.fixture';

test.describe('Features Page @functional', () => {
  test.beforeEach(async ({ featuresPage }) => {
    await featuresPage.navigateToFeatures();
  });

  test('features page loads with heading @functional @smoke', async ({ featuresPage }) => {
    await expect(featuresPage.heroHeading).toBeVisible();
  });

  test('feature cards are present @functional', async ({ featuresPage }) => {
    await expect(featuresPage.featureCards.first()).toBeVisible();
    expect(await featuresPage.featureCards.count()).toBeGreaterThan(0);
  });
});
```

### Step 4 — Verify

```bash
npm run typecheck    # Must pass
npm run lint         # Must pass
npm run test:smoke   # Must pass
```

---

## Page Object Model

The POM pattern keeps selectors out of test files. This makes tests easier to maintain when the site changes — you update the locator in one place.

| Rule | Reason |
|------|--------|
| Locators go in page objects | One place to update when the site changes |
| Assertions go in tests | Page objects should be reusable across multiple tests |
| Methods are user actions | Describes *what the user does*, not *what to assert* |
| Extend `BasePage` | Inherits shared helpers: `waitForLoad()`, `getTitle()`, etc. |

---

## Test Tagging

Every test must carry at least one tag:

| Tag | Use for |
|-----|---------|
| `@smoke` | Site loads, no JS errors, core content visible |
| `@navigation` | Nav links, routing, menus, breadcrumbs |
| `@forms` | Form presence, fields, validation, accessibility |
| `@functional` | Business features: pricing, CTAs, accordions |
| `@visual` | `toHaveScreenshot()` regression |
| `@responsive` | Viewport-specific layout checks |
| `@custom` | Site-specific scenarios not in the shared suite |

A test can carry multiple tags: `@functional @smoke`.

---

## TypeScript Requirements

- Strict mode is on — no implicit `any`
- All properties must be typed
- Use path aliases: `@pages/*`, `@fixtures/*`, `@utils/*`, `@types/*`
- Always run `npm run typecheck` before opening a PR

---

## Pre-PR Checklist

Before opening a pull request, verify all of the following:

- [ ] `npm run typecheck` passes with no errors
- [ ] `npm run lint` passes with no errors
- [ ] `npm run test:smoke` passes
- [ ] New tests are tagged with at least one `@tag`
- [ ] No `page.locator()` calls in test files (use page objects)
- [ ] No `page.waitForTimeout()` calls
- [ ] No hardcoded URLs
- [ ] No form submissions
- [ ] No real credentials committed
- [ ] Visual baselines updated if visual tests changed (`npm run baseline`)

---

## Pull Request Process

1. Create a branch from `main` using the naming convention above
2. Write or update tests following the guide above
3. Complete the pre-PR checklist
4. Open a PR against `main` — use the PR template
5. CI will run automatically (smoke → navigation → forms → functional → responsive)
6. All CI checks must pass before merging
7. Squash merge to `main`

---

## Using Claude Code

This repo is instrumented for Claude Code. You can use it to generate tests, analyze the site, and run reports:

```bash
# In the Claude Code terminal
/analyze-site              # Discover site structure
/generate-full-suite       # Generate all POM classes + tests
/run-smoke                 # Run and display smoke results
/generate-report           # Summarize the last test run
/update-baseline           # Refresh visual regression baselines
```

See [`Skills.md`](../Skills.md) for full command documentation and [`Agent.md`](../Agent.md) for agent documentation.

---

## Questions?

Open a GitHub issue with the `question` label, or reach out to the repo maintainer.

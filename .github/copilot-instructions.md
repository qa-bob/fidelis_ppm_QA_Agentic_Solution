# GitHub Copilot Instructions

This file provides context for GitHub Copilot working in this repository. For full project instructions, see [`CLAUDE.md`](../CLAUDE.md) and [`AGENTS.md`](../AGENTS.md).

---

## Project

**Playwright + TypeScript regression test suite** for [getfidelis.com](http://getfidelis.com/) using the Page Object Model (POM) and OOP design patterns. All test behavior is driven from `site.config.json`.

## Architecture

- Page objects live in `src/pages/` — one class per page, all extending `BasePage`
- Tests live in `tests/<category>/` — organized by `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`
- Fixtures in `src/fixtures/site.fixture.ts` supply page objects to tests via dependency injection
- TypeScript strict mode is enabled

## Key Rules

- **Never put `expect()` inside page objects** — assertions go in tests only
- **Never use `page.locator()` in test files** — all locators belong in page objects
- **Never submit forms** — validate fields, not submission behavior
- **Never hardcode URLs** — always use `baseURL` from the config
- **Never use `page.waitForTimeout()`** — use Playwright's built-in auto-waiting
- **Always tag tests** — every test needs at least one of: `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`
- **Always run `npx tsc --noEmit`** after writing TypeScript

## Locator Priority

1. `getByRole()` — most resilient
2. `getByLabel()`, `getByPlaceholder()`, `getByText()`
3. `locator('[data-testid="..."]')`
4. `locator('[class*="..."]')` — class substring match
5. CSS/XPath — last resort

## Example Page Object

```typescript
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

## Example Test

```typescript
import { test, expect } from '@fixtures/site.fixture';

test.describe('Features Page @functional', () => {
  test.beforeEach(async ({ featuresPage }) => {
    await featuresPage.navigateToFeatures();
  });

  test('feature cards are present @functional', async ({ featuresPage }) => {
    await expect(featuresPage.featureCards.first()).toBeVisible();
    expect(await featuresPage.featureCards.count()).toBeGreaterThan(0);
  });
});
```

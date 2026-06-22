---
paths:
  - "src/pages/**/*.ts"
  - "tests/**/*.spec.ts"
  - "playwright.config.ts"
---

# Playwright Rules

Rules that apply when writing Playwright page objects and test files.

---

## Locator strategy (priority order)

1. `getByRole()` — semantic, accessible, most resilient
2. `getByLabel()`, `getByPlaceholder()`, `getByText()` — human-readable
3. `locator('[data-testid="..."]')` — if test IDs are present in the DOM
4. `locator('[class*="..."]')` — class substring match (more resilient than exact class)
5. CSS/XPath — last resort; document why a better selector was not possible

## Page objects

- All locators are declared as `readonly Locator` class properties — never create locators inside method bodies
- Methods perform exactly one user action and call `waitForLoad()` after navigation
- No `expect()` inside page objects — assertions belong in tests only
- Extend `BasePage` from `src/pages/base.page.ts` — do not duplicate shared helpers

## Waiting

- Never use `page.waitForTimeout()` for test synchronization
- Prefer `waitForSelector`, `waitForURL`, or `waitForLoadState('networkidle')`
- Playwright's `expect(locator).toBeVisible()` auto-waits — use it in tests instead of manual waits

## Visual regression

- Screenshot tests use `toHaveScreenshot()` with descriptive names: `homepage-desktop.png`
- Always update baselines with `npm run baseline` after intentional visual changes
- Do not commit screenshot baselines for pages with random or animated content (`skipVisual: true` in config)

## Timeouts

- Default test timeout is 30 seconds (set in `playwright.config.ts`) — do not override per-test unless the action genuinely requires it
- Navigation timeouts: use `{ waitUntil: 'domcontentloaded' }` for fast pages, `'networkidle'` for SPAs

## Multi-viewport

- Tests run against three Playwright projects: `chromium-desktop` (1280×720), `mobile-chrome` (390×844), `tablet` (768×1024)
- Responsive tests (`@responsive`) must explicitly check the viewport using `page.viewportSize()`
- Do not hard-code viewport dimensions in test assertions — read them from the Playwright project config

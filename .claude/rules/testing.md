# Testing Rules

Rules that apply when Claude reads or writes files in `tests/` or `src/`.

---

- Every test must be tagged with at least one of: `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`, or `@custom`
- Tests are organized by category directory: `tests/smoke/`, `tests/navigation/`, `tests/forms/`, `tests/functional/`, `tests/visual/`, `tests/responsive/`
- One spec file per feature area — do not put unrelated tests in the same file
- Every `test.describe` block must have a clear name matching the page or feature being tested
- Tests must be independent — no shared state, no ordering dependencies
- Use `test.beforeEach` for navigation setup, not inside individual test bodies
- Never use `page.waitForTimeout()` — use `waitForSelector`, `waitForURL`, or Playwright's built-in auto-waiting
- Never submit forms — test field interactions and validation only
- Never create accounts, log in, or enter real credentials
- Import page objects via the custom fixture: `import { test, expect } from '@fixtures/site.fixture'`
- Never call `page.locator()` directly in a test body — all selectors belong in page objects
- Do not hardcode URLs — always use `baseURL` from the Playwright config or `siteConfig.url` from the fixture

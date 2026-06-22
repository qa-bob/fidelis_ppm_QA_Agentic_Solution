# AGENTS.md — AI Agent Instructions

This file provides instructions for AI coding agents (GitHub Copilot, Cursor, OpenAI Codex, etc.) working in this repository. **Claude Code reads `CLAUDE.md` instead** — that file is the authoritative source of project instructions for Claude.

> If you are using Claude Code, read [`CLAUDE.md`](./CLAUDE.md). This file exists for cross-tool compatibility.

---

## Project Overview

This is a **Playwright + TypeScript regression test suite** for [getfidelis.com](http://getfidelis.com/). It uses the **Page Object Model (POM)** and **Object-Oriented Programming (OOP)** patterns. Tests cover smoke, functional, visual, navigation, forms, and responsive categories.

The site under test is defined in `site.config.json`. All test behavior is driven from that config.

---

## Architecture Rules

### Page Object Model
- Every page or major section has its own class in `src/pages/`
- All classes extend `BasePage` from `./base.page`
- Locators are `readonly Locator` properties on the class
- Methods represent **user actions only** — no assertions
- Never put `expect()` inside page object methods

### Tests
- Import page objects via `src/fixtures/site.fixture.ts` — never use raw `page.locator()` in test bodies
- Every test must have at least one tag: `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`
- Never hardcode URLs — use `baseURL` from Playwright config
- Never submit forms — test field interactions and validation only
- Never create accounts or enter real credentials

### TypeScript
- Strict mode is enabled — no `any` without explicit justification
- All page object properties must be typed
- Run `npx tsc --noEmit` before finishing any change

---

## Key Files

| File | Purpose |
|------|---------|
| `site.config.json` | Site URL and test flags |
| `playwright.config.ts` | Browser projects (desktop, mobile, tablet) |
| `src/pages/base.page.ts` | Abstract base class for all page objects |
| `src/fixtures/site.fixture.ts` | Playwright custom fixtures |
| `tests/smoke/` | @smoke tests |
| `tests/navigation/` | @navigation tests |
| `tests/forms/` | @forms tests |
| `tests/functional/` | @functional tests |
| `tests/visual/` | @visual screenshot tests |
| `tests/responsive/` | @responsive layout tests |

---

## Do Not

- Submit any form
- Create accounts or log in (unless `auth.required: true` in config)
- Hardcode the base URL in tests
- Put assertions inside page object methods
- Use `page.waitForTimeout()` — use `waitForSelector` or Playwright auto-waiting
- Use `any` type without justification
- Write tests outside the `tests/` directory

---

## Available npm Scripts

```bash
npm test                    # Run all tests
npm run test:smoke          # @smoke tests only
npm run test:navigation     # @navigation tests only
npm run test:forms          # @forms tests only
npm run test:visual         # @visual tests only
npm run test:responsive     # @responsive tests only
npm run baseline            # Update visual snapshots
npm run lint                # ESLint
npm run typecheck           # TypeScript check
```

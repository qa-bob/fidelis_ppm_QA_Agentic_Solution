---
paths:
  - "src/**/*.ts"
  - "tests/**/*.ts"
  - "*.ts"
---

# TypeScript Rules

Rules that apply to all TypeScript files in this project.

---

- Strict mode is enabled — `"strict": true` in `tsconfig.json`. Never disable it.
- Never use `any` without an explicit comment explaining why
- All `Locator` properties on page objects must be `readonly`
- Page object constructors must type the `page` parameter as `Page` from `@playwright/test`
- All async methods must return typed promises: `Promise<void>`, `Promise<string>`, etc.
- Use path aliases defined in `tsconfig.json`: `@pages/*`, `@utils/*`, `@types/*`, `@fixtures/*`
- Run `npx tsc --noEmit` after every set of TypeScript changes — do not finish until it passes cleanly
- Do not use `require()` — use ES module `import` syntax throughout
- Exported types and interfaces belong in `src/types/`
- All page object classes must be exported with `export class`

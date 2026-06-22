# Skills.md — Claude Code Slash Commands Reference

This document describes all **slash commands** (skills) available in this repository. Slash commands are defined in `.claude/commands/` and are invocable from the Claude Code CLI with a `/` prefix.

For agent definitions (subagents Claude can spawn), see [`Agent.md`](./Agent.md).

---

## What Are Skills?

Skills are Markdown files in `.claude/commands/` that define repeatable workflows Claude executes on demand. They are invoked with a `/` prefix in the Claude Code terminal or chat interface.

Skills load **on demand** — they are not injected into every session. This keeps context clean and avoids token overhead.

Command files live at:
```
.claude/commands/
  analyze-site.md
  generate-full-suite.md
  generate-report.md
  run-smoke.md
  update-baseline.md
```

---

## Available Commands

### `/generate-full-suite`

**File:** `.claude/commands/generate-full-suite.md`

**What it does:** The primary command. Analyzes the live website and generates a complete, production-quality Playwright + TypeScript test suite.

**Steps:**
1. Reads `site.config.json` to get the URL and flags
2. Uses `WebFetch` to crawl the homepage and all linked pages
3. Records: page titles, nav links, forms, interactive elements, content sections
4. Plans and creates page object classes (`src/pages/`)
5. Updates `src/fixtures/site.fixture.ts` with new page objects
6. Writes test files across all six categories: smoke, navigation, forms, functional, visual, responsive
7. Updates `site.config.json` with discovered values
8. Runs `npx tsc --noEmit` and fixes any errors
9. Reports a summary: pages analyzed, classes created, test count, inaccessible pages

**Output:**
- New/updated files in `src/pages/`
- New/updated files in `tests/`
- Updated `site.config.json`
- TypeScript compilation passing

**Usage:**
```
/generate-full-suite
```

---

### `/analyze-site`

**File:** `.claude/commands/analyze-site.md`

**What it does:** Inspects the site defined in `site.config.json` and produces a structured report of pages, forms, navigation, and detectable elements. Does not write any test code.

**Output:**
- Canonical URL (after redirects)
- Meta description and OG tags
- Navigation structure (all nav links)
- Forms detected (fields, types, required flags)
- Interactive elements (accordions, carousels, modals, video)
- Industry inference
- Auth requirements
- Recommended updates to `site.config.json`

**Usage:**
```
/analyze-site
```

---

### `/run-smoke`

**File:** `.claude/commands/run-smoke.md`

**What it does:** Executes the smoke test suite (`npm run test:smoke`) and displays a formatted summary of results.

**Output:**
- Pass/fail count
- Names of any failed tests
- Console error summary (if any)
- Recommendation to run `/generate-full-suite` if baseline is missing

**Usage:**
```
/run-smoke
```

---

### `/update-baseline`

**File:** `.claude/commands/update-baseline.md`

**What it does:** Runs `npm run baseline` to refresh all visual regression snapshots. Use this after intentional visual changes to the site to prevent false positives.

**When to use:**
- After a site redesign or intentional visual change
- When onboarding the test suite to a new environment
- When the visual baseline is stale

**Usage:**
```
/update-baseline
```

---

### `/generate-report`

**File:** `.claude/commands/generate-report.md`

**What it does:** Generates a human-readable summary of the most recent test run from Playwright's JSON output.

**Output:**
- Total tests: passed / failed / skipped
- Per-category breakdown
- Failed test names with error excerpts
- Recommendations for next steps

**Usage:**
```
/generate-report
```

---

## Typical Workflow

```
# First time on a new site
/analyze-site              # Discover site structure
/generate-full-suite       # Build the full test suite

# Ongoing maintenance
/run-smoke                 # Quick health check
/generate-report           # After a full test run

# After site redesign
/analyze-site              # Re-check structure
/update-baseline           # Refresh visual snapshots
```

---

## Creating a New Skill

1. Create `.claude/commands/<command-name>.md`
2. Write the command instructions in plain Markdown
3. The file name (without `.md`) becomes the slash command: `/command-name`
4. Include:
   - A clear description of what the command does
   - Step-by-step instructions
   - Expected output or side effects
   - Any caveats or guard conditions

**Naming convention:** Use kebab-case. Keep names short and verb-first (e.g., `run-smoke`, `generate-report`).

**Size guidance:** Keep command files under 200 lines. For complex multi-phase workflows, break the work into agents (`.claude/agents/`) and have the command invoke them.

---

## Rules for All Commands

- Always read `site.config.json` first
- Use `WebFetch` to inspect the live site before writing selectors
- Never submit forms
- Never hardcode URLs — use `baseURL` from Playwright config
- Run `npx tsc --noEmit` after generating any TypeScript files
- Report what was done and what was skipped

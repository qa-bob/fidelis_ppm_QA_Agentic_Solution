# Agent.md — Claude Code Agent Reference

This document describes all Claude Code **subagents** defined in `.claude/agents/` and how to use them. Agents are specialized AI workers that Claude can invoke to perform specific tasks autonomously.

For slash commands (skills), see [`Skills.md`](./Skills.md).

---

## What Are Claude Code Agents?

Agents are defined as Markdown files in `.claude/agents/`. Each file gives a subagent a name, a role, defined capabilities, and step-by-step instructions. Claude Code can spawn these agents to perform work autonomously, in parallel with other agents.

Agent files live at:
```
.claude/agents/
  site-analyzer.md      # Crawls the site, generates site.config.json
  test-generator.md     # Generates site-specific Playwright test files
```

---

## Available Agents

### `site-analyzer`

**File:** `.claude/agents/site-analyzer.md`

**Role:** Crawls a live website and produces a fully-populated `site.config.json`.

**When to invoke:**
- Onboarding a new site (first-time config generation)
- Verifying an existing `site.config.json` after a site redesign
- Running the `/analyze-site` slash command

**What it does:**
1. Resolves the canonical URL (follows redirects)
2. Navigates with `networkidle` and dismisses cookie banners
3. Extracts navigation items, forms, industry keywords
4. Detects contact forms, auth requirements, and animation-heavy pages
5. Outputs a valid `site.config.json` with a confidence rating

**Output:**
```json
{
  "name": "Fidelis PPM",
  "url": "https://getfidelis.com",
  "description": "...",
  "industry": "...",
  "hasContactForm": true,
  "expectedNavItems": ["Home", "Features", "Pricing", "Contact"],
  "viewports": ["desktop", "mobile", "tablet"],
  "skipVisual": false,
  "skipForms": false,
  "auth": { "required": false, "loginUrl": "", "username": "", "password": "" }
}
```

**Edge cases handled:** SPAs, auth-gated content, redirect chains, missing `<nav>` elements.

---

### `test-generator`

**File:** `.claude/agents/test-generator.md`

**Role:** Reads a populated `site.config.json` and generates site-specific Playwright tests beyond the shared suite.

**When to invoke:**
- The site has unique functionality not covered by the generic test suites
- A client requested additional coverage (pricing calculator, demo flow, blog pagination)
- Site structure is unusual enough that generic selectors fail
- Writing regression tests for a discovered bug

**What it does:**
1. Reads `site.config.json` to understand the site structure
2. Identifies gaps in the shared test suites
3. Plans test scenarios and outputs a brief plan before writing code
4. Generates page object additions if needed
5. Writes spec files to `tests/custom/<scenario-name>.spec.ts`

**Output format:**
- Files tagged `@custom` in addition to any relevant category tag
- JSDoc header explaining what is tested and why it is site-specific
- Follows all POM conventions: no `expect()` in page objects, no fixed timeouts > 500ms

**Example output file:** `tests/custom/pricing-page.spec.ts`

---

## How to Invoke Agents

### Via a slash command

Several slash commands automatically invoke agents:

```
/analyze-site          → invokes site-analyzer
/generate-full-suite   → invokes site-analyzer + test-generator in sequence
```

### Via a Claude Code prompt

```
"Run the site-analyzer agent on http://getfidelis.com and update site.config.json"

"Use the test-generator agent to write tests for the pricing page"
```

### Via the Agent tool (for orchestration)

Claude's `/generate-full-suite` command chains both agents:
1. `site-analyzer` runs first to discover the site structure
2. `test-generator` runs second, using the updated `site.config.json`

---

## Creating a New Agent

1. Create `.claude/agents/<agent-name>.md`
2. Define these sections:
   - `## Role` — what the agent does
   - `## When to invoke` — trigger conditions
   - `## Capabilities` — what tools/actions it can take
   - `## Inputs` — required and optional inputs with types
   - `## Output` — what it produces
   - `## Step-by-step instructions` — the exact workflow
3. Reference the agent in any relevant slash commands in `.claude/commands/`

Agent files use the same Markdown format as CLAUDE.md. The more specific the instructions, the more reliably the agent performs.

---

## Agent Conventions

| Convention | Rule |
|------------|------|
| No form submission | Agents must never submit a form |
| No account creation | Agents must never create accounts or enter real credentials |
| Real selectors | Agents must use actual HTML selectors from the live site, not placeholders |
| TypeScript validity | Generated code must pass `npx tsc --noEmit` |
| Tagging | Every generated test must have at least one `@tag` |
| POM compliance | Locators go in page objects; assertions go in tests |

---
name: validate-da-test
description: Validate Digital Archive E2E tests against the live application. Explores pages via Playwright MCP, audits selectors, runs regression, offers selective execution.
argument-hint: "[test-path|directory|pages/*.ts|all]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Edit
  - Write
  - Agent
  - Skill
  - mcp__playwright__browser_navigate
  - mcp__playwright__browser_snapshot
  - mcp__playwright__browser_take_screenshot
  - mcp__playwright__browser_evaluate
  - mcp__playwright__browser_click
  - mcp__playwright__browser_fill_form
  - mcp__playwright__browser_wait_for
---

Invoke the `da-test-validator` skill to validate Digital Archive E2E tests against the live application.

The target test suite lives at: `C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e\`

Pass $ARGUMENTS as the validation scope. Use the Skill tool to invoke `da-test-validator`.

If no arguments provided, ask the user what to validate:
- A specific test file path (e.g., `tests/archives.spec.ts`)
- A test directory (e.g., `tests/management/`)
- A POM file for selector audit only (e.g., `pages/archives-page.ts`)
- `all` for full suite validation

---
name: create-da-test
description: Create Digital Archive E2E tests by exploring the live app first. Supports feature-based or scenario-based input.
argument-hint: "[page-name|\"scenario description\"|page-name category]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
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

Invoke the `da-test-creator` skill to create Digital Archive E2E tests by exploring the live application first.

The target test suite lives at: `C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e\`

Pass $ARGUMENTS as the creation scope. Use the Skill tool to invoke `da-test-creator`.

If no arguments provided, ask the user what to test:
- A page name: `login`, `archives`, `management`, `graph`, `recycle-bin`, `shares`, `processes`
- A page + category: `archives search`, `management security`, `login validation`
- A specific scenario: `"test that uploading a PDF and searching for its content works"`

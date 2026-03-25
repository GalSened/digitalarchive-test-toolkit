---
name: da-test-validator
description: >
  Validates Digital Archive Playwright E2E tests against the live application using a 3-phase
  pipeline: (1) Explore pages via Playwright MCP to build element maps, (2) Audit POM
  selectors against live DOM with confidence-scored fix proposals, (3) Selective test
  execution controlled by the user. Use when user discusses validating UI tests, checking
  if E2E tests work, verifying selectors, auditing page objects, or says "validate this test",
  "check these selectors", "are these tests correct", "audit the POM".
---

# DA Test Validator — Orchestration Pipeline

## 1. YOU ARE

A Digital Archive UI test validation specialist. You orchestrate a 3-phase pipeline using specialized agents to validate Playwright E2E tests against the live Digital Archive application.

You are an ORCHESTRATOR — you dispatch agents, you do not do exploration or auditing yourself.

## 2. MCP BROWSER vs PLAYWRIGHT TEST BROWSER — IMPORTANT CAVEAT

The Playwright MCP browser (used by agents for exploration) and the Playwright test browser (used by `npx playwright test`) are SEPARATE processes:

| Aspect | MCP Browser | Playwright Test |
|--------|-------------|-----------------|
| Viewport | Default | Per config (devices) |
| Locale | Default | en-US (from config) |
| Timeout | Default | 60s test / 10s expect |
| Auth | Manual login | Via fixture |

MCP validation provides ~90% confidence. Phase 3 (test execution) provides the remaining 10%. This is why Phase 3 exists — it is NOT redundant.

## 3. PRE-FLIGHT CHECKS (before anything else)

**Check 1: Playwright MCP**
Try using `mcp__playwright__browser_snapshot`. If it fails, tell user: "Playwright MCP server required. Please start it before running validation."

**Check 2: Read credentials**
Read env files in order (use Read tool):
1. `C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e\.env`
2. Fallback: `C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive.env`

Parse DA_BASE_URL, DA_USERNAME, DA_PASSWORD, DA_TENANT.
If neither exists or creds empty, tell user: "Digital Archive credentials not found. Create e2e/.env from .env.example and fill in credentials."

**Check 3: Parse scope from $ARGUMENTS**
- File path (e.g., `tests/archives.spec.ts`) — validate that specific test file
- Directory (e.g., `tests/management/`) — validate all tests in that directory
- POM path (e.g., `pages/archives-page.ts`) — audit POM selectors only, skip Phase 3
- `all` — full suite validation (all pages, all POMs, all tests)
- Empty — ask user what to validate

## 4. PHASE 1: EXPLORE

Determine which pages the target tests reference:
- Use Grep tool to find `from.*pages/` imports in the target test files
- Map each import to a page area (archives-page -> archives, management-page -> management, etc.)

For each referenced page, dispatch the `archive-explorer` agent using the **Agent tool**:

```
Use Agent tool:
  description: "Explore {page_name} page"
  prompt: |
    You are the archive-explorer agent. Navigate to Digital Archive and explore the {page_name} page.

    BASE_URL: {base_url}
    Username: {username}
    Password: {password}
    Tenant: {tenant}
    Target page: {page_name} ({url})

    Follow your full exploration protocol:
    1. Authenticate (if not already logged in)
    2. Navigate to the target page
    3. Take snapshot + screenshot
    4. Extract DOM structure via browser_evaluate
    5. Explore interactive states (modals, context menus)
    6. Return the structured page map
```

Collect the page map and screenshots from each explorer dispatch.

## 5. PHASE 2: COLLECT

### 5a. Selector Audit

For each POM file referenced by target tests, dispatch the `selector-auditor` agent:

```
Use Agent tool:
  description: "Audit {pom_file} selectors"
  prompt: |
    You are the selector-auditor agent. Validate all selectors in {pom_file_path}.

    The MCP browser is already authenticated and on the {page_name} page.

    POM file to audit: {full_path_to_pom}

    Page map from explorer (element inventory):
    {paste relevant page map summary}

    Follow your full audit protocol:
    1. Read the POM file
    2. Extract all selectors
    3. Evaluate each against the live DOM
    4. Classify each (VALID/MISSING/HIDDEN/STALE)
    5. Propose fixes for MISSING with confidence scores
    6. Check for wholesale fragility
    7. Return the audit report
```

### 5b. Apply HIGH Confidence Fixes (with user approval)

If the audit found MISSING selectors with HIGH confidence fixes:
1. Present a dry-run preview to the user:
   ```
   Proposed fixes (HIGH confidence -- require your approval):
     1. e2e/pages/archives-page.ts:18
        Old: page.locator('.search-box')
        New: page.locator('input[name="search"]')
        Confidence: 92% -- same input, same placeholder text

     Apply these fixes? [Y/n]
   ```
2. On approval, apply fixes using the Edit tool
3. After applying, dispatch `regression-runner` agent:
   ```
   Use Agent tool:
     description: "Regression check for {pom_file}"
     prompt: |
       You are the regression-runner agent. POM file {pom_file} was modified.
       Changes: {summary of fixes applied}
       Working directory: C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e\

       Follow your full regression protocol:
       1. Find all test files importing this POM
       2. Collect all affected tests
       3. Run affected tests
       4. Classify any failures
       5. Report recommendation (SAFE or REVERT)
   ```
4. If regression reports REVERT NEEDED, undo the fixes and report the conflict to user

### 5c. Collection Check

Run via Bash:
```bash
cd "C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e" && npx playwright test --list {target_test_paths} 2>&1
```

## 6. PHASE 3: SELECTIVE EXECUTE

Present the full validation report to the user:

```
+======================================+
|  DA TEST VALIDATION REPORT           |
+======================================+

PHASE 1 -- Exploration:
  Pages explored: {list}
  Screenshots taken: {count}

PHASE 2 -- Selector Audit:
  Selectors audited: {total}
    VALID: {N} | VALID_MULTI: {N} | MISSING: {N} | HIDDEN: {N}
  Fixes applied: {N} (HIGH confidence, user-approved)
  Regression status: {CLEAR | N/A}

  Tests collected: {N}/{N} ({errors} errors)

PHASE 3 -- Ready to execute:
  1. Run all {N} tests
  2. Run smoke tests only
  3. Run specific test file (enter path)
  4. Skip execution

Which option?
```

Execute the user's choice:
```bash
cd "C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e" && npx playwright test {selection} --reporter=list 2>&1
```

Report results. For failures: include test name, error message, and take a screenshot if possible.

## 7. ERROR HANDLING

| Situation | Action |
|-----------|--------|
| Digital Archive unreachable | Abort at Phase 1 with "DA unreachable" |
| Login fails | Report "Authentication failed" and abort |
| Playwright MCP not connected | Detect at pre-flight, report requirement |
| No POM for target page | Report "No POM found for {page}", skip audit |
| Collection fails | Report error details, suggest fixes |
| All selectors pass but tests fail | Phase 3 catches this -- report execution results |

## IMPORTANT RULES

- Do NOT create git commits
- This skill ORCHESTRATES — it dispatches agents, it does not do exploration or auditing itself
- Always present dry-run preview before applying POM fixes
- Always get user approval before execution in Phase 3
- If invoked automatically by da-test-creator, still present the report but default to "run all"

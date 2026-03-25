---
name: da-test-inventory
description: >
  Generates or updates the bilingual test inventory for the Digital Archive E2E test suite.
  Runs generate_inventory.py with --app digitalarchive to produce the HTML inventory report.
  Also validates that all test titles follow the "Verify that..." convention required for
  Hebrew translation. Use when user says "generate inventory", "update inventory",
  "build test inventory", "inventory report", or "test catalog".
---

# DA Test Inventory — Generation & Validation

## 1. YOU ARE

A test inventory specialist for the Digital Archive test suites. You generate bilingual (English + Hebrew) HTML inventory reports and ensure all tests follow the naming conventions required by the inventory generator.

## 2. CRITICAL RULES

- All Playwright test titles MUST start with "Verify that..." for the Hebrew translator to work
- The inventory generator lives at: `C:\Users\gals\repos\comsigntrust-automation-tests\generate_inventory.py`
- The E2E suite lives at: `C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e\`
- Output HTML goes to: `C:\Users\gals\repos\comsigntrust-automation-tests\docs\inventory-digitalarchive.html`

## 3. PIPELINE

### Step 1: VALIDATE test title conventions

Scan all `.spec.ts` files for test titles that do NOT start with "Verify that":

```bash
cd "C:\Users\gals\repos\comsigntrust-automation-tests\apps\digitalarchive\digitalarchive-e2e"
grep -rn "test('" tests/ | grep -v "Verify that" | grep -v "test.describe" | grep -v "test.beforeEach" | grep -v "test.skip"
```

If any violations found:
1. List each violation with file:line and current title
2. Propose the corrected "Verify that..." title
3. Ask user to approve before fixing
4. Apply fixes with Edit tool

### Step 2: GENERATE inventory

Run the inventory generator for the Digital Archive app:

```bash
cd "C:\Users\gals\repos\comsigntrust-automation-tests"
python generate_inventory.py --app digitalarchive
```

### Step 3: REPORT

Present the results:

```
INVENTORY GENERATION REPORT
===========================

Title Convention:
  Total tests scanned: {N}
  Following "Verify that..." pattern: {N}
  Violations fixed: {N}

Generated Output:
  File: docs/inventory-digitalarchive.html
  Suites included: {list}
  Total scenarios: {N}

  Per suite:
    - Digital Archive API: {N} scenarios
    - Browser E2E Tests (Playwright): {N} scenarios
```

### Step 4: OPEN (optional)

If user wants to preview, open the HTML file:

```bash
start "C:\Users\gals\repos\comsigntrust-automation-tests\docs\inventory-digitalarchive.html"
```

## 4. ADDING NEW TESTS TO INVENTORY

When new tests are added (via `/create-da-test` or manually), this skill should be run to:
1. Validate the new tests follow naming conventions
2. Regenerate the inventory to include them
3. Report the updated counts

## 5. TEST TITLE CONVENTION

Every Playwright `test()` title MUST start with "Verify that" followed by a description:

**Good:**
```typescript
test('Verify that the login page loads with form visible', async () => {
test('Verify that creating a user with required fields adds it to the table', async () => {
test('Verify that XSS payloads in the search field are not executed', async () => {
```

**Bad (will produce broken Hebrew):**
```typescript
test('page loads with table visible', async () => {
test('create user and verify in table', async () => {
test('XSS in search does not break the page', async () => {
```

## 6. TypeScript PARSER

The inventory generator includes a TypeScript parser (`extract_ts_test_data()`) that:
- Finds all `.spec.ts` files recursively
- Parses `test.describe('Title', ...)` as test class groups
- Parses `test('Title', ...)` as individual test entries
- Feeds titles through `translate_to_hebrew()` for bilingual output
- Classifies tests using the same `classify_tests()` logic as Python suites

## IMPORTANT RULES

- Do NOT create git commits — only generate the inventory and report results
- Always validate naming conventions BEFORE generating
- If violations exist, fix them first, then generate

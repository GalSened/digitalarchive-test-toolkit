---
name: review-da-tests
description: Static code review of DA E2E tests against actual source code. Validates selectors, assertions, flows, and boundaries without a running app.
argument-hint: "[file-path|all]"
allowed-tools:
  - Read
  - Edit
  - Glob
  - Grep
  - Bash
  - Agent
---

Invoke the `da-test-review` skill to perform static validation of Digital Archive E2E tests.

Cross-references test code against React source (components, schemas, repositories, i18n) to verify correctness without a running app.

The 6-layer method:
1. Selector verification (POM selectors → JSX elements)
2. Assertion correctness (expect() → actual component behavior)
3. Flow verification (multi-step tests → handler chains)
4. Validation boundary accuracy (boundary values → yup schemas)
5. False test detection (tests that always pass regardless)
6. Business logic completeness (missing coverage gaps)

Pass $ARGUMENTS as scope:
- `all` — review entire test suite
- A specific file path — review that file only
- No arguments — review all and generate full report

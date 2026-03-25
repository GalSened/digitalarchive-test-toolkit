---
name: generate-da-inventory
description: Generate or update the bilingual test inventory report for Digital Archive test suites. Validates naming conventions and produces HTML output.
argument-hint: "[--fix|--validate-only|--open]"
allowed-tools:
  - Read
  - Edit
  - Glob
  - Grep
  - Bash
---

Invoke the `da-test-inventory` skill to generate the Digital Archive test inventory.

The inventory generator lives at: `C:\Users\gals\repos\comsigntrust-automation-tests\generate_inventory.py`
The output goes to: `C:\Users\gals\repos\comsigntrust-automation-tests\docs\inventory-digitalarchive.html`

Use the Skill tool to invoke `da-test-inventory`.

Pass $ARGUMENTS as options:
- No arguments: validate conventions + generate inventory
- `--fix`: auto-fix naming convention violations before generating
- `--validate-only`: only check naming conventions, don't generate
- `--open`: generate and open the HTML report in browser

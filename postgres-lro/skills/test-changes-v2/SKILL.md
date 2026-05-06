---
name: test-changes-v2
description: Run the complete test suite for the entire repository using make test. This runs all tests including database tests and is more thorough than test-changes.
trigger: When the user explicitly asks to run all tests, run the full test suite, or wants comprehensive test coverage after changes
---

## Instructions

1. **Analyze changes**:
   - Run `git diff --stat` to see what changed
   - Provide a brief summary of the modified files

2. **Run full test suite**:
   ```bash
   make test
   ```
   - This runs the complete test suite for the entire repository
   - Includes all database tests, integration tests, and unit tests
   - Uses the project's test infrastructure (with-postgres.sh script)

3. **Report**:
   - Show which files were changed
   - Display the full test output
   - Report pass/fail status with detailed output
   - If tests fail, highlight the failures and suggest next steps

## When to use

- After completing a plan or major feature implementation
- When explicitly asked to run all tests or the full test suite
- Before creating a pull request
- When you need comprehensive test coverage validation
- When targeted testing (test-changes) isn't sufficient

## When NOT to use

- During rapid iteration/development (use `/test-changes` instead)
- For quick validation of small changes
- When only specific packages need testing

## Examples

**Full test suite**:
```bash
make test
```

**Test specific package** (if PKG parameter is supported):
```bash
make test PKG=lro
```

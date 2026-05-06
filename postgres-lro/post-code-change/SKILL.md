---
name: post-code-change
description: Automatically run the complete post-code-change workflow (update godocs, add unit tests, run full test suite) after modifying Go files
trigger: After any code changes to Go files, or when user explicitly requests the complete workflow
---

## Instructions

This skill orchestrates the complete post-code-change workflow for Go code modifications. It runs three skills in sequence to ensure code quality, documentation, and test coverage.

### Workflow Steps

Execute these skills in the following order:

1. **Update godocs** (`/update-godocs`)
   - Updates documentation for modified functions
   - Ensures godocs accurately reflect implementation
   - Follows Go documentation conventions

2. **Add/update unit tests** (`/add-unit-tests`)
   - Generates or updates unit tests for changed code
   - Ensures test coverage for all code paths
   - Uses proper test patterns for Go

3. **Run full test suite** (`/test-changes-v2`)
   - Runs the complete test suite for the entire repository
   - Uses `make test` to run all tests including database tests
   - Provides comprehensive validation of all changes

### Execution

Run each skill sequentially using the Skill tool:

```
Skill(skill: "update-godocs")
Skill(skill: "add-unit-tests")
Skill(skill: "test-changes-v2")
```

Wait for each skill to complete before starting the next one, as they may depend on previous steps.

### When to Use

- After implementing new functions or methods
- After modifying existing Go code
- After refactoring
- When user explicitly runs `/post-code-change`
- As part of the pre-commit workflow

### When NOT to Use

- For non-Go files
- When only documentation files are changed
- When user explicitly asks to skip the workflow
- For trivial changes like formatting or comments

### Success Criteria

The workflow succeeds when:
1. All godocs are updated and accurate
2. Unit tests exist for changed code paths
3. All tests in the complete test suite pass

### Error Handling

If any skill in the workflow fails:
1. Report which skill failed
2. Show the error message
3. Ask user how to proceed (fix and retry, skip, or abort)

## Notes

- This skill is designed to be run automatically after code changes
- It ensures consistency in documentation, testing, and code quality
- Running this workflow before committing helps catch issues early
- The workflow runs the full test suite to ensure comprehensive validation of all changes

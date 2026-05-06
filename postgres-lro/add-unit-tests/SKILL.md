---
name: add-unit-tests
description: Generate or update unit tests for modified/new functions, ensuring all code paths are covered with proper test patterns
trigger: When functions are added or modified and need test coverage, or when user explicitly requests test generation
---

## Goal

Generate comprehensive unit tests for changed functions by analyzing code paths and creating test cases that cover happy paths, error cases, and edge cases.

## Workflow

1. Use `git diff` to identify modified/new functions
2. Read function implementations to understand code paths
3. Find existing tests in `*_test.go` files and identify gaps
4. Generate test cases for uncovered paths
5. Write/update test files
6. Report what was generated

## Codebase Patterns

**Naming**:
- Methods: `TestType_MethodName`
- Functions: `TestFunctionName`
- Error cases: append suffix (e.g., `TestService_ClaimWorkItem_NoWorkItem`)

**Structure**:
- Use table-driven tests with `t.Run()` for subtests
- Test struct fields: `name`, `input`, `setupMock`, `expected`, `expectError`

**Mocking**:
- Function-based mocks: `type MockClient struct { SomeMethodFunc func(...) ... }`
- NO external libraries (no testify/mockery)

**Assertions**:
- Manual `if err != nil` checks with `t.Fatalf()`
- For gRPC: check `status.FromError(err)` and `st.Code()`

**Coverage**:
- Happy path + all error paths
- Edge cases: nil inputs, empty values, boundaries
- All conditional branches

## Notes

- NO external assertion libraries
- Preserve existing tests, only add missing scenarios
- This skill only generates tests; use `/test-changes` to run them

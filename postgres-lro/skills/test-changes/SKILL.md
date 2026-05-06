---
name: test-changes
description: Run targeted Go tests only for code paths that were modified, minimizing test execution time. Automatically detects changed files and runs tests for affected packages.
trigger: After implementing a plan, or when the user asks to test their changes quickly
---

## Instructions

1. **Analyze changes**:
   - Run `git diff --stat` and `git diff` to see what changed
   - Categorize: business logic, database operations, test code, mocks/helpers, or deletions

2. **Database test detection**:
   - `repository_test.go` contains slow database tests (~5-6 min)
   - Run database tests if:
     - `repository.go` was modified, OR
     - Changes in `service.go` (or other files) modify calls to repository functions
     - Changes affect the call chain leading to database operations
   - Skip with `-skip "TestDB|TestRepository"` if changes don't touch repository or its call chain
   - To detect: check `git diff -U0` for repository method calls in modified functions

3. **Determine test strategy**:
   - **Business logic code**: Parse `git diff -U0` for function names in `@@` headers, find matching test functions
   - **Database operations**: Modified `repository.go` or calls to repository methods → run database tests
   - **Test files**: Run only modified test functions
   - **Mocks/helpers**: Verify compilation with `go build`, or run dependent tests if actively used
   - **Deletions**: Verify unused with grep, then just compile

4. **Run tests**:
   ```bash
   # Non-repository changes (skip database tests)
   go test -timeout 10m -v -skip "TestDB|TestRepository" -run <TestPattern> ./lro
   
   # Repository changes (include database tests)
   go test -timeout 10m -v -run <TestPattern> ./lro
   
   # Compilation only
   go build ./lro
   ```

5. **Report**:
   - State change type detected
   - List tests run or if just compiled
   - Show pass/fail (detailed output only on failure)

## Examples

**Business logic code**:
```bash
go test -timeout 10m -v -skip "TestDB|TestRepository" -run "TestResolveVersion|TestEnqueueWorkItem" ./lro
```

**Database operations**:
```bash
go test -timeout 10m -v -run TestClaimWorkItem ./lro
```

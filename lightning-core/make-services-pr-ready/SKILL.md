---
name: make-services-pr-ready
description: Prepare services for PR by running build, check, and check-coverage, then fixing any issues. Usage: `/make-services-pr-ready [changed|lro|service-names...]` - defaults to changed services if no args provided.
---

# Make Services PR Ready

This skill prepares services for a pull request by running build, tests, linter checks, and coverage validation, then automatically fixing any issues found.

## Usage Modes

The skill supports three modes with explicit parameters:

1. **Changed Services Only** (default): Process only services with changes compared to main
   - Usage: `/make-services-pr-ready changed` or `/make-services-pr-ready`
   
2. **All LRO Services**: Process ALL services using postgres-lro, regardless of changes
   - Usage: `/make-services-pr-ready lro` or `/make-services-pr-ready all-lro`

3. **Specific Services**: Process an explicitly named list of services
   - Usage: `/make-services-pr-ready service-name-1 service-name-2 service-name-3`

### Parameter Parsing

- If no arguments provided: Default to "changed" mode
- If first argument is `lro` or `all-lro`: Use "all LRO services" mode
- If first argument is `changed`: Use "changed services only" mode
- Otherwise: Treat all arguments as service names for "specific services" mode

## Steps

### 1. Service Selection

**For "Changed Services Only" mode:**
```bash
git diff main --name-only | grep "^services/" | cut -d'/' -f2 | sort -u
```

**For "All LRO Services" mode:**
- Use the `list-lro-services` skill to identify all services using postgres-lro
- Extract both LRO services and worker services from the results

**For "Specific Services" mode:**
- Use the explicitly provided list of service names

### 2. Run Validation in Parallel

- Create a TodoWrite task list with all target services to track progress
- Launch validation commands for ALL services in parallel using `run_in_background: true`
- For each service, run:
  ```bash
  cd /Users/kunal.narang/code/lightning-core/services/<service-name> && VERBOSE=true make build && make check && make check-coverage 2>&1
  ```
- Use a single message with multiple Bash tool calls to launch all services concurrently
- The system will notify when each background task completes

### 3. Process Results as Tasks Complete

- When notified of task completion, read the output file from the task
- Check the exit code:
  - Exit code 0: Service passed all checks
  - Exit code != 0: Service has failures that need fixing

### 4. Handle Build Failures

- Read the build error output from the task output file
- Identify compilation errors (syntax errors, type mismatches, missing imports, etc.)
- Fix the issues in the affected files
- Re-run `make build` to verify the fix
- Continue until build succeeds

### 5. Handle Test Failures

Parse test output from `make check` to identify failing tests:
- Read the test files and implementation to understand the failure
- Fix the root cause in either the test or the implementation
- Re-run `make check` to verify the fix
- Continue until all tests pass

### 6. Handle Linter Warnings

Parse linter output (golangci-lint or similar) from `make check`:
- Fix each warning by:
  - Removing unused variables/imports
  - Fixing formatting issues
  - Addressing code quality issues
  - Handling error returns properly
- Re-run `make check` to verify the fix
- Continue until no warnings remain

### 7. Adjust Coverage Thresholds

After all background tasks complete, review all task outputs:

**For each service, check if it has code changes:**
```bash
git diff main --name-only | grep "^services/<service-name>/"
```

**ONLY proceed with coverage adjustment if the service has code changes:**
- Parse the output to find the actual coverage percentage
- Extract the coverage value (e.g., "coverage: 58.5% of statements")
- Calculate the correct threshold: `floor(actual_coverage) - 1.0`
  - Example: If actual is 58.5% or 58.0%, set threshold to 57.0%
  - Example: If actual is 61.7%, set threshold to 60.0%
- Compare with current threshold in the Makefile (look for `TEST_COVERAGE_THRESHOLD := XX.X`)
- If the current threshold doesn't match the formula, update it:
  - Read the Makefile
  - Edit the `TEST_COVERAGE_THRESHOLD` line
  - Verify with `make check-coverage`

**Note:** Skip coverage adjustment for services with no code changes, even in "All LRO Services" mode.

### 8. Progress Tracking

- Use TodoWrite to track progress across all services
- Create tasks for each service: "Process <service-name>"
- Update task status as background tasks complete
- Mark services as completed when all checks pass
- Report any services that couldn't be fixed automatically

## Coverage Threshold Update Details

When updating coverage thresholds in the Makefile:

**Find the threshold:** Look for patterns like:
- `go test -coverprofile=coverage.out -coverpkg=./... ./... && go tool cover -func=coverage.out | grep total | awk '{if ($$3+0 < 57.0) exit 1}'`
- Or similar patterns checking coverage percentage

**Parse actual coverage:** From output like:
```
total:                                          (statements)            58.5%
```
Extract the `58.5`

**Calculate new threshold:**
```
actual = 58.5
new_threshold = floor(58.5) - 1.0 = 58.0 - 1.0 = 57.0
```

**Update Makefile:** Replace the old threshold (e.g., `60.0`) with the new one (e.g., `57.0`)

## Parallel Execution Strategy

**ALWAYS run services in parallel** to maximize efficiency:

1. Launch ALL service validation commands in a **single message** with multiple Bash tool calls
2. Set `run_in_background: true` on all Bash calls
3. Use absolute paths when changing directories: `cd /Users/kunal.narang/code/lightning-core/services/<service-name>`
4. Redirect stderr to stdout with `2>&1` to capture all output
5. System will automatically notify when each task completes - do NOT poll or wait actively
6. Process results as notifications arrive
7. After all tasks complete, batch-update all coverage thresholds that need adjustment

### Example Parallel Launch

```
# After getting the list of services, launch them ALL in one message:
Bash(command: "cd /Users/kunal.narang/code/lightning-core/services/service-1 && VERBOSE=true make build && make check && make check-coverage 2>&1", run_in_background: true)
Bash(command: "cd /Users/kunal.narang/code/lightning-core/services/service-2 && VERBOSE=true make build && make check && make check-coverage 2>&1", run_in_background: true)
Bash(command: "cd /Users/kunal.narang/code/lightning-core/services/service-3 && VERBOSE=true make build && make check && make check-coverage 2>&1", run_in_background: true)
# ... continue for all services in the same message
```

This launches all services concurrently and processes results as they complete.

## Output Format

Provide a summary with:
- Mode used (based on arguments: `changed`, `lro`, or service names)
- List of services processed
- Status for each service:
  - ✅ Build: passing/fixed
  - ✅ Tests: passing/fixed
  - ✅ Linter: clean/fixed
  - ✅ Coverage: threshold updated to match actual coverage (or skipped if no changes)
- Summary of all fixes applied
- Any services requiring manual intervention

## Notes

- Always use absolute paths for service directories (e.g., `cd /Users/kunal.narang/code/lightning-core/services/di-segmentation-lro`)
- Use `VERBOSE=true` for better error messages
- Fix issues iteratively - don't batch fixes across multiple issues
- If a service can't be automatically fixed after 3 attempts, mark it for manual review
- Coverage thresholds should only be lowered, never raised by this skill
- Always verify fixes by re-running the failed command
- Coverage threshold adjustment only runs for services with actual code changes in the current branch
- When reading background task outputs, use the output file path provided in the task notification
- Batch all Edit calls for coverage threshold updates to improve efficiency

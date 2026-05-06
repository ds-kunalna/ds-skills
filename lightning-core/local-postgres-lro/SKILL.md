---
description: Replace postgres-lro dependency with local path for testing, or revert to remote version
---

# Local postgres-lro Testing

This skill helps you quickly switch between the remote and local versions of postgres-lro for testing.

## Usage

- `/local-postgres-lro` or `/local-postgres-lro enable` - Replace with local path and run go mod tidy
- `/local-postgres-lro disable` - Revert to remote version and run go mod tidy
- `/local-postgres-lro enable --no-tidy` - Replace without running go mod tidy
- `/local-postgres-lro disable --no-tidy` - Revert without running go mod tidy

## Implementation

Parse the args to determine the action and whether to skip tidy (default is to run tidy).

### Enable (default)

1. Read go.mod
2. Check if the replace directive already exists
3. If not, add this replace directive after the existing replace statements:
   ```
   // local development
   replace bitbucket.dentsplysirona.com/libgo/postgres-lro => /Users/kunal.narang/code/postgres-lro
   ```
4. Unless the `--no-tidy` flag is present:
   - Run `make generate-rpcs`
   - Delete `pkg/ortho-gateway` and `pkg/outsim-gateway` directories
   - Run `go mod tidy`
5. Report the change to the user

### Disable

1. Read go.mod
2. Remove the line containing `replace bitbucket.dentsplysirona.com/libgo/postgres-lro => /Users/kunal.narang/code/postgres-lro`
3. Also remove the `// local development` comment if it's on the line immediately before and has no other replace directives after it
4. Unless the `--no-tidy` flag is present:
   - Run `make generate-rpcs`
   - Delete `pkg/ortho-gateway` and `pkg/outsim-gateway` directories
   - Run `go mod tidy`
5. Report the change to the user

## Notes

- The local path is: `/Users/kunal.narang/code/postgres-lro`
- The remote path is: `bitbucket.dentsplysirona.com/libgo/postgres-lro`
- go mod tidy may fail due to unrelated dependency issues - this is not a problem with the replace directive itself

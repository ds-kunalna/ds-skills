---
name: update-godocs
description: Automatically update godocs for modified functions to accurately reflect their behavior, implementation, and parameters
trigger: After any code change to Go files, after plan implementation, or when user explicitly requests godoc updates
---

## Instructions

When updating godocs for modified code:

1. **Identify changed functions** by:
   - Using `git diff` to find modified Go files
   - Parsing the diff to find changed functions/methods
   - Look for `func` declarations that were added or modified
   - Extract function signatures including receivers, parameters, and return types

2. **Analyze function behavior** by:
   - Reading the complete function implementation
   - Understanding what the function does, not just what it's named
   - Identifying key behaviors: side effects, error conditions, special cases
   - Understanding parameter usage and return value meanings
   - Checking for any exported types, constants, or variables

3. **Update godocs following Go conventions**:
   - Start with the function/type/variable name (e.g., "ResolveVersion resolves...")
   - Describe what it does, not how (focus on behavior, not implementation)
   - Document parameters if their purpose isn't obvious
   - Document return values, especially error conditions
   - Keep it concise but complete - one paragraph for simple functions, more for complex ones
   - Use proper Go comment style: `// FunctionName does something`
   - For exported package-level items, ensure every exported element has a godoc

4. **Special cases**:
   - **Methods**: Include receiver context if relevant (e.g., "EnqueueWorkItem adds a work item to the service's queue")
   - **Constructors**: Document what the function creates and any initialization behavior
   - **Interfaces**: Document the contract, not the implementation
   - **Error returns**: Always document when and why errors are returned
   - **Nil handling**: Document if nil parameters are allowed or if nil can be returned

5. **Quality checks**:
   - Ensure godoc starts with the function name
   - Verify the description matches the actual implementation
   - Check that all exported functions, types, and variables have godocs
   - Run `go doc` to verify the documentation displays correctly
   - Ensure line length is reasonable (wrap at ~80-100 characters)

6. **Apply updates**:
   - Use the Edit tool to update godocs in place
   - Only modify the comment block above the function/type
   - Preserve existing useful context, update inaccurate parts
   - If no godoc exists, add one above the declaration

## Example

For a function like:
```go
func ResolveVersion(ctx context.Context, version string) (string, error) {
    if version == "latest" {
        return r.getLatestVersion(ctx)
    }
    return version, nil
}
```

Good godoc:
```go
// ResolveVersion resolves a version string to an exact version.
// If version is "latest", it queries for the most recent version.
// Returns the resolved version string or an error if resolution fails.
func ResolveVersion(ctx context.Context, version string) (string, error) {
```

Bad godoc:
```go
// ResolveVersion resolves version
func ResolveVersion(ctx context.Context, version string) (string, error) {
```

## Workflow

1. After code changes, detect modified .go files
2. For each file, identify changed functions/methods
3. Read and analyze each changed function
4. Draft improved godocs that accurately reflect behavior
5. Apply updates using Edit tool
6. Report what was updated

## Notes

- This skill runs automatically after code changes to keep docs in sync
- Focus on accuracy - godocs should match implementation, not aspirations
- Follow standard Go documentation practices per Effective Go
- Don't add godocs to unexported (private) functions unless they're complex
- Verify updates don't break `go build` or `golint` checks

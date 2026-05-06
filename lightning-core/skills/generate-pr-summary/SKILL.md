---
name: generate-pr-summary
description: Generate a concise summary of changes in the current branch compared to the main branch, suitable for PR descriptions
trigger: When the user asks to generate a PR summary, summarize changes, or prepare PR description
---

## Instructions

When asked to generate a PR summary:

1. **Identify the base branch**:
   - Check the current branch: `git branch --show-current`
   - Use `main` as the base branch (or ask user if different)

2. **Analyze the changes**:
   - Get commit history: `git log main..HEAD --oneline` to see all commits in the branch
   - Get file changes: `git diff --stat main...HEAD` to see which files changed
   - Get detailed diff: `git diff main...HEAD` to understand the nature of changes

3. **Categorize the changes**:
   - **New features**: New functions, files, or functionality added
   - **Bug fixes**: Fixes to existing issues or incorrect behavior
   - **Refactoring**: Code improvements without behavior changes
   - **Tests**: New or updated test coverage
   - **Documentation**: Changes to comments, godocs, or README files
   - **Configuration**: Changes to build files, configs, or dependencies

4. **Generate the summary** with this structure:
   ```markdown
   ## Summary
   [1-2 sentence overview of what this PR does]

   ## Changes
   - [Bullet point for each major change, grouped by category]
   - [Focus on WHAT and WHY, not implementation details]
   - [Use present tense: "Add", "Fix", "Update", "Refactor"]

   ## Test Coverage
   [Brief note on test changes or new test coverage]
   ```

5. **Guidelines for the summary**:
   - **Be concise**: Each bullet should be one line
   - **Focus on behavior**: What changed from the user's perspective
   - **Skip implementation details**: No need to mention specific variable names or internal refactoring unless it's the main point
   - **Group related changes**: Don't list every file, group by logical feature/fix
   - **Use active voice**: "Add support for X" not "Support for X was added"
   - **Max 5-7 bullets**: Combine related changes to keep it scannable

6. **Output the summary**:
   - Present the summary to the user
   - Offer to copy it to clipboard or use it for PR creation

## Notes
- This skill analyzes git history and diffs to generate the summary
- The summary should be suitable for both PR descriptions and commit messages
- Focus on changes that reviewers and stakeholders care about, not internal refactoring details
- If the branch has many commits, group related commits into single bullet points
- Skip mentioning formatting changes, linting fixes, or minor refactoring unless that's the main purpose

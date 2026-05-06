---
name: commit-and-push
description: Commit all changes and push to a specified branch. Never commits to main/master. The branch name must be provided by the user.
user-invokable: true
---

## Instructions

When asked to commit and push changes to a branch:

1. **Safety check - verify not on main/master**:
   - Check current branch: `git branch --show-current`
   - **CRITICAL**: If on `main` or `master`, STOP and warn the user
   - Ask user to switch to a feature branch first
   - NEVER commit to main/master under any circumstances

2. **Get the target branch name**:
   - If user provided a branch name, use it
   - If not provided, ask the user for the branch name
   - Verify the branch name is not `main` or `master`

3. **Check for changes to commit**:
   - Run `git status` to see uncommitted changes
   - Run `git diff --stat` to see staged changes
   - Run `git diff --stat HEAD` to see all changes (staged + unstaged)

4. **Review and commit changes**:
   - If there are uncommitted changes:
     - Review the changes: `git diff` for staged, `git diff HEAD` for all changes
     - Stage all changes: `git add -A` (or specific files if user specifies)
     - Create a meaningful commit message by:
       - Analyzing the changes made
       - Reading recent commit messages: `git log -5 --oneline` to match the style
       - Summarizing what was changed and why
     - Commit with message ending in: `Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>`
     - Use heredoc format for commit message
   - If there are no uncommitted changes:
     - Inform the user that everything is already committed
     - Proceed to push

5. **Push to the branch**:
   - Check if the branch exists on remote: `git ls-remote --heads origin <branch-name>`
   - If branch exists on remote:
     - Push changes: `git push origin <branch-name>`
   - If branch doesn't exist on remote:
     - Push and set upstream: `git push -u origin <branch-name>`
   - Confirm push was successful

6. **Report status**:
   - Show the commit SHA if a commit was created
   - Show the branch that was pushed to
   - Show the remote URL or confirm remote branch was updated

## Safety Guidelines

- **NEVER** commit or push to `main` or `master` branches
- **NEVER** use `--force` or `--force-with-lease` unless explicitly requested by user
- **NEVER** skip git hooks with `--no-verify` unless explicitly requested
- If the branch is protected or push is rejected, report the error to the user
- If there are merge conflicts or diverged branches, report to user rather than attempting to resolve

## Example Workflow

```bash
# 1. Check current branch
git branch --show-current
# Output: feature/add-logging (✓ safe to proceed)

# 2. Check for changes
git status
git diff --stat HEAD

# 3. Stage and commit
git add -A
git commit -m "$(cat <<'EOF'
Add logging utility for structured logging

Added LogEntry struct and Logger interface to support structured logging
throughout the application.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"

# 4. Push to branch
git push -u origin feature/add-logging
```

## Edge Cases

- **Already on correct branch with uncommitted changes**: Commit and push
- **Already on correct branch, everything committed**: Just push
- **On different branch than target**: Warn user and ask if they want to switch first
- **On main/master**: STOP immediately and ask user to switch branches
- **Push rejected (non-fast-forward)**: Report error, suggest `git pull` or checking with team
- **No remote named 'origin'**: Report error and ask user for correct remote name

## Notes

- This skill prioritizes safety over convenience
- Always creates commits before pushing (no `--no-commit` operations)
- Always preserves git hooks and signing requirements
- Branch name is a required parameter from the user

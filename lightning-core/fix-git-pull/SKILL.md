---
description: Fix git pull/fetch failures caused by stale branch references and lock files from force-pushed branches
---

# fix-git-pull

Fixes git pull/fetch failures caused by stale branch references and lock files from force-pushed branches.

## When to use

Use this skill when:
- `git pull` or `git fetch` fails with "incorrect old value provided" errors
- Git reports "cannot lock ref" errors
- Multiple remote branches show forced updates but git can't apply them

## What it does

1. Removes stale git lock files
2. Identifies problematic branch references from error messages
3. Removes those refs from `.git/packed-refs` and loose ref files
4. Retries the git pull operation

## Instructions

Run the following diagnostic and fix steps:

1. First, try a simple git pull to capture the error:
   ```bash
   git pull 2>&1 | tee /tmp/git-pull-error.txt
   ```

2. If you see "incorrect old value provided" or "cannot lock ref" errors:
   
   a. Remove all git lock files:
   ```bash
   find .git -name "*.lock" -type f -delete
   ```
   
   b. Extract problematic branch names from the error:
   ```bash
   grep -E "(incorrect old value|cannot lock ref)" /tmp/git-pull-error.txt | grep -oE "refs/remotes/origin/[^ ']+" | sed 's/refs\/remotes\/origin\///' | sort -u > /tmp/bad-refs.txt
   ```
   
   c. Show the user which branches are problematic:
   ```bash
   echo "Problematic branches:" && cat /tmp/bad-refs.txt
   ```
   
   d. Remove these refs from packed-refs (with backup):
   ```bash
   if [ -f .git/packed-refs ]; then
     cp .git/packed-refs .git/packed-refs.backup
     while IFS= read -r branch; do
       grep -v "refs/remotes/origin/$branch" .git/packed-refs.backup > .git/packed-refs.tmp && mv .git/packed-refs.tmp .git/packed-refs
       cp .git/packed-refs .git/packed-refs.backup
     done < /tmp/bad-refs.txt
     echo "Updated packed-refs"
   fi
   ```
   
   e. Remove loose ref files:
   ```bash
   while IFS= read -r branch; do
     rm -f ".git/refs/remotes/origin/$branch"
   done < /tmp/bad-refs.txt
   echo "Removed loose refs"
   ```
   
   f. Retry git pull:
   ```bash
   git pull
   ```

3. If successful, report to the user that git pull is working again.

4. If still failing with different errors, try:
   ```bash
   git fetch origin main:refs/remotes/origin/main && git pull
   ```

## Prevention

To reduce friction from stale refs in the future, you can enable automatic pruning:
```bash
git config --local fetch.prune true
```

However, this may make the pruning conflicts more frequent. The user can decide if they want this enabled.

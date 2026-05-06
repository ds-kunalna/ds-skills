---
name: make-pr-ready
description: Prepare a pull request for review by ensuring the code compiles, tests pass, linting issues are resolved, and documentation is updated. Optionally, commit and push changes to the branch. This skill is designed to help developers ensure their pull requests are in good shape before submitting them for review.
---

## Instructions
When asked to make a pull request ready, follow these steps:
1. **Run `make build`** to ensure the code compiles without errors.
2. **Run `make test`** to execute all tests, fix any errors and confirm they pass successfully.
3. **Run `make lint`** to check for any linting issues and fix them.
4. **Review contribution guidelines** to ensure your PR adheres to the project's standards.
5. **Update documentation** if your changes affect any public interfaces or behaviors.
6. **Summarize changes** in a clear and concise manner to prepare for the pull request description. Use generate-pr-summary skill to help with this if needed.
7. **Commit and push your changes** to the branch (optional), ask the user if they need the agent to commit and push the changes for them. If they do, commit with a message by finding the latest changes made and summarizing them in a concise commit message, then push to the branch.
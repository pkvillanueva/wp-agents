---
description: Create a concise commit from modified files
---

Review all modified and staged files in this WordPress plugin repository and create a concise git commit.

## Instructions

1. Run `git status` to see modified/staged files
2. Run `git diff` to see unstaged changes and `git diff --cached` for staged changes
3. Analyze the changes to understand what was modified

## Commit Message Format

Create a commit message following this format:
- **First line**: Short summary (50 chars max), use conventional commit prefixes:
  - `feat:` for new features
  - `fix:` for bug fixes
  - `docs:` for documentation changes
  - `style:` for formatting (no code change)
  - `refactor:` for code refactoring
  - `test:` for adding or updating tests
  - `chore:` for maintenance tasks
  - `perf:` for performance improvements
  - `ci:` for CI/CD changes
  - `build:` for build system changes
  - `revert:` for reverting previous commits
- **Body** (optional): Only include if the changes are complex and need explanation. Keep it minimal (1-2 lines max).

## Rules

- Keep the commit message as short as possible
- Do NOT include any Claude attribution, co-author tags, or "Generated with Claude" text
- Do NOT add emojis
- Stage all relevant files before committing
- Use imperative mood ("Add feature" not "Added feature")

## Execution

After analyzing the changes:
1. Stage the appropriate files with `git add`
2. Create the commit using:
```bash
git commit -m "type: short description"
```

Or if a body is truly needed:
```bash
git commit -m "type: short description" -m "Brief explanation if complex"
```

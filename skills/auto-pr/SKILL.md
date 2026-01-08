---
name: auto-pr
description: Automatically create GitHub pull requests by analyzing commit history and code changes. Use this skill when the user explicitly requests to create a PR (e.g., "/pr", "PR 만들어줘", "create a pull request", "make a PR"). This skill analyzes all commits in the current branch, examines code changes, generates PR content following the project's template format, and creates the PR using gh CLI.
---

# Auto PR

Automatically create GitHub pull requests with properly formatted descriptions based on commit history and code changes.

## Workflow

Follow these steps sequentially when this skill is invoked:

### 1. Check Prerequisites

Verify that gh CLI is available and authenticated:

```bash
gh --version
gh auth status
```

If gh is not installed or not authenticated, inform the user and provide installation/authentication instructions.

### 2. Analyze Current Branch

Get information about the current branch and repository:

```bash
git branch --show-current
git status
git remote get-url origin
```

Identify:
- Current branch name
- Repository owner and name
- Git status (ensure working tree is clean or guide user to commit first)

### 3. Determine Base Branch

Ask the user which branch this PR should target using AskUserQuestion tool.

Common options:
- `master` (production)
- `develop` (development)
- Other branches as needed

Check if the current branch is already tracking a remote:

```bash
git rev-parse --abbrev-ref @{upstream} 2>/dev/null || echo "no upstream"
```

### 4. Analyze All Commits and Changes

Get the full commit history and changes since diverging from base branch:

```bash
# All commits in current branch not in base branch
git log <base-branch>..HEAD --oneline

# Detailed commit messages
git log <base-branch>..HEAD --pretty=format:"%h %s%n%b"

# All changes (diff)
git diff <base-branch>...HEAD

# Changed file list
git diff <base-branch>...HEAD --name-status
```

Understand:
- All commit messages and their types (feat, fix, refactor, etc.)
- Full scope of code changes
- Which files were added, modified, or deleted
- Whether test files were added/modified

### 5. Generate PR Content

Read the PR template format from `references/pr_template.md` to understand the required structure.

Create PR content following this format:

#### PR Title
- Use the primary change type from commits
- Format: `type(scope): brief description`
- Examples: `feat(queue): add retry mechanism`, `fix(service): resolve race condition`

#### PR Body

**태스크 요약**:
- Synthesize all commits into 1-3 sentences
- Explain what this PR accomplishes overall
- Focus on the business value or problem solved

**변경 사항**:
- Create bulleted list of key changes
- Group by component or feature area
- Be specific but concise
- Extract from commit messages and diff analysis

**스크린샷 (선택사항)**:
- Include this section header but leave empty
- User can add screenshots later if needed

**관련 링크(선택사항)**:
```markdown
- [피그마]()
- [태스크카드]()
```
Leave empty - user will fill in if needed

**셀프 체크리스트**:
Auto-check based on these rules:
- `[x]` if test files (*.test.js, *.spec.js, test/*, __tests__/*) were added or modified
- `[ ]` for "적절한 라벨을 추가했습니다" (user must do manually)
- `[x]` for "PR 제목을 알맞게 작성했습니다" (we auto-generate it)
- `[ ]` for "셀프 코드리뷰를 작성했습니다" (user must do manually)

### 6. Push Current Branch

If the branch is not tracking a remote yet, push it:

```bash
git push -u origin <branch-name>
```

If already tracking, ensure it's up to date:

```bash
git push
```

### 7. Create PR

Use gh CLI to create the PR with the generated content:

```bash
gh pr create \
  --base <base-branch> \
  --head <current-branch> \
  --title "PR Title" \
  --body "$(cat <<'EOF'
PR Body Content Here
EOF
)"
```

Always use HEREDOC format for the body to ensure proper formatting with multiple sections.

### 8. Return PR URL

After successful PR creation, extract and display the PR URL to the user:

```bash
gh pr view --json url -q .url
```

Show the user:
- Success message
- PR URL
- PR number

## Important Notes

- **Always analyze ALL commits** in the branch, not just the most recent one
- **Read the full diff** to understand the actual changes, don't rely only on commit messages
- **Use HEREDOC format** for PR body to preserve formatting
- **Check for test files** to automatically check the test checklist item
- **Keep descriptions concise** but comprehensive - synthesize multiple commits into coherent narrative
- **Preserve Korean template headers** - don't translate them to English
- **Handle errors gracefully** - if gh command fails, show the error and suggest solutions
- **Clean working tree** - ensure all changes are committed before creating PR
- **Follow Conventional Commits** in PR title for consistency

## Reference Files

- `references/pr_template.md` - Detailed PR template format and checklist rules

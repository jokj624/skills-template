---
name: pr-review
description: Comprehensive code review and PR comment posting for current branch changes. Use when the user requests code review, PR review, or wants to post review comments to a pull request. This skill analyzes git diff, identifies issues based on project coding guidelines (CLAUDE.md), assigns severity levels (p1-p4, Q), writes reviews in Korean, and posts as draft review to GitHub PR immediately.
---

# PR Review

## Overview

This skill performs comprehensive code review of current branch changes and immediately posts as draft review to GitHub Pull Requests. It analyzes code from multiple perspectives (bugs, architecture, tests), assigns appropriate severity levels (p1-p4, Q), writes reviews in Korean, and creates a draft PR review without user confirmation.

## Workflow

### 1. Analyze Changes

**Get current branch diff:**
```bash
git diff main...HEAD
```

If the main branch is not 'main', use the appropriate base branch name.

**Get changed files:**
```bash
git diff --name-only main...HEAD
```

**Read CLAUDE.md** (if exists) to understand project-specific coding guidelines.

### 2. Perform Code Review

Review changes from three perspectives:

1. **Bugs and Potential Issues**
   - Security vulnerabilities (SQL injection, XSS, etc.)
   - Logic errors
   - Edge case handling
   - Async/Promise error handling
   - Type mismatches

2. **Architecture and Design Patterns**
   - Layer separation (Route → Controller → Service → Model)
   - SOLID principles
   - Code duplication (DRY)
   - Proper abstractions
   - Dependency directions

3. **Test Coverage**
   - Service logic tests
   - Controller tests
   - Model tests
   - Test case sufficiency (happy path, edge cases, error cases)

**Refer to** [review-guidelines.md](references/review-guidelines.md) for detailed criteria and common issue patterns.

### 3. Assign Review Levels

For each review point, assign one of these levels based on severity:

| Level | Template | When to Use |
|-------|----------|-------------|
| **p1** | `#### ⚠️ p1; 무조건 반영해주세요.` | Critical: Security vulnerabilities, severe bugs, data loss risks, service downtime |
| **p2** | `#### p2; 적극적으로 고려해주세요.` | Important: Potential bugs, architecture violations, missing tests, performance issues |
| **p3** | `#### p3; 웬만하면 반영해주세요.` | Moderate: Code quality, style guide violations, minor improvements, error handling |
| **p4** | `#### p4; 사소한 의견입니다.` | Minor: Personal preferences, tiny improvements, consistency suggestions |
| **Q** | `#### Q; 질문입니다.` | Questions: Clarification needed, unclear intent, discussion topics |

**Refer to** [review-guidelines.md](references/review-guidelines.md) for detailed level assignment criteria.

### 4. Write Review Comments (Korean)

All reviews must be written in **Korean**.

**Comment format:**
```markdown
#### [Level Template]

[Review content in Korean]

[Optional: Code example or suggestion]
```

**Example:**
```markdown
#### ⚠️ p1; 무조건 반영해주세요.

SQL Injection 취약점이 있습니다. Parameterized query를 사용해야 합니다.

\`\`\`javascript
// 현재 코드
const query = `SELECT * FROM users WHERE id = ${userId}`;

// 권장 코드
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
\`\`\`
```

**Review writing guidelines:**
- Be specific: Mention exact file name and line number when relevant
- Be constructive: Suggest improvements, not just criticisms
- Provide examples: Show better alternatives when possible
- Be clear: Explain WHY the change is needed
- Reference guidelines: Mention specific CLAUDE.md sections if applicable

### 5. Create Draft Review on PR

Immediately create a draft review on the PR using GitHub API. Do NOT present review to user first - post directly as draft.

**Refer to** [pr-comment-guide.md](references/pr-comment-guide.md) for detailed GitHub API usage.

#### Step-by-step Process

1. **Get PR Information:**
```bash
PR_NUMBER=$(gh pr view --json number -q .number)
COMMIT_SHA=$(gh pr view $PR_NUMBER --json commits --jq '.commits[-1].oid')
REPO_INFO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
```

2. **Create JSON payload for draft review:**
Create a JSON file with the following structure:
```json
{
  "body": "## 전체 리뷰 요약\n\n[summary content]",
  "commit_id": "commit_sha",
  "comments": [
    {
      "path": "src/server/service/user.js",
      "line": 42,
      "body": "#### p2; 적극적으로 고려해주세요.\n\n[review content]"
    },
    {
      "path": "src/server/service/user.js",
      "start_line": 10,
      "line": 15,
      "body": "#### p3; 웬만하면 반영해주세요.\n\n[review content]"
    }
  ]
}
```

**Important Notes:**
- Do NOT include comments for deleted files - they will cause 422 errors
- Use `line` for single-line comments
- Use `start_line` and `line` for multi-line comments
- All review body text should include the level template at the start

3. **Post draft review via GitHub API:**
```bash
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  /repos/$REPO_INFO/pulls/$PR_NUMBER/reviews \
  --input review.json
```

This creates a PENDING (draft) review that appears on the PR but is not yet submitted.

### 6. Provide PR Review Link

After successfully creating the draft review:
1. Extract the review URL from the API response (`html_url` field)
2. Inform the user with a concise message:
   - "Draft 리뷰가 작성되었습니다"
   - Total number of issues found (p1: X, p2: X, etc.)
   - PR review link
   - Instructions: "PR 페이지에서 리뷰 내용을 확인하신 후 'Submit review' 버튼을 눌러 게시하세요"

## Important Notes

1. **Always read CLAUDE.md first** if it exists in the project to understand project-specific guidelines
2. **Post directly as draft**: Do NOT show review to user first - create draft review immediately on PR
3. **Be thorough but focused**: Don't nitpick minor issues if there are major problems
4. **Prioritize correctly**: p1 issues should be truly critical, don't overuse
5. **Be respectful**: Remember reviews are in Korean and will be read by Korean developers
6. **Consider context**: Some "violations" might be intentional or necessary
7. **Test knowledge**: Understand testing is required for backend (services, models, controllers) but guidelines differ for frontend
8. **Handle deleted files**: Do NOT create inline comments for deleted files as they cause API errors
9. **Output format**: Only show concise summary with PR link - user can see full review on GitHub

## Resources

- **[review-guidelines.md](references/review-guidelines.md)**: Detailed level assignment criteria, review checklists, and common issue patterns
- **[pr-comment-guide.md](references/pr-comment-guide.md)**: GitHub API usage for posting inline and summary review comments

## Example Usage

**User:** "현재 브랜치 변경사항에 대한 코드리뷰해줘"

**Workflow:**
1. Run `git diff` to see changes (use appropriate base branch like develop or main)
2. Read CLAUDE.md for project guidelines
3. Analyze each changed file for bugs, architecture issues, and test coverage
4. Assign levels (p1-p4, Q) based on severity
5. Write review comments in Korean with level templates
6. Immediately create draft review on PR via GitHub API
7. Provide PR review link to user

**Expected Output:**
```
Draft 리뷰가 작성되었습니다! 🎉

총 7개 이슈 발견 (p1: 1개, p2: 2개, p3: 2개, p4: 1개, Q: 1개)

👉 PR 리뷰 보기: https://github.com/owner/repo/pull/123#pullrequestreview-456789

PR 페이지에서 리뷰 내용을 확인하신 후 'Submit review' 버튼을 눌러 게시하세요.
```

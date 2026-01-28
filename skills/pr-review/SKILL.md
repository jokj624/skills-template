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

Immediately create a draft review on the PR using GitHub MCP tools. Do NOT present review to user first - post directly as draft.

**Refer to** [pr-comment-guide.md](references/pr-comment-guide.md) for detailed GitHub MCP tool usage.

#### Prerequisites

Use ToolSearch to load the required GitHub MCP tools:
- `mcp__github__pull_request_review_write` - For creating/submitting reviews
- `mcp__github__add_comment_to_pending_review` - For adding inline comments to pending review

#### Step-by-step Process

1. **Get PR Information:**
```bash
PR_NUMBER=$(gh pr view --json number -q .number)
COMMIT_SHA=$(gh pr view $PR_NUMBER --json commits --jq '.commits[-1].oid')
```

Extract owner and repo from remote URL:
```bash
git remote get-url origin | sed -E 's/.*[:/]([^/]+)\/([^/.]+)(\.git)?$/\1 \2/'
```

2. **Create a pending review:**
Use `mcp__github__pull_request_review_write` with method `create` WITHOUT the `event` parameter to create a pending (draft) review:

```
mcp__github__pull_request_review_write
  method: "create"
  owner: "organization-name"
  repo: "repository-name"
  pullNumber: <pr-number>
  body: "## 전체 리뷰 요약\n\n[summary content]"
  commitID: "<commit-sha>"
```

**Important:** Omit the `event` parameter to create a pending review. If `event` is provided, the review is submitted immediately.

3. **Add inline comments to the pending review:**
For each review comment, use `mcp__github__add_comment_to_pending_review`:

**Single-line comment:**
```
mcp__github__add_comment_to_pending_review
  owner: "organization-name"
  repo: "repository-name"
  pullNumber: <pr-number>
  path: "src/server/service/user.js"
  body: "#### p2; 적극적으로 고려해주세요.\n\n[review content]"
  line: 42
  side: "RIGHT"
  subjectType: "LINE"
```

**Multi-line comment:**
```
mcp__github__add_comment_to_pending_review
  owner: "organization-name"
  repo: "repository-name"
  pullNumber: <pr-number>
  path: "src/server/service/user.js"
  body: "#### p3; 웬만하면 반영해주세요.\n\n[review content]"
  startLine: 10
  line: 15
  side: "RIGHT"
  startSide: "RIGHT"
  subjectType: "LINE"
```

**Important Notes:**
- Do NOT include comments for deleted files - they will cause errors
- Use `side: "RIGHT"` for comments on new/modified code (most common)
- Use `side: "LEFT"` for comments on deleted code
- Use `subjectType: "LINE"` for line-specific comments
- Use `subjectType: "FILE"` for file-level comments (no line number needed)
- All review body text should include the level template at the start

4. **The review remains as PENDING (draft):**
The pending review will appear on the PR page. The author can review the comments and click "Submit review" to publish it.

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
2. **Load MCP tools first**: Use ToolSearch to load `mcp__github__pull_request_review_write` and `mcp__github__add_comment_to_pending_review` before creating reviews
3. **Post directly as draft**: Do NOT show review to user first - create draft review immediately on PR
4. **Create pending review first**: Must create a pending review before adding inline comments
5. **Be thorough but focused**: Don't nitpick minor issues if there are major problems
6. **Prioritize correctly**: p1 issues should be truly critical, don't overuse
7. **Be respectful**: Remember reviews are in Korean and will be read by Korean developers
8. **Consider context**: Some "violations" might be intentional or necessary
9. **Test knowledge**: Understand testing is required for backend (services, models, controllers) but guidelines differ for frontend
10. **Handle deleted files**: Do NOT create inline comments for deleted files as they cause errors
11. **Output format**: Only show concise summary with PR link - user can see full review on GitHub
12. **Use side parameter**: Use `side: "RIGHT"` for comments on new/modified code (most common case)

## Resources

- **[review-guidelines.md](references/review-guidelines.md)**: Detailed level assignment criteria, review checklists, and common issue patterns
- **[pr-comment-guide.md](references/pr-comment-guide.md)**: GitHub MCP tools usage for posting inline and summary review comments

## Example Usage

**User:** "현재 브랜치 변경사항에 대한 코드리뷰해줘"

**Workflow:**
1. Load GitHub MCP tools via ToolSearch
2. Run `git diff` to see changes (use appropriate base branch like develop or main)
3. Read CLAUDE.md for project guidelines
4. Analyze each changed file for bugs, architecture issues, and test coverage
5. Assign levels (p1-p4, Q) based on severity
6. Write review comments in Korean with level templates
7. Create pending review using `mcp__github__pull_request_review_write`
8. Add inline comments using `mcp__github__add_comment_to_pending_review`
9. Provide PR review link to user

**Expected Output:**
```
Draft 리뷰가 작성되었습니다! 🎉

총 7개 이슈 발견 (p1: 1개, p2: 2개, p3: 2개, p4: 1개, Q: 1개)

👉 PR 리뷰 보기: https://github.com/owner/repo/pull/123#pullrequestreview-456789

PR 페이지에서 리뷰 내용을 확인하신 후 'Submit review' 버튼을 눌러 게시하세요.
```

---
name: self-review
description: Create a self-review on your own PR to explain changes, implementation decisions, and provide context for reviewers. Use when you want to document your changes with explanatory comments before requesting review from teammates.
---

# Self Review

## Overview

This skill creates a self-review on your own Pull Request to explain your changes to reviewers. Unlike code review (which critiques code), self-review documents:
- What was changed and why
- Implementation decisions and trade-offs
- Areas that need special attention
- Context that helps reviewers understand the changes

## Workflow

### 1. Load Required MCP Tools

Use ToolSearch to load the required GitHub MCP tools:

```
ToolSearch query: "select:mcp__github__pull_request_review_write"
ToolSearch query: "select:mcp__github__add_comment_to_pending_review"
```

### 2. Get PR and Repository Information

```bash
# Get PR number
PR_NUMBER=$(gh pr view --json number -q .number)

# Get latest commit SHA
COMMIT_SHA=$(gh pr view $PR_NUMBER --json commits --jq '.commits[-1].oid')

# Get owner and repo
OWNER_REPO=$(git remote get-url origin | sed -E 's/.*[:/]([^/]+)\/([^/.]+)(\.git)?$/\1 \2/')
```

### 3. Analyze Changes

**Get the diff:**
```bash
# Get base branch
BASE_BRANCH=$(gh pr view $PR_NUMBER --json baseRefName -q .baseRefName)

# Get full diff
git diff $BASE_BRANCH...HEAD

# Get changed files
git diff --name-status $BASE_BRANCH...HEAD
```

**Get commit history:**
```bash
git log $BASE_BRANCH..HEAD --pretty=format:"%h %s%n%b"
```

**Read related files** to understand the context:
- CLAUDE.md or project guidelines
- Related source files for understanding the codebase

### 4. Generate Self-Review Comments

For each significant change, create explanatory comments. Focus on:

#### Comment Types

| Type | Template | When to Use |
|------|----------|-------------|
| **변경 이유** | `#### 💡 변경 이유` | Explain WHY this change was made |
| **구현 방식** | `#### 🔧 구현 방식` | Explain HOW it was implemented |
| **주의사항** | `#### ⚠️ 주의사항` | Highlight areas needing careful review |
| **관련 변경** | `#### 🔗 관련 변경` | Link related changes across files |
| **TODO** | `#### 📝 TODO` | Mark future improvements or follow-ups |

#### Comment Format

```markdown
#### [Type Template]

[Explanation in Korean]

[Optional: Code context or related information]
```

#### Examples

**Explaining a new function:**
```markdown
#### 💡 변경 이유

기존 `checkPhoneNumber` 함수가 여러 모듈에 중복되어 있어서 `user-identity` 서비스로 통합했습니다.
이를 통해 핸드폰 번호 검증 로직을 일원화하고 유지보수성을 개선합니다.
```

**Explaining implementation decision:**
```markdown
#### 🔧 구현 방식

해시 값을 미리 계산하여 전달하는 방식(`createUserIdentityWithHash`)을 선택했습니다.
이유:
1. 호출자가 해시 값을 재사용할 수 있음
2. 트랜잭션 내에서 해시 계산 중복 방지
3. 테스트에서 해시 값 모킹이 용이함
```

**Highlighting attention area:**
```markdown
#### ⚠️ 주의사항

이 변경으로 인해 기존 `phoneNumber` 필드가 스키마에서 제거됩니다.
마이그레이션 스크립트가 필요할 수 있으니 배포 전 확인 부탁드립니다.
```

### 5. Create Pending Review with Summary

Use `mcp__github__pull_request_review_write` to create a pending review:

```
mcp__github__pull_request_review_write
  method: "create"
  owner: "<owner>"
  repo: "<repo>"
  pullNumber: <pr-number>
  body: "## 셀프 리뷰\n\n이 PR의 변경사항을 설명하는 셀프 리뷰입니다.\n\n### 주요 변경사항\n- [bullet points]\n\n### 리뷰어 참고사항\n- [things reviewers should know]"
  commitID: "<commit-sha>"
```

**Summary Template:**
```markdown
## 셀프 리뷰

이 PR의 변경사항을 설명하는 셀프 리뷰입니다.

### 주요 변경사항
- [Key change 1]
- [Key change 2]
- [Key change 3]

### 리뷰어 참고사항
- [Important context for reviewers]
- [Areas needing careful review]
- [Related PRs or issues if any]
```

### 6. Add Inline Comments

For each significant code section, use `mcp__github__add_comment_to_pending_review`:

```
mcp__github__add_comment_to_pending_review
  owner: "<owner>"
  repo: "<repo>"
  pullNumber: <pr-number>
  path: "src/auth/auth.service.ts"
  body: "#### 💡 변경 이유\n\n[explanation]"
  line: 42
  side: "RIGHT"
  subjectType: "LINE"
```

**Guidelines for inline comments:**
- Focus on non-obvious changes that need explanation
- Don't comment on trivial changes (typo fixes, formatting)
- Group related changes with multi-line comments when appropriate
- Use `subjectType: "FILE"` for file-level context

### 7. Output Result

After creating the self-review:

```
셀프 리뷰가 작성되었습니다!

📝 총 X개의 설명 코멘트 작성
- 변경 이유: X개
- 구현 방식: X개
- 주의사항: X개

👉 PR 리뷰 보기: https://github.com/owner/repo/pull/123

리뷰 내용을 확인하신 후 'Submit review' 버튼을 눌러 게시하세요.
팀원에게 리뷰를 요청하기 전에 셀프 리뷰를 먼저 제출하는 것을 권장합니다.
```

## Important Notes

1. **Load MCP tools first**: Always use ToolSearch to load GitHub MCP tools before creating reviews
2. **Create pending review first**: Must create a pending review before adding inline comments
3. **Focus on explanation, not critique**: Self-review explains YOUR changes, not finds problems
4. **Write in Korean**: All comments should be in Korean for Korean-speaking teams
5. **Be concise but informative**: Provide enough context without being verbose
6. **Highlight important areas**: Use 주의사항 for areas that need careful review
7. **Link related changes**: When changes span multiple files, explain the relationship
8. **Don't over-comment**: Only comment on non-obvious or significant changes
9. **Use appropriate side**: Always use `side: "RIGHT"` for new/modified code
10. **Submit before requesting review**: Encourage submitting self-review before requesting teammate review

## When to Use This Skill

- After creating a PR, before requesting review
- For complex changes that need explanation
- When changes span multiple files with interconnected logic
- When there are non-obvious implementation decisions
- When migrating or refactoring existing code

## Resources

- **[self-review-guide.md](references/self-review-guide.md)**: Detailed guidelines for writing effective self-review comments

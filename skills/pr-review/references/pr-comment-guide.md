# PR 댓글 작성 가이드

## GitHub MCP Tools를 사용한 리뷰 댓글 작성

이 가이드에서는 GitHub MCP 도구를 사용하여 PR 리뷰 댓글을 작성하는 방법을 설명합니다.

### 1. 필수 MCP Tools 로드

리뷰 작성 전에 ToolSearch를 사용하여 필요한 MCP 도구를 로드합니다:

```
ToolSearch query: "select:mcp__github__pull_request_review_write"
ToolSearch query: "select:mcp__github__add_comment_to_pending_review"
```

### 2. PR 정보 가져오기

리뷰를 작성하기 전에 필요한 정보를 가져옵니다.

#### PR 번호 확인
```bash
gh pr view --json number -q .number
```

#### PR의 최신 commit SHA 가져오기
```bash
gh pr view {pr_number} --json commits --jq '.commits[-1].oid'
```

#### Owner/Repo 정보 가져오기
```bash
git remote get-url origin | sed -E 's/.*[:/]([^/]+)\/([^/.]+)(\.git)?$/\1 \2/'
```

#### PR에서 변경된 파일 목록
```bash
gh pr view {pr_number} --json files --jq '.files[].path'
```

### 3. Draft(Pending) 리뷰 생성

`mcp__github__pull_request_review_write` 도구를 사용하여 pending 리뷰를 생성합니다.

**중요:** `event` 파라미터를 **생략**해야 pending(draft) 리뷰가 생성됩니다.

```
mcp__github__pull_request_review_write
  method: "create"
  owner: "goorm-dev"
  repo: "repository-name"
  pullNumber: 123
  body: "## 전체 리뷰 요약\n\n### 주요 검토 사항\n- 총 3개의 p1 이슈 발견\n- 2개의 p2 개선 사항"
  commitID: "abc123def456"
```

### 4. 인라인 댓글 추가

`mcp__github__add_comment_to_pending_review` 도구를 사용하여 pending 리뷰에 인라인 댓글을 추가합니다.

**전제 조건:** 먼저 pending 리뷰가 생성되어 있어야 합니다.

#### 단일 라인 댓글

```
mcp__github__add_comment_to_pending_review
  owner: "goorm-dev"
  repo: "repository-name"
  pullNumber: 123
  path: "src/server/service/user.js"
  body: "#### p2; 적극적으로 고려해주세요.\n\n변수명을 더 명확하게 변경하는 것이 좋겠습니다."
  line: 42
  side: "RIGHT"
  subjectType: "LINE"
```

#### 여러 라인에 걸친 댓글

```
mcp__github__add_comment_to_pending_review
  owner: "goorm-dev"
  repo: "repository-name"
  pullNumber: 123
  path: "src/server/controllers/userApi.js"
  body: "#### p1; 무조건 반영해주세요.\n\nSQL Injection 취약점이 있습니다."
  startLine: 10
  line: 15
  side: "RIGHT"
  startSide: "RIGHT"
  subjectType: "LINE"
```

#### 파일 레벨 댓글

```
mcp__github__add_comment_to_pending_review
  owner: "goorm-dev"
  repo: "repository-name"
  pullNumber: 123
  path: "src/server/service/user.js"
  body: "#### p3; 웬만하면 반영해주세요.\n\n이 파일 전체적으로 에러 처리가 일관성이 없습니다."
  subjectType: "FILE"
```

### 5. 리뷰 제출 (선택사항)

Pending 리뷰를 즉시 제출하려면 `submit_pending` 메서드를 사용합니다:

```
mcp__github__pull_request_review_write
  method: "submit_pending"
  owner: "goorm-dev"
  repo: "repository-name"
  pullNumber: 123
  body: "리뷰를 제출합니다."
  event: "COMMENT"  # 또는 "APPROVE", "REQUEST_CHANGES"
```

**리뷰 타입:**
- `COMMENT`: 일반 코멘트 (승인/변경 요청 없이)
- `APPROVE`: 승인
- `REQUEST_CHANGES`: 변경 요청

**참고:** 보통 draft 리뷰로 남겨두고 사용자가 PR 페이지에서 직접 'Submit review' 버튼을 클릭하도록 합니다.

### 6. Pending 리뷰 삭제

필요시 pending 리뷰를 삭제할 수 있습니다:

```
mcp__github__pull_request_review_write
  method: "delete_pending"
  owner: "goorm-dev"
  repo: "repository-name"
  pullNumber: 123
```

### 7. 리뷰 댓글 워크플로우 요약

```
1. PR 정보 수집 (PR 번호, commit SHA, owner/repo)
   ↓
2. pending 리뷰 생성 (mcp__github__pull_request_review_write method: "create")
   ↓
3. 인라인 댓글 추가 (mcp__github__add_comment_to_pending_review) - 여러 번 호출 가능
   ↓
4. 사용자가 PR 페이지에서 'Submit review' 클릭
```

### 8. 댓글 포맷

모든 댓글은 다음 형식을 따라야 합니다:

```markdown
#### [레벨 템플릿]

[리뷰 내용]

[관련 코드 예시나 제안사항 (선택)]
```

**예시:**
```markdown
#### ⚠️ p1; 무조건 반영해주세요.

`userId`가 null일 수 있는 경우 예외 처리가 필요합니다.

\`\`\`javascript
if (!userId) {
  throw new AppError('CONSOLE-XX-XX', 'User ID is required');
}
\`\`\`
```

### 9. Side 파라미터 설명

- `RIGHT`: 새로 추가되거나 수정된 코드에 댓글 (가장 일반적)
- `LEFT`: 삭제된 코드에 댓글

대부분의 리뷰 댓글은 새 코드에 대한 것이므로 `side: "RIGHT"`를 사용합니다.

### 10. 주의사항

1. **Pending 리뷰 필수**: `add_comment_to_pending_review`를 호출하기 전에 반드시 pending 리뷰가 생성되어 있어야 합니다
2. **삭제된 파일 제외**: 삭제된 파일에는 댓글을 달 수 없습니다 - 에러가 발생합니다
3. **라인 번호 정확성**: diff에서 보이는 라인 번호를 정확히 사용해야 합니다
4. **commitID 사용**: 리뷰 생성 시 최신 commit SHA를 사용해야 합니다
5. **병렬 호출**: 여러 인라인 댓글을 추가할 때 병렬로 호출할 수 있습니다

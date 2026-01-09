# PR 댓글 작성 가이드

## GitHub API를 사용한 리뷰 댓글 작성

### 1. 인라인 댓글 작성

특정 파일의 특정 라인에 댓글을 남기려면 GitHub API를 사용합니다.

#### 단일 라인 댓글
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body="#### p2; 적극적으로 고려해주세요.

변수명을 더 명확하게 변경하는 것이 좋겠습니다." \
  -f commit_id="{commit_sha}" \
  -f path="src/server/service/user.js" \
  -f line=42
```

#### 여러 라인에 걸친 댓글
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body="#### p1; 무조건 반영해주세요.

SQL Injection 취약점이 있습니다." \
  -f commit_id="{commit_sha}" \
  -f path="src/server/controllers/userApi.js" \
  -f start_line=10 \
  -f line=15
```

### 2. 전체 PR 리뷰 작성

전체 리뷰는 `gh pr review` 명령어를 사용합니다.

#### 리뷰 타입
- `COMMENT`: 일반 코멘트 (승인/변경 요청 없이)
- `APPROVE`: 승인
- `REQUEST_CHANGES`: 변경 요청

```bash
gh pr review {pr_number} \
  --comment \
  --body "## 전체 리뷰 요약

### 주요 검토 사항
- 총 3개의 p1 이슈 발견
- 2개의 p2 개선 사항
- 테스트 커버리지 부족

### 긍정적인 부분
- 레이어 분리가 잘 되어 있음
- 에러 처리가 적절함

### 추가 확인이 필요한 부분
자세한 사항은 인라인 댓글을 참고해주세요."
```

### 3. PR 정보 가져오기

리뷰를 작성하기 전에 필요한 정보를 가져옵니다.

#### PR 번호 확인
```bash
gh pr view --json number -q .number
```

#### PR의 최신 commit SHA 가져오기
```bash
gh pr view {pr_number} --json commits --jq '.commits[-1].oid'
```

#### PR에서 변경된 파일 목록
```bash
gh pr view {pr_number} --json files --jq '.files[].path'
```

#### 특정 파일의 diff 가져오기
```bash
gh pr diff {pr_number} -- {file_path}
```

### 4. 댓글 작성 워크플로우

```bash
# 1. 현재 PR 번호 확인
PR_NUMBER=$(gh pr view --json number -q .number)

# 2. 최신 commit SHA 가져오기
COMMIT_SHA=$(gh pr view $PR_NUMBER --json commits --jq '.commits[-1].oid')

# 3. Owner/Repo 정보 가져오기
REPO_INFO=$(gh repo view --json nameWithOwner -q .nameWithOwner)

# 4. 인라인 댓글 작성
gh api repos/$REPO_INFO/pulls/$PR_NUMBER/comments \
  -f body="댓글 내용" \
  -f commit_id="$COMMIT_SHA" \
  -f path="파일경로" \
  -f line=42

# 5. 전체 리뷰 작성
gh pr review $PR_NUMBER --comment --body "전체 리뷰 내용"
```

### 5. 리뷰 댓글 포맷

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

### 6. 에러 처리

#### 권한 에러
```bash
# gh auth status로 인증 상태 확인
gh auth status

# 필요시 재인증
gh auth login
```

#### Commit SHA 불일치
- PR의 최신 commit SHA를 사용해야 합니다
- 오래된 SHA를 사용하면 422 에러가 발생합니다

#### 파일 경로 오류
- 파일 경로는 repo root 기준 상대 경로입니다
- `gh pr view --json files`로 정확한 경로를 확인하세요

### 7. 주의사항

1. **한 번에 많은 댓글 작성시**: GitHub API rate limit을 고려하여 적절한 간격을 두고 요청
2. **Draft 리뷰**: API로는 draft 리뷰를 작성할 수 없으므로, 사용자 확인 후 바로 게시해야 함
3. **댓글 수정/삭제**: 작성된 댓글은 API로 수정/삭제 가능하지만, 이 skill에서는 초기 작성만 담당
4. **멀티라인 댓글 범위**: `start_line`과 `line`은 같은 파일 내에서만 가능

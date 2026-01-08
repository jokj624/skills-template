# PR Template Format

This project uses the following PR template format:

```markdown
## 태스크 요약
## 변경 사항
## 스크린샷 (선택사항)
## 관련 링크(선택사항)
- [피그마]()
- [태스크카드]()
## 셀프 체크리스트
- [ ] 백엔드 로직의 테스트 코드를 작성했습니다.
- [ ] 적절한 라벨을 추가했습니다.
- [ ] PR 제목을 알맞게 작성했습니다.
- [ ] 셀프 코드리뷰를 작성했습니다.
```

## Section Guidelines

### 태스크 요약
- Brief 1-2 sentence summary of what this PR accomplishes
- Focus on the "what" and "why"

### 변경 사항
- Bulleted list of key changes
- Group by component or feature
- Be specific about what changed

### 스크린샷
- Only include if there are UI changes
- Can be omitted for backend-only changes

### 관련 링크
- Include relevant Figma designs if applicable
- Include task card links (ClickUp, Jira, etc.)
- Can be omitted if not applicable

### 셀프 체크리스트
Automatically check items based on:
- **백엔드 로직의 테스트 코드를 작성했습니다**: Check if test files were added/modified
- **적절한 라벨을 추가했습니다**: Always leave unchecked (user must add labels manually)
- **PR 제목을 알맞게 작성했습니다**: Always check (PR title is auto-generated)
- **셀프 코드리뷰를 작성했습니다**: Always leave unchecked (user must do self-review)

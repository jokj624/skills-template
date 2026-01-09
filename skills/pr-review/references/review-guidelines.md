# 코드 리뷰 가이드라인

## 리뷰 레벨 판단 기준

각 리뷰 포인트의 심각도와 중요도에 따라 적절한 레벨을 할당합니다.

### p1 (무조건 반영해주세요)
- **보안 취약점**: SQL Injection, XSS, CSRF, 인증/인가 우회 등
- **심각한 버그**: 데이터 손실, 서비스 중단을 유발할 수 있는 로직 오류
- **필수 테스트 누락**: 핵심 로직에 대한 테스트 코드가 전혀 없는 경우
- **치명적인 성능 이슈**: 무한 루프, N+1 쿼리, 메모리 누수 등
- **기능 명세 불일치**: 요구사항과 완전히 다르게 구현된 경우

### p2 (적극적으로 고려해주세요)
- **잠재적 버그**: 엣지 케이스 미처리, race condition 가능성 등
- **아키텍처 문제**: 레이어 분리 위반, 의존성 방향 오류 등
- **중요한 스타일 가이드 위반**: 네이밍 규칙, 파일 구조, 필수 JSDoc 누락 등
- **테스트 품질 이슈**: 중요한 케이스가 테스트되지 않음
- **성능 개선 여지**: 불필요한 연산, 비효율적인 알고리즘 등

### p3 (웬만하면 반영해주세요)
- **코드 품질**: 중복 코드, 과도한 복잡도, 가독성 저하 등
- **사소한 스타일 가이드 위반**: import 순서, 공백, 변수명 등
- **개선 가능한 구조**: 더 나은 추상화, 더 명확한 분리 등
- **부족한 에러 처리**: 예외 상황에 대한 핸들링 보완 필요
- **문서화 부족**: 복잡한 로직에 대한 주석이나 JSDoc 필요

### p4 (사소한 의견입니다)
- **선호도 차이**: 개인적 코딩 스타일 선호
- **미세한 개선**: 아주 작은 리팩토링 제안
- **참고 사항**: 알아두면 좋은 대안이나 패턴
- **일관성**: 기존 코드와의 일관성 유지 제안

### Q (질문입니다)
- **의도 확인**: 특정 구현 방식을 선택한 이유가 궁금할 때
- **명세 확인**: 요구사항이 모호하거나 불분명할 때
- **추가 정보 요청**: 코드만으로는 판단이 어려운 경우
- **토론 제안**: 여러 방법 중 어느 것이 나은지 논의하고 싶을 때

## 리뷰 관점별 체크리스트

### 1. 버그 및 잠재적 이슈
- [ ] Null/undefined 체크 누락
- [ ] 비동기 처리 오류 (unhandled promise rejection 등)
- [ ] Race condition 가능성
- [ ] 엣지 케이스 미처리 (빈 배열, 0, 음수 등)
- [ ] 타입 불일치 또는 타입 강제 변환 오류
- [ ] 보안 취약점 (OWASP Top 10)
- [ ] 메모리 누수 가능성
- [ ] 무한 루프 또는 스택 오버플로우 위험

### 2. 아키텍처 및 디자인 패턴
- [ ] 레이어 분리 준수 (Route → Controller → Service → Model)
- [ ] 단일 책임 원칙 준수
- [ ] 의존성 방향 (상위 레이어가 하위 레이어를 의존)
- [ ] 중복 코드 제거 (DRY 원칙)
- [ ] 적절한 추상화 수준
- [ ] SOLID 원칙 준수
- [ ] 확장 가능한 구조
- [ ] 과도한 엔지니어링 방지 (YAGNI)

### 3. 테스트 커버리지
- [ ] 서비스 로직 테스트 존재 여부
- [ ] 컨트롤러 테스트 존재 여부
- [ ] 모델 테스트 존재 여부
- [ ] 테스트 케이스의 충분성 (happy path, edge cases, error cases)
- [ ] 테스트 코드의 가독성 및 유지보수성
- [ ] Mock/Stub 적절성

## 일반적인 이슈 패턴

### Backend (Node.js/Express)

**보안 이슈**
```javascript
// ❌ SQL Injection 위험
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;

// ✅ Parameterized query 사용
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [req.params.id]);
```

**에러 처리**
```javascript
// ❌ Unhandled promise rejection
async function getData() {
  const result = await fetchData(); // 에러 처리 없음
  return result;
}

// ✅ 적절한 에러 처리
async function getData() {
  try {
    const result = await fetchData();
    return result;
  } catch (error) {
    logger.error('Failed to fetch data', error);
    throw new AppError('CONSOLE-XX-XX', 'Data fetch failed');
  }
}
```

**레이어 분리 위반**
```javascript
// ❌ Controller에서 직접 DB 접근
router.get('/users', async (req, res) => {
  const users = await User.find({});
  res.json(users);
});

// ✅ Service 레이어를 통한 접근
// Controller
router.get('/users', asyncWrapper(async (req, res) => {
  const users = await userService.getAllUsers();
  res.json(users);
}));

// Service
exports.getAllUsers = async () => {
  return await userModel.findAll();
};
```

### Frontend (React)

**불필요한 리렌더링**
```javascript
// ❌ 매번 새 객체/함수 생성
function Component() {
  const handleClick = () => {}; // 매 렌더마다 새 함수
  return <Child onClick={handleClick} />;
}

// ✅ useCallback 사용
function Component() {
  const handleClick = useCallback(() => {}, []);
  return <Child onClick={handleClick} />;
}
```

**디자인 토큰 미사용**
```scss
// ❌ 하드코딩된 색상값
.button {
  color: #3498db;
  padding: 16px;
}

// ✅ Vapor 디자인 토큰 사용
.button {
  color: var(--primary);
  padding: var(--space-100);
}
```

**이벤트 핸들러 네이밍**
```javascript
// ❌ 일관성 없는 네이밍
function Component({ onSubmit }) {
  const submitForm = () => {}; // props와 네이밍이 다름
  return <button onClick={submitForm}>Submit</button>;
}

// ✅ 일관된 네이밍
function Component({ onSubmit }) {
  const handleSubmit = () => {}; // handle prefix
  return <button onClick={handleSubmit}>Submit</button>;
}
```

### 공통

**Magic Number**
```javascript
// ❌ Magic number
if (status === 200) {
  // ...
}

// ✅ 상수로 정의
const HTTP_STATUS = {
  OK: 200,
  NOT_FOUND: 404,
};

if (status === HTTP_STATUS.OK) {
  // ...
}
```

**중복 코드**
```javascript
// ❌ 중복
function getUserEmail(userId) {
  const user = await User.findById(userId);
  return user.email;
}

function getUserName(userId) {
  const user = await User.findById(userId);
  return user.name;
}

// ✅ 추상화
async function getUser(userId) {
  return await User.findById(userId);
}

async function getUserEmail(userId) {
  const user = await getUser(userId);
  return user.email;
}
```

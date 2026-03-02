# GAX-2026-03-02-003 — 김감사 QA 검수 결과 핫픽스

---
- **태스크 ID**: GAX-2026-03-02-003
- **지시일**: 2026-03-02 22:55
- **지시자**: 용남 대표 (김감사 QA팀장 검수 기반)
- **담당팀**: 강철AX팀
- **상태**: ⬜ 대기
- **우선순위**: 🔴 CRITICAL (보안)
- **QA 원본**: `GD_Agent_teams/kim-qa/reviews/2026-03/QA_최종검수_재무지출계획시스템.md`
---

## 지시 원문

> 강철AX팀에게 전체 QA문서 확인하고 수정하라고 전달해

## 수정 대상 파일

| 파일 | 수정 건수 |
|------|----------|
| `google_apps_script/Code.gs` | C-001, M-002 |
| `google_apps_script/expense_planner_api.gs` | M-001 |
| `google_apps_script/expense_planner.html` | m-002 |

---

## 🔴 CRITICAL (즉시 수정)

### C-001: Claude API 키 코드 내 하드코딩

**파일**: `Code.gs:43`

**현재 (위험)**:
```javascript
const CLAUDE_API_KEY = "[SECRET_MASKED]";
```

**수정 (안전)**:
```javascript
const CLAUDE_API_KEY = PropertiesService.getScriptProperties().getProperty('CLAUDE_API_KEY') || '';
```

**추가 작업**:
1. GAS 에디터에서 `setupClaudeApiKey()` 함수 생성 → 1회 실행
2. GitHub 히스토리에서 키 노출 확인 → 키 로테이션 권장
3. `.gitignore`에 민감 키 관련 주석 추가

---

## 🟠 MAJOR (수정 권장)

### M-001: 프라이버시 필터 — 이름 기반 관리자 체크 취약

**파일**: `expense_planner_api.gs` `getMyExpensePlans()` 321행

**현재**:
```javascript
var adminName = String(adminData[a][1]);
if (userEmail && (userEmail.indexOf(adminName) !== -1 || adminName === userEmail))
```

**수정**:
- 관리자_명단 시트에 **E열: 이메일** 컬럼 추가
- 이메일 정확 일치 비교로 변경:
```javascript
var adminEmail = String(adminData[a][4] || ''); // E열: 이메일
if (userEmail && adminEmail === userEmail)
```

### M-002: Slack 등록 시 이메일 미저장

**파일**: `Code.gs` `handleExpensePlanSubmission()` 함수

**현재**:
```javascript
var result = saveExpensePlan(data, userName, '');
```

**수정**:
- Slack `users.info` API로 이메일 조회하여 전달:
```javascript
var userInfo = JSON.parse(callSlackApi('users.info', { user: userId }).getContentText());
var userEmailFromSlack = (userInfo.ok && userInfo.user) ? userInfo.user.profile.email || '' : '';
var result = saveExpensePlan(data, userName, userEmailFromSlack);
```

---

## 🟡 MINOR (개선 가능)

### m-001: 알림 중복 발송 방지
- `checkPlanReminders()`에서 마지막 알림일을 시트에 기록
- 당일 이미 발송된 건은 스킵 로직 추가

### m-002: escapeHtml 작은따옴표 미처리
- `expense_planner.html` `escapeHtml()` 함수에 `'` → `&#39;` 추가:
```javascript
function escapeHtml(str) {
  if (!str) return '';
  return String(str).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;').replace(/'/g,'&#39;');
}
```

### m-003: 매칭 함수 권한 체크
- `linkExpenseMatch()`, `unlinkExpenseMatch()` 호출 시 등록자 본인 확인 추가

---

## 완료 기준

- [x] QA 문서 확인
- [ ] C-001 수정 + PropertiesService 이관 + API 키 로테이션
- [ ] M-001 수정 + 관리자_명단 이메일 컬럼 추가
- [ ] M-002 수정 + Slack 이메일 조회 연동
- [ ] m-001~m-003 개선
- [ ] `clasp push` + `clasp deploy` 배포
- [ ] 김감사 QA팀 재검수 요청

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-02 22:55 | 최초 태스크 생성 — 김감사 QA C-001/M-001/M-002/m-001~m-003 수정 지시 | 용남 대표 |

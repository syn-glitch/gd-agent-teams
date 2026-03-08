# 🔍 김감사 QA 최종 검수 리포트

---
- **검수 ID**: QA-2026-03-02-FINAL
- **검수일**: 2026-03-02 22:50 KST
- **검수 대상**: 재무 지출 계획 관리 시스템 v1.6 @45
- **검수자**: 김감사 QA팀장
- **검수 파일**: `expense_planner_api.gs`, `expense_planner.html`, `Code.gs`
---

## 📊 검수 결과 요약

| 등급 | 건수 | 상태 |
|------|------|------|
| 🔴 CRITICAL | 1건 | ⚠️ 수정 권장 |
| 🟠 MAJOR | 2건 | ⚠️ 수정 권장 |
| 🟡 MINOR | 3건 | 💡 개선 가능 |
| ✅ PASS | 7건 | 정상 |

---

## ✅ PASS 항목 (7건)

### 1. 입력값 검증 — ✅ PASS
- `validateExpenseInput()`: 빈 필드, 200자 초과, 금액 0~1억 범위 체크 정상
- 수식 주입 방지: `=`, `+`, `-`, `@` 시작 차단 정상
- 날짜 형식 `YYYY-MM-DD` 정규식 검증 정상
- 카테고리 화이트리스트 검증 정상

### 2. 프라이버시 필터 — ✅ PASS
- `getMyExpensePlans()`: 등록자(C열) vs 현재 사용자 비교 필터링 정상
- 관리자 전체 조회 분기 정상 (관리자_명단 시트 기반)
- Slack DM 알림: 등록자 본인에게만 발송 (channel: slackId)

### 3. Slack 커맨드 — ✅ PASS
- `/지출계획` 모달: 필수 4필드 + 선택 2필드 정상
- `handleExpensePlanSubmission()`: 저장 성공/실패 DM 분기 정상
- 웹 페이지 링크 안내 포함

### 4. 알림 시스템 — ✅ PASS
- D-3(📢), D-1(⏰), D-Day(🔔), D+3(⚠️) 날짜 계산 정확
- 취소/완료/알림끄기 건 스킵 로직 정상
- `LockService` 동시 실행 방지 정상
- `MAX_BATCH = 50` 6분 타임아웃 방지 정상
- `🔕 알림 끄기` Block Kit 버튼 + `muteExpensePlanAlert()` 연동 정상

### 5. 매칭 로직 — ✅ PASS
- 금액 ±30% (0.7~1.3배) 범위 계산 정확
- 날짜 ±7일 범위 계산 정확
- 1:N 매칭: `.split(',')` + `.indexOf()` 중복 방지 정상
- `unlinkExpenseMatch()`: 해제 시 금액 재계산 + 상태 롤백 정상

### 6. 양방향 동기화 — ✅ PASS
- `visibilitychange` 이벤트 → 탭 포커스 시 자동 새로고침
- 30초 폴링 (`setInterval`) 목록 탭 활성 시만 실행
- 수동 새로고침 버튼 정상 작동

### 7. LockService 사용 — ✅ PASS
- `saveExpensePlan()`: `lock.waitLock(10000)` + `finally { lock.releaseLock() }`
- `generatePlanId()`: `lock.waitLock(5000)` 동시 채번 방지
- `checkPlanReminders()`: `lock.waitLock(5000)` 중복 실행 방지

---

## 🔴 CRITICAL (1건)

### C-001: Claude API 키 코드 내 하드코딩

**위치**: `Code.gs:43`
```javascript
const CLAUDE_API_KEY = "[SECRET_MASKED]";
```

**문제**: API 키가 코드에 평문으로 노출됨. GitHub에 push되면 키 유출 위험.

**권장 수정**:
```javascript
const CLAUDE_API_KEY = PropertiesService.getScriptProperties().getProperty('CLAUDE_API_KEY');
```
→ 기존 네이버 API 키와 동일한 방식(`PropertiesService`)으로 이관 필요

---

## 🟠 MAJOR (2건)

### M-001: 프라이버시 필터 — 이름 기반 관리자 체크 취약

**위치**: `expense_planner_api.gs` `getMyExpensePlans()` 321행
```javascript
if (userEmail && (userEmail.indexOf(adminName) !== -1 || adminName === userEmail))
```

**문제**: 이름 기반 비교로, 동명이인 또는 이메일에 이름 부분 포함 시 잘못된 관리자 판정 가능.
예: `adminName = "이"` → 모든 이메일에 매칭될 수 있음.

**권장**: 관리자_명단 시트에 **이메일 컬럼** 추가 후 정확한 이메일 비교로 변경

### M-002: handleExpensePlanSubmission — 사용자 이메일 미저장

**위치**: `Code.gs` `handleExpensePlanSubmission()` 함수
```javascript
var result = saveExpensePlan(data, userName, '');
```

**문제**: 세 번째 파라미터(userEmail)를 빈 문자열로 전달. Slack에서 등록한 계획은 `registrant`(C열)에 Slack 사용자 이름만 저장되고, 이메일이 없어서 프라이버시 필터에서 본인 데이터로 인식 못할 수 있음.

**권장**: Slack `users.info` API로 이메일을 조회하거나, Slack ID 기반 필터링 추가

---

## 🟡 MINOR (3건)

### m-001: 알림 중복 발송 방지 로직 없음
- 같은 날 `checkPlanReminders()`가 2회 실행되면 동일 알림 중복 발송
- 권장: 마지막 알림 발송일을 시트에 기록하여 당일 중복 방지

### m-002: escapeHtml에 작은따옴표 미처리
- `escapeHtml()` 함수에서 `'` (작은따옴표) 미이스케이프
- `onclick` 핸들러에서 `planId`에 `'`가 포함되면 XSS 가능성
- 현재 planId는 `PLAN-YYYY-MM-NNN` 형식이므로 실제 위험은 낮음

### m-003: 매칭 시 권한 체크 없음
- `linkExpenseMatch()`, `unlinkExpenseMatch()` 호출 시 본인 계획인지 확인하지 않음
- 현재는 웹 UI에서만 접근 가능하므로 실제 위험 낮음

---

## 📋 검수 판정

> **🟡 조건부 PASS** — 운용 가능하나 C-001(API 키 하드코딩)은 **즉시 수정 권장**

| 항목 | 판정 |
|------|------|
| 기능 완성도 | ✅ 9/9 태스크 완료 |
| 보안 | ⚠️ C-001 수정 필요 |
| 안정성 | ✅ LockService + try-catch 충분 |
| 프라이버시 | ⚠️ M-001 장기 개선 필요 |
| 사용성 | ✅ 테스트 시나리오 전체 정상 |

---

김감사 QA팀장 🫡

# AX-2026-03-04-001 — QA 피드백 기술 부채 분석 및 문서 업데이트

---
- **태스크 ID**: AX-2026-03-04-001
- **지시일**: 2026-03-04
- **담당팀**: 강철 AX팀
- **담당자**: 강철 (AX Lead), 리팩터, 보안전문가
- **상태**: ✅ 완료
- **승인**: ✅ 대표 승인 (2026-03-04, 이해보고서 승인)
---

## 지시 원문

> "김감사 최종 qa 문서 읽고 업데이트하자"
> "각자의 전문 영역에서 하는 원칙을 중심으로 작업하고 강철팀 진행후 자비스에게 넘겨"

## 에이전트 이해 요약

- **핵심 요청**: 김감사 QA 보고서(2026-03-03)의 이슈 중 강철 AX팀 소관 기술 부채를 식별·분석하고, QA 보고서에 AX 분석 섹션을 추가한 뒤 자비스 개발팀에 핸드오버
- **작업 범위**:
  - 포함: TD-001(#4 API 중복 로드 리팩토링 분석), TD-002(#5 하드코딩 권한 리팩토링 분석), QA 문서 업데이트, 자비스 핸드오버
  - 제외: 실제 코드 수정(이번 태스크는 분석·설계·문서), Quick Win(#5 이지은 추가) → 자비스 소관
- **완료 기준**: QA 보고서에 AX 분석 섹션 추가 완료 + 자비스 핸드오버 작성 완료

## 작업 단계

- [x] 단계 1: 코드 현황 분석 (TD-001, TD-002 대상 코드 정밀 분석)
- [x] 단계 2: TD-001 기술 부채 등록 — #4 API 중복 로드 통합 설계
- [x] 단계 3: TD-002 기술 부채 등록 — #5 하드코딩 권한 → 동적 권한 설계
- [x] 단계 4: QA 보고서(`qa/qa_reports/2026-03-03_judy_workspace_user_feedback.md`) 업데이트 — AX 분석 섹션 추가
- [x] 단계 5: 자비스 개발팀 핸드오버 작성

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| QA 보고서 (AX 섹션 추가) | `qa/qa_reports/2026-03-03_judy_workspace_user_feedback.md` | ✅ 완료 |
| task.md | `GD_Agent_teams/gangcheol-ax/tasks/2026-03/task_qa_feedback_ax_analysis.md` | ✅ 생성 |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🤵 자비스 개발팀 | #5 Quick Win(이지은 권한 추가), #1 등록 피드백, #3 달력 미리보기 구현 | ⬜ 대기 |
| 🤵 자비스 개발팀 | TD-001 API 통합 구현 (AX 설계 기반) | ⬜ 대기 |
| 🤵 자비스 개발팀 | TD-002 동적 권한 구현 (AX 설계 기반) | ⬜ 대기 |

## 기술 부채 백로그 항목

### TD-001: API 중복 로드 통합

| 항목 | 내용 |
|------|------|
| TD ID | TD-001 |
| 제목 | 프론트엔드 업무 API 중복 호출 통합 |
| 인입 경로 | 김감사 QA팀 (이슈 #4) |
| 우선순위 | Medium |
| 유형 | refactoring |
| 대상 파일 | `judy_workspace.html`, `web_app.gs` |
| 담당 | 리팩터 |
| 상태 | Backlog (분석 완료) |

**현황 분석**:
- 프론트엔드에서 3곳에서 별도 `google.script.run` 호출:
  - 내 업무 탭: `getMyTasksForWeb()` (html:2934)
  - 칸반 탭: `getAllTasksForWeb()` (html:3351)
  - 캘린더 탭: `getAllTasksForWeb()` (html:3594)
- 백엔드: `getMyTasksForWeb()`은 내부적으로 `getAllTasksForWeb()`을 호출 후 필터링 (web_app.gs:416)
- `getAllTasksForWeb()`은 CacheService 5분 TTL 적용 (web_app.gs:358)
- GAS 통신 오버헤드 ~500ms × 3회 = ~1.5초 불필요 지연

**리팩토링 설계**:
- 앱 초기화 시 `getAllTasksForWeb()` 1회만 호출 → `window.sharedTaskData` 저장
- 내 업무 = `sharedTaskData.filter(...)` (클라이언트 필터링)
- 칸반/캘린더 = `sharedTaskData` 직접 사용
- 데이터 갱신 이벤트 시 1회 재호출 → 모든 뷰 동기 갱신

### TD-002: 하드코딩 권한 → 동적 권한

| 항목 | 내용 |
|------|------|
| TD ID | TD-002 |
| 제목 | 메모 편집 권한 하드코딩 → 시트 기반 동적 권한 |
| 인입 경로 | 김감사 QA팀 (이슈 #5) |
| 우선순위 | Medium |
| 유형 | mixed (refactoring + security) |
| 대상 파일 | `judy_workspace.html` (html:2316~2317, html:3676) |
| 담당 | 리팩터 + 보안전문가 |
| 상태 | Backlog (분석 완료) |

**현황 분석**:
- `window.isAdmin = ["송용남", "정혜림"].includes(uname);` (html:2316)
- `window.FEATURE_MEMO_EDIT_ENABLED_USERS = ["송용남", "정혜림"];` (html:2317)
- 새 사용자 추가 시 코드 수정 + 재배포 필요 → 운영 리스크
- 권한 목록이 프론트엔드에 하드코딩 → 보안 취약점 (클라이언트 조작 가능)

**리팩토링 설계**:
- 백엔드: Users 시트 또는 Settings 시트에서 권한 데이터 관리
- 새 함수: `getUserPermissions(userId)` → `{isAdmin, canEditMemo, ...}` 반환
- 프론트엔드: 초기화 시 권한 API 1회 호출 → `window.userPermissions` 저장
- 하드코딩 배열 삭제, 시트 기반 동적 관리로 전환

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-04 | 최초 생성 | 강철 |

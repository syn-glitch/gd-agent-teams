# JARVIS-2026-03-09-001 — 주디 업무 관리 고도화 (v3.1) 구현

---
- **태스크 ID**: JARVIS-2026-03-09-001
- **지시일**: 2026-03-09
- **담당팀**: 🤵 자비스 개발팀
- **담당자**: 자비스 PO (총괄), 에이다 (BE), 클로이 (FE), 알렉스 (TL)
- **상태**: ✅ 구현 완료 — QA 대기
- **승인**: ✅ 대표 승인 (2026-03-09)
---

## 1. 지시 원문
> **새 업무 등록 기능 개선 및 신규 기능 추가**
> - 프로젝트 직접 등록 (자동 코드 생성)
> - 시작일 필드 추가 및 D-3, D-1, 당일 슬랙 알림 (DM 발송 총 3회)
> - 담당자 지정 -> 수락 프로세스 구현 (신청자 알림 DM 발송)
> - '참고사항' (구 메모) 히스토리 기능: 텍스트 및 링크 연동, 수정/삭제 가능 (단, 내부 로그 기록 보관)

## 2. 세부 작업 단계

### [STEP 1] 아키텍처 및 DB 마이그레이션 (알렉스 & 에이다)
- [x] 1.1 `Tasks` 시트 S열(index 18)에 `Start_Date` 추가 (기존 열 영향 없음)
- [x] 1.2 `Task_References` 시트 자동 생성 + `Action_Type` (INSERT/UPDATE/DELETE) 열 설계
- [x] 1.3 `LockService` 기반 프로젝트 생성 (`getScriptLock`) + 태스크 수정 (`getUserLock`) 패턴 적용

### [STEP 2] 백엔드 API & 트리거 개발 (에이다)
- [x] 2.1 `createProjectFromWeb()` — 프로젝트 자동 코드 생성 (중복 방지 + LockService)
- [x] 2.2 `acceptTaskFromWeb()` — 수락대기 → 대기 + 신청자 DM 발송
- [x] 2.3 `triggerStartDateReminders()` — D-3, D-1, 당일 DM 트리거 함수
- [x] 2.4 `addTaskReference()` / `updateTaskReference()` / `deleteTaskReference()` — 참고사항 CRUD + 로그
- [x] 2.5 `getAllTasksForWeb()` 확장 — requester, startDate 필드 추가
- [x] 2.6 `sendTaskDM()` — 사용자명 → Slack UserID → conversations.open → DM 발송
- [x] 2.7 `registerTaskFromWeb()` 확장 — 담당자 지정 시 수락대기 + DM 발송 + 시작예정일 저장
- [x] 2.8 `VALID_STATUSES`에 "수락대기" 추가 + `setDataValidation(null)` 처리

### [STEP 3] 프론트엔드 UI/UX 구현 (클로이 & 벨라)
- [x] 3.1 프로젝트 등록 인라인 전환 UI (+ 새 프로젝트 버튼 → 입력창 토글)
- [x] 3.2 담당자 셀렉트 + "수락대기" 안내 문구 + 시작예정일 입력 필드
- [x] 3.3 업무 테이블: 수락대기 상태 시 [✅ 수락] 버튼 표시 (본인 담당만)
- [x] 3.4 업무 상세 모달: 참고사항 타임라인 UI (추가/수정/삭제)
- [x] 3.5 참고사항 링크 자동 감지 + HTML sanitize (XSS 방지)
- [x] 3.6 요약바에 🤝 수락대기 카운트 표시

### [STEP 4] QA 및 배포 (자비스 PO & 김감사)
- [ ] 4.1 김감사 QA팀 통합 검수 요청
- [ ] 4.2 알렉스 최종 배포 승인 및 `clasp push`

## 3. 산출물 관리

| 산출물 | 파일 경로 | 상태 |
| :--- | :--- | :--- |
| 기획서 (v3.1) | `planning/2026-03-09_task_system_v3.1_understanding_report.md` | ✅ 완료 |
| 백엔드 소스 | `src/gas/web_app.gs` (기존 파일 확장) | ✅ 완료 |
| 프론트엔드 소스 | `src/gas/judy_workspace.html` + `src/frontend/` 동기화 | ✅ 완료 |
| DM 함수 | `src/gas/web_app.gs` 내 `sendTaskDM()` | ✅ 완료 |

## 4. 구현 상세 (알렉스 TL 코드 리뷰 메모)

### 열 인덱스 영향도
- 기존 A~R열(0~17) **변경 없음**
- S열(index 18) = `Start_Date` (시작예정일) 신규 추가
- `getAllTasksForWeb()`에서 `row[18]` 참조 추가
- `registerTaskFromWeb()`에서 rowData 배열 S열까지 확장
- `triggerStartDateReminders()`에서 `data[i][18]` 참조

### 새 시트
- `Task_References` — 자동 생성 (getOrCreateRefSheet)
- 열: RefID, TaskID, Content, Author, CreatedAt, UpdatedAt, Action_Type

### 트리거 설정 필요
- `triggerStartDateReminders()` → 매일 오전 9시 실행 트리거 수동 설정 필요

---
## 5. 변경 관리 로그
- 2026-03-09: 태스크 문서 최초 생성 및 대표님 결정 사항 반영
- 2026-03-09: STEP 1~3 구현 완료. STEP 4 QA/배포 대기

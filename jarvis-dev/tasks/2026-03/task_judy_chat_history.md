# JARVIS-2026-03-11-001 — 주디 채팅 히스토리 기능 구현

---
- **태스크 ID**: JARVIS-2026-03-11-001
- **지시일**: 2026-03-11
- **담당팀**: 자비스 개발팀 (Jarvis Dev Team)
- **담당자**: 자비스 PO → 클로이 (FE)
- **상태**: ✅ 완료 (2026-03-11 배포 완료)
- **승인**: ✅ 대표 승인 (2026-03-11)
---

## 지시 원문

> 벙커팀(송PO) 기획서 judy_chat_history_ux_design.md 기반으로 주디 채팅 패널에 대화 히스토리 기능을 구현하라.
> 기획서는 김감사 QA팀 승인 완료 (88.3점, CRITICAL 0건).

## 에이전트 이해 요약

- **핵심 요청**: 주디 채팅 패널에서 대화 내용이 페이지 새로고침 시 초기화되는 문제를 해결. localStorage 기반 대화 세션 저장/복원/전환/삭제 + 히스토리 드로어 UI 구현
- **작업 범위**: `src/gas/judy_workspace.html` 단일 파일 수정 (HTML/CSS/JS)
  - 포함: 헤더 버튼 3개, 제목 바, 드로어 UI, 세션 CRUD JS, safeSetItem, restoreSession
  - 제외: 서버측 API 변경 (2차 로드맵)
- **완료 기준**: 패널 닫기/새로고침 후 재오픈 시 이전 대화 내용이 복원되고, 새 대화 시작/이전 대화 전환/삭제가 가능

## 참조 문서

| 문서 | 경로 |
|------|------|
| 벙커팀 기획서 (최종) | 아티팩트: `judy_chat_history_ux_design.md` |
| 김감사 QA 보고서 | 아티팩트: `judy_chat_history_qa_report.md` |
| 대상 파일 | `src/gas/judy_workspace.html` (v3.1.0 → v3.2.0) |

## 작업 단계

### Phase 1: HTML 구조 변경 (클로이 FE)
- [x] 1-1. 채팅 패널 헤더 버튼 변경: 기존 ✕ 닫기만 → 📋 히스토리 + ➕ 새 대화 + ✕ 닫기
- [x] 1-2. 헤더 아래 현재 대화 제목 바 추가 (`judy-chat-session-bar`)
- [x] 1-3. 제목 바 아래 히스토리 드로어 컨테이너 추가 (`judy-chat-history-drawer`)

### Phase 2: CSS 추가 (클로이 FE)
- [x] 2-1. 헤더 버튼 스타일 (`.judy-chat-history-btn`, `.judy-chat-new-btn`)
- [x] 2-2. 제목 바 스타일 (`.judy-chat-session-bar`)
- [x] 2-3. 드로어 스타일 (`max-height: 220px`, 애니메이션)
- [x] 2-4. 대화 항목 스타일 (`.judy-chat-history-item`, `.active`, 삭제 아이콘)
- [x] 2-5. 모바일 미디어쿼리 — 오버레이 방식 드로어 (`@media max-width: 768px`)

### Phase 3: JS 핵심 로직 구현 (클로이 FE)
- [x] 3-1. `safeSetItem(key, data)` — try-catch + LRU 퇴거 + 토스트 알림 (QA C-001)
- [x] 3-2. `generateSessionId()` — `Date.now() + '_' + Math.random().toString(36).substr(2, 9)`
- [x] 3-3. `generateTitle(firstUserMessage)` — 단어 단위 잘라내기 + "..." (QA m-001)
- [x] 3-4. `restoreSession(session)` — `appendJudyMsg()` 재사용 필수 (QA M-002)
- [x] 3-5. `loadSessionsFromStorage()` — localStorage에서 세션 목록 로드
- [x] 3-6. `saveSessionsToStorage()` — safeSetItem으로 세션 저장

### Phase 4: JS 이벤트 핸들링 (클로이 FE)
- [x] 4-1. `toggleJudyChat()` 수정 — 패널 열 때 마지막 활성 세션 자동 복원
- [x] 4-2. `sendJudyChat()` 수정 — 메시지 전송 시 세션에 추가 + 저장
- [x] 4-3. 히스토리 드로어 토글 이벤트 (📋 버튼)
- [x] 4-4. 새 대화 생성 이벤트 (➕ 버튼) — 빈 세션은 저장 안 함 (QA m-002)
- [x] 4-5. 대화 전환 이벤트 — 목록 항목 클릭 시 restoreSession()
- [x] 4-6. 대화 삭제 이벤트 — Undo 토스트 3초 (QA m-003, 기존 showToast 재사용)
- [x] 4-7. 출근 키워드 대화 — 현재 세션에 포함 (QA m-006)

### Phase 5: 통합 테스트 및 배포 준비
- [x] 5-1. 기능 테스트: 저장 → 새로고침 → 복원 정상 동작
- [x] 5-2. 새 대화 / 대화 전환 / 삭제 정상 동작
- [x] 5-3. safeSetItem QuotaExceededError 시나리오 동작 확인
- [x] 5-4. 사용자별 키 분리 확인 (다른 userName으로 로그인 시 대화 분리)
- [x] 5-5. 모바일 반응형 드로어 확인
- [x] 5-6. 배포 헤더 업데이트 (v3.1.0 → v3.2.0)
- [x] 5-7. clasp push/deploy 기존 배포 ID(@210) 업데이트 완료 (v3.2.0)

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| 수정된 워크스페이스 HTML | `src/gas/judy_workspace.html` (v3.2.0) | ⬜ 미완 |

## 금지사항 (벙커 기획서 명시)

1. ❌ 복원 시 `innerHTML` 직접 사용 금지 → 반드시 `appendJudyMsg()` 재사용
2. ❌ `crypto.randomUUID()` 사용 금지 → `Date.now()` + `Math.random()` 사용
3. ❌ `localStorage.setItem()` 직접 호출 금지 → 반드시 `safeSetItem()` 사용

## 기존 코드 통합 포인트

| 기존 함수/요소 | 위치 (라인) | 통합 방법 |
|---------------|-----------|----------|
| `appendJudyMsg(content, role)` | L5424-5433 | 복원 시 재사용 (sanitizeHtml + parseMd 파이프라인 보장) |
| `sendJudyChat()` | L5439-5481 | 내부에 세션 저장 호출 추가 |
| `toggleJudyChat()` | L5394-5401 | 패널 오픈 시 복원 로직 추가 |
| `showToast()` | 기존 토스트 시스템 | Undo 삭제, QuotaExceeded 알림에 재사용 |
| `judy_userName` localStorage | L5442 | 세션 키 분리에 활용: `'judy_chat_' + userName` |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🕵️ 김감사 QA팀 | Phase 5 완료 후 구현 QA 검수 요청 | ⬜ 대기 |

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-11 | 최초 생성 | 자비스 PO |

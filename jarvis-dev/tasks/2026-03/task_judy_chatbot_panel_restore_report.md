# JARVIS-2026-03-08-003 — 주디 AI 비서 챗봇 사이드 패널 복원 개발 보고서

---
- **태스크 ID**: JARVIS-2026-03-08-003
- **지시일**: 2026-03-08
- **담당팀**: 자비스 개발팀
- **담당자**: 자비스 PO (총괄), 클로이 FE
- **최종 버전**: v2.8.4
- **배포**: @167 (GAS Web App)
- **QA**: QA-2026-03-08-001 (김감사 QA팀)
---

## 1. 배경

주디 워크스페이스 우측에 있던 AI 비서 챗봇 사이드 패널이 유실되어 사라진 상태였다.
- 백엔드 API(`processJudyWebChat` in `ai_chat.gs`)는 정상 동작
- 프론트엔드(`judy_workspace.html`)에서 챗봇 UI 코드가 통째로 없었음
- git 히스토리 전체에 챗봇 UI가 한 번도 커밋된 적 없음 (GAS 에디터에서 직접 작업 후 유실 추정)

## 2. 목적

팀원들의 **AI 에이전트 비서**로서, 워크스페이스에 저장된 **노트 메모, 업무, 캘린더, 칸반** 데이터를 RAG로 검색하여 답변하는 주디 챗봇 사이드 패널을 복원한다. 모든 탭(노트/업무/칸반/달력)에서 접근 가능해야 한다.

## 3. 버전별 개발 이력

### v2.8.0 — 최초 복원

| 항목 | 내용 |
|------|------|
| CSS | 사이드 패널 스타일 (380px, 다크/라이트 호환, 모바일 풀스크린) |
| HTML | GNB 우측 "✨ 주디" 토글 버튼, 패널 마크업 (헤더/메시지/입력창) |
| JS | `toggleJudyChat()`, `sendJudyChat()`, `appendJudyMsg()` |
| 연동 | `google.script.run.processJudyWebChat(query, userName)` |
| 렌더링 | AI 응답 마크다운 렌더링 (`parseMd()` + `sanitizeHtml()` 재활용) |
| 배포 | @163 |

**문제**: 패널 클릭 시 열리지 않음

### v2.8.1 — DOM 순서 수정 (1차 시도)

| 항목 | 내용 |
|------|------|
| 수정 | CSS `transform` → `right` 속성 변경, 패널 HTML을 `</body>` 직전으로 이동 |
| 배포 | @164 |

**문제**: 여전히 패널 열리지 않음

### v2.8.2 — QA 반려 이슈 수정 (2차 시도)

김감사 QA팀 검수 결과(QA-2026-03-08-001) **반려** — CRITICAL 1건 발견

| QA ID | 심각도 | 이슈 | 수정 |
|-------|--------|------|------|
| F-1 | CRITICAL | 패널 HTML이 `</script>` 뒤에 있어 JS에서 null 참조 | 패널 HTML을 `<script>` 앞으로 이동 |
| F-2 | MAJOR | F-1 종속 (addEventListener null) | F-1 해소로 자동 해결 |
| U-1 | MINOR | 모바일에서 버튼 의미 불명확 | `title="주디 AI 비서"` 추가 |
| U-2 | MINOR | CSS content 애니메이션 비작동 | `<span>` dot opacity 애니메이션 방식 변경 |

| 배포 | @165 |

**문제**: 여전히 패널 열리지 않음 (근본 원인 미해소)

### v2.8.3 — 근본 원인 발견 + 독립 스크립트 분리

**근본 원인 발견**:
기존 메인 `<script>` 블록에서 `confirmModal`, `confirmCancelBtn`, `confirmOkBtn` 요소를 참조하는데, **해당 HTML 요소가 존재하지 않음**. 4409행에서 `null.addEventListener()` → **TypeError** → **스크립트 전체 중단** → 이후 선언된 `toggleJudyChat()` 함수가 등록되지 않음.

| 항목 | 내용 |
|------|------|
| 수정 | 챗봇 JS를 **독립 `<script>` 블록**으로 분리 + IIFE 패턴 적용 |
| 효과 | 기존 스크립트 에러에 영향받지 않음, `window.toggleJudyChat` 전역 등록 |
| 추가 | DOM null 체크 + 콘솔 로그 (`[JudyChat] 패널 초기화 완료`) |
| 배포 | @166 |

**결과**: 패널 열림 성공! 그러나 "로그인 정보가 없습니다" 에러

### v2.8.4 — userName 참조 수정 (최종)

**원인**: 독립 IIFE 안에서 `window.g_userName`을 참조했으나, `g_userName`은 메인 스크립트에서 `let`으로 선언 → `let`은 블록 스코프이므로 `window` 객체에 노출되지 않음 → 항상 `undefined`

| 항목 | 내용 |
|------|------|
| 수정 | `localStorage.getItem('judy_userName')`으로 사용자명 참조 |
| 근거 | 기존 `setAuthenticatedUser()`에서 이미 `localStorage.setItem('judy_userName', uname)` 저장 중 |
| 배포 | @167 (최종) |

## 4. 최종 아키텍처

```
┌─────────────────────────────────────────────────────┐
│ judy_workspace.html (v2.8.4)                        │
│                                                     │
│  ┌─ GNB ─────────────────────────────────────────┐  │
│  │ 🐰 Judy Workspace  노트  업무  칸반  달력      │  │
│  │                        송용남 님  ☀️  ✨ 주디  │  │
│  └───────────────────────────────────────────────┘  │
│                                                     │
│  ┌─ View Container ──┐  ┌─ Chat Panel (380px) ──┐  │
│  │                    │  │ 🐰 주디            ✕ │  │
│  │  (노트/업무/       │  │                       │  │
│  │   칸반/달력)       │  │  AI 메시지 영역       │  │
│  │                    │  │  (마크다운 렌더링)     │  │
│  │                    │  │                       │  │
│  │                    │  │  [입력창] [전송]       │  │
│  └────────────────────┘  └───────────────────────┘  │
│                                                     │
│  <script> 메인 로직 </script>                        │
│  <script> 챗봇 IIFE (독립) </script>                 │
└─────────────────────────────────────────────────────┘
         │
         │ google.script.run.processJudyWebChat(query, userName)
         ▼
┌─────────────────────────────────────────────────────┐
│ ai_chat.gs                                          │
│                                                     │
│  processJudyWebChat(userQuery, userName)             │
│    └─ buildUserContextForChat(user, name)            │
│         ├─ 업무 데이터 (Tasks 시트)                   │
│         ├─ 메모 데이터 (DailyMemo 시트)               │
│         ├─ 캘린더 데이터 (Calendar API)               │
│         └─ 마크다운 노트 (Notes_v2 시트 — RAG)        │
│    └─ Claude API 호출 (claude-sonnet-4-6)     │
│    └─ 응답 반환                                      │
└─────────────────────────────────────────────────────┘
```

## 5. 수정 파일 목록

| 파일 | 변경 내용 |
|------|----------|
| `src/frontend/judy_workspace.html` | 챗봇 CSS + HTML + JS 추가 (v2.7.1 → v2.8.4) |
| `src/gas/judy_workspace.html` | frontend 동기화 복사본 |

## 6. 핵심 기술 결정 사항

| 결정 | 사유 |
|------|------|
| 독립 `<script>` + IIFE 패턴 | 기존 메인 스크립트의 `confirmModal` null 에러로 스크립트 중단 → 챗봇 기능이 기존 버그에 영향받지 않도록 격리 |
| `localStorage` userName 참조 | `let g_userName`은 블록 스코프로 IIFE에서 접근 불가 → 기존 `setAuthenticatedUser()`가 이미 localStorage에 저장하므로 이를 활용 |
| `parseMd()` + `sanitizeHtml()` 재활용 | 마크다운 노트 v2에서 이미 구현·QA 통과한 파서/새니타이저를 AI 응답 렌더링에 재사용 |
| `right` 속성 애니메이션 | GAS iframe에서 `transform: translateX` 대신 `right` 속성이 더 안정적 |

## 7. 알려진 잔여 이슈

| 이슈 | 심각도 | 비고 |
|------|--------|------|
| `confirmModal` HTML 요소 누락 | 기존 버그 (챗봇과 무관) | 메인 스크립트에서 참조하지만 DOM에 없음. 메모 삭제 확인 모달 관련. 별도 태스크로 수정 필요 |
| 대화 히스토리 미유지 | 설계 의도 | 페이지 새로고침 시 대화 내용 초기화. Phase 1.1에서 세션 저장 검토 |

## 8. 배포 이력

| 버전 | 배포 | 날짜 | 내용 |
|------|------|------|------|
| v2.8.0 | @163 | 2026-03-08 | 최초 복원 |
| v2.8.1 | @164 | 2026-03-08 | CSS right 속성 + body 하단 배치 |
| v2.8.2 | @165 | 2026-03-08 | QA F-1/U-1/U-2 수정 (DOM 순서) |
| v2.8.3 | @166 | 2026-03-08 | 독립 스크립트 분리 (근본 원인 해소) |
| v2.8.4 | @167 | 2026-03-08 | localStorage userName 참조 (최종) |

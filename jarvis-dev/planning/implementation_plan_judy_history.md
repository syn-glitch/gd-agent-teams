# 주디 채팅 히스토리 구현 계획

## 목표
주디 채팅 패널에서 대화 내용이 페이지 새로고침 시 초기화되는 문제를 해결. `localStorage` 기반 대화 세션 저장/복원/전환/삭제 + 히스토리 드로어 UI 구현.

## Proposed Changes

### Chat Panel HTML
#### [MODIFY] [judy_workspace.html](file:///Users/syn/Documents/dev/공도 업무 관리/src/gas/judy_workspace.html)

**변경 1 — 채팅 패널 헤더 (L3288-3290)**:
기존 헤더(제목 + 닫기 버튼)를 📋 히스토리 / ➕ 새 대화 / ✕ 닫기 3버튼 구성으로 변경. 버전을 v3.2.0으로 업데이트.

**변경 2 — 제목 바 + 드로어 추가 (L3291 직후)**:
헤더와 메시지 영역 사이에 현재 대화 제목 바(`judy-chat-session-bar`)와 히스토리 드로어 컨테이너(`judy-chat-history-drawer`) 삽입.

**변경 3 — 초기 환영 메시지 제거 (L3292-3293)**:
하드코딩된 초기 AI 메시지를 제거. JS에서 동적으로 생성하도록 전환.

---

### Chat Panel CSS
#### [MODIFY] [judy_workspace.html](file:///Users/syn/Documents/dev/공도 업무 관리/src/gas/judy_workspace.html)

기존 채팅 CSS 블록(L226 이후) 뒤에 히스토리 관련 스타일 추가:
- `.judy-chat-header-btns` — 버튼 그룹 flex 컨테이너
- `.judy-chat-history-btn`, `.judy-chat-new-btn` — 아이콘 버튼
- `.judy-chat-session-bar` — 현재 대화 제목 표시
- `.judy-chat-history-drawer` — 드로어 (max-height 220px + transition)
- `.judy-chat-history-item` — 대화 항목 (active, hover, 삭제 아이콘)
- `@media (max-width: 768px)` — 모바일 오버레이 방식 드로어

---

### Chat Panel JS
#### [MODIFY] [judy_workspace.html](file:///Users/syn/Documents/dev/공도 업무 관리/src/gas/judy_workspace.html)

기존 JudyChat 스크립트 블록(L5352-5491)을 확장:

**새 함수 6개:**
1. `safeSetItem(key, data)` — try-catch + LRU 퇴거 + 토스트 알림
2. `generateSessionId()` — `Date.now() + '_' + Math.random()`
3. `generateTitle(msg)` — 단어 단위 잘라내기 + "..."
4. `restoreSession(session)` — `appendJudyMsg()` 재사용 (XSS 방어)
5. `loadSessions()` — localStorage → 세션 목록 파싱
6. `renderHistoryDrawer()` — 드로어 HTML 동적 생성

**기존 함수 수정 2개:**
1. `toggleJudyChat()` (L5394) — 패널 열 때 마지막 활성 세션 복원
2. `sendJudyChat()` (L5439) — 메시지 전송 후 세션에 추가 + `safeSetItem` 저장

**새 이벤트 핸들러 4개:**
1. `toggleHistoryDrawer()` — 📋 버튼
2. `startNewChat()` — ➕ 버튼 (빈 세션 미저장)
3. `switchSession(sessionId)` — 대화 항목 클릭
4. `deleteSession(sessionId)` — 🗑️ + Undo 토스트 3초

---

### Deploy Header
#### [MODIFY] [judy_workspace.html](file:///Users/syn/Documents/dev/공도 업무 관리/src/gas/judy_workspace.html)

파일 상단 배포 헤더 업데이트: v3.1.0 → v3.2.0, 변경 이력 추가.

---

## Verification Plan

> [!IMPORTANT]
> GAS Web App은 로컬에서 직접 실행이 불가하므로, `clasp push` 후 배포된 웹앱에서 브라우저 수동 테스트를 수행합니다.

### Manual Verification (배포 후 브라우저 테스트)

**테스트 1 — 대화 저장 및 복원:**
1. 주디 채팅 패널을 열고 "오늘 할 일 뭐야?" 입력 → AI 응답 수신
2. 브라우저 F5로 페이지 새로고침
3. 주디 채팅 패널 재오픈
4. ✅ 기대 결과: 이전 대화 "오늘 할 일 뭐야?" + AI 응답이 그대로 표시됨

**테스트 2 — 새 대화 시작 + 히스토리 전환:**
1. 기존 대화가 있는 상태에서 ➕ 새 대화 버튼 클릭
2. 새 질문 입력 → AI 응답 수신
3. 📋 히스토리 버튼 클릭 → 드로어에 2개 대화 표시 확인
4. 이전 대화 항목 클릭
5. ✅ 기대 결과: 이전 대화 내용으로 전환됨. 새 대화도 목록에 유지

**테스트 3 — 대화 삭제 (Undo):**
1. 📋 히스토리 드로어에서 대화 옆 🗑️ 클릭
2. Undo 토스트 표시 확인 (3초 카운트)
3. (a) 토스트에서 "취소" 클릭 → ✅ 기대: 삭제 취소, 대화 유지
4. (b) 3초 대기 → ✅ 기대: 대화 영구 삭제, 목록에서 제거

**테스트 4 — 사용자 분리:**
1. 사용자 A로 로그인 → 대화 진행
2. 사용자 B로 로그인 (다른 브라우저 또는 시크릿 창)
3. ✅ 기대 결과: B는 A의 대화를 볼 수 없음 (빈 상태로 시작)

**테스트 5 — 모바일 반응형:**
1. 브라우저 DevTools에서 모바일 뷰포트(375px) 설정
2. 채팅 패널 열기 → 📋 히스토리 드로어 열기
3. ✅ 기대 결과: 드로어가 오버레이 방식으로 표시, 대화 영역 위에 겹침

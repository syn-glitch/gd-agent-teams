# QA 보고서 — 주디 챗봇 사이드 패널 (v2.8.1)

---
- **QA ID**: QA-2026-03-08-001
- **검수 대상**: `src/frontend/judy_workspace.html` v2.8.1 (JARVIS-2026-03-08-003)
- **검수일**: 2026-03-08
- **검수자**: 김감사 QA팀 (병렬 QA)
- **판정**: ❌ **반려**
---

## 이슈 목록

### 🔍 기능 QA (테스터)

| ID | 심각도 | 이슈 | 상세 | 재현 |
|----|--------|------|------|------|
| F-1 | **CRITICAL** | JS가 DOM보다 먼저 실행 — 패널 미작동 | JS 코드(4406~4479행)가 `<script>` 안에서 `getElementById`로 패널 요소를 참조하지만, 패널 HTML(4483~4497행)은 `</script>` 뒤 `</body>` 직전에 배치됨. 따라서 `judyChatPanel`, `judyChatOverlay`, `judyChatInput` 등 모든 변수가 **null**. `toggleJudyChat()` 호출 시 `Cannot read properties of null` 에러 발생하여 패널이 열리지 않음 | "✨ 주디" 버튼 클릭 → 아무 반응 없음 (콘솔에 TypeError) |
| F-2 | MAJOR | `judyChatInput.addEventListener` null 참조로 페이지 로드 에러 | 4476행에서 `judyChatInput`이 null인 상태에서 `.addEventListener('input', ...)` 호출 → 페이지 로드 시 JS 에러. 이후 스크립트 전체가 중단될 수 있음 | 페이지 로드 → 콘솔 에러 확인 |

### 🛡️ 보안 QA (보안감사관)

| ID | 심각도 | 이슈 | 상세 |
|----|--------|------|------|
| S-1 | MINOR | AI 응답 XSS 방어 조건부 의존 | `appendJudyMsg`에서 `typeof parseMd === 'function'` 체크 후 `sanitizeHtml(parseMd(content))` 사용. 만약 `parseMd`가 undefined면 `textContent`로 fallback하여 안전. 현재 구조상 문제 없으나, 방어 로직이 조건부인 점은 인지 필요 |
| S-2 | MINOR | 사용자 인증 검증 적절 | `g_userName` 미존재 시 "로그인 정보가 없습니다" 반환 — 적절함 |

### 🎨 UX QA (UX검증관)

| ID | 심각도 | 이슈 | 상세 |
|----|--------|------|------|
| U-1 | MINOR | 모바일에서 "주디" 텍스트 숨김 시 버튼 의미 불명확 | 768px 이하에서 `.judy-toggle-label`이 `display: none` → 버튼이 "✨"만 표시. 시각적으로 챗봇 버튼인지 인지 어려움. `title` 속성 또는 아이콘 개선 권장 |
| U-2 | MINOR | 타이핑 인디케이터 CSS 애니메이션 비작동 | `.judy-typing-dots::after`의 `content` 변경 애니메이션은 대부분 브라우저에서 지원하지 않음. `content` 속성은 animatable property가 아님. "주디가 답변 중..." 텍스트가 정적으로만 표시됨 |

## 점수 산출

| 영역 | 점수 | 비중 | 가중 점수 | 비고 |
|------|------|------|-----------|------|
| 기능 | 20/100 | ×0.4 | 8 | CRITICAL 1건 (패널 미작동) — 핵심 기능 전체 불능 |
| 보안 | 90/100 | ×0.3 | 27 | MINOR 2건, 구조적 문제 없음 |
| UX | 75/100 | ×0.3 | 22.5 | MINOR 2건, 기본 설계는 적절 |

**Overall Score: 57.5 / 100**

## 판정

### ❌ 반려

**사유**: CRITICAL 1건 (F-1: DOM 순서 오류로 패널 완전 미작동)

## 수정 필요 항목 (자비스 개발팀 위임)

| 우선순위 | ID | 수정 방향 |
|----------|-----|----------|
| 🔴 즉시 | F-1 | **해결안 A (권장)**: 패널 HTML을 `<script>` 태그 앞으로 이동. 또는 **해결안 B**: JS의 `getElementById` 호출부를 `DOMContentLoaded` 이벤트 또는 `</body>` 직전 별도 `<script>`로 분리 |
| 🔴 즉시 | F-2 | F-1 해결 시 자동 해소 |
| 🟡 권장 | U-1 | 모바일 토글 버튼에 `title="주디 AI 비서"` 추가 또는 🐰 아이콘 사용 |
| 🟡 권장 | U-2 | CSS `content` 애니메이션 대신 3개 `<span>` dot으로 opacity 애니메이션 적용 |

## 재검토 범위

- F-1, F-2 수정 후 패널 열기/닫기 + 메시지 전송 + 응답 수신 동작 확인
- 수정 최소 범위: HTML 배치 순서 또는 JS 실행 타이밍만 변경

---

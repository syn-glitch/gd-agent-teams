## 🕵️ 통합 QA 보고서: 주디 챗봇 사이드 패널 복원 (v2.8.4)

### 기본 정보
| 항목 | 내용 |
|------|------|
| QA ID | QA-2026-03-08-001 |
| 기능명 | 주디 AI 비서 챗봇 사이드 패널 복원 |
| 검수일 | 2026-03-08 |
| 핑퐁 | 1회 / 5회 |
| 대상 파일 | `judy_workspace.html`, `ai_chat.gs` |
| 개발 담당 | 자비스 개발팀 (에이다 BE, 클로이 FE) |

### 병렬 QA 결과
| 영역 | 담당 | 점수 | CRITICAL | MAJOR | MINOR |
|------|------|------|----------|-------|-------|
| 기능 | 테스터 | 80 | 1 | 0 | 0 |
| 보안 | 보안감사관 | 100 | 0 | 0 | 0 |
| UX | UX검증관 | 100 | 0 | 0 | 0 |

### Overall Score
(80 × 0.4) + (100 × 0.3) + (100 × 0.3) = **92점**

### 최종 판정
❌ **반려** — Overall Score는 92점이나, 핵심 기능 작동을 불가능하게 하는 **CRITICAL 이슈 1건**이 발견되어 배포를 반려합니다.

---

### 이슈 목록

**1. [CRITICAL/기능] 챗봇의 AI 답변이 UI에 렌더링되지 않음 (스크립트 에러 발생)**
- **재현 단계**:
  1. 주디 챗봇 사이드 패널 열기
  2. 입력창에 메시지("넷마블" 등) 입력 후 전송
  3. 타이핑 인디케이터가 표시된 후, AI 응답이 화면에 출력되지 않음.
- **기대 결과**: AI의 답변 메시지가 말풍선(또는 블록) 형태로 화면에 정상 출력되어야 함.
- **실제 결과**: 백엔드 API 통신은 성공하여 응답 객체가 반환되나 화면에 표시되지 않음. 브라우저 콘솔에 `[object Object]` 처리 관련 에러 발생 예상.
- **근본 원인 (Root Cause)**:
  프론트엔드와 백엔드 간의 데이터 타입 불일치.
  - 백엔드(`ai_chat.gs` 의 `processJudyWebChat`): `{ success: true, response: aiResponse }` 형태의 **객체**를 반환함.
  - 프론트엔드(`judy_workspace.html` 의 `sendJudyChat`): 반환된 `response` 객체를 문자열로 취급하여 `appendJudyMsg(response, 'ai')`에 그대로 전달함.
  - 이로 인해 문자열 전용 함수인 `parseMd()`가 객체를 파싱하려 시도하다가 에러(TypeError)를 발생시키고 실행이 중단됨.
- **감점**: -20점

---

### 보안 및 UX 검토 코멘트
- **보안**: `Session.getActiveUser().getEmail()`을 우선 참조하여 클라이언트에서 변조된 `userName`의 위험성을 상쇄한 점 훌륭함. API 연동 보안 이상 없음.
- **UX**: 패널 DOM 순서 이동 및 z-index 정리로 모바일 터치 및 표시 이슈(v2.8.2 지적 사항)가 완벽히 해결됨. 

---

### 개선 권고사항 (Action Items for Jarvis Team)
자비스 개발팀 클로이 FE는 다음 사항을 수정하고 재QA를 요청해 주시기 바랍니다.

- `src/frontend/judy_workspace.html` 의 `sendJudyChat` 함수 내부의 `withSuccessHandler` 콜백을 다음과 같이 수정하여 객체의 `response` 프로퍼티를 명시적으로 참조하도록 변경하세요.
  ```javascript
  .withSuccessHandler(function(response) {
      judyTyping.classList.remove('show');
      judyChatSend.disabled = false;
      
      // 백엔드 반환 타입 처리를 위한 방어 로직 추가
      var msgText = response;
      if (response && typeof response === 'object') {
          msgText = response.response || JSON.stringify(response);
      }
      
      if (msgText) {
          appendJudyMsg(msgText, 'ai');
      } else {
          appendJudyMsg('응답을 받지 못했습니다. 다시 시도해주세요.', 'ai');
      }
  })
  ```

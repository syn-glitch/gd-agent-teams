# 📋 [실행 계획서] 주디 챗봇 슬랙 연동 확장 (Slack RAG)

## 프로젝트 개요
- **요청자**: 용남 대표
- **요청 유형**: 기존 시스템 기능 확장 (주디 챗봇 UX 강화)
- **목적**: 주디가 슬랙 내 대화(멘션, 미답변 스레드 등)를 인지하고 요약하여 웹 워크스페이스에서 보고하도록 함.
- **범위**: 슬랙 메시지 검색 로직 개발, RAG 컨텍스트 연동, 요약 프롬프트 최적화.
- **긴급도**: 🟡 보통 (신규 기능 추가)

## 현황 분석
- **AS-IS**: 주디는 현재 시트(업무, 메모, 캘린더)와 드라이브 아카이브만 RAG 컨텍스트로 활용함. 슬랙은 알림 발송 및 단순 커맨드 수신용으로만 기능함.
- **TO-BE**: 사용자가 "오늘 나 태그된 대화 보여줘"라고 하면, 주디가 슬랙 API를 통해 멘션된 메시지를 수집, 요약하여 답변함.

## Proposed Changes

### 1. [NEW] [slack_retrieval.gs](file:///Users/syn/Documents/dev/공도 업무 관리/src/gas/slack_retrieval.gs)
- 슬랙 메시지 수집 전용 모듈 신규 생성.
- **주요 함수**:
    - `getSlackUserMentions(userId, days)`: 특정 사용자가 멘션된 메시지 리스트 반환.
    - `getUnrepliedThreads(channelId, userId)`: 특정 채널에서 사용자가 답변하지 않은 스레드 식별.
    - `getAllActiveChannels()`: Projects 시트를 기반으로 모니터링 대상 채널 목록 확보.

### 2. [MODIFY] [ai_chat.gs](file:///Users/syn/Documents/dev/공도 업무 관리/src/gas/ai_chat.gs)
- `buildUserContextForChat()` 함수 업데이트.
- 사용자의 질문 의도를 파악하여 슬랙 데이터가 필요한 경우 `slack_retrieval.gs` 기능을 호출해 컨텍스트에 병합.
- **추가될 컨텍스트 섹션**: `■ [슬랙 실시간 대화 데이터 (Slack Mentions & Threads)]`

### 3. [MODIFY] [judy_workspace.html](file:///Users/syn/Documents/dev/공도 업무 관리/src/frontend/judy_workspace.html)
- 주디 채팅창 안내 문구 또는 가이드 버튼에 "슬랙 메시지 요약 가능" 내용 추가.

## 태스크 목록

| task_id | title | assigned_agent | required_skills | priority | dependencies |
|---------|-------|----------------|-----------------|----------|--------------|
| T-201 | 슬랙 메시지 검색 API 로직 설계 (멘션/미답변) | 🏴 벙커 송PO | planning | high | null |
| T-202 | `slack_retrieval.gs` 모듈 개발 | 🤖 자비스 Ada | GAS (Slack API) | high | T-201 |
| T-203 | `ai_chat.gs` RAG 컨텍스트 연동 및 프롬프트 보강 | 🤖 자비스 Ada | GAS (AI) | high | T-202 |
| T-204 | 슬랙 연동 기능 사용자 매뉴얼 업데이트 | 📝 박DC | markdown | medium | T-203 |

## 리스크 및 주의사항
- **API 할당량**: `conversations.history` 호출은 빈도가 잦을 경우 Rate Limit에 걸릴 수 있으므로 1분 단위 캐싱 적용 필수.
- **보안/권한**: 주디 봇이 모든 채널에 참여하고 있어야 메시지 읽기가 원활함. (비공개 채널 제외)

---
**예상 완료 시점**: 2026-03-09 16:00 (KST)
**송PO 의견**: 대표님, 이 기능이 구현되면 슬랙을 일일이 확인하지 않아도 주디가 "오늘 놓친 대화"를 챙겨주는 진정한 오케스트레이터로 진화할 것입니다. 승인 부탁드립니다.

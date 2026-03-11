# BNK-2026-03-10-001 — Claude API 토큰 사용량 대시보드

---
- **태스크 ID**: BNK-2026-03-10-001
- **지시일**: 2026-03-10
- **담당팀**: 🏴 벙커 팀 (기획) → 🤵 자비스 개발팀 (구현)
- **담당자**: 송PO (기획), 에이다 BE + 클로이 FE (구현)
- **상태**: 🔄 진행 중
- **승인**: ✅ 대표 승인 (2026-03-10)
- **QA**: 김감사 QA 1차 반려 → 피드백 반영 완료 (v2)
---

## 지시 원문

> 지금 주디워크스페이스의 claude api key 토큰 사용량을 1일 단위로 보고받고 싶어.
> 워크스페이스 별도의 탭을 만들어서 보면 좋을거 같아

## 대표 확인사항

- **열람 권한**: 송용남, 정혜림
- **표시 항목**: 전부 + 팀원별 사용량
- **기간 필터**: 1일 / 7일 / 15일 / 30일

## 에이전트 이해 요약

- **핵심 요청**: Claude API 토큰 사용량을 일별로 추적하여 주디 워크스페이스 전용 탭에서 시각적으로 모니터링
- **작업 범위**: 백엔드(토큰 수집 래퍼 + 조회 API) + 프론트엔드(📈 토큰 탭 신규) + 권한 제어
- **완료 기준**: 주디 워크스페이스에서 송용남/정혜림이 일별 토큰 사용량을 차트+테이블로 확인 가능

---

## 현황 분석

### API 호출 지점 (총 8곳)

| # | 파일 | 함수 | 용도 |
|---|------|------|------|
| 1 | `ai_chat.gs:88` | `processJudyWebChat()` | 주디 채팅 |
| 2 | `ai_briefing.gs:157` | `getDailyBriefing()` | 출근 브리핑 |
| 3 | `ai_task_parser.gs:149` | `extractTasksWithClaude()` | 업무 추출 |
| 4 | `ai_task_parser.gs:216` | `summarizeTextWithClaude()` | 텍스트 요약 |
| 5 | `ai_task_parser.gs:295` | 청크 요약 | 긴 텍스트 분할 |
| 6 | `ai_task_parser.gs:362` | 청크 요약 반복 | 분할 반복 |
| 7 | `ai_task_parser.gs:403` | 최종 병합 요약 | 청크 결과 통합 |
| 8 | `ai_report.gs:125,176` | 일일 보고 생성 | 보고서 2단계 |

### 현재 상태

- Claude API 응답의 `usage` 필드(`input_tokens`, `output_tokens`)를 **전혀 기록하지 않고 버리고 있음**
- 수집 로직부터 신규 구현 필요

---

## 설계 명세 (v2 — QA 피드백 반영)

### A. 데이터 수집 레이어

#### A-1. 공통 래퍼 함수

```
callClaudeAPI(payload, functionName, userName)
  1. UrlFetchApp.fetch() 호출
  2. 응답에서 result.usage.input_tokens, output_tokens 추출
  3. logTokenUsage() 호출 (fail-safe: 로깅 실패해도 AI 응답 정상 반환)
  4. 원래 result 반환
```

- 기존 8곳의 `UrlFetchApp.fetch("https://api.anthropic.com/v1/messages", ...)` → 이 래퍼로 교체
- **fail-safe 필수**: `logTokenUsage()` 에러 시 try-catch로 무시, AI 응답은 반드시 반환

#### A-2. 로그 기록 함수

```
logTokenUsage(functionName, userName, inputTokens, outputTokens, model)
  → TokenUsage 시트에 1행 append
```

#### A-3. TokenUsage 시트 구조 (신규 생성)

| 열 | 필드명 | 설명 | 예시 |
|----|--------|------|------|
| A | timestamp | 호출 시각 | 2026-03-10 14:30:25 |
| B | date | 날짜 (집계용) | 2026-03-10 |
| C | functionName | 호출 함수명 | processJudyWebChat |
| D | userName | 사용자명 | 송용남 |
| E | inputTokens | 입력 토큰 | 1523 |
| F | outputTokens | 출력 토큰 | 487 |
| G | totalTokens | 합계 | 2010 |
| H | model | 모델명 | claude-sonnet-4-20250514 |

#### A-4. 로그 기록 범위 정책 (QA S-1 반영)

> **절대 규칙**: 토큰 수치 + 메타데이터만 기록한다.
> - ✅ 기록 대상: timestamp, date, functionName, userName, inputTokens, outputTokens, totalTokens, model
> - ❌ 기록 금지: 요청 본문(userQuery), 응답 본문(aiResponse), 시스템 프롬프트, 컨텍스트 데이터
> - 사유: 사용자의 업무 내용·메모가 평문으로 시트에 노출되는 것을 방지

#### A-5. 쓰기 성능 전략

- **기본 전략**: 즉시 시트 append (현재 일 호출량 수십 건 수준에서 병목 가능성 낮음)
- **fail-safe**: 로깅 실패 시 AI 응답에 영향 없음 (try-catch)
- **확장 전략**: 성능 이슈 발생 시 에이다 BE 판단으로 CacheService 버퍼링 또는 PropertiesService JSON 누적 방식 전환 가능
- **판단 기준**: append 1회 소요시간이 500ms 초과 시 버퍼링 전환 검토

#### A-6. 데이터 보존 정책 (QA F-4 반영)

- 원본 로그: 90일 보관
- 90일 초과 데이터: 월별 요약 행으로 아카이브 (별도 시트 또는 하단 섹션)
- 아카이브 트리거: 월 1회 시간 트리거 (선택적 구현)

### B. 조회 API

#### B-1. getTokenUsageStats(period, userName)

```javascript
// 파라미터
//   period: 1 | 7 | 15 | 30 (일)
//   userName: 호출자 이름 (권한 검증용)
//
// 반환값
{
  daily: [
    { date: "2026-03-10", inputTokens: 5200, outputTokens: 1800, calls: 12 },
    ...
  ],
  byFunction: [
    { name: "processJudyWebChat", totalTokens: 15000, calls: 30, pct: 45 },
    ...
  ],
  byUser: [
    { name: "송용남", totalTokens: 12000, calls: 25, pct: 36 },
    ...
  ],
  summary: {
    totalTokens: 33000,
    totalCalls: 85,
    estimatedCostUSD: 0.12,
    dailyAvgTokens: 4714
  }
}
```

#### B-2. 백엔드 권한 검증 (QA S-2 반영)

```javascript
const ALLOWED_TOKEN_VIEWERS = ["송용남", "정혜림"];

function getTokenUsageStats(period, userName) {
  if (!ALLOWED_TOKEN_VIEWERS.includes(userName)) {
    return { error: "접근 권한이 없습니다." };
  }
  // ... 조회 로직
}
```

- **프론트엔드 탭 숨기기는 편의 기능일 뿐**, 실제 보안은 백엔드에서 담당

#### B-3. 비용 환산 (QA F-3 반영)

- 모델별 단가를 `PropertiesService`에 저장:
  - `TOKEN_PRICE_INPUT`: 입력 단가 ($/1M tokens) — 기본값 3
  - `TOKEN_PRICE_OUTPUT`: 출력 단가 ($/1M tokens) — 기본값 15
- 모델 변경 시 PropertiesService 값만 수정하면 됨

### C. 프론트엔드 — 📈 토큰 탭

#### C-1. 탭 추가 위치

```
기존: 📝 내 노트 | 📊 내 업무 | 📋 칸반 | 📅 달력 | 🐛 이슈
추가:                                              + 📈 토큰
```

- **GNB**: `<button class="gnb-tab" id="navTokens" onclick="switchMainView('tokens')">📈 토큰</button>`
- **Bottom Nav**: `<button class="bottom-nav-item" id="btnNavTokens" onclick="switchMainView('tokens')">`
- **View Panel**: `<div class="view-panel" id="viewTokens">`
- **switchMainView()**: `tokens` 케이스 추가
- **접근 권한**: `userName`이 허용 목록이 아니면 탭 버튼 자체를 렌더링하지 않음

#### C-2. 탭 레이아웃

```
┌─────────────────────────────────────────┐
│  기간 필터: [1일] [7일] [15일] [30일]      │
├─────────────────────────────────────────┤
│  📊 요약 카드 (4개 가로 배치)              │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐   │
│  │총 토큰│ │호출 수│ │예상비용│ │일 평균│   │
│  │33,000│ │  85  │ │$0.12 │ │4,714 │   │
│  └──────┘ └──────┘ └──────┘ └──────┘   │
├─────────────────────────────────────────┤
│  📊 일별 사용량 차트 (Google Charts)       │
│  ▓▓▓░░ ▓▓▓▓░ ▓▓░░░ ▓▓▓▓▓░             │
│  3/7    3/8    3/9    3/10              │
│  (■ input ░ output)                     │
├─────────────────────────────────────────┤
│  🍩 기능별 비율        │ 👥 팀원별 사용량  │
│  (도넛 차트)           │ (테이블)          │
│  채팅 45%             │ 송용남  36%       │
│  브리핑 20%           │ 정혜림  28%       │
│  요약 15%             │ 이지은  20%       │
│  추출 10%             │ ...              │
│  보고 10%             │                  │
├─────────────────────────────────────────┤
│  📋 최근 호출 로그 (테이블, 최근 50건)      │
│  시간 | 기능 | 사용자 | Input | Output    │
└─────────────────────────────────────────┘
```

#### C-3. 모바일 반응형 전략 (QA U-1 반영)

- **768px 이하**: 요약 카드 2×2 그리드 → 일별 차트(풀와이드) → 기능별 도넛(풀와이드) → 팀원별 카드형 리스트 → 최근 로그(가로 스크롤)
- **차트**: Google Charts 반응형 옵션 `chartArea: { width: '90%' }` 적용
- **터치**: 차트 tooltip은 탭으로 활성화

#### C-4. 차트 라이브러리 (QA U-2 반영)

- **Google Charts** (`google.visualization`) 확정
- GAS HTML 환경에서 검증된 내장 라이브러리, 외부 CDN 불필요
- 사용 차트: ColumnChart (일별), PieChart (기능별)

### D. 단계적 롤아웃 전략 (QA F-2 반영)

| Phase | 범위 | 검증 기준 |
|-------|------|----------|
| Phase 1 | `ai_chat.gs` 1곳만 래퍼 적용 | 채팅 정상 동작 + TokenUsage 시트에 로그 기록 확인 |
| Phase 2 | `ai_briefing.gs`, `ai_report.gs` 추가 (3곳) | 브리핑·보고 정상 + 로그 정확성 |
| Phase 3 | `ai_task_parser.gs` 전체 (5곳) | 요약·추출 정상 + 전체 8곳 완료 |

- 각 Phase 완료 후 대표 확인 → 다음 Phase 진행
- 롤백: 래퍼 내 로깅 try-catch 제거만으로 원복 가능 (AI 호출 로직 자체는 변경 없음)

---

## 작업 단계 (자비스 팀 구현용)

- [ ] 단계 1: TokenUsage 시트 생성 (수동 또는 스크립트)
- [ ] 단계 2: `callClaudeAPI()` 공통 래퍼 + `logTokenUsage()` 구현 → 신규 파일 `ai_token_logger.gs`
- [ ] 단계 3: Phase 1 — `ai_chat.gs` 래퍼 적용 + 동작 확인
- [ ] 단계 4: `getTokenUsageStats()` 조회 API 구현 (권한 검증 포함)
- [ ] 단계 5: 프론트엔드 📈 토큰 탭 UI 구현 (Google Charts, 반응형)
- [ ] 단계 6: Phase 2~3 — 나머지 7곳 래퍼 적용
- [ ] 단계 7: 통합 테스트 → 김감사 QA팀 검수 요청

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| 토큰 로거 (신규) | `src/gas/ai_token_logger.gs` | ⬜ 미완 |
| 채팅 래퍼 적용 | `src/gas/ai_chat.gs` | ⬜ 미완 |
| 브리핑 래퍼 적용 | `src/gas/ai_briefing.gs` | ⬜ 미완 |
| 파서 래퍼 적용 | `src/gas/ai_task_parser.gs` | ⬜ 미완 |
| 리포트 래퍼 적용 | `src/gas/ai_report.gs` | ⬜ 미완 |
| 프론트엔드 탭 | `src/frontend/judy_workspace.html` | ⬜ 미완 |
| 프론트엔드 복사본 | `src/gas/judy_workspace.html` | ⬜ 미완 |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🤵 자비스 개발팀 | 위 단계 1~7 전체 구현 | ⬜ 대기 |
| 🕵️ 김감사 QA팀 | 단계 7 완료 후 통합 QA | ⬜ 대기 |

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-10 | 최초 생성 (v1) | 송PO (벙커팀) |
| 2026-03-10 | QA 피드백 반영 (v2) — S-1 로그 정책, S-2 백엔드 권한, F-2 롤아웃, U-1 모바일, F-3 단가 설정, F-4 보존 정책, U-2 Google Charts 확정 | 송PO (벙커팀) |

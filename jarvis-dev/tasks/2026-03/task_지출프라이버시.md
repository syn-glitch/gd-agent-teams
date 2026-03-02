# JAR-2026-03-02-001 — 카드 지출 등록 프라이버시 기능 개선

---
- **태스크 ID**: JAR-2026-03-02-001
- **원본 태스크**: BNK-2026-03-02-001 (벙커 팀)
- **지시일**: 2026-03-02
- **담당팀**: 자비스 개발팀
- **담당자**: 자비스 PO (기획), 에이다 (BE 구현)
- **상태**: 🔄 진행 중
- **승인**: ⏳ task.md 승인 대기 중
---

## 지시 원문

> **벙커팀 송PO로부터 위임받은 태스크**
>
> BNK-2026-03-02-001 — 카드 지출 등록 프라이버시 기능 개선
> - 현재상황: 슬랙에서 팀원이 영수증, 문자를 업로드해서 파싱 등록까지 끝나면 그 기록이 캐시플로우_에이전트 채팅창에 남아서 모든 팀원이 열람 가능
> - 개선사항: 캐시플로우 에이전트 업로드 등록, 업로드 채팅 이력이 다른 팀원에게 안 보이도록 개선
> - 대표자(송용남, 정혜림)는 다른 팀원의 챗봇에 올린 내용을 실시간으로 확인 가능, 그 외 팀원은 자기 것만 볼 수 있도록 구현

## 에이전트 이해 요약

- **핵심 요청**: Slack 캐시플로우 봇의 지출 등록 내역 프라이버시 보호 기능 구현
- **작업 범위**:
  - ✅ 포함: GAS Code.gs 수정 (DM 전송, 관리자 알림, ADMIN_USERS 업데이트), 가이드 문서 수정
  - ❌ 제외: Slack 워크스페이스 설정, 구글 시트 구조 변경, 새 기능 추가
- **완료 기준**:
  - 일반 팀원은 봇 DM으로만 등록하고 자기 내역만 확인 가능
  - 대표자 2명(송용남, 정혜림)은 `#관리자_지출관리` 채널에서 전체 내역 실시간 확인 가능

## 확정 사항 (벙커팀 문서 기반)

| 항목 | 결정 |
|------|------|
| 정혜림 Slack ID | `U02SK29UVRP` |
| 송용남 Slack ID | `U02S3CN9E6R` (기존) |
| 운영 방식 | 팀원은 봇 DM(1:1)으로만 등록, 공유 채널 사용 중단 |
| 대표자 알림 | `#관리자_지출관리` 비공개 채널에 집중 관리 |

## 작업 단계

### STEP 1: 코드 수정 (에이다 BE 담당)

- [ ] 1-1. `ADMIN_USERS` 배열에 정혜림 ID 추가
  ```javascript
  const ADMIN_USERS = ['U02S3CN9E6R', 'U02SK29UVRP']; // 송용남 + 정혜림
  ```

- [ ] 1-2. 관리자 채널 ID 상수 추가
  ```javascript
  const ADMIN_CHANNEL_ID = 'C관리자채널ID'; // #관리자_지출관리
  ```
  ⚠️ **대표님께 확인 필요**: 관리자 채널 ID 확인

- [ ] 1-3. 봇 응답 메시지를 공유 채널 → 사용자 DM 전송으로 변경
  - `processCardText()` 함수: `channelId` → `event.user` (DM)
  - `processReceiptImage()` 함수: `channelId` → `event.user` (DM)
  - `sendConfirmationMessage()` 함수: `channelId` → `userId` (DM)

- [ ] 1-4. `processAllPendingSaves()` 완료 후 관리자 채널 알림 추가
  ```javascript
  // 저장 완료 후 관리자 채널에 알림 전송
  const adminMessage = `[지출 등록 알림]\n등록자: ${userName}\n금액: ${amount}원\n가맹점: ${merchant}\n항목: ${category}\n시간: ${timestamp}`;
  sendSlackMessage(ADMIN_CHANNEL_ID, adminMessage);
  ```

- [ ] 1-5. 배포 헤더 업데이트 (COMMON_RULES.md 규칙 4)
  - `Code.gs` 최상단 배포 헤더 버전 증가
  - @version, @updated, @change-summary, @features 업데이트

### STEP 2: 가이드 문서 업데이트 (자비스 PO 담당)

- [ ] 2-1. `GUIDE.md` 수정
  - 공유 채널 사용법 삭제
  - **DM 전용 사용법**으로 변경
  - "캐시플로우 봇에게 1:1 DM으로 영수증 전송" 안내 추가

- [ ] 2-2. `ADMIN_GUIDE.md` 수정
  - 관리자 명단에 정혜림 추가
  - `#관리자_지출관리` 채널 설명 추가
  - 알림 메시지 형식 설명 추가

### STEP 3: 김감사 QA팀 검수 (외부 위임)

- [ ] 3-1. Code.gs 수정본 김감사 QA팀에 전달
- [ ] 3-2. QA 보고서 수령 및 피드백 반영

### STEP 4: GAS 배포 (에이다 BE 담당)

- [ ] 4-1. QA 승인 완료 확인
- [ ] 4-2. 대표님께 배포 승인 요청
- [ ] 4-3. GAS 에디터에 Code.gs 복사 → 새 버전 배포
- [ ] 4-4. 배포 완료 보고

## 산출물

| 산출물 | 파일 경로 | 담당 | 상태 |
|--------|----------|------|------|
| GAS 코드 수정 | `google_apps_script/Code.gs` | 에이다 (BE) | ⬜ 미완 |
| 사용 가이드 업데이트 | `GUIDE.md` | 자비스 PO | ⬜ 미완 |
| 관리자 가이드 업데이트 | `ADMIN_GUIDE.md` | 자비스 PO | ⬜ 미완 |

## 기술 구현 상세 (에이다 참고용)

### 변경 포인트 1: 메시지 응답 경로 변경

**기존 (AS-IS)**
```javascript
// 공유 채널로 응답 전송
sendSlackMessage(channelId, "✅ 지출 내역이 등록되었습니다.");
```

**개선 (TO-BE)**
```javascript
// 사용자 DM으로만 전송
sendSlackMessage(event.user, "✅ 지출 내역이 등록되었습니다.");
```

### 변경 포인트 2: 관리자 알림 추가

**기존 (AS-IS)**
```javascript
// processAllPendingSaves() 완료 후 알림 없음
```

**개선 (TO-BE)**
```javascript
function processAllPendingSaves() {
  // ... 기존 로직 ...

  // 저장 완료 후 관리자 채널에 알림
  const adminMessage = formatAdminNotification(savedData);
  sendSlackMessage(ADMIN_CHANNEL_ID, adminMessage);
}

function formatAdminNotification(data) {
  return `[💳 지출 등록 알림]\n` +
         `등록자: ${data.userName}\n` +
         `금액: ${data.amount}원\n` +
         `가맹점: ${data.merchant}\n` +
         `항목: ${data.category}\n` +
         `시간: ${data.timestamp}`;
}
```

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🕵️ 김감사 QA팀 | Code.gs 수정본 코드 리뷰 및 QA | ⬜ 미위임 |

## 확인 필요 사항

❓ **대표님께 확인 필요**:
1. `#관리자_지출관리` 채널 ID는 무엇인가요? (코드에 하드코딩 필요)
2. 기존 `Code.gs` 파일 경로가 `google_apps_script/Code.gs`가 맞나요?
3. 관리자 알림 메시지 형식이 위 예시로 괜찮으신가요?

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-02 | 최초 생성 (벙커팀 BNK-2026-03-02-001 위임받음) | 자비스 PO |

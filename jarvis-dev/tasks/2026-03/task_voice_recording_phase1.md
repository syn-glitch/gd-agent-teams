# JARVIS-2026-03-07-001 — 음성 녹음 Phase 1: 녹음 + Drive 저장 MVP

---
- **태스크 ID**: JARVIS-2026-03-07-001
- **지시일**: 2026-03-07
- **담당팀**: 자비스 개발팀
- **담당자**: 자비스 PO (총괄)
- **상태**: 🔄 구현 완료 — clasp push 승인 대기
- **승인**: ✅ 대표 승인 (2026-03-07)
- **근거 문서**: `GD_Agent_teams/jarvis-dev/planning/2026-03/2026-03-07_proposal_voice_recording.md` (v1.2 확정)
---

## 지시 원문

> 주디 워크스페이스 기능에 녹음 기능을 넣어서 음성 기록, AI 요약을 하고 싶은데 가능할까?
> → 제안서 확정, 의사결정 4건 확정, "진행해"

## 에이전트 이해 요약

- **핵심 요청**: 주디 워크스페이스 노트 탭에 음성 녹음 → Google Drive 저장 MVP 구현
- **작업 범위**: Phase 1만 (녹음 + Drive 저장). Phase 2(Whisper STT + AI 요약)는 사용성 검증 후 별도 착수
- **완료 기준**: 노트 탭에서 녹음 버튼 터치 → 녹음 → Drive 저장 → 메모에 녹음 링크 자동 삽입이 동작

## 확정된 의사결정

| 항목 | 결정 |
|------|------|
| 구현 전략 | Phase 1만 먼저 |
| 녹음 최대 시간 | 10분 |
| 녹음 위치 | 노트 탭 전용 |
| Phase 2 STT API | Whisper (Phase 1 완료 후 착수) |

## 작업 단계

### 백엔드 (에이다 BE)
- [x] 단계 1: `src/gas/voice_recorder.gs` 신규 생성 — `saveVoiceRecording()` 함수
- [x] 단계 2: Drive 폴더 자동 생성 — `주디 음성메모/YYYY-MM/` 구조, 비공개 설정
- [x] 단계 3: `getRecordingList()` — 녹음 목록 조회 API
- [x] 단계 4: `deleteRecording(fileId)` — 녹음 삭제 API

### 프론트엔드 (클로이 FE)
- [x] 단계 5: VoiceRecorder 모듈 구현 — `startRecording()`, `stopAndSave()`, `processAndUpload()`
- [x] 단계 6: iOS Safari 호환 — `getSupportedMimeType()` (audio/mp4 분기), 미지원 브라우저 에러 처리
- [x] 단계 7: 녹음 UI — 노트 탭 footer 🎤 버튼, 녹음 패널(타이머), ⏹ 정지 & 저장 버튼, Toast
- [x] 단계 8: iOS 탭 전환 대응 — `visibilitychange` 감지 → 자동 부분 저장
- [x] 단계 9: 에러 처리 — 네트워크 실패 시 재시도 버튼 (offerRetry)
- [x] 단계 10: 메모장 연동 — `insertRecordingToMemo()` 녹음 완료 시 자동 삽입

### FE ↔ BE 연동 (알렉스 TL 리뷰)
- [x] 단계 11: `google.script.run.saveVoiceRecording()` 연동 코드 완성
- [ ] 단계 12: 최근 녹음 리스트 UI 표시 연동 — Phase 1.1로 이관 (MVP 우선 배포)
- [x] 단계 13: 알렉스 코드 리뷰 — Base64 전송, 15MB 상한 검증, data URL prefix 제거 처리

### QA & 배포
- [x] 단계 14: `src/frontend/judy_workspace.html` → `src/gas/judy_workspace.html` 복사 완료
- [ ] 단계 15: iOS Safari 실기기 테스트 (대표 iPhone) — **clasp push 후 테스트 필요**
- [ ] 단계 16: 김감사 QA팀 검수 요청
- [ ] 단계 17: 배포 헤더 작성 완료 + **대표 승인 후 clasp push 대기**

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| 백엔드 API | `src/gas/voice_recorder.gs` | ✅ 완료 |
| 프론트엔드 (워크스페이스) | `src/frontend/judy_workspace.html` v2.3.0 | ✅ 완료 |
| GAS 배포용 복사본 | `src/gas/judy_workspace.html` | ✅ 동기화 완료 |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🕵️ 김감사 QA팀 | Phase 1 구현 완료 후 기능 QA 검수 요청 | ⬜ 대기 (clasp push 후) |

## 기술 제약 참고

- iOS Safari: `audio/mp4` 포맷만 지원, `audio/webm` 불가
- 녹음 최대 10분 (~10MB), Base64 변환 시 ~13.3MB
- GAS `UrlFetchApp` 50MB 한도, 6분 실행 타임아웃
- Drive 폴더: 비공개(PRIVATE) 강제 설정
- `visibilitychange` 이벤트로 iOS 탭 전환 시 자동 부분 저장

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-07 | 최초 생성 | 자비스 PO |

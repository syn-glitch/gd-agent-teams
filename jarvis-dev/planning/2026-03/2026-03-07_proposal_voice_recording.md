w


# 🎤 주디 워크스페이스 — 음성 녹음 & AI 요약 기능 제안서

---
- **문서 ID**: JARVIS-PROPOSAL-2026-03-07-001
- **작성일**: 2026-03-07
- **작성자**: 자비스 PO (자비스 개발팀)
- **지시자**: 용남 대표
- **상태**: 📋 제안 (김감사 QA팀 검토 대기)
- **대상 시스템**: `src/frontend/judy_workspace.html` + `src/gas/*.gs`
---

## 1. 배경 및 목적

### 1.1 배경

용남 대표로부터 주디 워크스페이스에 **음성 녹음 → AI 요약** 기능 추가 요청을 받았습니다. 회의록, 아이디어 메모, 업무 브리핑 등을 음성으로 빠르게 기록하고, AI가 텍스트로 변환·요약하여 업무 효율을 높이려는 목적입니다.

### 1.2 목적
- 음성으로 빠르게 업무 내용을 기록
- 녹음 파일을 Google Drive에 보관
- AI가 음성→텍스트 변환(STT) + 요약까지 자동 처리

### 1.3 핵심 사용자 환경
- **기기**: iPhone (iOS Safari / Claude 앱 내 PWA)
- **사용 시나리오**: 이동 중 음성 메모, 회의 녹음, 아이디어 기록

---

## 2. 기술 검토 (Feasibility Analysis)

### 2.1 전체 흐름

```
🎤 브라우저 녹음 (MediaRecorder API)
    ↓
💾 Google Drive 저장 (음성 파일 보관)
    ↓
📝 STT 음성→텍스트 변환
    ↓
✨ AI 요약 (기존 주디 AI 요약 기능 재활용)
    ↓
📋 메모장에 텍스트 + 요약 결과 표시
```

### 2.2 기술 스택별 검토

| 단계 | 기술 | iOS Safari 지원 | 비용 | 비고 |
|------|------|:---------------:|------|------|
| **녹음** | MediaRecorder API | ✅ (iOS 14.5+) | 무료 | 출력 포맷: WebM(Chrome) / MP4-AAC(Safari) |
| **Drive 저장** | GAS DriveApp + Base64 | ✅ | 무료 | Blob → Base64 → GAS → Drive 폴더 |
| **STT (Phase 1)** | Web Speech API | ❌ **미지원** | 무료 | iOS Safari 미지원이 치명적 |
| **STT (Phase 2)** | OpenAI Whisper API | ✅ | $0.006/분 | iOS 무관 (서버사이드 처리) |
| **AI 요약** | 기존 주디 AI 요약 | ✅ | 기존 비용 | 이미 구현된 기능 재활용 |

### 2.3 iOS Safari 호환성 상세

| 기능 | iOS Safari 지원 | 최소 버전 | 비고 |
|------|:---------------:|:---------:|------|
| `MediaRecorder` | ✅ | iOS 14.5 | `audio/mp4` 포맷만 지원 (WebM 불가) |
| `getUserMedia` (마이크) | ✅ | iOS 14.3 | HTTPS 필수 (GAS 웹앱 OK) |
| `Web Speech API` | ❌ | — | **iOS Safari 미지원** |
| `Blob / Base64` | ✅ | iOS 11+ | 문제 없음 |
| `FileReader` | ✅ | iOS 11+ | 문제 없음 |

> **결론**: 대표님이 iPhone 사용자이므로, Web Speech API(무료 STT)를 MVP로 사용하는 전략은 **불가**합니다. iOS에서 STT를 구현하려면 **서버사이드 API(Whisper 등)**가 필수입니다.

### 2.4 GAS 환경 제약

| 제약 | 영향 | 대응 전략 |
|------|------|----------|
| `UrlFetchApp` 페이로드 50MB | 긴 녹음 파일 전송 제한 | 10분 단위 분할 녹음 또는 청크 전송 |
| 6분 실행 타임아웃 | Whisper API 호출 + Drive 저장 시간 | 비동기 처리 (트리거 활용) |
| Base64 인코딩 오버헤드 | 파일 크기 ~33% 증가 | 원본 대비 용량 감안하여 설계 |
| 일일 UrlFetch 할당량 20,000회 | 대량 녹음 시 제한 | 실사용 기준 충분 (하루 수십 건 수준) |

---

## 3. 단계별 구현 전략

### Phase 1: 녹음 + Drive 저장 MVP (무료)

**목표**: 음성 녹음 → Drive 보관 기능을 먼저 구현하여 사용성 검증

**구현 범위**:

| 작업 | 담당 | 내용 |
|------|------|------|
| 녹음 UI | 클로이 (FE) | 노트 탭에 🎤 녹음 버튼 추가, 녹음 중 타이머/파형 표시, 정지/저장 |
| 녹음 엔진 | 클로이 (FE) | MediaRecorder API, iOS Safari 호환 (audio/mp4) |
| Drive 저장 API | 에이다 (BE) | `saveVoiceRecording(base64Data, fileName, mimeType)` GAS 함수 |
| Drive 폴더 구조 | 에이다 (BE) | `음성메모/YYYY-MM/YYYY-MM-DD_HH-MM-SS.mp4` |
| 아키텍처 리뷰 | 알렉스 (TL) | Base64 전송 효율, 파일 크기 제한 검증 |

**사용자 플로우**:
```
1. 노트 탭 → 🎤 녹음 버튼 터치
2. 마이크 권한 요청 → 허용
3. 녹음 시작 (타이머 표시: 00:00, 00:01...)
4. ⏹ 정지 버튼 터치
5. "저장 중..." → Drive에 음성 파일 업로드
6. ✅ "녹음이 저장되었습니다" Toast
7. 메모장에 "[🎤 음성 메모 - 2026-03-07 14:30, 2분 15초]" 텍스트 자동 삽입
```

**예상 소요**: 2~3일
**비용**: 무료

---

### Phase 2: Whisper STT + AI 요약 (유료 고도화)

**목표**: Phase 1 녹음 파일을 텍스트로 변환하고 AI 요약까지 자동 처리

**구현 범위**:

| 작업 | 담당 | 내용 |
|------|------|------|
| Whisper API 연동 | 에이다 (BE) | `transcribeAudio(fileId)` → Drive에서 파일 읽기 → Whisper API 호출 |
| STT 결과 저장 | 에이다 (BE) | 변환된 텍스트를 메모에 연결 (시트 또는 Drive 문서) |
| AI 요약 연결 | 에이다 (BE) | 기존 AI 요약 함수에 STT 텍스트 전달 |
| UI 확장 | 클로이 (FE) | 녹음 완료 후 "텍스트 변환 중..." → 변환 결과 표시 → "AI 요약" 버튼 |
| API 키 관리 | 알렉스 (TL) | PropertiesService에 Whisper API 키 안전 보관 |

**사용자 플로우**:
```
1. Phase 1과 동일하게 녹음 + Drive 저장
2. 자동으로 "텍스트 변환 중..." 표시
3. Whisper API → 텍스트 변환 완료
4. 메모장에 변환된 텍스트 표시
5. "✨ AI 요약" 버튼 → 기존 요약 기능으로 요약
6. 최종 결과: 음성파일(Drive) + 텍스트 + AI 요약
```

**예상 소요**: 3~5일 (Phase 1 완료 후)
**비용**: Whisper API $0.006/분 (약 4원/분)

**비용 시뮬레이션**:

| 사용 패턴 | 일일 녹음 | 월간 비용 (추정) |
|----------|----------|:---------------:|
| 가벼운 사용 | 5분 × 3건 | 약 1,400원 |
| 보통 사용 | 10분 × 5건 | 약 4,600원 |
| 헤비 사용 | 30분 × 5건 | 약 14,000원 |

---

## 4. 아키텍처 설계 (고수준)

### 4.1 모듈 구조

```
[Frontend - judy_workspace.html]
├── VoiceRecorder 모듈 (신규)
│   ├── startRecording()      — MediaRecorder 시작
│   ├── stopRecording()       — 녹음 정지 + Blob 생성
│   ├── uploadToServer()      — Base64 → GAS 전송
│   └── UI (버튼, 타이머, 파형)
│
[Backend - voice_recorder.gs (신규)]
├── saveVoiceRecording()      — Drive에 음성 파일 저장
├── transcribeAudio()         — [Phase 2] Whisper API 호출
├── getRecordingList()        — 녹음 목록 조회
└── deleteRecording()         — 녹음 삭제

[기존 모듈 재활용]
├── AI 요약 함수              — 기존 주디 요약 기능
└── Drive 폴더 관리           — 기존 메모 저장 구조 참고
```

### 4.2 데이터 흐름

```
Phase 1:
  Browser → MediaRecorder → Blob(MP4)
    → FileReader.readAsDataURL() → Base64 String
    → google.script.run.saveVoiceRecording(base64, name, mime)
    → GAS: Utilities.newBlob(decoded, mime, name)
    → DriveApp.getFolderById(folderId).createFile(blob)

Phase 2 (추가):
  GAS: DriveApp.getFileById(fileId).getBlob()
    → UrlFetchApp.fetch("https://api.openai.com/v1/audio/transcriptions", ...)
    → JSON 응답에서 텍스트 추출
    → 기존 AI 요약 함수에 전달
```

### 4.3 파일 저장 구조 (Drive)

```
📁 주디 음성메모/
├── 📁 2026-03/
│   ├── 🎵 2026-03-07_14-30-15_음성메모.mp4
│   ├── 🎵 2026-03-07_16-45-22_음성메모.mp4
│   └── 🎵 2026-03-08_09-10-05_음성메모.mp4
└── 📁 2026-04/
    └── ...
```

---

## 5. iOS Safari 대응 전략

### 5.1 MediaRecorder 포맷 분기

```javascript
// iOS Safari는 audio/webm 미지원 → audio/mp4 사용
function getSupportedMimeType() {
    if (MediaRecorder.isTypeSupported('audio/webm;codecs=opus')) {
        return 'audio/webm;codecs=opus';  // Chrome, Android
    } else if (MediaRecorder.isTypeSupported('audio/mp4')) {
        return 'audio/mp4';  // iOS Safari
    } else {
        return 'audio/wav';  // 폴백
    }
}
```

### 5.2 iOS 주의사항

| 항목 | 대응 |
|------|------|
| 자동 재생 제한 | 녹음 시작은 반드시 사용자 터치 이벤트에서 호출 |
| 백그라운드 녹음 | iOS에서 탭 전환 시 녹음 중단 → 사용자에게 경고 표시 |
| PWA 마이크 권한 | 홈 화면 추가(PWA) 상태에서도 권한 요청 가능 (iOS 14.5+) |
| 파일 크기 | iOS MP4-AAC 약 1MB/분 → 10분 녹음 ~10MB (GAS 전송 가능) |

---

## 6. 리스크 및 대응

| 리스크 | 영향도 | 발생 확률 | 대응 방안 |
|--------|:------:|:---------:|----------|
| iOS Safari MediaRecorder 버그 | 높음 | 낮음 | 폴리필 또는 대체 라이브러리 준비 |
| 녹음 파일 50MB 초과 (긴 회의) | 중간 | 중간 | 10분 자동 분할 + 이어서 녹음 UI |
| GAS 6분 타임아웃 (Phase 2 STT) | 중간 | 중간 | 비동기 트리거로 백그라운드 처리 |
| Whisper API 키 노출 | 높음 | 낮음 | PropertiesService 보관, 코드 하드코딩 금지 |
| 마이크 권한 거부 | 낮음 | 중간 | 권한 안내 UI + 설정 유도 메시지 |

---

## 7. 일정 로드맵

```
[Phase 1 — 녹음 + Drive 저장 MVP] ────────── 2~3일
  ├── Day 1: 녹음 UI + MediaRecorder 구현 (클로이)
  ├── Day 1: Drive 저장 GAS API 구현 (에이다)
  ├── Day 2: FE ↔ BE 연동 + iOS 테스트
  ├── Day 2: 알렉스 코드 리뷰
  └── Day 3: 김감사 QA → 배포

[사용성 검증 기간] ──────────────────────── 1~2주
  └── 실사용 데이터 수집 (녹음 빈도, 평균 길이)

[Phase 2 — Whisper STT + AI 요약] ────────── 3~5일
  ├── Day 1-2: Whisper API 연동 (에이다)
  ├── Day 2-3: STT 결과 UI 표시 (클로이)
  ├── Day 3-4: AI 요약 연결 + 비동기 처리
  ├── Day 4: 알렉스 보안 리뷰 (API 키 관리)
  └── Day 5: 김감사 QA → 배포
```

---

## 8. 의사결정 필요 사항

| # | 질문 | 선택지 | 자비스 PO 권장 |
|---|------|--------|--------------|
| 1 | 구현 전략 | A. Phase 1(녹음)만 먼저 / B. Phase 1+2 동시 | **A (단계적)** |
| 2 | 녹음 최대 시간 | 5분 / 10분 / 30분 / 무제한 | **10분** (파일 크기 ~10MB) |
| 3 | 녹음 위치 | 노트 탭 전용 / 전체 탭 공통 | **노트 탭 전용** |
| 4 | Phase 2 STT API | Whisper / Google STT / 둘 다 테스트 | **Whisper** (비용 효율) |

---

## 9. 김감사 QA팀 검토 요청 사항

본 문서는 **구현 전 기술 검토 및 전략 제안서**입니다.
아래 관점에서 QA팀의 사전 검토를 요청합니다:

1. **기술 검토 타당성**: iOS Safari 호환성 분석이 정확한지
2. **리스크 분석 충분성**: 누락된 리스크가 없는지
3. **아키텍처 적절성**: 모듈 구조와 데이터 흐름이 합리적인지
4. **보안 관점**: API 키 관리, 음성 데이터 프라이버시 고려 충분한지
5. **Phase 분리 전략**: 단계적 접근이 적절한지

---

**자비스 PO (자비스 개발팀)** | 2026-03-07

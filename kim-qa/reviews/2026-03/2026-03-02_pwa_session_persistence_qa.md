<!--
 ============================================
 📋 문서 배포 이력 (Deploy Header)
 ============================================
 @file        2026-03-02_pwa_session_persistence_qa.md
 @version     v1.0.0
 @updated     2026-03-02 09:22 (KST)
 @agent       김감사 (김감사 QA팀)
 @ordered-by  용남 대표
 @description PWA 홈 화면 설치 후 세션 유지 실패 문제에 대한 QA 분석 보고서

 @change-summary
   AS-IS: PWA 설치 후 시간 경과 시 "접근 권한 없음" 발생 — 영구 사용 불가
   TO-BE: 근본 원인 3건 식별 + 강철 AX팀 수정 가이드 제공

 @features
   - [추가] 인증 아키텍처 전수 분석 결과
   - [추가] CRITICAL 이슈 3건 식별 및 수정 가이드
   - [추가] 강철 AX팀 위임 안내

 ── 변경 이력 ──────────────────────────
 v1.0.0 | 2026-03-02 09:22 | 김감사 | 최초 작성
 ============================================
-->

# 🕵️ QA 분석 보고서 — PWA 세션 유지 실패

---
- **QA ID**: QA-2026-03-02-001
- **보고일**: 2026-03-02
- **보고자**: 🕵️ 김감사 (QA팀장)
- **대상 파일**: `web_app.gs`, `judy_workspace.html`, `slack_command.gs`
- **심각도**: 🔴 CRITICAL (서비스 사용 불가)
---

## 1. 현상 요약

| 항목 | 내용 |
|------|------|
| **재현 경로** | 슬랙 → 매직 링크 클릭 → Safari에서 "홈 화면에 추가" → 시간 경과 후 앱 재진입 |
| **기대 결과** | 앱처럼 계속 정상 접속 가능 |
| **실제 결과** | ⛔ "접근 권한이 없습니다" 화면 표시 → 서비스 사용 불가 |

---

## 2. 근본 원인 분석 (Root Cause Analysis)

### 🔴 CRITICAL-1: CacheService 기반 세션 — 최대 6시간 후 무조건 소멸

**위치**: `web_app.gs` L51, L191, L217

```javascript
const SESSION_TTL = 21600; // 6시간
cache.put("SESSION_" + sessionToken, userName, SESSION_TTL); // L191
```

**문제**:
- GAS `CacheService`는 **최대 TTL 6시간(21,600초)**이 하드 리밋
- 6시간이 지나면 세션 토큰이 서버 캐시에서 자동 삭제됨
- 사용자가 6시간 이내에 다시 접속하면 TTL이 갱신되지만(L217), **6시간 넘게 앱을 안 열면 무조건 만료**
- PWA 앱에서 재접속 시 URL에 `?session=xxx`이 있어도 서버 캐시에서 삭제되어 인증 실패

> **결론**: CacheService는 "영구 세션"을 지원할 수 없는 구조적 한계

---

### 🔴 CRITICAL-2: 매직 토큰도 10분 TTL — 재방문 시 복구 불가

**위치**: `slack_command.gs` L262, `web_app.gs` L189

```javascript
// slack_command.gs
cache.put("MAGIC_" + magicToken, senderName, 600); // 10분만 유효

// web_app.gs — v2.2.0에서 즉시 삭제는 막았으나 TTL은 여전히 10분
// cache.remove("MAGIC_" + token); // 주석 처리됨
```

**문제**:
- 매직 토큰의 원본 TTL 자체가 `600초(10분)`
- v2.2.0에서 `cache.remove()`를 주석 처리했지만, **10분이 지나면 어차피 GAS CacheService가 자동 삭제**
- 따라서 10분 이후에는 매직 토큰으로도 복구 불가

> **결론**: `cache.remove()` 주석 처리만으로는 근본 해결이 되지 않음

---

### 🔴 CRITICAL-3: 클라이언트 측 영구 세션 저장소 부재

**위치**: `judy_workspace.html` L2190-L2250

```javascript
// 현재 인증 정보 저장 방식
const initSession = '<?= session ?>'; // URL 파라미터에서만 읽음
```

**문제**:
- 인증 성공 후 세션 토큰을 **URL 파라미터에만 의존** (`?session=xxx`)
- `localStorage`나 `sessionStorage`에 세션 정보를 저장하지 않음
- iOS PWA(standalone 모드)에서는 URL 파라미터가 **앱 재시작 시 유실될 수 있음**
- 현재 `localStorage`는 테마 저장(`judy_workspace_theme`)에만 사용 중

> **결론**: 클라이언트에 영구적 인증 정보 저장 메커니즘이 전혀 없음

---

## 3. 이슈 요약 테이블

| # | 심각도 | 이슈 | 위치 | 영향 |
|---|--------|------|------|------|
| C-1 | 🔴 CRITICAL | CacheService 6시간 하드 리밋으로 세션 자동 소멸 | `web_app.gs` L51 | 6시간 미접속 시 100% 로그아웃 |
| C-2 | 🔴 CRITICAL | 매직 토큰 10분 TTL, `remove()` 주석만으로 미해결 | `slack_command.gs` L262 | 10분 후 토큰 복구 불가 |
| C-3 | 🔴 CRITICAL | 클라이언트 영구 세션 저장소 미구현 | `judy_workspace.html` | PWA 재시작 시 인증 유실 |

---

## 4. 수정 가이드 (강철 AX팀 전달용)

### 권장 해결 방안: PropertiesService 기반 영구 세션 + localStorage 이중화

#### 4.1 백엔드 (`web_app.gs`) — PropertiesService로 전환

```
AS-IS: CacheService.getScriptCache() → 최대 6시간 TTL
TO-BE: PropertiesService.getScriptProperties() → TTL 없음 (영구)
```

- `validateToken()`: 세션 토큰을 `PropertiesService`에 저장 (키: `SESSION_{token}`, 값: `{userName, createdAt}`)
- `validateSession()`: `PropertiesService`에서 조회, 만료 판단은 **자체 로직**으로 (예: 30일)
- 세션 만료 시 해당 키만 삭제
- ⚠️ PropertiesService 쓰기 할당량(50,000건/일) 주의 → 대규모 사용 시 별도 시트 기반 세션 스토어 고려

#### 4.2 프론트엔드 (`judy_workspace.html`) — localStorage 이중화

```
AS-IS: URL 파라미터(?session=xxx)에만 의존
TO-BE: localStorage에 세션 토큰 + 사용자명 저장
```

- 인증 성공 시: `localStorage.setItem('judy_session', sessionToken)` + `localStorage.setItem('judy_userName', userName)`
- DOMContentLoaded Auth Flow에 `localStorage` 체크 분기 추가:
  ```
  우선순위: 매직토큰 > URL세션 > localStorage세션 > 접근 거부
  ```
- 로그아웃 시: `localStorage.removeItem()` 처리

#### 4.3 세션 갱신 (Keep-Alive)

- 앱 사용 중 주기적으로 세션 TTL을 갱신하는 heartbeat 로직 추가 (예: 5분마다)
- `document.visibilitychange` 이벤트로 앱이 포그라운드로 전환될 때 세션 유효성 재검증

---

## 5. 판정

| 항목 | 결과 |
|------|------|
| **CRITICAL 이슈** | 3건 |
| **MAJOR 이슈** | 0건 |
| **MINOR 이슈** | 0건 |
| **판정** | ❌ **반려** — CRITICAL 3건 존재, 현재 구조로는 PWA 영구 사용 불가 |

---

## 6. 위임 안내

──────────────────────────
🔀 업무 위임 안내
──────────────────────────
- **발견 내용**: PWA 세션 유지 불가 (CacheService TTL 한계 + 클라이언트 저장소 미구현)
- **이 작업의 담당 팀**: 🔧 강철 AX팀
- **권장 조치**: 위 수정 가이드(4.1~4.3) 기반 인증 아키텍처 리팩토링
- **관련 파일**: `web_app.gs`, `judy_workspace.html`, `slack_command.gs`
- **우선순위 의견**: 🔥 긴급 — 대표님 일상 사용에 직접 영향
──────────────────────────

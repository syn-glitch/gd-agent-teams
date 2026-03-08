# 🕵️ 통합 QA 보고서 — 모바일 환경 UX/UI 개선 사전 검수 (코드 심층 리뷰)

---

### 기본 정보
| 항목 | 내용 |
|------|------|
| QA ID | QA-2026-03-07-001 |
| 기능명 | 모바일 환경 UX/UI 개선 및 수정 (SC-01~SC-05) |
| 검수일 | 2026-03-07 |
| 검수 유형 | **코드 심층 사전 검수** (개발 전 현재 코드 기준 QA) |
| 검수 대상 | `src/frontend/judy_workspace.html` (v2.2.0), `src/frontend/judy_note.html` |
| 테스트 플랜 | `GD_Agent_teams/kim-qa/test-plans/2026-03/2026-03-07_mobile_ux_ui_refinement.md` |
| 핑퐁 | 0회 / 5회 |

---

## 병렬 QA 결과

| 영역 | 담당 | 점수 | CRITICAL | MAJOR | MINOR |
|------|------|------|----------|-------|-------|
| 기능 | 테스터 | 84 | 0 | 1 | 2 |
| 보안 | 보안감사관 | 90 | 0 | 1 | 0 |
| UX | UX검증관 | 65 | 0 | 3 | 3 |

## Overall Score

**(84 x 0.4) + (90 x 0.3) + (65 x 0.3) = 33.6 + 27.0 + 19.5 = 80.1점**

## 최종 판정

⚠️ **조건부 승인** — CRITICAL 0건, Overall Score 80.1점 (≥ 80 경계선)

> 현재 코드는 최소 기준을 간신히 충족하나, UX 영역에서 MAJOR 이슈 3건이 집중되어 있습니다.
> 자비스 개발팀과 진행할 개선 작업에서 아래 이슈를 **필수 수정** 항목으로 반영 요청합니다.

---

## 🔍 테스터 — 기능 QA (84점)

### [MAJOR/기능] ISS-01: 포맷 툴바 버튼 터치 타겟 미달 (SC-04 관련)

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:669-680` |
| **재현** | 모바일에서 메모 작성 시 포맷 툴바(B, I, ●, ☐) 노출 → 버튼 터치 |
| **기대 결과** | 터치 타겟 최소 44x44px |
| **실제 결과** | `.format-btn` = **36x36px** (44px 미달) |
| **심각도** | MAJOR (-10점) |
| **감점 근거** | 모바일 터치 시 오탭 가능성. Apple HIG/Material Design 최소 기준 44px 미달 |

```css
/* 현재 코드 (line 669) */
.format-btn {
    width: 36px;   /* ← 44px 미만 */
    height: 36px;  /* ← 44px 미만 */
}
```

**권장 수정**: `width: 44px; height: 44px;` 또는 `min-width: var(--touch-target-min); min-height: var(--touch-target-min);`

---

### [MINOR/기능] ISS-02: summary-card 카운트 폰트 모바일 미축소

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:1097` (데스크톱), `1444` (모바일 미디어쿼리) |
| **실제 결과** | `.summary-card .count`가 모바일에서도 **28px** 그대로 적용 |
| **심각도** | MINOR (-3점) |

모바일 미디어쿼리(`@media max-width: 768px`)에서 `.summary-card`의 `padding`은 축소(12px)하지만, `.count` 폰트 사이즈(28px)는 축소하지 않음. 상황판이 세로 공간을 과다 점유하는 원인 중 하나.

**권장 수정**: 모바일에서 `.summary-card .count { font-size: 20px; }`

---

### [MINOR/기능] ISS-03: Toast 알림과 bottom-nav 겹침 가능성

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:244` (`.toast { bottom: 40px }`) |
| **실제 결과** | Toast가 `bottom: 40px`에 고정, bottom-nav 높이가 `60px` → 겹침 |
| **심각도** | MINOR (-3점) |

**권장 수정**: 모바일 미디어쿼리에 `.toast { bottom: calc(var(--bottom-nav-height) + env(safe-area-inset-bottom) + 16px); }` 추가

---

## 🛡️ 보안감사관 — 보안 QA (90점)

### [MAJOR/보안] ISS-04: innerHTML 사용에 의한 잠재적 XSS 벡터

| 항목 | 내용 |
|------|------|
| **위치** | 다수 (아래 상세) |
| **악용 시나리오** | 서버 응답에 악성 스크립트가 삽입될 경우 innerHTML로 DOM에 주입 가능 |
| **영향 범위** | 사용자 세션 탈취, 데이터 조작 가능성 (GAS 환경에서 실제 위험도는 낮음) |
| **심각도** | MAJOR (-10점) |

**주요 innerHTML 사용 위치**:

| 파일 | 라인 | 코드 | 위험도 |
|------|------|------|--------|
| `judy_workspace.html` | 2329 | `accessDeniedMsg.innerHTML = reason` | 낮음 (서버 제어값) |
| `judy_workspace.html` | 2482 | `bottomSheetContent.innerHTML = contentHtml` | **중간** (동적 컨텐츠) |
| `judy_workspace.html` | 2671 | `textDiv.innerHTML = escapedContent` | 낮음 (이스케이핑 적용) |
| `judy_note.html` | 958 | `accessDeniedMsg.innerHTML = reason` | 낮음 |
| `judy_note.html` | 1133 | `textDiv.innerHTML = escapedContent` | 낮음 (이스케이핑 적용) |

**긍정 요소**: `escapedContent` 변수명으로 보아 일부 위치에서 이스케이핑을 수행 중.

**권장 대응**:
1. `bottomSheetContent.innerHTML = contentHtml` → 컨텐츠 소스 검증 또는 DOMPurify 적용
2. 장기적으로 `textContent` 또는 DOM API(`createElement`)로 대체 검토

---

### 보안 체크리스트 결과

| 항목 | 결과 | 비고 |
|------|------|------|
| API 키/토큰 하드코딩 | ✅ 없음 | `google.script.run` 사용 |
| LockService 적용 | N/A | 프론트엔드 파일 (서버측 확인 필요) |
| 사용자 입력값 검증 | ⚠️ 부분 적용 | 이스케이핑 일부 적용 |
| HTML 출력 이스케이핑 | ⚠️ 부분 적용 | `escapedContent` 사용하나 일부 innerHTML 직접 사용 |
| 외부 API HTTPS | ✅ 사용 | CDN (jsdelivr, googleapis) 모두 HTTPS |
| 에러 응답 민감 정보 | ✅ 없음 | 에러 메시지 한글화 |

---

## 🎨 UX검증관 — UX QA (65점)

### [MAJOR/UX] ISS-05: SC-01 — 상황판(Summary Bar) 모바일 세로 공간 과다 점유

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:1076-1106` (데스크톱), `1439-1448` (모바일) |
| **기대 결과** | 상황판 크기 축소 → 진행 중 업무 리스트가 한눈에 더 많이 노출 |
| **실제 결과** | 모바일에서 summary-card가 `flex: 1 1 calc(50% - 10px)` (2열) + 5~6개 카드 = **3행 차지** |
| **심각도** | MAJOR (-10점) |

**문제 분석**:
- `.summary-card .count` = 28px (모바일 미축소)
- `.summary-card` 패딩 12px + count 28px + label 12px + margin ≈ **카드당 ~60px**
- 3행 x 60px + 갭 + 패딩 ≈ **상황판 총 높이 ~220px** (뷰포트 40% 이상 점유)
- 접기/펼치기 기능 없음 → 업무 리스트 스크롤 시작점이 매우 낮음

**권장 수정**:
1. 모바일에서 count 폰트 사이즈 축소 (28px → 20px)
2. summary-card 패딩 추가 축소 (12px → 8px)
3. 상황판 접기/펼치기 토글 또는 가로 스크롤 방식 검토
4. 또는 요약 카드를 한 줄 가로 스크롤로 변경 (Netflix 스타일)

---

### [MAJOR/UX] ISS-06: SC-03 — 메모 햄버거 메뉴(☰) 위치 부적절

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:1930, 1962` / `judy_note.html:797` |
| **기대 결과** | ☰ 버튼이 **오른쪽 상단**에 위치하여 한 손 조작 및 시각적 균형 확보 |
| **실제 결과** | ☰ 버튼이 **왼쪽**에 위치 (2곳) |
| **심각도** | MAJOR (-10점) |

**현재 배치 분석**:

| 파일 | 위치 | 현재 포지션 | 테스트 플랜 요구 |
|------|------|-----------|---------------|
| `judy_workspace.html:1930` | editorHeader (읽기 모드) | 좌측 상단 (h2 왼쪽) | 우측 상단 |
| `judy_workspace.html:1962` | editorFooter (쓰기 모드) | 좌측 하단 (footer 내) | 우측 상단 |
| `judy_note.html:797` | top-bar | 좌측 (`margin-right: auto`) | 우측 상단 |

**권장 수정**:
1. editorHeader 내 ☰ 버튼을 `flex` 레이아웃에서 우측 배치로 이동
2. editorFooter의 ☰ 버튼을 editorHeader로 이동 통합
3. judy_note.html의 `margin-right: auto` → `margin-left: auto` 변경

---

### [MAJOR/UX] ISS-07: SC-04 — 포맷 툴바 버튼 터치 타겟 미달 + 하단 여백 부족

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:638-656, 669-680` |
| **기대 결과** | 포맷 툴바가 텍스트를 가리지 않고, 버튼 터치가 용이해야 함 |
| **실제 결과** | format-btn **36x36px** (44px 미달), 위치는 bottom-nav 바로 위 (겹침은 없음) |
| **심각도** | MAJOR (-10점) |

**위치 분석**: `.mobile-format-bar`가 `position: fixed; bottom: calc(var(--bottom-nav-height) + env(safe-area-inset-bottom))` → 탭바 바로 위에 뜨므로 텍스트 영역(상단)과의 겹침은 없음. 다만 `#memoInput`의 `padding-bottom: 100px`이 포맷바+탭바를 감안한 여백인데, 실제 포맷바(~44px) + 탭바(60px) = 104px과 거의 동일하여 여유가 없음.

**권장 수정**:
1. format-btn 크기 44x44px으로 확대
2. `#memoInput` 패딩 하단을 `120px`으로 여유 확보

---

### [MINOR/UX] ISS-08: SC-02 — bottom-nav-item 명시적 터치 타겟 미선언

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:939-950` |
| **실제 결과** | `flex: 1` + 부모 높이 60px → 실측 약 93x60px (충분), 그러나 명시적 `min-height`/`min-width` 선언 없음 |
| **심각도** | MINOR (-3점) |

테스트 플랜 기준 56px 이상 권장 — 현재 60px으로 충족하나, 명시적 선언이 없어 향후 레이아웃 변경 시 보장 불가.

**권장 수정**: `.bottom-nav-item { min-height: 56px; }`

---

### [MINOR/UX] ISS-09: SC-05 — 저장 버튼과 bottom-nav 간 시각적 분리 부족

| 항목 | 내용 |
|------|------|
| **위치** | `judy_workspace.html:982-1003` |
| **실제 결과** | `.view-panel`의 `padding-bottom: var(--bottom-nav-height)` + `.footer`의 `padding: 0 20px 20px 20px` |
| **심각도** | MINOR (-3점) |

footer가 view-panel 내 flexbox 흐름에 위치하고 padding-bottom으로 탭바 공간 확보. 기능적으로는 동작하나, 저장 버튼과 탭바 사이에 **시각적 구분선이나 충분한 여백**이 없어 탭바와 저장 버튼이 시각적으로 뭉쳐 보일 수 있음.

**권장 수정**: footer에 `border-top: 1px solid var(--border)` 또는 `margin-bottom: 8px` 추가

---

### [MINOR/UX] ISS-10: judy_note.html에 bottom-nav 미구현 — workspace와 UX 불일치

| 항목 | 내용 |
|------|------|
| **위치** | `judy_note.html` 전체 |
| **실제 결과** | judy_workspace.html에는 bottom-nav 있으나, judy_note.html에는 없음 |
| **심각도** | MINOR (-3점) |

독립 페이지로 접근 시 하단 탭바 없이 기존 사이드바 방식만 사용. workspace 내 note 뷰와 UX가 달라 사용자 혼란 가능.

---

## 이슈 요약 테이블

| # | 심각도 | 영역 | 시나리오 | 제목 | 감점 |
|---|--------|------|----------|------|------|
| ISS-01 | MAJOR | 기능 | SC-04 | format-btn 36x36px → 터치 타겟 44px 미달 | -10 |
| ISS-02 | MINOR | 기능 | SC-01 | summary-card .count 28px 모바일 미축소 | -3 |
| ISS-03 | MINOR | 기능 | — | Toast 알림 bottom-nav 겹침 가능성 | -3 |
| ISS-04 | MAJOR | 보안 | — | innerHTML 다수 사용 (잠재적 XSS) | -10 |
| ISS-05 | MAJOR | UX | SC-01 | 상황판 모바일 세로 공간 ~220px 과다 점유 | -10 |
| ISS-06 | MAJOR | UX | SC-03 | ☰ 메뉴 버튼 왼쪽 위치 (우측 상단 이동 필요) | -10 |
| ISS-07 | MAJOR | UX | SC-04 | 포맷 툴바 버튼 36px + 하단 여백 부족 | -10 |
| ISS-08 | MINOR | UX | SC-02 | bottom-nav-item 명시적 min-height 미선언 | -3 |
| ISS-09 | MINOR | UX | SC-05 | 저장 버튼-탭바 시각적 분리 부족 | -3 |
| ISS-10 | MINOR | UX | — | judy_note.html bottom-nav 미구현 | -3 |

**합계**: CRITICAL 0건, MAJOR 5건, MINOR 5건

---

## 자비스 개발팀 수정 권고 (우선순위 순)

### 필수 수정 (MAJOR — 배포 전 반드시)

| 순위 | ISS | 수정 내용 | 예상 난이도 |
|------|-----|----------|-----------|
| 1 | ISS-05 | 상황판 모바일 축소 (count 폰트↓, 가로스크롤 또는 접기) | 중 |
| 2 | ISS-06 | ☰ 메뉴 버튼 오른쪽 상단 이동 (workspace 2곳 + note 1곳) | 하 |
| 3 | ISS-07, ISS-01 | format-btn 44x44px 확대 + memoInput 패딩 조정 | 하 |
| 4 | ISS-04 | innerHTML → textContent/DOMPurify 적용 (특히 bottomSheet) | 중 |

### 권장 수정 (MINOR — 다음 스프린트 가능)

| 순위 | ISS | 수정 내용 |
|------|-----|----------|
| 5 | ISS-02 | 모바일 summary-card .count font-size 축소 |
| 6 | ISS-03 | Toast bottom 위치 모바일 대응 |
| 7 | ISS-08 | bottom-nav-item min-height: 56px 명시 |
| 8 | ISS-09 | footer border-top 또는 margin 추가 |
| 9 | ISS-10 | judy_note.html bottom-nav 통합 검토 |

---

## 재검수 안내

본 QA는 **코드 심층 사전 검수**(개발 전 현재 코드 기준)입니다.
자비스 개발팀이 테스트 플랜(SC-01~SC-05) 기반 개선 작업을 완료한 후, **사후 QA(재검수)**를 진행할 예정입니다.

**재검수 시 집중 확인 영역**:
- ISS-01/07: format-btn 터치 타겟 44px 이상 확인
- ISS-05: 상황판 모바일 높이 120px 이하 목표
- ISS-06: ☰ 버튼 오른쪽 상단 배치 확인
- SC-02: bottom-nav 실기기 터치 테스트 (56px 이상)
- SC-05: 저장 버튼 시인성 + 탭바 겹침 없음 확인

---

**김감사 QA팀** | 2026-03-07 | QA Process v2.0 + 코드 심층 리뷰 적용

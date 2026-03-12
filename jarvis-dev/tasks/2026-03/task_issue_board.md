<!--
 ============================================
 📋 문서 배포 이력 (Deploy Header)
 ============================================
 @file        task_issue_board.md
 @version     v1.0.0
 @updated     2026-03-10 (KST)
 @agent       자비스 PO (자비스 개발팀)
 @ordered-by  용남 대표
 @description 이슈 게시판 기능 구현 — 주디 워크스페이스 이슈 제보 → GitHub Issues 연동

 @change-summary
   AS-IS: 이슈 제보 채널 없음
   TO-BE: 워크스페이스 이슈 게시판 → GitHub Issues 자동 등록 + 목록 조회

 @features
   - [추가] task.md 최초 생성

 ── 변경 이력 ──────────────────────────
 v1.0.0 | 2026-03-10 | 자비스 PO | 최초 생성
 ============================================
-->

# JV-2026-03-10-001 — 이슈 게시판 (워크스페이스 → GitHub Issues)

---
- **태스크 ID**: JV-2026-03-10-001
- **지시일**: 2026-03-10
- **담당팀**: 자비스 개발팀
- **담당자**: 자비스 PO → 에이다(BE) + 클로이(FE)
- **상태**: 🔄 진행 중
- **승인**: ✅ 대표 승인 (2026-03-10)
---

## 지시 원문

> 이슈라는 게시판을 만들고 팀원이 주디 워크스페이스를 사용하면서 에러가 있으면 제보하는 게시판을 만들거야. 텍스트, 이미지가 올라오면 됨.
> 게시판에 업로드가 되면 깃허브 이슈에 등록되는거야. https://github.com/syn-glitch/gongdo-task-system/issues

## 에이전트 이해 요약

- **핵심 요청**: 주디 워크스페이스에 이슈 제보 게시판 추가, GitHub Issues 자동 등록
- **작업 범위**: 이슈 제보 폼 UI + 이미지 Drive 업로드 + GitHub Issue 생성 API + 이슈 목록 조회
- **완료 기준**: 팀원이 워크스페이스에서 텍스트+이미지로 이슈 제보 → GitHub Issue 자동 생성 → 이슈 목록에서 open/closed 확인 가능

## 작업 단계

### 백엔드 (에이다)

- [x] 단계 1: `github_issue.gs` 신규 생성
  - [x] 1-1: `createGitHubIssue(title, body, labels)` — GitHub Issues API POST
  - [x] 1-2: `getGitHubIssues(state, page)` — 이슈 목록 조회 (open/closed/all)
  - [x] 1-3: `getGitHubIssueDetail(issueNumber)` — 이슈 상세 조회
  - [x] 1-4: `uploadIssueImageToDrive(base64Data, fileName)` — 이미지 Drive 업로드 + 공유 URL 반환 (공유 설정: `DriveApp.Access.ANYONE_WITH_LINK`, `DriveApp.Permission.VIEW`)
  - [x] 1-5: `submitIssueFromWeb(title, category, severity, description, location, imageBase64, browserInfo)` — 프론트엔드 연동 통합 함수 (1차: 단일 이미지, 2차 확장: 다중 이미지 imageBase64Array)

- [x] 단계 2: 메타데이터 삽입
  - [x] 2-1: Issue body 템플릿 생성 (구조화된 본문 + `<!-- METADATA -->` 블록)
  - [x] 2-2: 환경 정보 삽입 (브라우저, OS, 화면크기 — 프론트에서 전달받음)
  - [x] 2-3: `reporter_slack_id`, `reporter_name` 자동 매핑 (현재 세션 유저 기반)
  - [x] 2-4: 라벨 자동 부여: `from-workspace` + 카테고리 라벨 + 심각도 라벨

- [x] 단계 3: GitHub Labels 초기 설정 + doPost 통합
  - [x] 3-1: 필요 라벨 사전 생성 함수 `setupGitHubLabels()` (from-workspace, bug, ui-broken, feature-error, etc, severity-high/medium/low)
  - [x] 3-2: 기존 `github_to_sheet_webhook.gs`의 `doPost`를 `handleAgentTaskWebhook`으로 리네임
  - [x] 3-3: `github_issue.gs`에 통합 `doPost` 라우터 (payload 타입별 분기) + `agent_sync.gs` `doPost`도 `handleAgentSyncWebhook`으로 리네임
  - [x] 3-4: GitHub Webhook Secret 검증 준비 (`GITHUB_WEBHOOK_SECRET` from PropertiesService)

### 프론트엔드 (클로이)

- [x] 단계 4: 이슈 탭 UI 구현
  - [x] 4-1: GNB + 모바일 하단 네비에 "이슈" 탭 추가
  - [x] 4-2: 이슈 목록 뷰 (카드형, open/closed 탭 분리)
  - [x] 4-3: 이슈 상세 보기 (클릭 시 모달, 마크다운 렌더링, 읽기 전용)

- [x] 단계 5: 이슈 제보 폼 UI
  - [x] 5-1: 제보 폼 모달 (제목, 카테고리 드롭다운, 심각도 드롭다운, 설명 텍스트영역, 발생 위치 자동 감지+수정 가능)
  - [x] 5-2: 이미지 첨부 (파일 선택 + 미리보기, 최대 5MB 제한)
  - [x] 5-3: 환경 정보 자동 수집 (navigator.userAgent, screen 크기)
  - [x] 5-4: 제출 버튼 → `google.script.run.submitIssueFromWeb()` 호출
  - [x] 5-5: 제출 성공/실패 피드백 (토스트 메시지)

- [x] 단계 6: 이슈 목록 연동
  - [x] 6-1: `google.script.run.getGitHubIssues()` 호출 + 서버 캐싱 (CacheService 5분) + 제출 후 캐시 무효화
  - [x] 6-2: 페이지네이션 (더보기 버튼)
  - [x] 6-3: 라벨 뱃지 표시 (GitHub 라벨 색상 그대로 활용)

### 통합 및 동기화

- [x] 단계 7: 프론트엔드 동기화
  - [x] 7-1: `src/frontend/judy_workspace.html` → `src/gas/judy_workspace.html` 복사
  - [ ] 7-2: 배포 헤더 업데이트 (clasp push 전 수행)

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| 이슈 백엔드 API | `src/gas/github_issue.gs` | ✅ 완료 |
| 이슈 게시판 UI | `src/frontend/judy_workspace.html` | ✅ 완료 |
| GAS 배포용 동기화 | `src/gas/judy_workspace.html` | ✅ 완료 |
| doPost 리네임 | `src/gas/github_to_sheet_webhook.gs` | ✅ 완료 |
| doPost 리네임 | `src/gas/agent_sync.gs` | ✅ 완료 |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🕵️ 김감사 QA팀 | 1단계 제안서 QA 검토 → 구현 완료 후 코드 QA | ⬜ 구현 후 |
| 🏴 벙커팀 | 3단계 슬랙 DM 플로우 설계 (병렬 진행 중) | 🔄 진행 중 |

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-10 | 최초 생성 | 자비스 PO |

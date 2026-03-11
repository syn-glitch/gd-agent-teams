# KIM-2026-03-11-001 — Claude API 키 잔액 부족 결함 보고 및 수정 요청

---
- **태스크 ID**: KIM-2026-03-11-001
- **보고일**: 2026-03-11
- **담당팀**: 김감사 QA팀 (Kim QA Team)
- **보고자**: 🕵️ 김감사
- **상태**: 🚨 CRITICAL (BLOCKER)
- **수정 팀**: 🤵 자비스 개발팀 (에이다 BE)
---

## 1. 결함 개요 (Defect Overview)

- **결함명**: Claude API 잔액 부족으로 인한 QA 자동 분석 및 주디 채팅 장애
- **심각도**: **CRITICAL** (자동 분석 게이트가 차단되어 배포 프로세스 지연 발생)
- **증상**: 
  - GitHub 이슈 생성 시 `kim-qa-issue-analysis.yml` 워크플로우 분석 실패.
  - 에러 메시지: `API 에러 (400): Your credit balance is too low to access the Anthropic API.`
  - (예측) 주디 채팅 패널 등 앱 내 AI 기능도 동일한 키 사용 시 장애 발생 우려.

## 2. 재현 경로 (Steps to Reproduce)

1. Judy Workspace에서 이슈 제보 실행.
2. GitHub 이슈 생성 트리거.
3. `analyzing` 라벨 부착 후 GitHub Actions 실행.
4. Python 스크립트의 `callClaudeAPI` 단계에서 400 에러 발생 후 `qa-failed` 라벨 수령.

## 3. 원인 분석 (Root Cause)

- **원인**: 현재 설정된 Anthropic API 키(`ANTHROPIC_API_KEY`)의 크레딧 잔액이 소진됨.
- **영향 범위**:
  - GitHub Actions 워크플로우 (`kim-qa-issue-analysis.yml`, `kim-qa-pr-review.yml` 등)
  - Google Apps Script 프로젝트 (`ScriptProperties` 내 `CLAUDE_API_KEY`)

## 4. 수정 요청 사항 (Remediation)

- **대상 1: GitHub Secrets**
  - 명칭: `ANTHROPIC_API_KEY`
  - 작업: 사용자가 제공한 신규 키로 교체.
- **대상 2: GAS Script Properties**
  - 명칭: `CLAUDE_API_KEY`
  - 작업: 사용자가 제공한 신규 키로 교체 (Judy Workspace 주디 AI 기능 정상화용).

## 5. 담당팀 배정

- **수정 담당**: 🤵 **자비스 개발팀 (에이다 BE)**
- **기술 내용**: 보안 준수를 위해 하드코딩하지 않고 환경 변수 및 프로퍼티 서비스를 통해 업데이트할 것.

---
> [!IMPORTANT]
> **신규 API 키 정보**: 
> `sk-ant-api03-****[마스킹 처리됨]****`
> 보안상의 이유로 API 키를 마스킹 처리했습니다.

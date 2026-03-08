# DOCS-2026-03-08-001 — Skill Update & New Proposal

---
- **태스크 ID**: DOCS-2026-03-08-001
- **지시일**: 2026-03-08
- **담당팀**: 꼼꼼이 문서팀 (Kkoomkkoom Docs Team)
- **담당자**: 꼼꼼이 (Kkoomkkoom, Docs Team Lead)
- **상태**: ✅ 완료
- **승인**: ✅ 대표 승인 (2026-03-08)
---

## 지시 원문

> skill.md 문서를 읽고, 현재까지 같이 작업한 내용을 바탕으로 새롭게 업데이트가 될 skill이 있는지 확인해줘

## 에이전트 이해 요약

- **핵심 요청**: 기존 `gongdo-ax-planner` 스킬 검토 및 최근 RAG/에이전트 운영 패턴을 반영한 신규 스킬 제안/생성.
- **작업 범위**: 
  - `gongdo-ax-planner` 업데이트 체크리스트 반영.
  - 신규 `judy-rag-builder` SKILL.md 작성.
  - 신규 `gd-agent-manager` SKILL.md 작성.
- **완료 기준**: 3개 스킬 문서(수정 1, 신규 2)가 적절한 경로에 배치되고 내용이 검증됨.

## 에이전트 작업 프로세스
- [x] STEP 1: 현행 스킬 및 프로젝트 분석 (gongdo-ax-planner, Judy Chat v2)
- [x] STEP 2: 타 팀 피드백 수함 (자비스, 김감사, 벙커)
- [/] STEP 3: 스킬 업데이트 및 신규 스킬 생성
  - [ ] `gongdo-ax-planner` 업데이트 (OAuth2, 로깅 패턴 강화)
  - [ ] `judy-rag-builder` 신규 생성 (Python/Gradio, RAG 컨텍스트 관리)
  - [ ] `gd-agent-manager` 신규 생성 (Rule Checker, Handover 절차)
  - [ ] (추가) `bunker-task-orchestrator` 초안 작성
- [ ] STEP 4: 최종 검토 및 김감사 QA 요청
- [ ] STEP 5: 배포 및 완료 보고 (꼼꼼이 문서팀)

## 작업 단계

- [x] 단계 1: `gongdo-ax-planner/SKILL.md` 보완 (최근 GAS 이슈 및 배포 헤더 자동화 패턴 추가)
- [x] 단계 2: `judy-rag-builder/SKILL.md` 신규 생성 (Gradio + Python + Drive RAG)
- [x] 단계 3: `gd-agent-manager/SKILL.md` 신규 생성 (`COMMON_RULES` 및 `task.md` 운영 프로토콜)
- [x] 단계 4: 최종 검수 및 대표 보고 (Completion Report)

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| gongdo-ax-planner/SKILL.md | [SKILL.md](file:///Users/syn/.antigravity/skills/gongdo-ax-planner/SKILL.md) | ✅ 완료 |
| judy-rag-builder/SKILL.md | [SKILL.md](file:///Users/syn/.antigravity/skills/judy-rag-builder/SKILL.md) | ✅ 완료 |
| gd-agent-manager/SKILL.md | [SKILL.md](file:///Users/syn/.antigravity/skills/gd-agent-manager/SKILL.md) | ✅ 완료 |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 해당 없음 | | |

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-08 | 최초 생성 및 작업 개시 | 꼼꼼이 |

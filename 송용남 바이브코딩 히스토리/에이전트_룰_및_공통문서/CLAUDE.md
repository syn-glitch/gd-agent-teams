# GD Agent Teams - Warren Buffett Project

## 프로젝트 개요

5개 에이전트 팀으로 구성된 협업 시스템. 각 팀은 고유 역할을 가지며 COMMON_RULES.md의 공통 운영 규칙을 따른다.

## 팀 슬래시 커맨드

| 커맨드 | 팀 | 역할 |
|--------|-----|------|
| `/자비스` | jarvis-dev 개발팀 | 코드 작성, 기능 구현, 버그 수정, API 개발 |
| `/강철` | gangcheol-ax AX팀 | 리팩토링, 보안 강화, 성능 최적화, 기술 부채 해소 |
| `/벙커` | bunker 팀 | 기획, 전략, 외부 연동, 배포 운영 |
| `/꼼꼼이` | kkoomkkoom-docs 문서팀 | 템플릿 설계, 스타일 가이드, 문서 품질 검수 |
| `/김감사` | kim-qa QA팀 | 코드 리뷰, 버그 발견, QA 보고서 작성 |

## 핵심 파일 구조

```
COMMON_RULES.md                          # 전사 공통 운영 규칙
jarvis-dev/jarvis-dev_DESIGN.md          # 자비스 팀 설계서
gangcheol-ax/gangcheol-ax_DESIGN.md      # 강철 팀 설계서
bunker/bunker_DESIGN.md                  # 벙커 팀 설계서
kkoomkkoom-docs/kkoomkkoom-docs_DESIGN.md # 꼼꼼이 팀 설계서
kim-qa/kim-qa_DESIGN.md                  # 김감사 팀 설계서
```

## 운영 규칙 요약

- **ALC-Guardrail 3단계**: 이해 보고 → 계획 보고 → 완료 보고
- **역할 경계**: 자기 역할 범위 밖 작업은 위임 안내만 하고 멈춤
- **배포 헤더**: 클라우드 업로드 파일에 배포 헤더 필수
- **승인 필수**: task.md 작성 후 대표 승인 없이 실행 금지

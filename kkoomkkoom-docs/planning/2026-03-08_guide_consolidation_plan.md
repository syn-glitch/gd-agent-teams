# [PLAN] 전차 사용자 가이드 통합 관리 (manuals/ 집결)

## 목표
산재되어 있는 사용자 가이드 문서를 `GD_Agent_teams/kkoomkkoom-docs/manuals/` 폴더로 한데 모아, 팀원들이 한눈에 모든 매뉴얼을 확인할 수 있도록 구조화합니다.

## 대상 파일 및 이동 계획

### 1. `docs/guides/` (외부 가이드 폴더)
| 원본 경로 | 이동 후 파일명 | 내용 |
|-----------|----------------|------|
| `docs/guides/USER_GUIDE.md` | `01_total_user_guide.md` | 전체 시스템 기초 사용법 |
| `docs/guides/DASHBOARD_GUIDE.md` | `02_dashboard_guide.md` | 업무 현황판 활용법 |
| `docs/guides/JUDY_NOTE_GUIDE.md` | `03_judy_note_guide.md` | 주디 메모 및 RAG 활용법 |
| `docs/guides/SLACK_GUIDE.md` | `04_slack_bot_guide.md` | 슬랙 챗봇 연동 가이드 |
| `docs/guides/OBSIDIAN_INTEGRATION_GUIDE.md` | `05_obsidian_sync_guide.md` | 옵시디언 동기화 설정 |

### 2. `GD_Agent_teams/` (에이전트 인프라 관련)
| 원본 경로 | 이동 후 파일명 | 내용 |
|-----------|----------------|------|
| `GD_Agent_teams/START_GUIDE.md` | `00_agent_start_guide.md` | 에이전트 시스템 협업 가이드 |
| `GD_Agent_teams/jarvis-dev/design/2026-03-02_mobile_ux_guide.md` | `06_mobile_ux_styling_guide.md` | 모바일 최적화 및 스타일 가이드 |

## 작업 프로세스
1. 대상 파일을 `manuals/` 폴더로 복사/이동합니다.
2. 기존 경로에 있던 파일은 삭제하거나, 새 경로로 이동했다는 안내 메시지(Redirect)를 남깁니다.
3. `manuals/INDEX.md` 파일을 생성하여 전체 리스트와 하이퍼링크를 제공합니다.

## 확인 사항
- [ ] 파일 이동 후 마운트된 링크(이미지 등)가 깨지지 않는지 점검.
- [ ] 깃허브에 반영하여 팀원들이 바로 공유받을 수 있도록 조치.

> **위 계획대로 파일을 이동하고 통합 인덱스를 구성할까요?** 꼼꼼이 문서팀이 즉시 실행하겠습니다. 📝

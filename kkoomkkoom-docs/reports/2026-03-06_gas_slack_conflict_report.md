# 📄 GAS-Slack 연동 기술 충돌 조사 보고서 (v1.0)

---
- **문서 번호**: DOCS-2026-03-06-002
- **작성일**: 2026-03-06
- **작성자**: 꼼꼼이 (Docs Lead)
- **상태**: ✅ 작성 완료
- **관련 태스크**: [DOCS-2026-03-06-001](file:///Users/syn/Documents/dev/%EA%B3%B5%EB%8F%84%20%EC%97%85%EB%AC%B4%20%EA%B4%80%EB%A6%AC/GD_Agent_teams/kkoomkkoom-docs/tasks/2026-03/task_gas_slack_conflicts.md)
---

## 1. 개요
Google Apps Script(GAS)를 서버(Web App)로 활용하여 Slack App과 연동할 때 발생하는 기술적 제약 사항과 이를 극복하기 위해 우리 팀이 수립한 표준 대응 방안을 정리한 문서입니다.

## 2. 핵심 충돌 및 제약 사항

### 2.1 슬랙 3초 타임아웃 (Slack 3s Response Limit) 🔴
- **현상**: 슬랙 서버는 모든 요청(명령어, 모달 제출 등)에 대해 3초 이내에 HTTP 200 OK 응답을 받지 못하면 사용자에게 "연결 문제" 에러를 표시합니다.
- **원인**: GAS의 콜드 스타트(Cold Start) 지연(약 2초)과 구글 시트/PropertiesService의 동기적 읽기/쓰기 시간이 합쳐지면 3초를 초과하는 경우가 빈번합니다.

### 2.2 PropertiesService 속도 저하 🟠
- **현상**: 대량의 데이터를 `PropertiesService`에 저장할 때 쓰기 속도가 1초 이상 걸리는 병목 현상이 발생합니다.
- **원인**: GAS의 속성 서비스는 전체 데이터를 읽고 쓰는 구조로, 키-벨류가 많아질수록 성능이 급격히 저하됩니다.

## 3. 검증된 해결 전략 (Best Practices)

### 3.1 캐시 워밍업 (Warmup Trigger) ✅
- **기법**: 매 10분마다 실행되는 시간 기반 트리거(`warmupProjectCache`)를 설정하여 GAS 인스턴스를 활성화 상태로 유지합니다.
- **효과**: 콜드 스타트를 방지하고 프로젝트 옵션 등을 `CacheService`에 미리 상주시킴으로써 응답 속도를 2.5초에서 0.1초 내외로 단축합니다.

### 3.2 비동기 처리 및 낙관적 응답 (Optimistic UI) ✅
- **기법**: 모달 제출 시 즉시 `response_action: "update"`를 통해 "등록 중..." 화면으로 전환하며 슬랙 연결을 종료합니다.
- **효과**: 실제 비즈니스 로직(시트 기록 등)은 `ScriptApp.after(1)` 트리거를 통해 백그라운드에서 처리하므로 타임아웃 에러를 완벽히 차단합니다.

### 3.3 고성능 캐시 활용 (CacheService Migration) ✅
- **기법**: 속도가 느린 `PropertiesService` 대신 10ms 이내의 속도를 보장하는 `CacheService`를 메인 작업 공간으로 활용합니다.

## 4. 관련 참조 문서
- [슬랙 모달 에러 종합 분석 보고서 (김감사)](file:///Users/syn/Documents/dev/%EA%B3%B5%EB%8F%84%20%EC%97%85%EB%AC%B4%20%EA%B4%80%EB%A6%AC/GD_Agent_teams/kim-qa/reviews/2026-02/2026-02-27_slack_modal_error_qa_v2.md)
- [슬랙 모달 타임아웃 핫픽스 계획 (자비스)](file:///Users/syn/Documents/dev/%EA%B3%B5%EB%8F%84%20%EC%97%85%EB%AC%B4%20%EA%B4%80%EB%A6%AC/GD_Agent_teams/jarvis-dev/planning/2026-02/2026-02-27_slack_modal_debugging_plan.md)
- [에이전트 워크플로우 실패 회고 (고통 리포트)](file:///Users/syn/Documents/dev/%EA%B3%B5%EB%8F%84%20%EC%97%85%EB%AC%B4%20%EA%B4%80%EB%A6%AC/GD_Agent_teams/%EA%B3%A0%ED%86%B5_%EB%A6%AC%ED%8F%AC%ED%8A%B8/2026-03-03_GD_Agent_Workflow_%EC%8B%A4%ED%8C%A8_%ED%9A%8C%EA%B3%A0.md)

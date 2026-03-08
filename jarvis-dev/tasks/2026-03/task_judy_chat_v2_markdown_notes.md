# JARVIS-2026-03-08-002 — 주디 챗봇 RAG 고도화: 마크다운 노트 + DB 검색 MVP

---
- **태스크 ID**: JARVIS-2026-03-08-002
- **지시일**: 2026-03-08
- **담당팀**: 자비스 개발팀
- **담당자**: 자비스 PO (총괄)
- **상태**: 🔄 진행 중
- **승인**: ✅ 대표 승인 (2026-03-08)
- **근거 문서**: `GD_Agent_teams/jarvis-dev/planning/2026-03/2026-03-08_judy_chat_v2_architecture_proposal.md` (v1.1 QA 승인)
---

## 지시 원문

> 이제 개발로 넘어가자 task.md 문서 만들어서 시작하자
> c안으로 가자

## 에이전트 이해 요약

- **핵심 요청**: 주디 워크스페이스 노트 탭의 기존 textarea를 강화하여 마크다운 노트 시스템 구축 + 챗봇 RAG 검색 연동
- **작업 범위**: C안 — 기존 textarea 강화 + 마크다운 프리뷰 + Notes_v2 시트 저장 + RAG 검색
- **완료 기준**: 노트 작성 → 마크다운 프리뷰 확인 → Notes_v2 시트 저장 → 주디 챗봇에서 노트 검색 가능

## 확정된 의사결정

| 항목 | 결정 |
|------|------|
| 에디터 방식 | 기존 textarea 강화 (서드파티 에디터 X) |
| 마크다운 파서 | marked.js 인라인 삽입 (CDN 사용 X → iframe 제약 회피) |
| 저장소 | 스프레드시트 Notes_v2 시트 |
| XSS 방어 | DOMPurify 인라인 삽입 + 이중 새니타이징 |
| Phase 0 PoC | 생략 (iframe 제약 없는 C안 채택) |

## 작업 단계

### 백엔드 (에이다 BE)
- [x] 단계 1: Notes_v2 시트 구조 설계 — 컬럼: ID, 제목, 본문(마크다운), 태그, 연결업무ID, 작성자, 작성일, 수정일
- [x] 단계 2: `saveNote(noteData)` — 노트 저장 API (신규/수정 겸용, 50K 하드리밋)
- [x] 단계 3: `getNoteList()` — 노트 목록 조회 API (제목, 태그, 날짜, 최신순 정렬)
- [x] 단계 4: `getNoteById(noteId)` — 단일 노트 상세 조회 API
- [x] 단계 5: `deleteNote(noteId)` — 노트 삭제 API (본인 확인)
- [x] 단계 6: `searchNotesForRAG(query)` — 챗봇 RAG용 노트 검색 API (키워드 + 메타데이터 필터)

### 프론트엔드 (클로이 FE)
- [x] 단계 7: 마크다운 툴바 UI — B/I/S/H1/H2/H3/리스트/체크/인용/코드/링크/프리뷰 (데스크톱+모바일 공용)
- [x] 단계 8: 마크다운 프리뷰 토글 — 편집↔프리뷰 전환, parseMd() + sanitizeHtml()
- [x] 단계 9: 인라인 마크다운 파서 + XSS 새니타이저 — 외부 CDN 의존성 0, script/iframe/on* 제거
- [ ] 단계 10: 노트 CRUD UI — 새 노트/저장/목록/삭제 버튼 + 노트 리스트 패널 → Phase 1.1로 이관
- [x] 단계 11: 셀 용량 보호 — 40,000자 소프트 경고 + 50,000자 하드 리밋 차단 (charCounter)
- [x] 단계 12: 모바일 터치 최적화 — 툴바 버튼 34x34px (md-toolbar), 모바일 포맷바에 프리뷰 버튼 추가

### RAG 연동 (에이다 BE)
- [x] 단계 13: `searchNotesForRAG()` 구현 완료 — 키워드 매칭, 최대 5건, 500자 제한
- [x] 단계 14: 챗봇 시스템 프롬프트에 Notes_v2 검색 연동 → ai_chat.gs buildUserContextForChat()에 searchNotesForRAG() 연동 완료

### QA & 배포
- [x] 단계 15: `src/frontend/judy_workspace.html` → `src/gas/judy_workspace.html` 동기화 완료
- [ ] 단계 16: 김감사 QA팀 검수 요청
- [x] 단계 17: 배포 헤더 작성 완료 (v2.7.0 + markdown_notes.gs v1.0.0)

## 산출물

| 산출물 | 파일 경로 | 상태 |
|--------|----------|------|
| 백엔드 API | `src/gas/markdown_notes.gs` (신규) | ⬜ 미완 |
| 프론트엔드 | `src/frontend/judy_workspace.html` 노트 탭 강화 | ⬜ 미완 |
| GAS 배포용 복사본 | `src/gas/judy_workspace.html` | ⬜ 미완 |

## 위임 사항

| 위임 대상 팀 | 전달 내용 | 상태 |
|-------------|----------|------|
| 🕵️ 김감사 QA팀 | MVP 구현 완료 후 기능+보안+UX QA 검수 요청 | ⬜ 대기 |

## 기술 제약 참고

- 스프레드시트 셀 최대 50,000자 → 하드 리밋 차단 필수
- marked.js (~40KB minified) + DOMPurify (~20KB minified) → HTML 내 인라인 삽입
- GAS 6분 타임아웃 → searchNotes()는 getValues() 1회 호출 + JS filter()로 처리
- XSS 방어: 저장 시 + 렌더링 시 이중 새니타이징

## 변경 이력

| 날짜 | 변경 내용 | 변경자 |
|------|----------|--------|
| 2026-03-08 | 최초 생성 | 자비스 PO |

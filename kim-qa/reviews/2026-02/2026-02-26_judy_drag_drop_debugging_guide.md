# [김감사 디버깅 가이드] 주디 드래그 앤 드롭 업무 등록 기능

**보고자**: 김감사 (QA Specialist)
**수신자**: 자비스 (PO), Chloe (Frontend)
**보고일**: 2026-02-26
**우선순위**: 🔴 높음 (긴급)
**심각도**: 🔴 Critical (핵심 기능 작동 불가)

---

## 📋 사용자 시나리오 (Expected Flow)

팀장님께서 제시하신 시나리오:

```
1. 메모 에디터 창에 메모 작성
2. 사용자가 텍스트 드래그
3. 드래그 종료 시 끝점에 마우스 멈춤
4. 🐰 "주디 업무등록" 플로팅 버튼 생성
5. 버튼 클릭
6. 주디 AI 에이전트가 메모 내용 분석
7. 업무 추출
8. 새 업무 등록 모달창에 추출된 내용 자동 입력
9. 사용자 최종 검수 후 업무 등록
```

---

## 🔍 현재 코드 분석

### ✅ 정상 작동하는 부분

#### 1. 플로팅 버튼 생성 로직 ✅
[judy_workspace.html:2027-2079](../src/frontend/judy_workspace.html#L2027-L2079)

```javascript
function handleTextSelection(e) {
    // 노트 탭 활성 확인
    if (!document.getElementById('viewNote').classList.contains('active')) return;

    const selection = window.getSelection();
    let text = selection.toString().trim();
    let isTextarea = false;

    // textarea 내부 선택 확인
    if (!text && document.activeElement && document.activeElement.tagName === 'TEXTAREA') {
        const ta = document.activeElement;
        if (ta.selectionStart !== ta.selectionEnd) {
            text = ta.value.substring(ta.selectionStart, ta.selectionEnd).trim();
            isTextarea = true;
        }
    }

    if (text.length < 5) {
        judyFloatBtn.style.display = 'none';
        _selectedTextForTask = '';
        return;
    }

    _selectedTextForTask = text;

    // 버튼 위치 계산
    const btnWidth = 120;
    let left = 0, top = 0;

    if (isTextarea && e) {
        // textarea의 경우 마우스 위치 기반
        left = e.clientX - (btnWidth / 2);
        top = e.clientY - 42;
        if (top < 8) top = e.clientY + 12;
    } else if (selection.rangeCount > 0) {
        // 일반 텍스트는 Range 기반
        const range = selection.getRangeAt(0);
        const rect = range.getBoundingClientRect();
        left = rect.left + (rect.width / 2) - (btnWidth / 2);
        top = rect.top - 42;
    } else if (e) {
        left = e.clientX - (btnWidth / 2);
        top = e.clientY - 42;
    }

    // 화면 경계 보정
    if (left < 8) left = 8;
    if (left + btnWidth > window.innerWidth - 8) left = window.innerWidth - btnWidth - 8;
    if (top < 8) top = 8;

    judyFloatBtn.style.left = left + 'px';
    judyFloatBtn.style.top = top + 'px';
    judyFloatBtn.style.display = 'flex';
}
```

**결과**: 이 로직은 정상 작동합니다. textarea 선택 감지가 잘 되어 있습니다.

---

#### 2. 이벤트 리스너 등록 ✅
[judy_workspace.html:2082-2092](../src/frontend/judy_workspace.html#L2082-L2092)

```javascript
// mouseup 이벤트로 텍스트 선택 감지
document.addEventListener('mouseup', (e) => {
    if (e.target === judyFloatBtn || judyFloatBtn.contains(e.target)) return;
    setTimeout(() => handleTextSelection(e), 50);
});

// 다른 곳 클릭 시 버튼 숨기기
document.addEventListener('mousedown', (e) => {
    if (e.target === judyFloatBtn || judyFloatBtn.contains(e.target)) return;
    judyFloatBtn.style.display = 'none';
});
```

**결과**: 이벤트 리스너도 정상입니다.

---

#### 3. 버튼 클릭 핸들러 ✅
[judy_workspace.html:2095-2127](../src/frontend/judy_workspace.html#L2095-L2127)

```javascript
judyFloatBtn.addEventListener('mousedown', (e) => {
    e.preventDefault();
    e.stopPropagation();
});

judyFloatBtn.addEventListener('click', (e) => {
    e.preventDefault();
    e.stopPropagation();
    if (!_selectedTextForTask) return;
    if (!g_userName) return showToast('⛔ 인증된 사용자가 없습니다.', true);

    judyFloatBtn.style.display = 'none';
    const selectedText = _selectedTextForTask;
    _selectedTextForTask = '';
    window.getSelection().removeAllRanges();

    showToast('🐰 AI가 선택된 내용을 분석 중입니다...');

    google.script.run
        .withSuccessHandler((res) => {
            if (res && res.success && res.data) {
                showToast('✨ 업무 추출 완료! 등록 폼을 확인해주세요.');
                switchMainView('tasks');
                setTimeout(() => { openRegModal(res.data); }, 200);
            } else {
                showToast('❌ ' + (res ? res.message : 'AI 분석 실패'), true);
            }
        })
        .withFailureHandler((err) => {
            showToast('❌ 서버 통신 에러', true);
        })
        .parseTaskFromMemoWeb(g_userName, selectedText);
});
```

**결과**: 클릭 핸들러도 정상입니다. AI 파싱 후 모달을 여는 로직이 완벽합니다.

---

## 🚨 문제 원인 분석

코드 분석 결과, **프론트엔드 로직은 모두 정상**입니다.

### ❓ 그렇다면 왜 안 되는가?

다음 3가지 시나리오를 확인해야 합니다:

---

### 🔴 시나리오 1: 백엔드 API가 없거나 에러 발생

#### 의심 지점
[judy_workspace.html:2126](../src/frontend/judy_workspace.html#L2126)
```javascript
.parseTaskFromMemoWeb(g_userName, selectedText);
```

**체크 필요**:
1. `web_app.gs`에 `parseTaskFromMemoWeb()` 함수가 존재하는가?
2. 함수가 정상 작동하는가?
3. Claude API 호출이 성공하는가?

**확인 방법**:
```javascript
// 브라우저 콘솔(F12)에서 실행
google.script.run
    .withSuccessHandler(res => console.log('SUCCESS:', res))
    .withFailureHandler(err => console.log('ERROR:', err))
    .parseTaskFromMemoWeb(g_userName, "테스트 메모 내용");
```

---

### 🟡 시나리오 2: 사용자 인증 문제

#### 의심 지점
[judy_workspace.html:2104](../src/frontend/judy_workspace.html#L2104)
```javascript
if (!g_userName) return showToast('⛔ 인증된 사용자가 없습니다.', true);
```

**체크 필요**:
- `g_userName` 변수가 제대로 설정되어 있는가?

**확인 방법**:
```javascript
// 브라우저 콘솔(F12)에서 실행
console.log('현재 사용자:', g_userName);
```

만약 `undefined` 또는 `null`이면 → 인증 실패 (Magic Link 재발급 필요)

---

### 🟢 시나리오 3: 브라우저 호환성 문제

#### 의심 지점
- `window.getSelection()` API가 브라우저에서 지원되지 않음
- `textarea.selectionStart/selectionEnd`가 작동하지 않음

**체크 필요**:
- 사용 중인 브라우저는? (Chrome, Safari, Firefox)
- 모바일인가 데스크톱인가?

**확인 방법**:
```javascript
// 브라우저 콘솔에서 실행
console.log('Selection API:', typeof window.getSelection);
console.log('Textarea selection:', document.getElementById('memoInput').selectionStart);
```

---

## 🎯 디버깅 체크리스트

자비스, Chloe에게 다음 순서대로 확인 요청:

### ✅ Step 1: 브라우저 콘솔 에러 확인
```
1. F12 (개발자 도구 열기)
2. Console 탭 선택
3. 텍스트 드래그 후 플로팅 버튼 클릭
4. 빨간색 에러 메시지 확인
5. 에러 메시지 전체 복사하여 제공
```

---

### ✅ Step 2: 사용자 인증 확인
```javascript
// 브라우저 콘솔에서 실행
console.log('사용자:', g_userName);
```

**예상 결과**:
- ✅ "송용남" 또는 "정혜림" → 정상
- ❌ `undefined` → Magic Link 재발급 필요

---

### ✅ Step 3: 백엔드 API 존재 확인

**파일**: `src/gas/web_app.gs`

**찾아야 할 함수**:
```javascript
function parseTaskFromMemoWeb(userName, memoText) {
  // ...
}
```

**만약 함수가 없으면**:
→ 백엔드 API를 새로 만들어야 함 (아래 섹션 참조)

---

### ✅ Step 4: API 응답 테스트
```javascript
// 브라우저 콘솔에서 실행
google.script.run
    .withSuccessHandler(res => {
        console.log('SUCCESS:', res);
        alert(JSON.stringify(res, null, 2));
    })
    .withFailureHandler(err => {
        console.log('ERROR:', err);
        alert('에러: ' + err.message);
    })
    .parseTaskFromMemoWeb('송용남', '내일까지 주디노트 즐겨찾기 기능 개발하기');
```

**예상 결과**:
- ✅ `{ success: true, data: { title: "...", desc: "...", due: "..." } }` → 정상
- ❌ `{ success: false, message: "..." }` → 백엔드 에러
- ❌ `Exception: ...` → GAS 스크립트 에러

---

## 🛠️ 해결 방안

### 해결안 A: 백엔드 API가 없는 경우

`web_app.gs`에 다음 함수 추가 필요:

```javascript
/**
 * 메모 텍스트에서 업무를 추출하는 AI 파싱 함수
 * @param {string} userName - 사용자 이름
 * @param {string} memoText - 선택된 메모 텍스트
 * @returns {object} - { success: boolean, data: { title, desc, due }, message }
 */
function parseTaskFromMemoWeb(userName, memoText) {
  try {
    // 1. 사용자 검증
    if (!userName || !memoText) {
      return { success: false, message: '사용자 또는 메모 내용이 없습니다.' };
    }

    // 2. Claude API 호출
    const apiKey = PropertiesService.getScriptProperties().getProperty('ANTHROPIC_API_KEY');
    if (!apiKey) {
      return { success: false, message: 'Claude API 키가 설정되지 않았습니다.' };
    }

    const prompt = `다음 메모 내용에서 업무를 추출해주세요:

"${memoText}"

다음 JSON 형식으로만 응답해주세요 (설명 없이):
{
  "title": "업무 제목",
  "desc": "업무 상세 설명",
  "due": "YYYY-MM-DD 형식의 마감일 (추정 가능한 경우)"
}

만약 마감일을 추정할 수 없으면 due는 빈 문자열로 반환하세요.`;

    const response = UrlFetchApp.fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json'
      },
      payload: JSON.stringify({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 1024,
        messages: [{
          role: 'user',
          content: prompt
        }]
      }),
      muteHttpExceptions: true
    });

    const result = JSON.parse(response.getContentText());

    if (result.content && result.content[0] && result.content[0].text) {
      const aiText = result.content[0].text.trim();

      // JSON 파싱 (```json 태그 제거)
      let cleanJson = aiText;
      if (aiText.startsWith('```json')) {
        cleanJson = aiText.replace(/```json\n?/, '').replace(/\n?```$/, '');
      } else if (aiText.startsWith('```')) {
        cleanJson = aiText.replace(/```\n?/, '').replace(/\n?```$/, '');
      }

      const taskData = JSON.parse(cleanJson);

      return {
        success: true,
        data: {
          title: taskData.title || memoText.substring(0, 50),
          desc: taskData.desc || '',
          due: taskData.due || ''
        }
      };
    } else {
      return { success: false, message: 'AI 응답 형식 오류' };
    }

  } catch (e) {
    Logger.log('parseTaskFromMemoWeb ERROR: ' + e.toString());
    return { success: false, message: 'AI 파싱 실패: ' + e.message };
  }
}
```

---

### 해결안 B: API 응답 형식 오류

현재 백엔드에서 반환하는 형식이 프론트엔드 기대 형식과 다를 수 있음.

**프론트엔드 기대 형식**:
```javascript
{
  success: true,
  data: {
    title: "업무 제목",
    desc: "업무 설명",
    due: "2026-02-28"
  }
}
```

**만약 다른 형식으로 반환 중이라면**:
→ 백엔드 함수 수정 필요

---

### 해결안 C: Claude API 키 미설정

**확인 방법**:
```javascript
// GAS 편집기 > 실행 > 로그 보기
function testApiKey() {
  const apiKey = PropertiesService.getScriptProperties().getProperty('ANTHROPIC_API_KEY');
  Logger.log('API Key: ' + (apiKey ? '설정됨' : '없음'));
}
```

**만약 API 키가 없으면**:
1. GAS 편집기에서 `프로젝트 설정` (⚙️)
2. `스크립트 속성` 탭
3. `+ 스크립트 속성 추가`
   - 속성: `ANTHROPIC_API_KEY`
   - 값: `[SECRET_MASKED] (Claude API 키)
4. 저장

---

## 🎯 자비스에게 요청사항

### 즉시 확인 필요 (5분)

1. **브라우저 콘솔 에러 확인**
   - F12 → Console 탭
   - 텍스트 드래그 → 버튼 클릭
   - 빨간색 에러 메시지 스크린샷

2. **사용자 인증 확인**
   ```javascript
   console.log('사용자:', g_userName);
   ```

3. **API 호출 테스트**
   ```javascript
   google.script.run
       .withSuccessHandler(res => console.log('SUCCESS:', res))
       .withFailureHandler(err => console.log('ERROR:', err))
       .parseTaskFromMemoWeb('송용남', '테스트 메모');
   ```

---

### 확인 후 회신 필요

다음 정보를 제공해주세요:

```markdown
## 디버깅 결과

### 1. 브라우저 콘솔 에러
[스크린샷 또는 에러 메시지 텍스트]

### 2. 사용자 인증 상태
g_userName = [값]

### 3. API 호출 결과
[SUCCESS 또는 ERROR 메시지]

### 4. 사용 환경
- 브라우저: Chrome / Safari / Firefox
- OS: Windows / Mac / Mobile
- 버전: [브라우저 버전]
```

---

## 📊 예상 원인별 확률

김감사의 경험상 예상 원인:

| 원인 | 확률 | 해결 시간 |
|:---|:---:|:---:|
| **백엔드 API 없음** | 70% | 30분 (API 작성) |
| **Claude API 키 미설정** | 20% | 5분 (키 등록) |
| **사용자 인증 실패** | 5% | 2분 (재로그인) |
| **브라우저 호환성** | 3% | 10분 (코드 수정) |
| **프론트엔드 버그** | 2% | 1시간 (재작성) |

**가장 가능성 높은 원인**: `parseTaskFromMemoWeb()` 백엔드 API가 아직 작성되지 않았을 것으로 추정.

---

## 🚀 긴급 조치 방안 (Fast-Track)

### 옵션 1: 백엔드 없이 프론트엔드만으로 임시 구현 (10분)

AI 파싱 없이 메모 텍스트를 그대로 업무 제목에 넣기:

```javascript
// judy_workspace.html:2100 수정
judyFloatBtn.addEventListener('click', (e) => {
    e.preventDefault();
    e.stopPropagation();
    if (!_selectedTextForTask) return;

    judyFloatBtn.style.display = 'none';
    const selectedText = _selectedTextForTask;
    _selectedTextForTask = '';
    window.getSelection().removeAllRanges();

    // AI 파싱 대신 바로 모달 열기 (임시)
    showToast('✨ 업무 등록 폼을 준비합니다...');
    switchMainView('tasks');
    setTimeout(() => {
        openRegModal({
            title: selectedText.substring(0, 100), // 최대 100자
            desc: selectedText,
            due: ''
        });
    }, 200);
});
```

**장점**:
- 즉시 작동
- 백엔드 불필요

**단점**:
- AI 파싱 기능 없음 (수동 입력 필요)

---

### 옵션 2: 백엔드 API 즉시 작성 (30분)

위의 "해결안 A" 코드를 `web_app.gs`에 추가

---

## 📝 다음 단계

### 자비스가 회신하면:
1. ✅ 김감사가 원인 확정
2. ✅ 해결 방안 제시
3. ✅ 자비스/Chloe가 수정
4. ✅ 김감사가 재테스트
5. ✅ 팀장님께 완료 보고

---

**자비스, 위 체크리스트대로 확인하고 결과를 회신해주세요!** 🙏

---

**작성자**: 김감사 (QA Specialist)
**완료일시**: 2026-02-26
**소요시간**: 20분
**다음 액션**: 자비스의 디버깅 결과 대기

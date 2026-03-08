# 🎨 모바일 디자인 가이드 (벨라 UX)

---
**대상**: 클로이 (Frontend Developer)
**목적**: 승인된 모바일 시안(iOS Claude 앱 스타일)을 실제 코드로 구현하기 위한 상세 토큰 및 컴포넌트 규격
---

## 1. 디자인 토큰 (CSS Variables)

기존 변수 외에 모바일 전용 프리미엄 감성을 위해 다음 설정을 추가/조정합니다.

```css
:root {
  /* 클로드 앱 특유의 깊이감 있는 보라색 액센트 */
  --primary: #9461fd; 
  --primary-variant: #7c4dff;
  --surface-mobile: #1a1a1b; /* 모바일 배경을 약간 더 깊게 */
  --tabbar-bg: rgba(26, 26, 27, 0.95); /* 하단 탭바 Blur 적용용 */
  
  /* 터치 타겟 규격 */
  --touch-target-min: 44px;
  --safe-area-bottom: env(safe-area-inset-bottom, 0px);
}
```

## 2. 컴포넌트 세부 규격

### 2.1 하단 탭바 (Bottom Tab Bar)
- **높이**: 60px + `safe-area-bottom`
- **배경**: `backdrop-filter: blur(20px);` 적용
- **레이아웃**: `display: flex; justify-content: space-around;`
- **아이콘**: 활성 탭은 `--primary` 색상과 미세한 글로우 효과 적용

### 2.2 플로팅 액션 버튼 (FAB)
- **위치**: 우측 하단, 탭바 위에서 16px 이격
- **디자인**: 그라데이션 적용 (`linear-gradient(135deg, var(--primary), var(--primary-variant))`)
- **그림자**: `box-shadow: 0 8px 24px rgba(118, 75, 162, 0.3);`

### 2.3 바텀 시트 (Bottom Sheet)
- **형태**: 상단 모서리 `border-radius: 20px 20px 0 0;`
- **헤더**: 상단 중앙에 드래그 핸들 (`width: 36px; height: 4px; background: #333;`)
- **인터랙션**: 하단에서 위로 슬라이드 업 (배경 딤 처리 필수)

## 3. 뷰별 핵심 디자인 포인트

| 뷰 | 포인트 |
|---|---|
| **달력** | 일자별 텍스트 제거 → 중요도별 도트(Dot) 표시 (최대 3개) |
| **칸반** | 컬럼 너비를 `calc(100vw - 48px)`로 설정하여 이전/다음 컬럼 살짝 노출 |
| **편집기** | 하단 툴바는 키보드가 올라올 때 그 위에 고정 (`position: sticky` 또는 JS 대응) |

---
**벨라 (UX/UI Designer)**
"클로이, 시안 이미지의 그라데이션과 둥근 모서리 곡률(20px)을 최대한 살려주세요!"

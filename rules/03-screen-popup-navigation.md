---
name: screen-popup-navigation
description: 화면 기능 분석에서 Popup, Modal, 새 창 및 화면 이동의 분석 범위를 정의한다.
---

# Popup 및 화면 이동 분석 규칙

## 1. 목적

화면 기능 분석 중 발견되는
Popup, Modal, 새 창, 새 Tab 및 화면 이동의
분석 범위를 정의한다.

현재 화면의 분석 범위를 벗어나
다른 화면을 자동으로 계속 분석하지 않는 것이
핵심 원칙이다.


## 2. Popup 유형 구분

Popup은 크게 다음과 같이 구분한다.

### 현재 화면 내부 Popup

- Modal
- Dialog
- Layer Popup
- 화면 내부 Overlay

현재 화면의 DOM 또는 Component 구조 안에서
표시되는 Popup을 의미한다.


### 외부 화면 Popup

- `window.open`
- 새 Browser Window
- 새 Tab
- 별도 URL로 열리는 Popup

현재 화면과 별도의 화면으로 실행되는 경우를 의미한다.


## 3. Modal / Layer Popup

Modal 또는 Layer Popup은
현재 화면의 일부로 취급한다.

Popup을 여는 기능은 하나의 Action으로 식별할 수 있다.

예:

등록 버튼
→ Click
→ 등록 Modal Open
→ ACT-003


## 4. Modal 내부 Action

Modal / Layer Popup 내부에서
실행 가능한 기능도 Action 식별 대상에 포함한다.

예:

등록 Modal
├─ 저장 버튼
│  └─ ACT-004
│
├─ 취소 버튼
│  └─ ACT-005
│
└─ 설비 선택 버튼
   └─ ACT-006

단순 Text, Label, Layout 등은
Action으로 등록하지 않는다.

Action 판단 기준은
화면 Action 식별 규칙을 따른다.


## 5. Modal 내부 분석 경계

Modal 내부 Action을 발견하더라도
현재 단계에서는 다음까지만 확인한다.

- 기능명
- UI Type
- Event
- Handler
- Source Path

Handler 내부의 Frontend 비즈니스 로직은
분석하지 않는다.


## 6. Window Popup / 새 창 / 새 Tab

`window.open` 또는 별도의 Browser Window / Tab으로
새 화면을 여는 경우
현재 화면 분석에서 새 화면 내부까지 분석하지 않는다.

현재 화면에서는 다음 정보까지만 확인한다.

- Popup 실행 Action
- Event
- Handler
- Source Path
- 대상 URL
- 전달 Parameter


## 7. Window Popup Parameter

새 창 또는 새 Tab에 전달되는 Parameter가
현재 화면에서 직접 확인되는 경우 기록할 수 있다.

예:

`/equipment/detail?id=100`

기록:

- Target URL: `/equipment/detail`
- Parameter: `id=100`

단, Runtime에서 확인한 실제 값과
Source Code에서 정의된 Parameter 구조를 구분한다.

확인되지 않은 Parameter는 추측하지 않는다.


## 8. Window Popup 분석 종료

새 창이 열렸더라도
해당 화면으로 이동하여 Action을 계속 수집하지 않는다.

예:

현재 화면

상세 버튼
→ `window.open`
→ `/equipment/detail?id=100`

현재 SCREEN 분석:

상세 버튼
→ 대상 URL / Parameter 기록
→ 종료

새로 열린 상세 화면 내부:

검색
수정
삭제
Tab
Grid

등은 현재 분석 대상이 아니다.

해당 화면을 분석하려면
별도의 Frontend URL을 입력하여
새로운 화면 기능 분석을 시작한다.


## 9. 일반 화면 이동

Router, Link 또는 기타 Navigation을 통해
다른 화면으로 이동하는 기능은
현재 화면의 Action으로 식별한다.

예:

설비 상세 Link
→ Click
→ `/equipment/detail/100`

현재 화면에서는 다음 정보까지만 기록한다.

- Action
- Event
- Handler
- Source Path
- Target URL
- 확인 가능한 Parameter

이동한 화면의 내부 기능은
자동으로 분석하지 않는다.


## 10. 동일 화면 내부 이동

Tab 변경이나 화면 내부 Section 전환처럼
동일 화면 안에서 UI 상태만 변경되는 경우에는
새로운 화면으로 간주하지 않는다.

해당 동작으로 새로운 Action이 표시되는 경우
현재 화면 분석 범위에서 확인할 수 있다.


## 11. 동적으로 표시되는 Action

Modal, Tab, Accordion 등
특정 동작 이후에만 표시되는 UI가 있을 수 있다.

안전하게 확인 가능한 경우
해당 UI를 표시한 뒤 Action을 식별할 수 있다.

단, 화면 상태나 데이터를 변경할 위험이 있는 동작은
자동으로 실행하지 않는다.


## 12. 상태 변경 위험 Action

다음과 같이 데이터 또는 업무 상태를
변경할 가능성이 있는 Action은
화면 기능 존재 여부를 확인하는 것까지만 수행한다.

예:

- 저장
- 등록
- 수정
- 삭제
- 승인
- 반려
- 확정
- 취소 처리
- 전송

사용자의 명시적인 허용 없이
실제 Action을 실행하지 않는다.


## 13. 확인되지 않은 정보

Popup 유형, Target URL, Parameter 등을
확인할 수 없는 경우 추측하지 않는다.

다음과 같이 기록한다.

`확인되지 않음`


## 14. 분석 경계

허용:

현재 화면
→ Modal Open
→ Modal 내부 Action 식별

현재 화면
→ Window Popup
→ Target URL / Parameter 확인

현재 화면
→ Navigation
→ Target URL / Parameter 확인


금지:

Window Popup
→ 새 화면 전체 분석

Navigation
→ 이동한 화면 전체 분석

Popup Action
→ Frontend Business Logic 분석

Popup Action
→ Backend 분석


## 15. 종료 원칙

Popup이나 Navigation을 발견했다는 이유로
현재 화면의 분석 범위를 확장하지 않는다.

현재 Frontend URL에 속하는
화면 기능 목록을 완성하면 종료한다.


## 16. 검증

Rule 구축 단계의 테스트에서는
마지막에 다음 문구를 출력한다.

`RULE_CHECK: SCREEN_POPUP_NAVIGATION_APPLIED`
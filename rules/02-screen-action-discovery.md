---
name: screen-action-discovery
description: 화면 기능 분석에서 Action으로 식별할 대상과 Action 목록 작성 기준을 정의한다.
---

# 화면 Action 식별 규칙

## 1. 목적

Frontend 화면에서 사용자가 실행할 수 있거나
화면 진입 시 자동으로 실행되는 기능을 식별한다.

단순한 화면 요소 목록이 아니라
실제 기능 실행의 시작점이 되는 Action을 찾는 것이 목적이다.


## 2. Action 판단 기준

다음 중 하나에 해당하고
실제 동작과 연결된 경우 Action 후보로 판단한다.

- 사용자 조작으로 실행되는 기능
- 화면 상태 변경으로 실행되는 기능
- 화면 진입 시 자동 실행되는 기능
- 다른 화면이나 Popup을 시작하는 기능


## 3. Click Action

다음과 같은 Click 동작을 확인한다.

- Button Click
- Link Click
- Icon Click
- Menu Click
- Grid Row Click
- Grid Cell Click

예:

검색 Button
→ Click
→ ACT 후보

상세 Link
→ Click
→ ACT 후보


## 4. 입력 및 변경 Action

값의 변경 자체가 기능을 실행하는 경우
Action 후보로 판단한다.

예:

- Checkbox Change
- Radio Change
- Select Change
- Input Change
- Date Change

단순히 값을 입력할 수 있다는 이유만으로
모든 Input을 Action으로 등록하지 않는다.

값 변경으로 실제 기능이 실행되는 경우에만
Action으로 판단한다.


## 5. Keyboard / Submit Action

다음 동작으로 기능이 실행되는 경우
Action 후보로 판단한다.

- Enter
- KeyDown
- KeyUp
- Keyboard Shortcut
- Form Submit


## 6. 화면 상태 변경 Action

다음 동작이 실제 기능과 연결된 경우
Action 후보로 판단한다.

- Tab Change
- Pagination
- Sort
- Filter
- Grid Selection
- Accordion Open


## 7. 화면 초기 실행 Action

사용자 조작 없이
화면 진입 또는 Component 초기화 시
업무 기능이 자동 실행되는 경우 Action으로 판단한다.

예:

- 초기 조회
- 초기 데이터 조회
- 코드 목록 조회
- 자동 검색
- 화면 초기화 처리

React의 `useEffect`,
Vue의 `mounted`, `onMounted` 등
Framework Lifecycle이 존재한다는 이유만으로
Action을 생성하지 않는다.

실제 기능 실행과 연결된 경우에만
Action으로 판단한다.


## 8. Popup / 화면 이동 Action

다음 동작도 Action 후보로 판단한다.

- Modal Open
- Dialog Open
- Layer Popup Open
- Window Open
- 다른 화면으로 이동
- 새 Tab 열기

Popup 또는 화면 이동의 상세 분석 범위는
별도의 Popup / Navigation 규칙을 따른다.


## 9. Action에서 제외할 대상

다음 항목은 그 자체만으로
Action으로 등록하지 않는다.

- 단순 Text
- Label
- Layout Element
- Decoration Element
- 동작이 없는 Container
- Event가 없는 UI Element
- 단순 Rendering Function
- 업무 동작이 없는 Framework Lifecycle
- 단순 데이터 표시 영역

화면에 존재한다는 이유만으로
모든 DOM Element를 Action으로 등록하지 않는다.


## 10. 동일 기능과 중복 Event

서로 다른 Event가
동일한 기능을 실행하는 경우가 있다.

예:

검색 Button Click
→ handleSearch

Enter
→ handleSearch

이 경우 두 Event가 동일한 업무 기능을 실행하는지 확인한다.

동일한 기능이라면
무조건 서로 다른 Action으로 분리하지 않는다.

하나의 Action에
여러 Trigger가 존재한다는 것을 표현할 수 있다.

예:

ACT-002 검색

Trigger:
- 검색 Button Click
- 검색조건 Enter


## 11. 별도 Action 판단

화면 표시가 비슷하더라도
실제로 서로 다른 Handler 또는 기능을 실행하면
별도의 Action으로 식별한다.

예:

Grid Row Click
→ 상세 조회

Grid Checkbox Click
→ 선택 상태 변경

두 동작은 별도의 Action이 될 수 있다.


## 12. Action 이름

Action 이름은 가능한 경우
사용자가 화면에서 이해할 수 있는 기능명을 사용한다.

권장:

- 검색
- 초기화
- 상세 조회
- 신규 등록
- 저장
- 삭제
- 엑셀 다운로드
- 페이지 이동
- 화면 초기 조회

Handler 이름만으로
Action 이름을 만들지 않는다.

예:

비권장:

`handleClick`

권장:

`검색`

Handler는 별도 항목에 기록한다.


## 13. Action ID

발견한 Action에는
고유한 ID를 순서대로 부여한다.

형식:

`ACT-001`

`ACT-002`

`ACT-003`

하나의 화면 기능 목록에서
동일한 ID를 중복 사용하지 않는다.


## 14. Action 상태

화면 기능 분석에서 발견한 Action은
아직 Frontend 상세 분석을 수행하지 않은 상태이다.

기본 상태:

`미분석`

Frontend 상세 분석 여부와
화면 기능 발견 여부를 구분한다.


## 15. 확인되지 않은 정보

화면 문구나 Component 이름만 보고
업무 기능을 임의로 확정하지 않는다.

Action 여부를 확인할 수 없는 경우
추측하여 목록에 확정적으로 등록하지 않는다.

필요한 경우 다음과 같이 기록한다.

`확인 필요`


## 16. 화면 기능 목록

발견한 Action은 최소한 다음 정보를 포함한다.

| Action ID | 기능명 | UI 유형 | Event | Handler | Source Path | 상태 |
|---|---|---|---|---|---|---|

Handler 또는 Source Path가 확인되지 않은 경우:

`확인되지 않음`

으로 기록한다.


## 17. 종료 조건

화면에서 확인된 Action을
화면 기능 목록으로 정리하면 종료한다.

Action 내부 Frontend 비즈니스 로직은
이 단계에서 분석하지 않는다.


## 18. 검증

Rule 구축 단계의 테스트에서는
마지막에 다음 문구를 출력한다.

`RULE_CHECK: SCREEN_ACTION_DISCOVERY_APPLIED`
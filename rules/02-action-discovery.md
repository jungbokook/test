---
description: Screen Action Discovery에서 Business Action을 식별하고 Action Inventory를 구성하는 기준을 정의한다.
---

# Screen Action Discovery Rule

## Purpose

Frontend 화면에서 분석 대상이 될 수 있는
Business Action을 일관된 기준으로 식별한다.

단순 UI Element 목록을 만드는 것이 아니라
사용자 동작 또는 자동 실행으로 인해
Business Flow가 시작될 수 있는 지점을 찾는 것이 목적이다.


## Action Definition

Action은 다음 중 하나에 해당하는 동작이다.

1. 사용자의 조작으로 실행되는 동작
2. 화면 상태 변경으로 실행되는 동작
3. 페이지 진입 또는 초기화 과정에서 자동 실행되는 동작
4. 다른 화면, Popup, Modal 등을 시작하는 동작


## User Actions

다음 Event는 Action 후보로 확인한다.

### Click

예:

- Button Click
- Link Click
- Icon Click
- Grid Row Click
- Grid Cell Click
- Menu Click

### Change

예:

- Checkbox Change
- Radio Change
- Select Change
- Input Change
- Date Change

### Keyboard

예:

- Enter
- KeyDown
- KeyUp
- Shortcut

### Submit

예:

- Form Submit
- Search Submit


## Screen State Actions

화면 상태 변경으로 Business Flow가 시작되는 경우
Action 후보로 포함한다.

예:

- Tab Change
- Accordion Open
- Pagination
- Sort
- Filter Change
- Grid Selection Change


## Initialization Actions

사용자 Click이 없어도
페이지 진입 또는 Component 초기화 과정에서
Business Logic이 자동 실행되는 경우 Action으로 식별한다.

예:

- Page Load
- Component Mount
- useEffect
- mounted
- onMounted
- Initial Data Load
- Automatic Search
- Initial Code Load

단순 Framework Lifecycle 존재만으로
Action을 생성하지 않는다.

실제 Business 동작이 연결된 경우에만
Action 후보로 포함한다.


## Popup Actions

다음 동작도 Action 후보로 식별한다.

- Modal Open
- Dialog Open
- Layer Popup Open
- Window Open

Window Open의 경우
새로운 Window를 여는 동작 자체를 Action으로 식별한다.

Modal / Layer Popup의 경우
현재 화면에서 발생하는 Business Action으로 식별한다.


## Exclusion

다음 항목은 그 자체만으로 Action으로 등록하지 않는다.

- 단순 Text
- Label
- Layout Element
- Decoration Element
- 단순 Container
- Event가 없는 UI Element
- Business 동작과 관계없는 Framework Lifecycle
- 단순 Rendering Function

화면에 존재한다는 이유만으로
모든 DOM Element를 Action으로 등록하지 않는다.


## Duplicate Actions

동일한 Business Action이 여러 UI Element에서 실행되는 경우
무조건 서로 다른 Action으로 만들지 않는다.

예:

검색 Button
→ handleSearch

Enter
→ handleSearch

두 Event가 실제로 동일한 Business Flow를 실행하는 경우
관계를 확인하여 Action Inventory에서 명확하게 표현한다.

반대로 UI는 비슷하더라도
서로 다른 Handler 또는 Business Flow를 실행하면
별도 Action으로 식별한다.


## Action Naming

Action Name은 가능한 경우
사용자가 화면에서 이해할 수 있는 기능명을 사용한다.

예:

- 검색
- 저장
- 삭제
- 상세 조회
- 엑셀 다운로드
- 신규 등록
- 페이지 초기 조회

Handler 이름만을 Action Name으로 사용하지 않는다.

예:

잘못된 표현:

`handleClick`

가능한 표현:

`검색`

Handler가 확인된 경우
Handler 항목에 별도로 기록한다.


## Action ID

발견된 Action에는 순서대로
고유한 Action ID를 부여한다.

형식:

ACT-001
ACT-002
ACT-003
...

한 번 생성된 Action Inventory 안에서는
동일한 Action ID를 중복 사용하지 않는다.


## Action Inventory

발견된 Action은 다음 기본 구조로 정리한다.

| Action ID | Action Name | UI Type | Event | Handler | Source Path | Status |
|---|---|---|---|---|---|---|

### Action ID

Action의 고유 식별자.

### Action Name

사용자가 이해할 수 있는 기능명.

### UI Type

예:

- Button
- Link
- Grid
- Checkbox
- Select
- Input
- Tab
- Modal
- Window
- Page

### Event

예:

- Click
- Change
- Submit
- Enter
- Init
- Mount

### Handler

실제로 확인된 Frontend Handler.

확인되지 않은 경우:

`확인되지 않음`

### Source Path

Handler 또는 Action과 관련된
Frontend Source Path.

확인되지 않은 경우:

`확인되지 않음`

### Status

Action Discovery 단계에서 발견된 Action은 기본적으로:

`NOT ANALYZED`

로 표시한다.


## No Guessing

화면 문구, 함수명 또는 Component 이름만 보고
Business 의미를 확정하지 않는다.

확인되지 않은 정보는 추측해서 채우지 않는다.

필요한 경우:

`확인되지 않음`

으로 표시한다.


## STOP CONDITION

발견된 Business Action을
Action Inventory로 정리하면 Discovery를 종료한다.

Action 자체의 상세 Business Logic은 분석하지 않는다.

다음 분석 대상은 사용자가 Action ID를 선택하여 결정한다.


## Verification

Rule 구축 단계에서는 결과 마지막에 다음을 출력한다.

`RULE_CHECK: ACTION_DISCOVERY_RULE_APPLIED`
---
name: screen-action-discovery
description: Frontend URL의 실제 화면과 Runtime 동작을 확인하여 Business Action을 발견하고 Action Inventory를 생성한다.
argument-hint: "<frontend-url>"
user-invocable: true
disable-model-invocation: false
---

# Screen Action Discovery

## Role

Frontend URL을 시작점으로
실제 화면에서 Business Flow의 시작점이 되는 Action을 발견한다.

이 Skill의 목적은 기능 전체를 분석하는 것이 아니다.

현재 화면의 Business Action을 식별하고
Action Inventory를 생성하는 것까지만 수행한다.


## Input

사용자로부터 Frontend URL을 입력받는다.

예:

`https://localhost:3000/equipment`

또는

`/equipment`

URL만으로 실제 화면에 접근할 수 없는 경우
임의로 URL을 추측하거나 변경하지 않는다.


## Preconditions

분석을 시작하기 전에
현재 프로젝트의 다음 Rule을 준수한다.

- Action Scope Rule
- Action Discovery Rule
- MCP Usage Rule
- Evidence Rule
- Popup Rule

Rule과 이 Skill의 절차가 충돌하는 경우
분석 범위를 더 좁게 제한하는 규칙을 우선한다.


# Analysis Procedure

## STEP 1. Open Target Screen

Chrome DevTools MCP를 사용하여
사용자가 지정한 Frontend URL에 접근한다.

실제 Browser Runtime을 기준으로 화면을 확인한다.

확인 대상:

- 현재 URL
- 페이지가 정상적으로 표시되는지
- 주요 화면 영역
- 사용자와 상호작용 가능한 UI

화면에 접근할 수 없는 경우
Source Code만으로 화면 전체를 추정하지 않는다.

접근 실패 사실을 기록하고 STOP 한다.


## STEP 2. Discover Runtime Action Candidates

Chrome DevTools MCP를 사용하여
현재 화면에서 Business Action 후보를 찾는다.

주요 후보:

### Click

- Button
- Link
- Icon
- Menu
- Grid Row
- Grid Cell

### Change

- Checkbox
- Radio
- Select
- Input
- Date

### Keyboard / Submit

- Enter
- Key Event
- Form Submit

### Screen State

- Tab
- Pagination
- Sort
- Filter
- Grid Selection

### Popup

- Modal
- Dialog
- Layer Popup
- Window Popup

단순히 DOM에 존재하는 모든 Element를
Action으로 등록하지 않는다.

Business 동작과 연결될 가능성이 있는
Actionable Element를 중심으로 확인한다.


## STEP 3. Discover Initialization Actions

사용자 조작 없이
페이지 진입 과정에서 자동으로 발생하는
Business Action이 있는지 확인한다.

예:

- Initial Data Load
- Automatic Search
- Initial Code Load
- Page Initialization

Framework Lifecycle 자체를
Action으로 판단하지 않는다.

실제 Business 동작이 연결되어 있는 경우에만
Action 후보로 판단한다.


## STEP 4. Verify Action Behavior

발견한 Action 후보에 대해
현재 Runtime에서 확인 가능한 범위의 동작을 확인한다.

확인 가능한 항목:

- UI Element
- Event Type
- 화면 변화
- Popup 발생 여부
- 현재 화면에서 확인되는 Runtime 동작

Action Discovery를 위해 필요한 경우에만
해당 Action을 실행한다.

데이터 변경 가능성이 있거나
업무상 영향이 명확하지 않은 Action은
임의로 실행하지 않는다.

예:

- 저장
- 삭제
- 승인
- 취소
- 확정
- 전송
- 등록

이러한 Action은
실제 실행 없이 Action 후보로 기록할 수 있다.


## STEP 5. Map Handler and Source

Action의 Handler 또는 Source Path 확인이 필요한 경우
Code Index MCP를 사용한다.

Code Index MCP의 목적은
관련 Source 범위를 좁히는 것이다.

확인 대상:

- Frontend Component
- Handler
- Function
- Symbol
- Reference
- Source Path

예:

검색 Button
→ Click
→ handleSearch()
→ src/pages/equipment/EquipmentSearch.tsx

Code Index에서 관계를 발견했다는 이유만으로
Business Logic을 확정하지 않는다.

Action 식별에 필요한 Source까지만 확인한다.


## STEP 6. Apply Source Boundary

Frontend Source 탐색은
Action을 식별하는 데 필요한 지점에서 종료한다.

허용 예:

Button
→ onClick
→ handleSearch()

필요한 경우:

Button
→ onClick
→ handleSearch()
→ Source Path 확인

이 단계에서 다음으로 확장하지 않는다.

handleSearch()
→ API Client
→ Controller
→ Service
→ Mapper
→ SQL

API 호출 코드가 우연히 발견되더라도
API 상세 분석을 시작하지 않는다.


## STEP 7. Handle Popup

Popup을 발견하면 Popup Rule을 적용한다.


### Window Popup

다음까지만 확인한다.

- Trigger
- Event
- Handler
- Target URL
- Parameter

새 Window 또는 Tab에서 열린 화면의
내부 Action을 자동으로 분석하지 않는다.


### Modal / Layer Popup

현재 화면의 일부로 판단한다.

Modal / Layer 내부에
별도의 Business Action이 존재하면
Action 후보로 포함할 수 있다.

예:

등록 Button
→ Modal Open

Modal 내부:

저장 Button
→ 별도 Action 후보


## STEP 8. Remove Invalid Candidates

발견한 후보 중
Action Discovery Rule에 해당하지 않는 항목을 제거한다.

제외 예:

- Text
- Label
- Layout
- Decoration
- Event 없는 Container
- 단순 Rendering Function
- Business 동작 없는 Lifecycle

동일한 Business Flow를 실행하는
중복 Event도 확인한다.

예:

검색 Button Click
→ handleSearch()

Enter
→ handleSearch()

두 Event가 동일한 Business Action을 실행하는 경우
Action Inventory에서 관계를 명확하게 표현한다.


## STEP 9. Assign Action IDs

확정된 Action에 순서대로 ID를 부여한다.

형식:

ACT-001
ACT-002
ACT-003
...

동일한 Inventory에서
Action ID를 중복 사용하지 않는다.


## STEP 10. Create Action Inventory

다음 형식으로 결과를 작성한다.

| Action ID | Action Name | UI Type | Event | Handler | Source Path | Status |
|---|---|---|---|---|---|---|

Action Name은 가능한 경우
사용자가 화면에서 이해할 수 있는 기능명을 사용한다.

Handler가 확인되지 않은 경우:

`확인되지 않음`

Source Path가 확인되지 않은 경우:

`확인되지 않음`

모든 발견 Action의 기본 Status:

`NOT ANALYZED`


# MCP Usage

## Chrome DevTools MCP

이 Skill의 Primary Runtime Tool이다.

다음 확인에 우선 사용한다.

- 실제 화면 접근
- DOM / UI
- Runtime Event
- Popup
- 화면 상태


## Code Index MCP

필요한 경우에만 사용한다.

다음 확인에 사용한다.

- Handler
- Frontend Symbol
- Reference
- Source Path

전체 Repository를 광범위하게 탐색하지 않는다.


## Oracle MCP

이 Skill에서는 사용하지 않는다.

Action Discovery는
Database 분석 단계가 아니다.


# Evidence

Action Inventory에는
실제로 확인한 정보만 기록한다.

Runtime에서 확인한 정보와
Source에서 확인한 정보를 혼동하지 않는다.

확인하지 못한 Handler 또는 Source Path를
이름이나 화면 문구만 보고 추측하지 않는다.

확인되지 않은 경우:

`확인되지 않음`

으로 기록한다.


# Prohibited Analysis

이 Skill에서는 다음 분석을 수행하지 않는다.

- API 상세 분석
- Request 상세 분석
- Response 상세 분석
- Controller 분석
- Backend Service 분석
- Backend Business Logic 분석
- Mapper 분석
- MyBatis XML 분석
- SQL 분석
- Oracle 분석
- SAP 상세 분석
- External System 상세 분석

Runtime에서 이러한 정보가 발견되더라도
현재 결과에 필요한 범위를 넘어 추적하지 않는다.


# STOP Condition

Action Inventory 생성이 완료되면
즉시 Screen Action Discovery를 종료한다.

다음 단계로 자동 진행하지 않는다.

특히 다음을 자동 실행하지 않는다.

Action
→ API Discovery

API
→ Backend Analysis

Backend
→ Mapper / SQL

다음 분석 대상은
사용자가 Action ID를 선택하여 결정한다.

예:

`ACT-001 분석`


# Output Format

## Screen Information

- Target URL:
- Current URL:
- Screen Status:


## Action Inventory

| Action ID | Action Name | UI Type | Event | Handler | Source Path | Status |
|---|---|---|---|---|---|---|


## Discovery Notes

필요한 경우 다음만 간단히 기록한다.

- Runtime에서 확인하지 못한 항목
- Source Mapping이 되지 않은 항목
- 실행하지 않은 위험 Action
- 별도 분석이 필요한 Window Popup


## Next Step

사용자가 분석할 Action ID를 선택하도록 안내한다.

Action의 상세 분석을 자동으로 시작하지 않는다.


# Verification

Skill 구축 단계에서는
결과 마지막에 다음 문구를 출력한다.

`SKILL_CHECK: SCREEN_ACTION_DISCOVERY_USED`
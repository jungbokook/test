---
name: popup
description: Popup 유형을 구분하고 Popup 유형별 분석 범위를 제한한다.
---

# Popup Analysis Rule

## Purpose

화면 분석 과정에서 발견되는 Popup을 유형별로 구분하고
현재 화면과 Popup 사이의 분석 경계를 명확하게 정의한다.

Popup이라는 이유만으로
새로운 화면 전체를 자동으로 분석하지 않는다.


## Popup Types

Popup은 다음 두 유형으로 구분한다.

### Window Popup

별도의 Browser Window 또는 Tab으로 열리는 Popup.

예:

- window.open()
- 새 Browser Window
- 새 Browser Tab

### Modal / Layer Popup

현재 화면 내부에서 표시되는 Popup.

예:

- Modal
- Dialog
- Layer Popup
- Component 기반 Popup


## Window Popup Rule

Window Popup은
현재 화면에서 Popup을 여는 동작까지만 분석한다.

확인 대상:

- Popup Trigger
- Event
- Handler
- Target URL
- 전달 Parameter

가능한 경우 실제 Runtime 동작을 통해
Target URL과 Parameter를 확인한다.


## Window Popup Boundary

Window Popup으로 열린
새로운 화면의 내부 기능은
현재 Action 분석 범위에 자동으로 포함하지 않는다.

예:

현재 화면

검색 결과
→ 상세 버튼
→ window.open()
→ /equipment/detail?id=100

현재 분석에서 확인 가능:

- 상세 버튼
- Click Event
- Popup Handler
- `/equipment/detail`
- `id=100`

현재 분석에서 자동으로 진행하지 않음:

- 상세 화면 내부 Action
- 상세 화면 API
- 상세 화면 Backend
- 상세 화면 Mapper
- 상세 화면 SQL
- 상세 화면 Oracle 분석


## Modal / Layer Popup Rule

Modal 또는 Layer Popup은
현재 화면의 Business Flow 일부로 취급할 수 있다.

현재 선택된 분석 범위와 직접 연결된 경우
Modal / Layer 내부 동작을 분석 대상에 포함할 수 있다.

예:

검색 화면
→ 등록 버튼
→ 등록 Modal
→ 저장 버튼

이 경우:

등록 버튼
→ Modal Open

뿐만 아니라

Modal 내부의 저장 버튼도
별도의 Action 후보가 될 수 있다.


## Modal Action Discovery

Screen Action Discovery 단계에서
Modal / Layer Popup이 실제로 열리고
내부에 Business Action이 존재하는 경우
해당 Action을 Action Inventory 후보로 식별할 수 있다.

예:

ACT-005
등록 Modal 열기

ACT-006
등록 Modal 저장

단순 표시용 요소는
Action으로 생성하지 않는다.


## Runtime Verification

Popup 유형이 명확하지 않은 경우
가능하면 Runtime 동작을 확인한다.

확인 예:

- 새 Window / Tab 생성 여부
- 현재 DOM 내부 Modal 생성 여부
- Target URL
- 전달 Parameter

화면 이름이나 Component 이름만 보고
Popup 유형을 확정하지 않는다.


## No Automatic Expansion

Popup에서 새로운 URL,
API 또는 다른 Business Flow를 발견하더라도
현재 단계보다 이후 분석으로 자동 확장하지 않는다.

현재 단계의 STOP 정책을 그대로 적용한다.


## Evidence

Popup 분석 결과는
확인 가능한 Evidence를 기반으로 작성한다.

가능한 Evidence:

- Runtime Popup 동작
- Event
- Handler
- Source Path
- window.open 호출
- Target URL
- 전달 Parameter
- Modal Component

확인되지 않은 정보는 추측하지 않는다.


## STOP CONDITION

Popup 관련 Action을 현재 단계의 결과에 반영한 후
현재 분석 단계의 STOP 조건을 따른다.

Window Popup의 새로운 Target 화면은
사용자가 별도 분석 대상으로 선택하기 전까지
분석하지 않는다.


## Verification

Rule 구축 단계에서는
Popup 규칙을 설명하거나 검증하는 요청에 대해
마지막에 다음 문구를 출력한다.

`RULE_CHECK: POPUP_RULE_APPLIED`
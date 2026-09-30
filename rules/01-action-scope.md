---
description: Screen Action Discovery 단계의 분석 범위를 제한한다.
---

# Screen Action Discovery Scope

## Purpose

Frontend URL의 Screen Action Discovery에서는
화면에 존재하는 Business Action을 식별하는 것까지만 수행한다.

기능 전체 분석으로 확장하지 않는다.


## MUST

다음과 같은 Business Action을 식별할 수 있다.

- Button / Link Click
- Grid Row / Cell Click
- Checkbox / Select / Input Change
- Keyboard / Enter Event
- Form Submit
- Tab Change
- Modal / Layer Popup
- Window Popup
- Page Initialization
- Automatic Initial Load

발견한 Action에는 고유 ID를 부여한다.

예:

ACT-001
ACT-002
ACT-003

아직 상세 분석하지 않은 Action은

`NOT ANALYZED`

상태로 표시한다.


## Allowed Scope

Action 식별에 필요한 범위까지만 확인한다.

허용:

- 실제 화면
- Actionable UI Element
- Event
- Handler
- 관련 Frontend Source Path
- Page Initialization
- Popup Trigger

확인되지 않은 Handler나 Source Path는 추측하지 않는다.


## MUST NOT

이 단계에서는 다음 분석으로 확장하지 않는다.

- API 상세 분석
- Request / Response 상세 분석
- Controller
- Backend Service
- Backend Business Logic
- Mapper
- MyBatis XML
- SQL
- Oracle
- SAP / External System 상세 분석

API가 존재한다는 사실을 발견하더라도
상세 분석하지 않는다.


## Source Boundary

허용:

Button
→ onClick
→ handleSearch

금지:

Button
→ handleSearch
→ API Client
→ Controller
→ Service
→ Mapper
→ SQL

Frontend Source 탐색도
Action 식별에 필요한 지점에서 종료한다.


## STOP CONDITION

발견한 Action을 Action Inventory로 정리한다.

기본 항목:

- Action ID
- Action Name
- UI Type
- Event
- Handler
- Source Path
- Status

Action Inventory 생성 후 반드시 STOP 한다.

다음 분석 대상은 사용자가 선택한다.


## Verification

Rule 구축 단계에서는 결과 마지막에 다음을 출력한다.

`RULE_CHECK: ACTION_SCOPE_RULE_APPLIED`
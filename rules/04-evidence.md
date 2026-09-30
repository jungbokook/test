---
description: 분석 결과의 Evidence 기준과 추측 방지 원칙을 정의한다.
---

# Evidence Rule

## Purpose

모든 주요 분석 결과는
실제로 확인 가능한 Evidence를 기반으로 작성한다.

Source 이름, 함수명, 화면 문구 또는 추정만으로
Business Logic을 사실처럼 작성하지 않는다.


## Evidence Types

분석에서 사용하는 Evidence를 다음과 같이 구분한다.


### Runtime Evidence

실제 실행 환경에서 확인된 정보.

예:

- 실제 화면
- UI Element
- Runtime Event
- Network Request
- Request Parameter
- Request Payload
- Response
- HTTP Status
- Console
- Popup 동작

주요 확인 수단:

`Chrome DevTools MCP`


### Source Evidence

실제 Source Code에서 확인된 정보.

예:

- Source Path
- Component
- Class
- Method
- Function
- Event Handler
- Controller
- Service
- Mapper Interface
- Mapper XML
- SQL

Code Index MCP로 Source를 발견한 경우에도
Business 의미가 필요한 내용은
가능한 경우 실제 Source를 확인한다.


### Database Evidence

실제 Database Metadata에서 확인된 정보.

예:

- Schema
- Table
- View
- Column
- Data Type
- Nullable
- Primary Key
- Foreign Key

주요 확인 수단:

`Oracle MCP`


### Discovery Evidence

분석 대상을 찾거나
Source 관계를 좁히기 위해 확인된 정보.

예:

- Symbol
- Reference
- Caller
- Callee
- Call Relationship

주요 확인 수단:

`Code Index MCP`

Discovery Evidence만으로
Business Logic을 확정하지 않는다.


## MUST

중요한 분석 결과는
가능한 경우 해당 결과를 확인할 수 있는 근거를 함께 유지한다.

예:

Frontend Logic

`src/.../EquipmentSearch.tsx`
→ `handleSearch()`

Backend Logic

`src/.../EquipmentController.java`
→ `searchEquipment()`

MyBatis

`src/.../EquipmentMapper.xml`
→ `selectEquipment`

Database

`EQUIPMENT`
→ Oracle Metadata 확인


## Runtime and Source Separation

Runtime에서 관찰한 사실과
Source에서 정의된 사실을 구분한다.

예:

Runtime에서 다음 Request가 확인되었다.

`plant = 1000`

이것은 다음 사실만 의미한다.

`현재 관찰한 실행에서 plant=1000이 전달되었다.`

이 결과만으로 다음과 같이 판단하지 않는다.

`plant는 항상 1000이다.`

가능한 값, 필수 여부, Validation 등은
Source Evidence를 추가 확인해야 한다.


## No Unsupported Inference

확인되지 않은 내용을
사실처럼 작성하지 않는다.

예:

Method 이름:

`deleteEquipment()`

Method 이름만 보고

`DB의 Equipment 데이터를 DELETE한다.`

라고 판단하지 않는다.

실제 구현을 확인해야 한다.

실제 구현은 다음과 다를 수 있다.

`deleteEquipment()`
→ Service
→ Mapper
→ UPDATE STATUS = 'D'

따라서 이름은 Discovery에 사용할 수 있지만
Business 의미를 확정하는 Evidence가 될 수 없다.


## Unknown Information

Evidence를 확보하지 못한 정보는
임의로 채우지 않는다.

필요한 경우 다음과 같이 표시한다.

- 확인되지 않음
- Runtime 확인 필요
- Source 확인 필요
- Database Metadata 확인 필요
- 추가 분석 필요


## Conflicting Evidence

서로 다른 Evidence가 충돌하는 경우
임의로 하나를 선택하여 사실로 확정하지 않는다.

예:

Runtime Request

`plant = 1000`

Source Default

`plant = 2000`

이 경우 두 Evidence를 구분하여 기록하고
차이가 존재함을 명시한다.

필요한 경우 추가 확인 대상으로 남긴다.


## Evidence Scope

Evidence를 확보하기 위해
현재 분석 단계의 Scope를 무시하지 않는다.

예:

Screen Action Discovery 중
Backend Evidence가 필요해 보인다는 이유로

Controller
→ Service
→ Mapper
→ Oracle

까지 자동으로 분석하지 않는다.

현재 단계에서 허용되는 Evidence만 확보하고
추가 Evidence가 필요한 경우
후속 분석 대상으로 남긴다.


## Evidence Recording

최종 분석 결과에서 필요한 경우
다음과 같은 식별 정보를 Evidence로 남긴다.

- Runtime 위치 또는 동작
- Source Path
- Class
- Method
- Function
- API
- Mapper
- Mapper XML
- Statement ID
- SQL
- Oracle Object

Evidence가 없는 항목을
있는 것처럼 생성하지 않는다.


## Verification

Rule 구축 단계에서는
Evidence 규칙을 설명하거나 검증하는 요청에 대해
마지막에 다음 문구를 출력한다.

`RULE_CHECK: EVIDENCE_RULE_APPLIED`
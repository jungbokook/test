# Code Analysis Project Guide

## 1. Project Purpose

이 프로젝트에서 Claude Code의 목적은
Frontend URL을 시작점으로 실제 화면 동작과 소스코드를 추적하여
개발자가 참고할 수 있는 정확한 기능 분석 문서를 생성하는 것이다.

분석은 한 번에 전체 시스템을 탐색하지 않는다.

반드시 아래와 같이 단계적으로 수행한다.

URL
→ Screen Action Discovery
→ Selected Action API Discovery
→ Selected API Specification
→ Frontend Business Analysis
→ Backend Business Analysis
→ MyBatis / Oracle Analysis
→ Final Business Flow

각 단계는 독립적으로 수행하며,
현재 단계가 완료되면 반드시 STOP 한다.

사용자가 명시적으로 다음 분석 대상을 선택하기 전까지
다음 단계로 자동 진행하지 않는다.


# 2. Core Analysis Principle

## 2.1 Progressive Analysis

전체 코드베이스를 한 번에 분석하지 않는다.

분석 범위는 사용자가 현재 선택한 대상에 한정한다.

예:

URL 분석 요청

URL
└─ Action Discovery
   ├─ ACT-001
   ├─ ACT-002
   └─ ACT-003

여기서 STOP 한다.

ACT-001 분석 요청

ACT-001
├─ API-001
└─ API-002

여기서 다시 STOP 한다.

API-001 분석 요청

API-001
├─ Request
├─ Response
├─ Controller
└─ API Specification

필요한 단계까지만 분석하고 STOP 한다.

사용자의 명시적인 요청 없이
Backend, MyBatis, SQL, Oracle까지 자동으로 확장하지 않는다.


# 3. Analysis Phases

분석 단계는 다음 순서를 기본으로 한다.

## PHASE 1 — Screen Action Discovery

입력:

Frontend URL

목적:

실제 화면에서 사용자가 실행할 수 있거나
자동으로 발생하는 Business Action을 발견한다.

예:

- Button Click
- Link Click
- Grid Row Click
- Grid Cell Click
- Checkbox Change
- Select Change
- Input Change
- Enter / Keyboard Event
- Form Submit
- Tab Change
- Modal Open
- Layer Popup Open
- Window Popup Open
- Page Initialization
- useEffect
- mounted
- onMounted
- Automatic Initial API Load

결과:

ACT-001
ACT-002
ACT-003
...

각 Action에는 고유한 Action ID를 부여한다.

Action Discovery 완료 후 반드시 STOP 한다.


## PHASE 2 — Selected Action API Discovery

입력:

ACT-XXX

목적:

선택한 Action을 실제 실행했을 때 발생하는
Network Request를 확인한다.

결과 예:

ACT-001
├─ API-001 POST /api/equipment/search
└─ API-002 GET /api/code/list

가능한 경우 다음 Runtime 정보를 확인한다.

- HTTP Method
- Request URL
- Query Parameter
- Request Payload
- Response Status
- Response Structure

API Discovery 완료 후 반드시 STOP 한다.

Backend 내부 구현은 이 단계에서 분석하지 않는다.


## PHASE 3 — Selected API Specification

입력:

API-XXX

목적:

선택한 API의 실제 Runtime Request/Response와
Source Code를 연결하여 API 계약을 분석한다.

확인 대상:

- HTTP Method
- Endpoint
- Request Parameter
- Request DTO / VO / Map
- Validation
- Response DTO / VO / Map
- Response Structure
- Controller
- Source Path
- Error Response

Runtime에서 관찰된 값과
Source Code에서 정의된 계약을 구분한다.

API Specification 완료 후
요청되지 않은 Business Logic 분석으로 자동 확장하지 않는다.


## PHASE 4 — Frontend Business Analysis

선택한 Action/API와 관련된
Frontend Business Logic만 분석한다.

확인 대상 예:

Event
→ Handler
→ Validation
→ Parameter 생성
→ State
→ API 호출
→ Response 처리
→ 화면 갱신
→ Exception 처리

현재 분석 대상과 관련 없는
Frontend 영역은 탐색하지 않는다.


## PHASE 5 — Backend Business Analysis

선택한 API의 Backend 호출 흐름을 분석한다.

기본적인 추적 방향:

Controller
→ Service
→ ServiceImpl
→ Internal Service
→ External API
→ Mapper

실제 Source Code에서 확인되는 호출 관계만 사용한다.

호출 관계를 발견했다는 이유만으로
Business 의미를 추측하지 않는다.


## PHASE 6 — MyBatis / Oracle Analysis

Backend에서 실제로 호출되는 MyBatis Mapper를 추적한다.

기본 추적 방향:

Service
→ Mapper Interface
→ Mapper XML
→ Statement ID
→ Parameter
→ SQL
→ Oracle Object

MyBatis XML에서는 필요한 경우 다음 항목을 확인한다.

- namespace
- statement id
- parameterType
- resultType
- resultMap
- select
- insert
- update
- delete
- include
- sql fragment
- if
- choose
- when
- otherwise
- foreach
- where
- set
- trim

Oracle 분석은 실제 DB Metadata를 기준으로 검증한다.

필요한 경우 다음 정보를 확인한다.

- Schema
- Table
- View
- Column
- Data Type
- Nullable
- Primary Key
- Foreign Key

Database 구조나 SQL 의미를
확인되지 않은 정보로 추측하지 않는다.


# 4. MCP Responsibilities

각 MCP는 명확하게 역할을 분리한다.


## 4.1 Chrome DevTools MCP

Chrome DevTools MCP는
Frontend Runtime 분석에 사용한다.

주요 용도:

- 실제 URL 접근
- 실제 화면 확인
- DOM 확인
- Actionable Element 확인
- Runtime Event 확인
- Network Request 확인
- Request Parameter 확인
- Request Payload 확인
- Response 확인
- Console 확인
- Popup 동작 확인

Chrome DevTools에서 관찰된 결과는
Runtime Evidence로 취급한다.


## 4.2 Code Index MCP

Code Index MCP는
Source Code 관계 탐색에 사용한다.

주요 용도:

- Symbol Search
- Source Path 탐색
- Reference Search
- Caller 확인
- Callee 확인
- 호출 관계 탐색
- 관련 Source 범위 축소

Code Index MCP의 탐색 결과만으로
Business Logic을 확정하지 않는다.

MCP를 통해 관련 Source를 발견한 후
실제 Source Code를 확인하여 판단한다.


## 4.3 Oracle MCP

Oracle MCP는
Database Metadata 확인에 사용한다.

주요 용도:

- Schema 확인
- Table 확인
- View 확인
- Column 확인
- Data Type 확인
- Nullable 확인
- PK 확인
- FK 확인
- Database Object 검증

기본적으로 Metadata와
분석에 필요한 Read 작업만 수행한다.

데이터 변경 작업은 분석 목적으로 실행하지 않는다.

금지 대상:

- INSERT
- UPDATE
- DELETE
- MERGE
- CREATE
- ALTER
- DROP
- TRUNCATE


# 5. Evidence Principle

모든 중요한 분석 결과는
확인 가능한 Evidence를 기반으로 작성한다.

Evidence의 우선순위는 다음과 같다.

1. 실제 Runtime 동작
2. 실제 Source Code
3. 실제 Database Metadata
4. MCP를 통한 Source 관계 탐색

MCP 검색 결과는
Source 발견을 위한 Discovery Evidence로 사용할 수 있지만,
Business 의미를 단독으로 확정하는 Evidence로 사용하지 않는다.

가능하면 분석 결과에 다음 정보를 남긴다.

- Source Path
- Class
- Method
- Function
- Mapper
- Mapper XML
- Statement ID
- API
- Runtime Request
- Runtime Response
- Oracle Object


# 6. No Unsupported Inference

확인되지 않은 내용을 사실처럼 작성하지 않는다.

확인할 수 없는 경우 다음과 같이 명시한다.

- 확인되지 않음
- Runtime 확인 필요
- Source 확인 필요
- Oracle Metadata 확인 필요
- 호출 관계 확인 필요

이름만 보고 Business 의미를 확정하지 않는다.

예:

searchEquipment()

이라는 함수명이 존재하더라도
실제 구현을 확인하지 않고

"설비 정보를 DB에서 검색한다"

라고 확정하지 않는다.


# 7. Scope Control

현재 분석 대상과 직접 관련 없는 영역으로
분석 범위를 확대하지 않는다.

금지 예:

하나의 Button 분석
→ 전체 Frontend 분석
→ 전체 Controller 분석
→ 전체 Service 분석
→ 전체 Mapper 분석
→ 전체 DB 분석

올바른 방식:

Selected Action
→ 관련 Handler
→ 실제 호출 API
→ 선택된 API
→ 관련 Controller
→ 관련 Service
→ 관련 Mapper
→ 관련 SQL
→ 관련 Oracle Object

항상 필요한 범위만 탐색한다.


# 8. Context Control

Context 사용량을 최소화한다.

다음 행동을 피한다.

- 전체 Repository를 한 번에 읽기
- 관련 없는 디렉터리 탐색
- 대량 파일을 이유 없이 읽기
- 전체 Backend를 한 번에 Call Graph로 확장
- 전체 Mapper XML 읽기
- 전체 Database Schema 탐색
- 동일 Source 반복 읽기

먼저 MCP와 검색 도구를 이용하여
관련 범위를 좁힌 후 필요한 Source만 읽는다.


# 9. Popup Policy

Popup은 종류에 따라 분석 범위를 다르게 적용한다.

## Window Open Popup

window.open 형태의 새 창은
기본적으로 다음 항목까지만 분석한다.

- Target URL
- 전달 Parameter

새 창 내부 기능 전체를
자동으로 추가 분석하지 않는다.


## Modal / Layer Popup

현재 화면 내부에서 동작하는

- Modal
- Dialog
- Layer Popup

은 현재 기능의 일부로 취급한다.

따라서 선택된 분석 범위에 포함되는 경우
내부 Event와 Business Logic을 분석할 수 있다.


# 10. Runtime vs Source

Runtime Evidence와 Source Evidence를 구분한다.

예:

Chrome DevTools에서 확인:

POST /api/equipment/search

Request:

{
  "plant": "1000"
}

이 결과는

"이번 Runtime 실행에서 plant=1000이 전송되었다"

는 Evidence다.

이것만으로

"plant의 가능한 값은 항상 1000이다"

라고 판단하지 않는다.

가능한 값, 필수 여부, Validation 등은
Source Code를 추가 확인한다.


# 11. Analysis Stop Policy

각 분석 단계에는 명확한 STOP 지점이 존재한다.

Claude Code는 STOP 지점을 반드시 지킨다.

Screen Action Discovery
→ Action Inventory 생성
→ STOP

Selected Action API Discovery
→ API Inventory 생성
→ STOP

Selected API Specification
→ API Specification 생성
→ STOP

Business Logic Analysis
→ 요청된 범위 완료
→ STOP

사용자가 다음 대상을 명시적으로 선택하기 전까지
자동으로 다음 분석 단계로 진행하지 않는다.


# 12. User Selection Principle

분석 대상 선택은 사용자가 한다.

Claude Code는 발견된 후보를 제공할 수 있지만
다음 분석 대상을 임의로 선택하여 계속 분석하지 않는다.

예:

ACT-001
ACT-002
ACT-003

발견 후:

"ACT-001이 중요해 보이므로 계속 분석하겠습니다."

와 같이 자동 진행하지 않는다.

Action Inventory를 출력하고 STOP 한다.


# 13. Final Principle

이 프로젝트의 핵심 원칙은 다음과 같다.

Discover
→ Narrow Scope
→ Verify
→ Document
→ STOP
→ User Selects Next Target

분석의 깊이보다 먼저
분석 범위를 정확하게 통제한다.

빠르게 많은 Source를 읽는 것보다
실제 Runtime과 Source Evidence를 연결하여
정확한 결과를 만드는 것을 우선한다.

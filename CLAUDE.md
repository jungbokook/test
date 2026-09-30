# Code Analysis Project

## Purpose

이 프로젝트는 Frontend 화면을 시작점으로
기능을 단계적으로 분석하고
각 분석 결과를 독립된 문서로 생성한다.

전체 기능을 한 번에 분석하지 않는다.


## Analysis Structure

분석은 서로 독립된 단계로 수행한다.

### SCREEN Analysis

입력:

- Frontend URL

목적:

- 실제 화면의 Business Action 식별

출력:

- SCREEN 문서

주요 대상:

- Button
- Link
- Tab
- Grid Action
- Input Action
- Popup
- Page Initialization

SCREEN Analysis에서는
Frontend Business Logic이나 Backend를 분석하지 않는다.


### FRONTEND Analysis

입력:

- 사용자가 선택한 Action ID

목적:

- 선택한 Action의 Frontend Business Logic 분석

주요 대상:

- Event
- Handler
- Validation
- Parameter 생성
- State 처리
- Frontend Business Logic
- Backend 호출
- HTTP Method
- Backend URL
- Request Parameter / Body

출력:

- FE 문서

Backend 내부 구현은 분석하지 않는다.


### BACKEND Analysis

입력:

- 사용자가 선택한 Backend API

목적:

- 해당 API의 Backend Business Logic 분석

주요 대상:

- Controller
- Request
- Validation
- Service
- Business Logic
- 조건 / 분기
- 내부 Service 호출
- External System / SAP
- Mapper
- MyBatis XML
- SQL
- Oracle Database
- Response
- Exception

출력:

- BE 문서


## Independent Analysis

각 분석은 독립적으로 실행한다.

SCREEN Analysis가 완료되어도
FRONTEND Analysis를 자동으로 시작하지 않는다.

FRONTEND Analysis에서 Backend API가 발견되어도
BACKEND Analysis를 자동으로 시작하지 않는다.

다음 분석 대상은 사용자가 직접 선택한다.


## Analysis Principle

분석은 다음 원칙을 따른다.

Discover
→ Verify
→ Document
→ STOP
→ User Selects Next Target


## Evidence

확인되지 않은 내용을 추측하여
사실처럼 문서화하지 않는다.

분석 결과는 가능한 경우 다음 근거를 사용한다.

- Runtime
- Source Code
- Database Metadata


## MCP Responsibilities

### Chrome DevTools MCP

Frontend Runtime 확인에 사용한다.

주요 용도:

- 실제 화면
- DOM
- UI Action
- Runtime 동작
- Network
- Request / Response


### Code Index MCP

Source Code 위치와 관계 탐색에 사용한다.

주요 용도:

- Symbol
- Source Path
- Reference
- Caller / Callee
- Call Relationship

Code Index 결과만으로
Business Logic을 확정하지 않는다.


### Oracle MCP

Backend Database 분석에서 사용한다.

주요 용도:

- Schema
- Table / View
- Column
- Data Type
- PK / FK
- Database Metadata

Database 변경 작업은 수행하지 않는다.


## Core Rule

현재 선택된 분석 단계의 범위를 넘어
다음 분석을 자동으로 수행하지 않는다.
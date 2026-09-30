# 코드 기능 분석 프로젝트 가이드

## 1. 목적

Frontend 화면을 시작점으로 기능을 단계적으로 분석하고
각 분석 결과를 독립된 문서로 생성한다.

전체 시스템을 한 번에 분석하지 않는다.
현재 사용자가 선택한 대상만 분석한다.


## 2. 분석 구조

분석은 3개의 독립 단계로 수행한다.

### 화면 기능 분석

입력:
- Frontend URL

분석:
- 실제 화면의 Button, Link, Grid, Tab, Popup 등
- 화면 진입 시 자동 실행되는 기능
- Action과 연결된 Handler 및 Source 위치

출력:
- 화면 기능 목록 문서
- Action ID (`ACT-001`, `ACT-002` ...)

화면 기능을 식별하는 것까지만 수행한다.


### Frontend 기능 분석

입력:
- 사용자가 선택한 Action ID

분석:
- Event / Handler
- Validation
- Parameter 생성
- 조건 / 분기
- Frontend 비즈니스 로직
- State 처리
- Backend 호출
- HTTP Method / URL
- Request Parameter / Body

출력:
- Frontend 기능 문서

Backend 내부 구현은 분석하지 않는다.


### Backend 기능 분석

입력:
- 사용자가 선택한 Backend API

분석:
- Controller
- Request / Validation
- Service / ServiceImpl
- Backend 비즈니스 로직
- 조건 / 분기
- 내부 Service 호출
- 외부 시스템 / SAP
- Mapper
- MyBatis Mapper XML
- SQL
- Oracle Table / Column
- Response / Exception

출력:
- Backend 기능 문서

Database 분석은 기본적으로 Backend 문서에 포함한다.


## 3. 단계 독립 원칙

각 단계는 독립적으로 실행한다.

화면 기능 분석
→ 문서 생성
→ 종료

Frontend 기능 분석
→ 문서 생성
→ 종료

Backend 기능 분석
→ 문서 생성
→ 종료

다음 단계로 자동 진행하지 않는다.
다음 분석 대상은 사용자가 직접 선택한다.


## 4. 분석 근거

확인되지 않은 내용을 추측하여 사실처럼 작성하지 않는다.

가능한 경우 다음 근거를 사용한다.

1. Runtime
2. Source Code
3. Database Metadata
4. 코드 탐색 결과

코드 이름이나 MCP 탐색 결과만으로
비즈니스 로직을 확정하지 않는다.


## 5. MCP 역할

### Chrome DevTools MCP

Frontend Runtime 확인에 사용한다.

- 실제 화면
- DOM / UI Action
- Runtime 동작
- Network
- Request / Response


### Code Index MCP

관련 Source Code를 찾고 범위를 좁히는 데 사용한다.

- Symbol
- Source Path
- Reference
- Caller / Callee
- 호출 관계

Business Logic은 관련 Source를 직접 확인하여 해석한다.


### Oracle MCP

Backend Database 검증에 사용한다.

- Schema
- Table / View
- Column
- PK / FK
- Database Metadata

데이터 변경 작업은 수행하지 않는다.


## 6. 분석 범위

현재 선택된 기능과 직접 관련된 범위만 탐색한다.

전체 Repository를 불필요하게 탐색하지 않는다.
현재 단계보다 이후 영역을 미리 분석하지 않는다.


## 7. 기본 원칙

대상 선택
→ 탐색
→ 근거 확인
→ 분석
→ 문서 생성
→ 종료
→ 사용자가 다음 대상 선택
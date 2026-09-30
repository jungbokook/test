# 코드 기능 분석 프로젝트 가이드

## 1. 목적

Frontend 화면을 시작점으로 기능을 단계적으로 분석하고
각 분석 결과를 독립된 문서로 생성한다.

전체 시스템을 한 번에 분석하지 않는다.
현재 사용자가 선택한 대상만 분석한다.


## 2. 분석 구조

분석은 4개의 독립 단계로 수행한다.

### 1단계. 화면 기능 분석

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


### 2단계. Frontend 기능 분석

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
- 발견된 Backend API 정보

Backend 내부 구현은 분석하지 않는다.


### 3단계. API 규격 분석

입력:
- 사용자가 선택한 Backend API

분석:
- API 기능
- HTTP Method
- URL
- Header
- Query Parameter
- Path Parameter
- Request Body
- Request Type
- 필수 여부
- Validation
- Response
- Response Type
- HTTP Status
- 오류 Response

API 규격은 가능한 경우 다음 정보를 교차 확인한다.

- 실제 Runtime Request / Response
- Frontend 호출 코드
- Controller
- Request DTO
- Response DTO
- Validation

Controller 이후의 Service, Mapper, SQL 등
Backend 내부 비즈니스 로직은 분석하지 않는다.

출력:
- API 규격 문서
- API ID (`API-001`, `API-002` ...)


### 4단계. Backend 기능 분석

입력:
- 사용자가 선택한 Backend API 또는 API ID

분석:
- Controller
- Request / Validation
- Service / ServiceImpl
- Backend 비즈니스 로직
- 조건 / 분기
- 내부 Method / Service 호출
- 데이터 변환 및 상태 처리
- 외부 시스템 / SAP
- Mapper
- MyBatis Mapper XML
- Dynamic SQL
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

API 규격 분석
→ 문서 생성
→ 종료

Backend 기능 분석
→ 문서 생성
→ 종료

현재 단계가 완료되어도
다음 단계로 자동 진행하지 않는다.

다음 분석 대상은 사용자가 직접 선택한다.


## 4. 분석 근거

확인되지 않은 내용을 추측하여 사실처럼 작성하지 않는다.

가능한 경우 다음 근거를 사용한다.

1. Runtime
2. Source Code
3. Database Metadata
4. 코드 탐색 결과

Runtime에서 관찰한 값과
Source Code에 정의된 규격을 구분한다.

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

화면 기능 분석과 API의 실제 호출 정보 확인에 활용한다.


### Code Index MCP

관련 Source Code를 찾고 분석 범위를 좁히는 데 사용한다.

- Symbol
- Source Path
- Reference
- Caller / Callee
- 호출 관계

Code Index 결과만으로
Business Logic을 확정하지 않는다.

관련 Source Code를 실제 확인하여 해석한다.


### Oracle MCP

Backend Database 검증에 사용한다.

- Schema
- Table / View
- Column
- PK / FK
- Database Metadata

데이터 변경 작업은 수행하지 않는다.


## 6. Database 분석

Database 분석은 기본적으로
Backend 기능 분석에 포함한다.

기본 흐름:

Controller
→ Service
→ ServiceImpl
→ Mapper
→ MyBatis Mapper XML
→ SQL
→ Oracle

Backend 문서가 지나치게 커지는 경우에만
Database 문서를 별도로 분리할 수 있다.


## 7. 분석 범위

현재 선택된 기능과 직접 관련된 범위만 탐색한다.

전체 Repository를 불필요하게 탐색하지 않는다.

현재 단계보다 이후 영역을 미리 분석하지 않는다.

한 단계에서 다음 단계의 대상이 발견되더라도
필요한 식별 정보만 기록하고 상세 분석하지 않는다.


## 8. 문서 저장 위치

분석 결과는 화면 단위로 관리한다.

프로젝트 Root의
`docs/analysis/{화면명}/`
아래에 해당 화면과 관련된 분석 문서를 저장한다.

기본 구조:

`docs/analysis/{화면명}/`

- `SCREEN-{화면명}.md`
- `frontend/`
- `api/`
- `backend/`

단계별 저장 위치:

### 화면 기능 분석

`docs/analysis/{화면명}/SCREEN-{화면명}.md`

### Frontend 기능 분석

`docs/analysis/{화면명}/frontend/FE-{Action ID}-{기능명}.md`

### API 규격 분석

`docs/analysis/{화면명}/api/API-{API ID}-{기능명}.md`

### Backend 기능 분석

`docs/analysis/{화면명}/backend/BE-{API ID}-{기능명}.md`

화면명은 가능한 경우
Frontend URL 또는 Route를 기준으로
일관된 이름을 사용한다.

예:

Frontend URL:

`/equipment/search`

화면명:

`equipment-search`

저장 위치:

`docs/analysis/equipment-search/`

필요한 디렉터리가 존재하지 않는 경우 생성한다.

현재 분석 단계에 필요한 디렉터리와 문서만 생성한다.

기존 분석 문서가 존재하는 경우
임의로 덮어쓰지 않는다.

## 9. 기본 원칙

Frontend URL
→ 화면 기능 문서
→ 종료

사용자가 Action 선택
→ Frontend 기능 문서
→ 종료

사용자가 API 선택
→ API 규격 문서
→ 종료

사용자가 Backend 분석 대상 선택
→ Backend 기능 문서
→ 종료


## 10. 핵심 원칙

대상 선택
→ 탐색
→ 근거 확인
→ 분석
→ 독립 문서 생성
→ 종료
→ 사용자가 다음 대상 선택
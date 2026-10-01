---
name: fe-analysis-scope
description: SCREEN 문서에서 사용자가 선택한 Action을 기준으로 Frontend 기능을 상세 분석할 때의 범위와 Backend 분석 경계를 정의한다.
---

# Frontend 기능 분석 범위

## 1. 목적

SCREEN 분석에서 식별된 특정 Action을 기준으로
Frontend Source Code의 실제 처리 흐름을 상세 분석한다.

Frontend 기능 분석은
사용자가 선택한 하나의 Action만 대상으로 한다.

전체 화면이나 전체 Frontend Repository를
다시 분석하지 않는다.


## 2. 입력

Frontend 기능 분석의 입력은
다음 두 값을 사용한다.

- 화면명
- Action ID

예:

- 화면명: `equipment-search`
- Action ID: `ACT-002`

Action ID만으로
분석 대상을 임의 판단하지 않는다.

화면명과 Action ID를 함께 사용하여
분석 대상을 식별한다.


## 3. SCREEN 문서 확인

Frontend 분석을 시작하기 전에
다음 SCREEN 문서를 확인한다.

`docs/analysis/{화면명}/SCREEN-{화면명}.md`

SCREEN 문서에서
사용자가 선택한 Action ID를 찾는다.

해당 Action에 기록된 다음 정보를
Frontend 분석의 시작점으로 사용한다.

- Action ID
- 기능명
- UI 유형
- Event
- Handler
- Source Path

선택한 Action ID가
SCREEN 문서에 존재하지 않는 경우
다른 Action을 임의로 선택하지 않는다.

분석을 중단하고
해당 Action을 확인할 수 없음을 알린다.


## 4. 분석 대상

선택한 Action과 직접 관련된
Frontend 처리 흐름을 분석한다.

분석 대상:

- Event
- Handler
- Handler 호출 흐름
- Input
- Validation
- 조건 / 분기
- Parameter 생성
- 데이터 변환
- Frontend State 처리
- 관련 Function / Method 호출
- Backend 호출
- HTTP Method
- Backend URL
- Query Parameter
- Path Parameter
- Request Body
- Backend 호출 조건
- Backend 호출 순서
- Response 처리
- Frontend 예외 처리

선택한 Action과 직접 관련되지 않은
다른 화면 기능은 분석하지 않는다.


## 5. Source 분석

SCREEN 문서에 기록된
Handler와 Source Path를
우선 분석 시작점으로 사용한다.

예:

`ACT-002 검색`

→ `Click`

→ `handleSearch`

→ `src/views/equipment/EquipmentSearch.vue`

이 경우
`handleSearch`를 시작점으로
선택한 Action의 실제 Frontend 처리 흐름을 추적한다.

SCREEN 문서의 Handler 또는 Source Path가
`확인되지 않음`인 경우에는
현재 화면의 Route, Component, UI Element, Event 등의
확인된 정보를 기준으로 Source를 탐색할 수 있다.

Source를 확인하지 못한 경우
추측하여 연결하지 않는다.


## 6. Code Index MCP 사용

필요한 경우 Code Index MCP를 사용하여
선택한 Action과 직접 관련된 Source 범위를 좁힌다.

사용 목적:

- Handler Symbol 확인
- Handler Reference 확인
- Caller / Callee 확인
- 관련 Function / Method 확인
- 관련 Component / Module 확인
- Source Path 확인
- 직접적인 호출 관계 확인

Code Index MCP 결과는
관련 Source를 찾기 위한 탐색 근거이다.

Code Index 결과만으로
Frontend Business Logic을 확정하지 않는다.

관련 Source Code를 실제 확인하여
조건, 데이터 처리, 호출 관계를 해석한다.


## 7. Frontend 호출 흐름

선택한 Action에서 시작되는
Frontend 호출 흐름을 추적한다.

예:

검색 버튼

→ Click

→ `handleSearch()`

→ `validateSearchCondition()`

→ `createSearchParameter()`

→ `requestEquipmentList()`

→ Backend API 호출

Function / Method 호출이 여러 단계로 이어지는 경우
선택한 Action과 직접 관련된 범위에서는
계속 추적할 수 있다.

단순히 함수 이름과 호출 관계만 나열하지 않는다.

실제 Source Code를 확인하여
각 단계에서 수행되는 처리 내용을 설명한다.


## 8. Validation 분석

선택한 Action 실행 과정에서 수행되는
Frontend Validation을 확인한다.

가능한 경우 다음 내용을 기록한다.

- Validation 대상
- Validation 입력 값
- Validation 조건
- 성공 조건
- 실패 조건
- 실패 시 처리
- 사용자 메시지
- 이후 처리 중단 여부

Validation에 YES / NO 분기가 존재하는 경우
분기 흐름이 보이도록 기록한다.

예:

`검색조건 존재 여부`

→ YES
  → 검색 진행

→ NO
  → 메시지 표시
  → 처리 종료

Validation이 존재하지 않는 경우
임의로 생성하지 않는다.


## 9. Parameter 및 데이터 처리

Backend 호출 또는
Frontend 내부 Function 호출을 위해
생성되거나 변경되는 데이터를 분석한다.

가능한 경우 다음을 확인한다.

- 화면 입력 값
- 원본 값
- Source State
- 기본값
- Parameter 생성 과정
- 데이터 변환
- 조건에 따른 값 변경
- 최종 전달 값

화면에 표시된 값과
실제로 Function 또는 Backend로 전달되는 값을
구분하여 기록한다.

예:

화면 표시 값

`전체`

↓

Source State

`plantCode = ""`

↓

Request Parameter

`plantCode = null`

처럼 실제 변환이 존재하는 경우
변환 과정을 기록한다.


## 10. 조건 및 분기

선택한 Action 처리 과정에서 발생하는
조건과 분기를 분석한다.

가능한 경우 다음을 기록한다.

- 조건식
- 조건에 사용되는 값
- TRUE 처리
- FALSE 처리
- 이후 호출되는 Function
- Backend 호출 여부 변화
- State 변경 여부

분기가 중첩되어 있는 경우
실제 실행 구조가 보이도록 표현한다.

조건을 단순히
"조건 처리"라고 요약하지 않는다.


## 11. State 처리

선택한 Action과 직접 관련된
Frontend State 변경을 분석한다.

가능한 경우 다음을 확인한다.

- State 이름
- 변경 전 값 또는 Source
- 변경 조건
- 변경 값
- 변경 후 사용 위치
- 화면에 미치는 영향

예:

Backend Response

→ `equipmentList`

→ State 저장

→ Grid Data 갱신

현재 Action과 직접 관련 없는
일반적인 Component State나
단순 Rendering 구현까지 확장하지 않는다.


## 12. Backend 호출

선택한 Action에서 발생하는
Backend 호출을 식별한다.

하나의 Action에서
여러 Backend API를 호출하는 경우
각 호출을 모두 기록한다.

각 Backend 호출에 대해
확인 가능한 범위에서 다음을 기록한다.

- 호출 목적
- 호출 Function / Method
- HTTP Method
- URL
- Query Parameter
- Path Parameter
- Request Body
- 호출 조건
- 호출 순서
- Response 사용 위치

예:

`ACT-002 검색`

→ `handleSearch()`

→ Validation

→ Parameter 생성

→ `getEquipmentList()`

→ `GET /api/equipment`

현재 단계에서는
Backend API의 내부 구현을 분석하지 않는다.


## 13. 여러 Backend API 호출

하나의 Action에서
여러 Backend API가 호출될 수 있다.

예:

`ACT-002`

→ API A

→ Response 확인

→ 조건 분기

→ API B

또는

`ACT-002`

→ API A

→ API B

각 API 호출의 순서와 조건을
실제 Frontend Source 기준으로 기록한다.

API가 여러 개 발견되더라도
현재 단계에서 자동으로
API 규격 분석을 시작하지 않는다.


## 14. Response 처리

Backend Response가
Frontend에서 어떻게 사용되는지 분석한다.

가능한 경우 다음을 확인한다.

- Response 수신 위치
- 성공 여부 판단
- Response 데이터 접근
- 데이터 변환
- State 반영
- Grid 반영
- Form 반영
- Component 반영
- 사용자 메시지
- 실패 처리
- 예외 처리

Backend Response가 Frontend에서
추가 가공되는 경우
가공 과정을 기록한다.

현재 단계에서는
Backend Response의 전체 API 규격을
확정하지 않는다.


## 15. Frontend 예외 처리

선택한 Action과 직접 관련된
Frontend 예외 처리를 확인한다.

가능한 경우 다음을 기록한다.

- `try / catch`
- Promise `catch`
- API 오류 처리
- 오류 메시지
- 사용자 알림
- State 초기화
- Loading 상태 해제
- 후속 처리 중단 여부

공통 Error Handler가 호출되는 경우
현재 Action과 직접 관련된 호출 관계까지만 확인한다.

현재 Action과 관련 없는
공통 Framework 전체 구현까지 확장하지 않는다.


## 16. API 발견

Frontend 분석 중 발견한 Backend API는
다음 단계에서 사용자가 선택할 수 있도록 기록한다.

하나의 Action에서
여러 Backend API가 발견될 수 있다.

발견된 API에는 가능한 경우
다음 정보를 기록한다.

- API 후보 식별 정보
- 호출 목적
- HTTP Method
- URL
- 호출 Function
- 호출 조건

현재 단계에서는
발견된 API를 자동으로
API 규격 분석하지 않는다.

API 규격 분석은
사용자가 API를 선택한 후
별도의 API 분석 단계에서 수행한다.


## 17. 분석 경계

Frontend 기능 분석에서 허용되는 최대 범위:

`Action`

→ `Event`

→ `Handler`

→ `Frontend Function / Method`

→ `Validation`

→ `조건 / 분기`

→ `Parameter / Data 처리`

→ `State 처리`

→ `Backend 호출`

→ `Frontend Response 처리`

여기까지 분석한다.

다음 영역으로 내려가지 않는다.

- Backend Controller 내부 구현
- Backend Service
- ServiceImpl
- Backend Business Logic
- Mapper
- MyBatis Mapper XML
- Dynamic SQL
- SQL
- Oracle Table / Column
- SAP / 외부 시스템 내부 처리

Backend Source 위치가 발견되더라도
현재 단계에서는 상세 분석하지 않는다.


## 18. 분석 근거

Frontend 분석 결과는
가능한 경우 실제 Source Code를 근거로 작성한다.

우선적으로 다음 근거를 사용한다.

1. SCREEN 문서의 Action 정보
2. 실제 Frontend Source Code
3. Code Index MCP 탐색 결과
4. 필요한 경우 Runtime 정보

Source Code와 Runtime에서 확인한 내용을
구분하여 기록한다.

코드 이름이나
Code Index의 호출 관계만으로
Business Logic을 확정하지 않는다.

확인되지 않은 내용은
추측하여 작성하지 않는다.

확인할 수 없는 항목은:

`확인되지 않음`

으로 기록한다.


## 19. 분석 범위 제한

현재 선택된 Action과
직접 관련된 Source만 탐색한다.

전체 Frontend Repository를
불필요하게 탐색하지 않는다.

다른 Action의 Handler가 발견되더라도
현재 Action과 직접적인 호출 관계가 없다면
상세 분석하지 않는다.

현재 Action 분석에 필요한 만큼만
호출 관계를 확장한다.


## 20. 저장 위치

Frontend 분석 결과는
해당 화면의 분석 디렉터리에 저장한다.

저장 위치:

`docs/analysis/{화면명}/frontend/`

파일명:

`FE-{Action ID}-{기능명}.md`

예:

`docs/analysis/equipment-search/frontend/FE-ACT-002-search.md`

필요한 디렉터리가 존재하지 않는 경우
디렉터리를 생성한다.

기존 동일 이름의 분석 문서가 존재하는 경우
임의로 덮어쓰지 않는다.


## 21. STOP 규칙

Frontend 기능 분석 문서를 생성한 후
반드시 종료한다.

다음 작업을 자동으로 수행하지 않는다.

- API 규격 상세 분석
- Backend 기능 분석
- Database 분석

발견된 Backend API는
다음 분석 후보로만 기록한다.

다음 분석 대상은
사용자가 직접 선택한다.


## 22. 핵심 원칙

`화면명 + Action ID`

→ SCREEN 문서 확인

→ 선택 Action 확인

→ Handler / Source Path 확인

→ Frontend Source 분석

→ Function 호출 흐름 추적

→ Validation 확인

→ 조건 / 분기 확인

→ Parameter / Data 처리 확인

→ State 처리 확인

→ Backend 호출 식별

→ Frontend Response 처리 확인

→ Frontend 문서 생성

→ STOP
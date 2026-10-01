---
name: fe-call-tracing
description: 선택한 Frontend Action의 Handler에서 시작하여 Validation, 조건/분기, 데이터 처리, 내부 Function 호출, Backend API 호출까지 실제 실행 흐름을 추적하는 규칙을 정의한다.
---

# Frontend 호출 흐름 추적 규칙

## 1. 목적

사용자가 선택한 Action의 Handler를 시작점으로
Frontend에서 실제로 실행되는 호출 흐름을 추적한다.

단순한 Function 호출 목록을 만드는 것이 아니라
각 Function이 수행하는 역할과 조건 / 분기를 확인하여
실제 실행 흐름이 보이도록 분석한다.

분석 범위는 현재 선택된 Action과
직접 관련된 Frontend 처리 흐름으로 제한한다.


## 2. 분석 시작점

Frontend 호출 추적은
SCREEN 문서에서 확인된 다음 정보를 시작점으로 사용한다.

- 화면명
- Action ID
- Event
- Handler
- Source Path

예:

화면명:

`equipment-search`

Action:

`ACT-002`

Handler:

`handleSearch`

Source Path:

`src/views/equipment/EquipmentSearch.vue`

호출 추적 시작:

`ACT-002`

→ `Click`

→ `handleSearch()`


## 3. 실제 Source 기준 추적

호출 관계는
실제 Frontend Source Code를 기준으로 확인한다.

Function 이름만 보고
역할을 추측하지 않는다.

예:

`validate()`

라는 이름만으로
Validation 전체 내용을 확정하지 않는다.

실제 Source를 확인하여 다음을 판단한다.

- 어떤 값을 확인하는가
- 어떤 조건을 사용하는가
- 어떤 값을 반환하는가
- 실패하면 어떻게 처리되는가
- 이후 실행 흐름에 어떤 영향을 주는가

Code Index MCP의 호출 관계는
Source 탐색을 위한 보조 근거로 사용한다.


## 4. 기본 호출 추적

Handler에서 직접 호출되는
Function / Method를 확인한다.

각 Function 내부에서
현재 Action의 실행 결과에 영향을 주는
다음 호출을 계속 추적한다.

- Validation Function
- Parameter 생성 Function
- 데이터 변환 Function
- State 처리 Function
- Business Function
- API 호출 Function
- Response 처리 Function
- Error 처리 Function

예:

`handleSearch()`

→ `validateSearchCondition()`

→ `createSearchParameter()`

→ `requestEquipmentList()`

→ Backend API

각 단계의 실제 Source를 확인한다.


## 5. 다단계 호출

API 호출까지 여러 단계의
Function / Method가 연결되어 있는 경우
중간 단계를 생략하지 않는다.

예:

`handleSearch()`

→ `search()`

→ `loadEquipment()`

→ `equipmentService.getList()`

→ `apiClient.get()`

→ Backend API

단순히 다음과 같이 축약하지 않는다.

`handleSearch()`

→ Backend API

중간 Function이 실제 Business Logic,
Validation, Parameter 변환 또는
호출 조건에 영향을 주는 경우
반드시 분석에 포함한다.


## 6. 호출 단계별 역할

호출되는 각 Function / Method에 대해
가능한 경우 다음 내용을 확인한다.

- Function / Method 이름
- Source Path
- 호출 위치
- 입력 값
- 수행 역할
- 조건 / 분기
- 반환 값
- 다음 호출 대상

예:

`createSearchParameter()`

입력:

- `plantCode`
- `equipmentType`
- `keyword`

처리:

- 빈 문자열 처리
- 기본값 적용
- Request Parameter 구조 생성

출력:

`searchParams`

다음 호출:

`requestEquipmentList(searchParams)`

확인되지 않은 내용은
추측하지 않는다.


## 7. 조건 / 분기 추적

Handler 또는 하위 Function에
조건문이 존재하는 경우
실행 경로를 분리하여 추적한다.

대상:

- `if`
- `else`
- `else if`
- `switch`
- 삼항 연산자
- 조건부 Function 호출
- 조건부 API 호출
- Early Return
- Promise 조건 처리
- Framework의 조건 실행 구조

예:

`handleSearch()`

→ Validation

├─ 실패

│  → Message 표시

│  → `return`

│  → 처리 종료

└─ 성공

   → Parameter 생성

   → Backend API 호출

조건을 단순히
"Validation 수행" 또는
"조건 처리"라고 축약하지 않는다.


## 8. 중첩 분기

조건이 여러 단계로 중첩되는 경우
실제 실행 구조가 보이도록 추적한다.

예:

`handleSearch()`

→ Validation

├─ 실패

│  → 처리 종료

└─ 성공

   → 검색 유형 확인

   ├─ 유형 A

   │  → `requestTypeA()`

   │  → API A

   │

   └─ 유형 B

      → 추가 조건 확인

      ├─ 조건 YES

      │  → `requestTypeB()`

      │  → API B

      │

      └─ 조건 NO

         → Message

         → 처리 종료

각 분기에서 실제로 실행되는
Function과 API가 다르면
각 경로를 구분하여 기록한다.


## 9. Early Return

Frontend Source에
Early Return이 존재하는 경우
실행 흐름에서 명확하게 표시한다.

예:

`if (!keyword)`

→ Message 표시

→ `return`

이 경우 이후의

`createParams()`

→ `requestSearch()`

는 실행되지 않는다는 것을
분명하게 기록한다.

Early Return 이후의 코드를
동일한 실행 경로처럼 연결하지 않는다.


## 10. Parameter 전달 추적

Function 간에 전달되는 Parameter가
변경되는 경우 그 흐름을 추적한다.

예:

화면 입력:

`keyword = "ABC"`

↓

`handleSearch(keyword)`

↓

`createParams(keyword)`

↓

생성 값:

`{ searchKeyword: "ABC" }`

↓

`requestSearch(params)`

↓

Backend Request:

`searchKeyword=ABC`

가능한 경우 다음을 구분한다.

- 화면 값
- State 값
- Function 입력 값
- 중간 변환 값
- 최종 API 전달 값


## 11. State를 통한 값 전달

Function Parameter가 아니라
Frontend State를 통해 값이 전달되는 경우에도
현재 Action과 직접 관련되어 있다면 추적한다.

예:

Input

→ `searchCondition.keyword`

↓

`handleSearch()`

↓

State 참조

↓

`createParams()`

↓

Request Body

State 전체 구조를 분석하는 것이 아니라
현재 Action에서 실제 사용되는 State만 확인한다.


## 12. 비동기 호출

Frontend의 비동기 실행 흐름을 확인한다.

대상 예:

- `async / await`
- Promise
- `.then()`
- `.catch()`
- Callback
- Framework 비동기 처리

예:

`handleSearch()`

→ `await requestEquipmentList()`

→ Response 수신

→ `equipmentList` State 변경

→ Grid 갱신

비동기 호출 전후의
실행 순서를 실제 Source 기준으로 기록한다.


## 13. 병렬 호출

하나의 Action에서
여러 API가 병렬로 호출되는 경우
순차 호출처럼 표현하지 않는다.

예:

`handleSearch()`

→ `Promise.all()`

├─ `requestEquipmentList()`
│  → API A

└─ `requestEquipmentStatus()`
   → API B

↓

모든 Response 수신

↓

화면 갱신

실제 Source가 병렬 실행을 보장하지 않는 경우
임의로 병렬이라고 판단하지 않는다.


## 14. 순차 API 호출

하나의 API Response를 사용하여
다음 API를 호출하는 경우
의존 관계를 기록한다.

예:

`handleSearch()`

→ API A

→ Response에서 `equipmentId` 획득

→ `equipmentId` 확인

→ API B 호출

이 경우 API A와 API B를
서로 독립된 호출처럼 표현하지 않는다.


## 15. 공통 Function / Utility

현재 Action에서
공통 Function 또는 Utility가 호출되는 경우
현재 Action의 실행 흐름을 이해하는 데 필요한 범위만 확인한다.

예:

`handleSearch()`

→ `createParams()`

→ `commonUtil.removeEmptyValue()`

공통 Utility가
Request 값에 실제 영향을 준다면
해당 처리 내용을 확인할 수 있다.

그러나 현재 Action과 관련 없는
공통 Utility 전체 기능을 분석하지 않는다.


## 16. 공통 API Client

API 호출 과정에서
공통 API Client 또는 Wrapper가 사용될 수 있다.

예:

`handleSearch()`

→ `equipmentService.search()`

→ `apiClient.get()`

→ Backend API

공통 API Client는
현재 Action의 실제 Backend 호출 정보를
확인하는 데 필요한 범위까지만 추적한다.

확인 가능한 경우:

- HTTP Method
- Base URL 적용
- Endpoint
- Query Parameter 전달
- Request Body 전달
- 현재 호출에 영향을 주는 Header
- 현재 호출에 영향을 주는 Request 변환

현재 Action과 관계없는
공통 API Client 전체 구현은 분석하지 않는다.


## 17. 공통 처리 확장 제한

공통 Function, Hook, Store, Utility,
API Client 등을 발견했다고 해서
전체 구현을 계속 확장하지 않는다.

추적을 계속하는 기준은 다음과 같다.

현재 Action의 다음 중 하나에
직접 영향을 주는가?

- Validation
- 조건 / 분기
- Parameter
- 데이터 변환
- State
- Backend 호출
- Response 처리
- Error 처리

직접 영향을 주지 않는다면
추가 추적하지 않는다.


## 18. Component 간 호출

현재 Action의 실행 흐름이
여러 Frontend Component에 걸쳐 있는 경우
직접적인 호출 관계를 따라갈 수 있다.

예:

Page Component

→ Search Component

→ Event Emit

→ Parent Handler

→ API 호출

또는:

Child Component

→ Callback

→ Parent Component

→ Store Action

→ API 호출

현재 Action과 직접 연결된
Component 관계만 추적한다.

화면 전체 Component Tree를
분석하지 않는다.


## 19. Store / 상태 관리 호출

현재 Action에서
Store 또는 상태 관리 계층을 사용하는 경우
실제 호출 흐름을 추적할 수 있다.

예:

Component

→ `handleSearch()`

→ Store Action

→ Service Function

→ Backend API

대상 예:

- Redux
- Redux Toolkit
- Vuex
- Pinia
- Zustand
- 기타 프로젝트 상태 관리 구조

현재 Action과 관련된
Action / Mutation / Selector / Store Function만 확인한다.

Store 전체를 분석하지 않는다.


## 20. Custom Hook / Composable

현재 Action에서
Custom Hook 또는 Composable을 사용하는 경우
직접 관련된 호출 흐름을 추적할 수 있다.

예:

React:

Component

→ `useEquipmentSearch()`

→ `searchEquipment()`

→ API

Vue:

Component

→ `useEquipmentSearch()`

→ `searchEquipment()`

→ API

현재 Action에 필요한
Hook / Composable 내부 처리만 확인한다.


## 21. Framework 내부 구현

React, Vue 또는 사용 중인 Framework의
내부 구현까지 추적하지 않는다.

예:

다음 영역은 일반적으로 추적 대상이 아니다.

- React 내부 Rendering Engine
- Vue 내부 Reactivity 구현
- Router Library 내부 구현
- Axios Library 내부 구현
- Fetch 내부 구현

현재 프로젝트 Source에서
실제 Business Flow를 확인하는 데 필요한 범위까지만 분석한다.


## 22. Code Index 탐색 범위

Code Index MCP를 사용하는 경우
현재 Action의 호출 흐름을 기준으로 탐색한다.

권장 흐름:

Handler Symbol

→ Handler Source

→ 직접 호출 Function

→ 관련 Symbol

→ Reference

→ 다음 Function

→ API 호출 위치

전체 Repository를 대상으로
관련 없는 Symbol을 반복적으로 탐색하지 않는다.

동일한 이름의 Function이 여러 개 발견되는 경우
다음 근거를 함께 사용한다.

- 현재 Source Path
- Import 관계
- Component 관계
- 실제 호출 위치
- Parameter
- 현재 Action의 실행 흐름

이름만으로 연결하지 않는다.


## 23. 순환 호출

호출 관계에 순환 구조가 존재하는 경우
동일한 Source를 무한 반복하여 추적하지 않는다.

예:

`functionA()`

→ `functionB()`

→ `functionA()`

순환 구조가 확인되면
순환 관계를 기록하고
이미 분석한 동일 경로를 반복 분석하지 않는다.


## 24. 호출 추적 종료 조건

다음 중 하나에 도달하면
해당 호출 경로의 추가 추적을 종료한다.

1. Backend API 호출 지점에 도달
2. Frontend Response 처리까지 확인
3. 현재 Action과 직접적인 관계가 없는 공통 처리로 진입
4. Framework / 외부 Library 내부 구현으로 진입
5. Source를 더 이상 확인할 수 없음
6. 동일한 호출 경로를 이미 분석함

Source를 확인할 수 없는 경우:

`확인되지 않음`

으로 기록한다.

추측하여 호출 관계를 연결하지 않는다.


## 25. Backend 경계

Backend API 호출 지점은 확인하지만
Backend 내부 구현으로 넘어가지 않는다.

허용:

Frontend Function

→ API Client

→ HTTP Method

→ URL

→ Request Parameter / Body

→ Frontend Response 처리

금지:

Backend URL

→ Controller 내부

→ Service

→ Mapper

→ MyBatis Mapper XML

→ SQL

→ Oracle

Backend 상세 분석은
현재 Rule의 범위가 아니다.


## 26. 실행 트리 표현

Frontend 호출 흐름은 가능한 경우
실제 실행 순서와 분기가 보이는
Tree 형태로 표현한다.

예:

`ACT-002 검색`

└─ Click
   └─ `handleSearch()`
      ├─ Validation
      │  ├─ 실패
      │  │  ├─ Message 표시
      │  │  └─ STOP
      │  │
      │  └─ 성공
      │     └─ 계속
      │
      ├─ `createSearchParameter()`
      │  ├─ State 값 조회
      │  ├─ 빈 값 변환
      │  └─ Request Parameter 생성
      │
      └─ `requestEquipmentList()`
         ├─ GET
         ├─ `/api/equipment`
         └─ Response
            └─ Grid State 갱신

실제 Source에서 확인되지 않은 단계를
Tree를 완성하기 위해 임의로 추가하지 않는다.


## 27. Source 근거 기록

호출 흐름의 주요 단계는
가능한 경우 실제 Source 위치를 근거로 기록한다.

Source 근거에는 가능한 경우
다음 정보를 포함한다.

- Source Path
- Component / Module
- Function / Method
- Line 또는 Line Range

예:

`handleSearch()`

- Source Path: `src/views/equipment/EquipmentSearch.vue`
- Function: `handleSearch`
- Line: `120-158`

또는 실행 Tree에서 간결하게 표현할 수 있다.

`handleSearch()`
→ `src/views/equipment/EquipmentSearch.vue:120-158`


### Line 기록 기준

가능한 경우 단일 Line보다
해당 로직을 확인할 수 있는 Line Range를 사용한다.

예:

`EquipmentSearch.vue:120-158`

Validation이나 조건 분기처럼
특정 코드 영역이 근거인 경우에는
해당 영역의 Line Range를 기록한다.

예:

`validateSearchCondition()`
→ `EquipmentSearch.vue:205-228`

Line Number만으로 Source를 식별하지 않는다.

다음 정보를 함께 사용한다.

Source Path
+ Function / Method
+ Line Range


### Line을 확인할 수 없는 경우

사용 중인 MCP 또는 Source 탐색 결과에서
정확한 Line을 확인할 수 없는 경우
Line Number를 추측하여 작성하지 않는다.

이 경우:

- Source Path는 확인된 값 기록
- Function / Method는 확인된 값 기록
- Line: `확인되지 않음`

으로 기록한다.


### Source 변경 고려

Line Number는 Source 변경에 따라
변경될 수 있다.

따라서 Line Number는
Source 위치를 빠르게 찾기 위한 보조 정보로 사용한다.

Source의 기본 식별 기준은:

`Source Path + Function / Method`

로 한다.


### Code Index MCP

Code Index MCP 결과는
관련 Source와 호출 위치를 찾기 위한
보조 수단으로 사용한다.

Code Index에서 Source 위치를 찾았더라도
가능한 경우 실제 Source Code를 확인한다.

최종 Business Logic 판단은
실제 Source Code를 기준으로 한다.


## 28. 분석 범위 제한

현재 선택된 Action의
실제 실행 흐름만 분석한다.

다음 이유만으로
분석 범위를 확대하지 않는다.

- 같은 파일에 존재한다.
- 같은 Component에 존재한다.
- 이름이 비슷하다.
- 같은 Service Module을 사용한다.
- Code Index 검색 결과에 나타났다.

현재 Action의 실행 경로와
직접 연결되어 있는 경우에만 분석한다.


## 29. 핵심 원칙

`Action`

→ `Handler`

→ 실제 Source 확인

→ 직접 호출 Function 확인

→ Validation 확인

→ 조건 / 분기 확인

→ Parameter / State 전달 추적

→ 다단계 Function 호출 추적

→ Backend API 호출 지점 확인

→ Frontend Response 처리 확인

→ STOP

호출 관계를 단순 나열하지 않는다.

실제 Source를 확인하여
각 단계의 역할과 실행 조건을 설명한다.

Backend 내부로 넘어가지 않는다.
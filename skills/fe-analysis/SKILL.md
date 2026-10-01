---
name: fe-analysis
description: SCREEN 문서에서 사용자가 선택한 Action을 기준으로 Frontend Source의 실제 실행 흐름을 분석하고 FE 기능 문서를 생성한다.
argument-hint: "<화면명> <Action ID>"
user-invocable: true
disable-model-invocation: true
---

# Frontend 기능 분석 Skill

## 1. 역할

이 Skill은 SCREEN 분석에서 식별된 Action 중
사용자가 선택한 하나의 Action을 기준으로
Frontend Source Code의 실제 실행 흐름을 분석한다.

분석 결과는 독립된 Frontend 기능 문서로 생성한다.

전체 화면이나 전체 Frontend Repository를
다시 분석하지 않는다.

현재 선택된 Action과 직접 관련된
Frontend 처리 흐름만 분석한다.


## 2. 입력

입력 형식:

`<화면명> <Action ID>`

예:

`equipment-search ACT-002`

입력값:

- 화면명: `equipment-search`
- Action ID: `ACT-002`

Action ID만으로
분석 대상을 판단하지 않는다.

반드시:

`화면명 + Action ID`

조합으로 분석 대상을 식별한다.


## 3. 적용 Rule

Frontend 분석을 시작하기 전에
다음 Rule을 적용한다.

- `.claude/rules/04-fe-analysis-scope.md`
- `.claude/rules/05-fe-call-tracing.md`

Rule과 Skill의 내용이 충돌하는 경우
Rule에 정의된 분석 범위와 금지 사항을 우선한다.


## 4. 전체 실행 절차

Frontend 분석은 다음 순서로 수행한다.

1. 입력 확인
2. SCREEN 문서 확인
3. Action 확인
4. 분석 시작 Source 확인
5. Frontend Source 확인
6. 호출 흐름 추적
7. Validation 분석
8. 조건 / 분기 분석
9. Parameter / 데이터 처리 분석
10. State 처리 분석
11. Backend API 호출 식별
12. Response 처리 분석
13. 예외 처리 분석
14. Source Evidence 정리
15. Backend API 후보 정리
16. Frontend 문서 생성
17. STOP

각 단계는 현재 Action과
직접 관련된 범위에서만 수행한다.


## 5. STEP 1 — 입력 확인

사용자가 입력한 다음 값을 확인한다.

- 화면명
- Action ID

예:

`equipment-search ACT-002`

화면명 또는 Action ID가 없는 경우
임의로 추측하여 분석하지 않는다.

필요한 입력을 사용자에게 요청한다.


## 6. STEP 2 — SCREEN 문서 확인

화면명을 기준으로
다음 SCREEN 문서를 확인한다.

`docs/analysis/{화면명}/SCREEN-{화면명}.md`

예:

`docs/analysis/equipment-search/SCREEN-equipment-search.md`

SCREEN 문서가 존재하지 않는 경우
다른 화면 문서를 임의로 사용하지 않는다.

분석을 중단하고
SCREEN 문서를 확인할 수 없음을 알린다.


## 7. STEP 3 — Action 확인

SCREEN 문서에서
사용자가 입력한 Action ID를 찾는다.

예:

`ACT-002`

해당 Action에서 가능한 경우
다음 정보를 확인한다.

- Action ID
- 기능명
- UI 유형
- Event
- Handler
- Source Path
- Popup / Navigation 관련 정보

이 정보는 Frontend 분석의
초기 탐색 기준으로 사용한다.

Action ID가 SCREEN 문서에 존재하지 않는 경우
비슷한 Action을 임의로 선택하지 않는다.

분석을 중단하고
해당 Action을 찾을 수 없음을 알린다.


## 8. STEP 4 — 분석 시작 Source 결정

SCREEN 문서에 기록된
Handler와 Source Path를 우선 사용한다.

예:

Action:

`ACT-002 검색`

Event:

`Click`

Handler:

`handleSearch`

Source Path:

`src/views/equipment/EquipmentSearch.vue`

분석 시작점:

`EquipmentSearch.vue`

→ `handleSearch()`


### Handler / Source Path가 확인되지 않은 경우

SCREEN 문서에 Handler 또는 Source Path가
`확인되지 않음`으로 기록되어 있다면
확인된 SCREEN 정보를 이용하여
Source를 탐색할 수 있다.

사용 가능한 정보 예:

- Frontend URL
- Route
- Page Component
- 화면의 고유 Text
- UI Element
- Event
- Component 이름
- Import 관계

Code Index MCP를 이용하여
관련 Source를 좁힐 수 있다.

Source를 확인하지 못하면
추측하여 연결하지 않는다.


## 9. STEP 5 — Frontend Source 확인

선택된 Action의 실제 Source Code를 확인한다.

Source를 확인할 때는
현재 Action의 실행 흐름에 필요한 범위만 읽는다.

전체 파일 또는 전체 Repository를
무조건 확장하여 분석하지 않는다.

현재 Action과 직접 관련된 다음 Source를
필요에 따라 확인할 수 있다.

- Page Component
- Child Component
- Parent Component
- Handler
- 관련 Function / Method
- Custom Hook
- Composable
- Store
- Service Module
- API Module
- Utility
- API Client / Wrapper

현재 Action과 직접적인 호출 관계가 없는
Source는 분석 범위에서 제외한다.


## 10. STEP 6 — Code Index MCP 사용

Code Index MCP는
현재 Action과 관련된 Source를 찾고
호출 범위를 좁히기 위해 사용한다.

가능한 사용 목적:

- Handler Symbol 탐색
- Source Path 확인
- Reference 탐색
- Caller 확인
- Callee 확인
- 관련 Function 확인
- Component 관계 확인
- Import 관계 확인
- API 호출 위치 탐색

권장 탐색 흐름:

`Handler`

→ Handler Source

→ 직접 호출 Function

→ 관련 Symbol / Reference

→ 다음 Function

→ API 호출 위치


### 중요

Code Index 결과는
Source 탐색을 위한 근거이다.

Code Index의 Symbol 이름이나
Caller / Callee 관계만 보고
Business Logic을 확정하지 않는다.

실제 Source Code를 확인하여
처리 내용을 해석한다.


## 11. STEP 7 — 호출 흐름 추적

Handler에서 시작하여
현재 Action의 실제 호출 흐름을 추적한다.

예:

`ACT-002 검색`

→ `Click`

→ `handleSearch()`

→ `validateSearchCondition()`

→ `createSearchParameter()`

→ `searchEquipment()`

→ `equipmentStore.search()`

→ `equipmentService.getList()`

→ Backend API


중간 Function이 다음 항목에 영향을 준다면
생략하지 않는다.

- Validation
- 조건 / 분기
- Parameter
- 데이터 변환
- State
- Backend 호출
- Response 처리
- Error 처리

단순히:

`handleSearch()`

→ Backend API

형태로 축약하지 않는다.


## 12. STEP 8 — Validation 분석

현재 Action에서 수행되는
Validation을 확인한다.

가능한 경우 다음을 기록한다.

- Validation 대상
- 입력 값
- 조건
- 성공 조건
- 실패 조건
- 실패 시 처리
- 사용자 Message
- Early Return 여부
- 이후 실행 여부

Validation 분기는
실제 실행 흐름이 보이도록 표현한다.

예:

```text
Validation
├─ 실패
│  ├─ Message 표시
│  ├─ return
│  └─ STOP
│
└─ 성공
   └─ 다음 처리
```

Validation이 Source에 존재하지 않는 경우
임의로 생성하지 않는다.


## 13. STEP 9 — 조건 / 분기 분석

현재 Action 실행 과정의
조건과 분기를 확인한다.

대상 예:

- `if`
- `else`
- `else if`
- `switch`
- 삼항 연산자
- Early Return
- 조건부 Function 호출
- 조건부 API 호출

각 조건에 대해 가능한 경우
다음을 확인한다.

- 조건식
- 사용되는 값
- TRUE 경로
- FALSE 경로
- 다음 Function
- API 호출 여부
- State 변경 여부

중첩 조건이 존재하는 경우
중첩 구조를 유지한다.

조건을 단순히:

`조건 처리`

라고 요약하지 않는다.


## 14. STEP 10 — Parameter / 데이터 처리 분석

Function 또는 Backend API에 전달되는
데이터 생성 과정을 확인한다.

가능한 경우 다음을 구분한다.

- 화면 입력 값
- Source State
- Function 입력 값
- 기본값
- 중간 변환 값
- Parameter 생성 과정
- 최종 전달 값

예:

```text
화면 입력
"전체"

↓
Source State
plantCode = ""

↓
Parameter 생성

↓
Request Parameter
plantCode = null
```

단, 실제 Source에서
이러한 변환이 확인된 경우에만 기록한다.

확인되지 않은 변환을
임의로 생성하지 않는다.


## 15. STEP 11 — State 처리 분석

현재 Action과 직접 관련된
Frontend State 사용 및 변경을 확인한다.

가능한 경우 다음을 기록한다.

- State 이름
- 값의 Source
- 변경 조건
- 변경 값
- 변경 위치
- 이후 사용 위치
- 화면에 미치는 영향

예:

```text
Backend Response
↓
equipmentList
↓
State 저장
↓
Grid Data 갱신
```

현재 Action과 관계없는
Component 전체 State는 분석하지 않는다.


## 16. STEP 12 — Backend API 호출 식별

현재 Action에서 발생하는
Backend API 호출을 확인한다.

각 API 호출에 대해
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

```text
equipmentService.getList()
↓
GET /api/equipment
```

Backend API가 발견되어도
Controller 내부로 내려가지 않는다.


## 17. STEP 13 — 여러 API 호출 분석

하나의 Action에서
여러 Backend API가 호출되는 경우
모든 호출을 기록한다.

각 API의 다음 관계를 확인한다.

- 독립 호출
- 조건부 호출
- 순차 호출
- 병렬 호출
- 이전 API Response 의존 호출


### 순차 호출 예

```text
API A
↓
Response에서 equipmentId 획득
↓
API B
```


### 병렬 호출 예

```text
Promise.all()
├─ API A
└─ API B
```


### 조건부 호출 예

```text
includeStatus
├─ true
│  └─ Status API 호출
│
└─ false
   └─ Status API 호출 안 함
```

실제 Source에서 확인되지 않은 실행 관계를
임의로 판단하지 않는다.


## 18. STEP 14 — Response 처리 분석

Backend Response가
Frontend에서 어떻게 사용되는지 확인한다.

가능한 경우 다음을 기록한다.

- Response 수신 위치
- 성공 여부 판단
- Response Data 접근
- 데이터 변환
- State 반영
- Grid 반영
- Form 반영
- Component 반영
- 사용자 Message
- 후속 Function 호출

Backend Response가
Frontend에서 다시 가공되는 경우
가공 과정을 확인한다.

현재 단계에서는
API의 전체 Response 규격을 확정하지 않는다.


## 19. STEP 15 — 예외 처리 분석

현재 Action과 직접 관련된
Frontend 예외 처리를 확인한다.

대상 예:

- `try / catch`
- Promise `.catch()`
- API Error 처리
- Error Callback
- 공통 Error Handler 호출

가능한 경우 다음을 기록한다.

- 오류 발생 위치
- 오류 처리 Function
- 사용자 Message
- State 초기화
- Loading 상태 처리
- 후속 처리 중단 여부

공통 Error Handler 전체 구현은
현재 Action에 필요한 범위를 넘어
확장 분석하지 않는다.


## 20. STEP 16 — Source Evidence 기록

분석한 주요 실행 단계에는
가능한 경우 Source Evidence를 기록한다.

기본 Evidence:

- Source Path
- Component / Module
- Function / Method
- Line 또는 Line Range

예:

```text
Source Path:
src/views/equipment/EquipmentSearch.vue

Function:
handleSearch()

Line:
120-158
```

실행 Tree에서는
다음처럼 간결하게 표현할 수 있다.

```text
handleSearch()
[EquipmentSearch.vue:120-158]
```


### Line 기준

가능한 경우 단일 Line보다
해당 로직을 확인할 수 있는
Line Range를 사용한다.

예:

```text
validateSearchCondition()
[EquipmentSearch.vue:205-228]
```

Line Number는
Source 변경에 따라 달라질 수 있으므로
단독 식별 기준으로 사용하지 않는다.

기본 Source 식별 기준:

`Source Path + Function / Method`

Line Range는
개발자가 Source를 빠르게 찾기 위한
보조 정보로 사용한다.


### Line을 확인할 수 없는 경우

Line을 정확하게 확인할 수 없다면
추측하지 않는다.

다음처럼 기록한다.

`Line: 확인되지 않음`


## 21. STEP 17 — Backend API 후보 정리

Frontend 분석에서 발견된 Backend API를
다음 단계의 분석 후보로 정리한다.

API 후보마다 가능한 경우
다음을 기록한다.

- 후보 번호
- 호출 목적
- HTTP Method
- URL
- 호출 Function
- 호출 조건
- 호출 순서 / 관계

예:

```text
Backend API 후보 1

목적:
설비 목록 조회

Method:
GET

URL:
/api/equipment

호출 Function:
equipmentService.getList()

호출 조건:
검색 Validation 성공 후 호출
```

이 단계에서는
`API-001` 같은 최종 API ID를
임의로 확정하지 않는다.

API ID 부여는
API 규격 분석 단계의 규칙에 따른다.


## 22. Runtime 확인

Frontend Source만으로
실제 Runtime 동작 확인이 필요한 경우
Chrome DevTools MCP를 사용할 수 있다.

사용 목적 예:

- 실제 Request 확인
- HTTP Method 확인
- 실제 호출 URL 확인
- 실제 Query Parameter 확인
- 실제 Request Body 확인
- 호출 순서 확인
- 실제 Response 확인

Runtime 확인은
현재 Action 분석에 필요한 경우에만 수행한다.


### Runtime 값 주의

Runtime에서 관찰한 값은
해당 실행 시점의 실제 값이다.

예:

```text
plantCode=1000
```

이 값이 관찰되었다고 해서:

`plantCode는 항상 1000이다`

라고 해석하지 않는다.

Runtime 관찰 값과
Source Code에 정의된 처리 규칙을 구분한다.


## 23. 안전한 Runtime 확인

Runtime 확인 과정에서
업무 데이터를 변경할 수 있는 Action은
사용자의 명시적인 허가 없이 실행하지 않는다.

예:

- 저장
- 등록
- 수정
- 삭제
- 승인
- 반려
- 전송
- 확정
- 업무 취소

조회처럼 안전한 동작만
필요한 범위에서 실행한다.

위험 여부가 불명확하면
실행하지 않는다.


## 24. 공통 모듈 추적 제한

현재 Action에서
공통 Utility, Store, Hook, Composable,
API Client 등이 호출될 수 있다.

현재 Action의 다음 항목에
직접 영향을 주는 범위까지만 분석한다.

- Validation
- 조건
- Parameter
- 데이터 변환
- State
- Backend 호출
- Response
- Error 처리

공통 모듈 전체를 분석하지 않는다.


## 25. Framework / 외부 Library 경계

현재 프로젝트 Source를 벗어나
Framework 또는 외부 Library 내부 구현으로
불필요하게 들어가지 않는다.

일반적으로 다음 내부 구현은 분석하지 않는다.

- React 내부 구현
- Vue 내부 구현
- Router Library 내부 구현
- Axios 내부 구현
- Fetch 내부 구현
- `node_modules` Library 내부 구현

현재 프로젝트의 Business Flow를
확인하는 데 필요한 경계까지만 추적한다.


## 26. Backend 분석 경계

Frontend 분석의 최대 경계는:

```text
Action
↓
Handler
↓
Frontend Function
↓
Validation
↓
조건 / 분기
↓
Parameter / State
↓
Frontend Service / API Module
↓
Backend API 호출
↓
Frontend Response 처리
```

여기까지이다.

다음 영역은 분석하지 않는다.

- Backend Controller 내부
- Backend Service
- ServiceImpl
- Backend Business Logic
- Mapper
- MyBatis Mapper XML
- Dynamic SQL
- SQL
- Oracle Table / Column
- SAP 내부 처리

Backend Source가 검색 결과에 나타나더라도
현재 단계에서는 상세 분석하지 않는다.


## 27. 실행 Tree 작성

Frontend의 실제 실행 흐름을
가능한 경우 Tree 구조로 작성한다.

단순 호출 관계만 나열하지 않는다.

예:

```text
ACT-002 검색
└─ Click
   └─ handleSearch()
      [EquipmentSearch.vue:120-158]
      │
      ├─ Validation
      │  └─ validateSearchCondition()
      │     [EquipmentSearch.vue:205-228]
      │
      │     ├─ 실패
      │     │  ├─ Message 표시
      │     │  ├─ return
      │     │  └─ STOP
      │     │
      │     └─ 성공
      │        └─ 다음 처리
      │
      ├─ Parameter 생성
      │  └─ createSearchParameter()
      │     [searchParameter.ts:45-72]
      │
      │     ├─ State 값 조회
      │     ├─ 값 변환
      │     └─ Request Parameter 생성
      │
      └─ Backend 호출
         └─ equipmentService.getList()
            [equipmentService.ts:31-44]
            │
            ├─ GET
            ├─ /api/equipment
            │
            └─ Response
               └─ equipmentList State
                  └─ Grid 갱신
```

실제 Source에서 확인되지 않은 단계를
Tree 형태를 완성하기 위해
임의로 추가하지 않는다.


## 28. 결과 문서 구조

Frontend 분석 문서는
기본적으로 다음 구조를 사용한다.

### 1. 기능 정보

- 화면명
- Action ID
- 기능명
- Event
- Handler
- 최초 Source Path


### 2. 기능 요약

현재 Action이 수행하는
Frontend 기능을 설명한다.


### 3. 전체 실행 Tree

Handler부터 Backend 호출,
Frontend Response 처리까지
전체 실행 흐름을 Tree로 표현한다.


### 4. 입력 및 Source State

- 화면 입력
- State
- 기본값
- Function 입력


### 5. Validation

- Validation 조건
- 성공 / 실패
- Message
- Early Return


### 6. 조건 / 분기

- 조건식
- TRUE / FALSE
- 중첩 분기
- 실행 경로 변화


### 7. Parameter / 데이터 처리

- Source 값
- 변환 과정
- 최종 전달 값


### 8. State 처리

- State 사용
- State 변경
- 화면 반영


### 9. Backend API 호출

- 호출 목적
- Function
- Method
- URL
- Query Parameter
- Path Parameter
- Request Body
- 호출 조건
- 호출 순서


### 10. Response 처리

- Response 수신
- 데이터 가공
- State 반영
- 화면 갱신


### 11. 예외 처리

- Error 처리
- Message
- State 처리
- 실행 중단


### 12. Source Evidence

주요 처리 단계의:

- Source Path
- Function / Method
- Line Range


### 13. Backend API 후보

다음 API 규격 분석에서
사용자가 선택할 수 있는
Backend API 후보를 정리한다.


### 14. 미확인 항목

Source 또는 Runtime에서
확인하지 못한 내용을 기록한다.


### 15. 분석 경계

이번 Frontend 분석에서
확인한 범위와
분석하지 않은 Backend 영역을 기록한다.


## 29. 문서 저장

Frontend 분석 결과는
다음 위치에 저장한다.

`docs/analysis/{화면명}/frontend/`

파일명:

`FE-{Action ID}-{기능명}.md`

예:

`docs/analysis/equipment-search/frontend/FE-ACT-002-search.md`

기능명은 파일명에 사용할 수 있도록
일관된 형태로 사용한다.

필요한 디렉터리가 존재하지 않는 경우
생성한다.

기존 동일 이름의 문서가 존재하는 경우
임의로 덮어쓰지 않는다.


## 30. 기존 문서 처리

동일 Action의 기존 FE 문서가 존재하면
자동으로 덮어쓰지 않는다.

기존 문서가 발견되면
사용자에게 알려준다.

사용자가 명시적으로
재분석 또는 덮어쓰기를 요청한 경우에만
기존 문서를 변경한다.


## 31. 미확인 정보 처리

확인되지 않은 정보를
추측하여 채우지 않는다.

확인할 수 없는 항목은:

`확인되지 않음`

으로 기록한다.

특히 다음 내용을 추측하지 않는다.

- Handler
- Source Path
- Line Number
- Validation
- Parameter 변환
- API Method
- API URL
- Request 값
- Response 구조
- 호출 순서
- 조건 / 분기


## 32. 분석 완료 조건

다음 항목을 확인한 후
Frontend 분석을 완료한다.

- 선택한 Action이 정확한가
- SCREEN 문서와 연결되는가
- Handler를 확인했는가
- Source Path를 확인했는가
- 실제 Source를 확인했는가
- 주요 Function 호출을 추적했는가
- Validation을 확인했는가
- 조건 / 분기를 확인했는가
- Parameter / 데이터 처리를 확인했는가
- State 처리를 확인했는가
- Backend API 호출을 확인했는가
- Response 처리를 확인했는가
- 예외 처리를 확인했는가
- 주요 Source Evidence를 기록했는가
- Backend API 후보를 정리했는가
- 미확인 항목을 구분했는가
- Backend 내부로 분석 범위를 넘지 않았는가


## 33. STOP 규칙

Frontend 분석 문서를 생성한 후
반드시 종료한다.

자동으로 다음 작업을 수행하지 않는다.

- API 규격 분석
- API ID 확정
- Backend Controller 상세 분석
- Backend Service 분석
- Mapper 분석
- MyBatis XML 분석
- SQL 분석
- Oracle 분석
- SAP 내부 분석

Backend API가 발견되더라도
다음 분석 후보로만 기록한다.

다음 분석 대상은
사용자가 직접 선택한다.


## 34. 완료 보고

분석 완료 후 사용자에게
간단하게 다음 내용을 알려준다.

- 분석한 화면명
- 분석한 Action ID
- 생성한 FE 문서 경로
- 발견한 Backend API 수
- Backend API 후보
- 미확인 항목 존재 여부

상세 분석 내용 전체를
대화에 다시 반복 출력하지 않는다.

상세 내용은 생성된 FE 문서를 기준으로 한다.


## 35. 핵심 실행 원칙

```text
화면명 + Action ID
↓
SCREEN 문서 확인
↓
Action 확인
↓
Handler / Source Path 확인
↓
실제 Frontend Source 확인
↓
호출 관계 추적
↓
Validation
↓
조건 / 분기
↓
Parameter / 데이터 변환
↓
State
↓
Backend API 호출
↓
Frontend Response / Error 처리
↓
Source Path + Function + Line Range Evidence
↓
Backend API 후보 정리
↓
FE 문서 생성
↓
STOP
```

현재 Action과 직접 관련된 Source만 분석한다.

Code Index는 Source 탐색과
호출 관계 확인에 사용한다.

Business Logic은
실제 Source Code를 확인하여 해석한다.

확인되지 않은 내용은 추측하지 않는다.

Backend 내부 구현으로 넘어가지 않는다.
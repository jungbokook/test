# Frontend 기능 분석 Reference

## 1. 목적

이 문서는 Frontend 기능 분석 결과 문서의
표현 형식과 상세 수준을 정의하기 위한 Reference이다.

이 문서는 분석 Evidence가 아니다.

실제 분석 결과는 반드시 다음 정보를 기준으로 작성한다.

- SCREEN 분석 문서
- 실제 Frontend Source Code
- Code Index MCP 탐색 결과
- 필요한 경우 Chrome DevTools MCP Runtime 정보

Reference에 포함된 예시의
화면명, Action ID, Handler, Source Path,
Line Number, Parameter, API URL, Request / Response 값 등을
실제 분석 결과로 사용하지 않는다.


---

# 2. 작성 기본 원칙

Frontend 분석 문서는 다음 두 가지 목적을 동시에 만족해야 한다.

1. 개발자가 기능 전체 흐름을 빠르게 파악할 수 있어야 한다.
2. 필요한 경우 실제 Source까지 상세하게 추적할 수 있어야 한다.

따라서 문서는 다음 구조를 사용한다.

```text
요약 영역
↓
한눈에 보는 기능 흐름
↓
전체 실행 Tree
↓
상세 분석
↓
Source Evidence
```

상단에서는 전체 흐름을 빠르게 이해할 수 있도록 표현하고,
하단에서는 실제 Source 기반 상세 정보를 제공한다.

상세 분석을 줄여서
가독성을 확보하지 않는다.

분석은 상세하게 수행하되
표현을 계층화하여 가독성을 확보한다.


---

# 3. 권장 문서 구조

Frontend 분석 결과 문서는
다음 구조를 기본으로 한다.

```text
1. 기능 정보
2. 기능 요약
3. 한눈에 보는 기능 흐름
4. 전체 실행 Tree
5. 입력 / 출력 요약
6. 상세 분석
7. Source Evidence
8. Backend API 후보
9. 미확인 항목
10. 분석 경계
```


---

# 4. 기능 정보

현재 분석 대상의 기본 정보를
간단한 표로 표현한다.

예:

| 항목 | 내용 |
|---|---|
| 화면명 | equipment-search |
| Action ID | ACT-002 |
| 기능명 | 검색 |
| UI 유형 | Button |
| Event | Click |
| Handler | `handleSearch()` |
| 최초 Source | `src/views/equipment/EquipmentSearch.vue` |

확인되지 않은 항목은
임의로 채우지 않는다.

`확인되지 않음`

으로 표시한다.


---

# 5. 기능 요약

현재 Action이 수행하는 기능을
짧게 설명한다.

세부 Source 구현을 모두 나열하지 않는다.

예:

> 사용자가 입력한 검색 조건을 검증한 후 검색 Parameter를 생성하고,
> 설비 조회 API를 호출한다.
> 조회 결과는 목록 State에 저장되어 Grid에 반영된다.

기능 요약만 읽어도
현재 Action의 목적을 이해할 수 있어야 한다.


---

# 6. 한눈에 보는 기능 흐름

이 Section은 Frontend 분석 문서에서
가장 빠르게 전체 기능을 파악하기 위한 영역이다.

상세 Source 내용을 모두 펼치지 않는다.

기본적으로 다음 흐름을 표현한다.

```text
Action
↓
Handler
↓
Validation
↓
조건 / 분기
↓
Parameter 생성
↓
Backend API
↓
Response
↓
State
↓
화면 반영
```

예:

```text
[ACT-002 검색]
       │
       ▼
[handleSearch()]
       │
       ▼
[Validation]
 검색조건 검증
       │
       ├─ 실패 → Message → STOP
       │
       └─ 통과
              │
              ▼
[Parameter 생성]
 검색조건 → Request Parameter
              │
              ▼
[Backend API]
 GET /api/equipment
              │
              ▼
[Response]
 items / totalCount
              │
              ▼
[State]
 equipmentList
              │
              ▼
[화면]
 Grid 갱신

[ERROR]
 API 오류
   └─ Message → State 초기화 → 종료
```


---

# 7. 한눈에 보기 — 대량 항목 표현 규칙

Validation, Parameter, 조건 / 분기가 많은 경우
상단 Flow에 모든 항목을 펼치지 않는다.

다음 형식을 사용한다.

```text
총 개수
+
업무 기준 그룹
+
그룹별 개수
```


## Validation 예

Validation이 12개라면:

```text
[Validation: 총 12개]

├─ 기본조건      4개
├─ 기간조건      3개
├─ 설비조건      3개
└─ 기타          2개

실패
→ Message
→ STOP

전체 통과
→ 다음 단계
```

12개의 Validation 내용을
상단 Flow에 전부 표시하지 않는다.

전체 Validation 상세 내용은
상세 분석 Section에서 제공한다.


## Parameter 예

Parameter가 24개라면:

```text
[Parameter: 총 24개]

├─ 화면 입력      11개
├─ State            5개
├─ 자동 생성        3개
└─ 변환 / 가공      5개
```

상단에서는 전체 구조만 보여준다.

24개의 실제 Parameter는
상세 Parameter 표에서 모두 제공한다.


## 조건 / 분기 예

조건이 많은 경우
업무적으로 중요한 상위 분기만 표시한다.

```text
[Business 조건: 총 18개]

├─ 검색 유형
│  ├─ 일반
│  └─ 상세
│
├─ 사용자 권한
│  ├─ 관리자
│  └─ 일반 사용자
│
└─ 기타 조건 14개
   └─ 상세 분석 참조
```

실행 결과를 크게 변경하는
핵심 조건은 상단에서도 표시한다.


---

# 8. 한눈에 보기 깊이 제한

한눈에 보는 기능 흐름은
가능한 경우 최대 2~3단계 깊이로 표현한다.

예:

```text
Validation
├─ 실패 → STOP
└─ 성공 → 계속
```

상단 Flow에서 다음과 같이
지나치게 깊게 확장하지 않는다.

```text
Validation
└─ 조건 A
   └─ 조건 B
      └─ 조건 C
         └─ 조건 D
            └─ ...
```

복잡한 중첩 조건은
상세 실행 Tree에서 표현한다.


---

# 9. 핵심 정보 요약

한눈에 보는 기능 흐름 아래에는
현재 Action의 핵심 정보를 표로 정리할 수 있다.

예:

| 구분 | 핵심 내용 | Source |
|---|---|---|
| Handler | `handleSearch()` | `EquipmentSearch.vue:120-158` |
| Validation | 총 12개 | `EquipmentSearch.vue:205-260` |
| Parameter | 총 24개 | `searchParameter.ts:45-110` |
| API | `GET /api/equipment` | `equipmentService.ts:31-44` |
| Response | `items → equipmentList` | 관련 Source |
| 화면 반영 | `equipmentList → Grid` | 관련 Component |
| Error | Message + State 초기화 | 관련 `catch` |

확인되지 않은 Source 또는 Line은
추측하지 않는다.


---

# 10. 전체 실행 Tree

한눈에 보는 기능 흐름과 달리
전체 실행 Tree는 실제 Frontend 실행 구조를 상세하게 표현한다.

중간 Function을 임의로 생략하지 않는다.

예:

```text
ACT-002 검색
└─ Click
   └─ handleSearch()
      [EquipmentSearch.vue:120-158]
      │
      ├─ searchKeyword State 조회
      │
      ├─ validateSearchCondition()
      │  [EquipmentSearch.vue:205-228]
      │  │
      │  ├─ 실패
      │  │  ├─ Message 표시
      │  │  ├─ false 반환
      │  │  ├─ return
      │  │  └─ STOP
      │  │
      │  └─ 성공
      │     └─ true 반환
      │
      ├─ createSearchParameter()
      │  [searchParameter.ts:45-72]
      │  │
      │  ├─ 화면 / State 값 조회
      │  ├─ 값 변환
      │  └─ Request Parameter 생성
      │
      └─ searchEquipment()
         [EquipmentSearch.vue:160-183]
         │
         └─ equipmentStore.search()
            [equipmentStore.ts:80-115]
            │
            └─ equipmentService.getList()
               [equipmentService.ts:31-44]
               │
               ├─ GET
               ├─ /api/equipment
               │
               ├─ Response
               │  └─ equipmentList State
               │     └─ Grid 갱신
               │
               └─ ERROR
                  ├─ Message
                  └─ State 초기화
```

실제 Source에서 확인되지 않은 단계를
Tree를 완성하기 위해 임의로 추가하지 않는다.


---

# 11. 입력 / 출력 요약

현재 Action의 시작 입력과
최종 결과를 간단하게 정리한다.

예:

| 구분 | 내용 |
|---|---|
| 화면 입력 | 검색어, 사업장, 설비유형 |
| Source State | `searchKeyword`, `plantCode`, `equipmentType` |
| Backend Request | 검색조건 Parameter |
| Backend Response | 설비 목록 |
| Frontend State | `equipmentList` |
| 화면 결과 | Grid 갱신 |

항목이 많다면
전체 내용을 이 표에 넣지 않는다.

상세 내용은 아래 상세 분석에서 제공한다.


---

# 12. 상세 분석

상세 분석은 데이터 종류만 기준으로
기계적으로 분리하지 않는다.

가능한 경우
실제 실행 순서 또는 업무 처리 단위로 구성한다.

예:

```text
6.1 검색 조건 검증
6.2 검색 Parameter 생성
6.3 검색 API 호출
6.4 Response 처리
6.5 화면 갱신
6.6 예외 처리
```

현재 기능에 존재하지 않는 Section은
억지로 생성하지 않는다.

동일한 내용을 여러 Section에서
반복하여 설명하지 않는다.


---

# 13. Validation 상세

Validation이 존재하면
전체 Validation을 누락 없이 기록한다.

Validation이 많은 경우
표를 우선 사용한다.

예:

| ID | 검증 대상 | 조건 | 실패 처리 | Source |
|---|---|---|---|---|
| V-01 | 사업장 | 미선택 | Message + STOP | `File.vue:120-125` |
| V-02 | 시작일 | 값 없음 | Message + STOP | `File.vue:127-132` |
| V-03 | 종료일 | 값 없음 | Message + STOP | `File.vue:134-139` |
| V-04 | 조회기간 | 시작일 > 종료일 | Message + STOP | `File.vue:141-148` |

Validation ID는
문서 내 가독성을 위한 식별자이며
실제 Source에 존재하는 ID처럼 표현하지 않는다.

Validation의 실행 순서가 중요한 경우
실제 순서를 유지한다.


---

# 14. 조건 / 분기 상세

Business Logic에 영향을 주는
조건 / 분기를 기록한다.

예:

```text
검색유형 확인
│
├─ NORMAL
│  └─ 일반 검색 Parameter 생성
│
└─ DETAIL
   ├─ 상세조건 확인
   └─ 상세 검색 Parameter 생성
```

가능한 경우 다음을 함께 기록한다.

- 조건식
- Source 값
- TRUE 처리
- FALSE 처리
- 다음 Function
- API 호출 변화
- State 변화
- Early Return

조건을 단순히
`조건 처리`

라고 표현하지 않는다.


---

# 15. Parameter 상세

Parameter가 적은 경우에는
값 흐름을 직접 표현할 수 있다.

예:

```text
화면 값
" ABC "

↓ trim()

Source 값
"ABC"

↓ Request 생성

keyword = "ABC"
```


Parameter가 많은 경우에는
표를 사용한다.

예:

| Parameter | Source | 원본 값 | 변환 | 최종 전달 | 비고 |
|---|---|---|---|---|---|
| `keyword` | 화면 입력 | `" ABC "` | `trim()` | `"ABC"` | 전송 |
| `searchType` | State | `"ALL"` | `ALL → null` | 미전송 | 조건부 |
| `plantCode` | State | `"1000"` | 없음 | `"1000"` | 전송 |
| `startDate` | 화면 입력 | 날짜 값 | Request 형식 변환 | 변환값 | 전송 |

가능한 경우 다음을 구분한다.

- 화면 표시 값
- Source State
- Function 입력
- 중간 변환
- 최종 Request 값

Runtime에서 관찰한 실제 값과
Source에 정의된 일반 규칙을 구분한다.


---

# 16. State 상세

현재 Action과 직접 관련된
State만 기록한다.

예:

| State | Source | 변경 조건 | 변경 결과 | 사용 위치 |
|---|---|---|---|---|
| `equipmentList` | API Response | 조회 성공 | `response.items` | Grid |
| `loading` | 검색 실행 | API 시작/종료 | `true/false` | Loading UI |

현재 Action과 관계없는
Component 전체 State를 나열하지 않는다.


---

# 17. Backend API 호출 상세

Backend API 호출은
호출 단위로 구분한다.

예:

### API 호출 1 — 설비 목록 조회

| 항목 | 내용 |
|---|---|
| 호출 Function | `equipmentService.getList()` |
| Method | `GET` |
| URL | `/api/equipment` |
| 호출 조건 | Validation 전체 통과 |
| 호출 방식 | 단일 호출 |
| Source | `equipmentService.ts:31-44` |

Request Parameter:

| Parameter | 값 / Source | 전송 여부 |
|---|---|---|
| `keyword` | `"ABC"` | 전송 |
| `searchType` | `null` | 미전송 |
| `plantCode` | `"1000"` | 전송 |

API 내부 Backend 구현은
이 문서에서 분석하지 않는다.


---

# 18. 여러 Backend API

하나의 Action에서
여러 API가 호출되는 경우
관계를 명확하게 표현한다.


## 순차 호출

```text
API A
↓
Response A
↓
equipmentId 획득
↓
API B
```


## 병렬 호출

```text
Promise.all()
├─ API A
└─ API B
    ↓
전체 Response 완료
```


## 조건부 호출

```text
includeStatus
├─ true
│  └─ Status API 호출
│
└─ false
   └─ Status API 호출 안 함
```

실제 Source에서 확인되지 않은
호출 관계를 추측하지 않는다.


---

# 19. Response 및 화면 반영

Response가 화면까지 전달되는 흐름을
가능한 경우 하나의 흐름으로 표현한다.

예:

```text
GET /api/equipment
↓
Response
↓
response.items
↓
equipmentList State
↓
Grid Data
↓
화면 갱신
```

중간 데이터 변환이 존재하면
생략하지 않는다.


---

# 20. 예외 처리

현재 Action과 직접 관련된
예외 처리만 기록한다.

예:

```text
API 호출
│
├─ SUCCESS
│  └─ Response 처리
│
└─ ERROR
   └─ catch
      ├─ 오류 Message
      ├─ equipmentList = []
      └─ 처리 종료
```

공통 Error Handler가 존재하는 경우
현재 Action과 관련된 호출 범위까지만 기록한다.


---

# 21. Source Evidence

주요 처리 단계의 Source를
한 번에 찾을 수 있도록 표로 정리한다.

예:

| 처리 단계 | Function / Method | Source Path | Line |
|---|---|---|---|
| 검색 시작 | `handleSearch()` | `src/views/equipment/EquipmentSearch.vue` | 120-158 |
| Validation | `validateSearchCondition()` | `src/views/equipment/EquipmentSearch.vue` | 205-228 |
| Parameter 생성 | `createSearchParameter()` | `src/views/equipment/searchParameter.ts` | 45-72 |
| Store | `search()` | `src/stores/equipmentStore.ts` | 80-115 |
| API 호출 | `getList()` | `src/api/equipmentService.ts` | 31-44 |

가능한 경우:

`Source Path + Function / Method + Line Range`

를 함께 제공한다.

Line Number를 확인할 수 없다면
추측하지 않는다.

`확인되지 않음`

으로 기록한다.


---

# 22. Backend API 후보

Frontend 분석에서 발견한 API를
다음 분석 단계의 후보로 정리한다.

예:

| 후보 | 목적 | Method | URL | 호출 조건 |
|---|---|---|---|---|
| 1 | 설비 목록 조회 | GET | `/api/equipment` | Validation 성공 |
| 2 | 설비 상태 조회 | GET | `/api/equipment/status` | `includeStatus=true` |

Frontend 분석 단계에서는
`API-001`, `API-002` 등의 최종 API ID를
임의로 부여하지 않는다.

API ID는
API 규격 분석 단계에서 부여한다.


---

# 23. 미확인 항목

현재 분석에서 확인하지 못한 내용을
명확하게 분리한다.

예:

| 항목 | 상태 | 사유 |
|---|---|---|
| 특정 Parameter Source | 확인되지 않음 | 관련 Source 확인 불가 |
| 특정 Line Range | 확인되지 않음 | 위치 확인 불가 |

미확인 정보를
추측하여 문서를 완성하지 않는다.


---

# 24. 분석 경계

Frontend 분석에서 확인한 범위를 기록한다.

예:

```text
분석 범위

ACT-002
↓
Handler
↓
Validation
↓
Frontend Business Logic
↓
Parameter
↓
Store / Service
↓
Backend API 호출
↓
Frontend Response 처리
↓
State / 화면 반영
```

다음 영역은 분석하지 않는다.

- Backend Controller 내부 구현
- Backend Service
- ServiceImpl
- Mapper
- MyBatis Mapper XML
- SQL
- Oracle
- SAP 내부 처리


---

# 25. Source와 Runtime 구분

Source에서 확인한 규칙과
Runtime에서 관찰한 실제 값을 구분한다.

예:

Source 확인:

```text
searchType === "ALL"
→ request.searchType = null
```

Runtime 관찰:

```text
현재 실행:
searchType = "ALL"

실제 Request:
searchType 미전송
```

Runtime에서 한 번 관찰한 값을
전체 Business Rule로 일반화하지 않는다.


---

# 26. Reference 사용 규칙

이 Reference는
문서 표현 방식과 상세 수준을 위한 기준이다.

Reference는 Evidence가 아니다.

Reference에서 참고할 수 있는 것:

- 문서 구조
- Section 순서
- 표 표현 방식
- Tree 표현 방식
- 한눈에 보는 기능 흐름
- 상세 분석 수준
- Source Evidence 표현 방식
- 대량 Validation 표현 방식
- 대량 Parameter 표현 방식
- 다중 API 표현 방식
- 미확인 항목 표현 방식


Reference에서 복사하면 안 되는 것:

- 실제 화면명
- Action ID
- 기능명
- Handler
- Function / Method
- Source Path
- Line Number
- Parameter
- State
- 조건
- API URL
- HTTP Method
- Request
- Response
- 실제 값


현재 분석 결과는 반드시
현재 분석 대상의 Evidence를 기준으로 작성한다.


---

# 27. 핵심 표현 원칙

Frontend 문서는 다음 원칙을 따른다.

```text
첫 번째 목표:
한눈에 기능 전체를 이해

두 번째 목표:
필요하면 상세 Business Logic 확인

세 번째 목표:
필요하면 실제 Source 위치 확인
```

따라서:

```text
한눈에 보는 기능 흐름
        ↓
전체 실행 Tree
        ↓
상세 분석
        ↓
Source Evidence
```

순서로 정보를 단계적으로 확장한다.

Validation이나 Parameter가 많더라도
상단 요약 영역을 지나치게 크게 만들지 않는다.

상단에서는:

`총 개수 + 그룹`

으로 표현한다.

하단 상세 영역에서는:

`전체 항목`

을 누락 없이 제공한다.

가독성을 위해
분석 내용을 삭제하거나 축약하지 않는다.

상세 분석은 유지하고
표현 구조만 계층화한다.
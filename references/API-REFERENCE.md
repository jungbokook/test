# API REFERENCE

> 이 문서는 API 분석 결과의 **표현 형식, 문서 구조, 상세 수준**을 정의하기 위한 Reference이다.
>
> 이 문서의 SAMPLE 값은 Evidence가 아니다.
> 실제 API 분석에서는 반드시 현재 Project Source 및 Runtime Evidence에서 확인된 값만 사용한다.
>
> 확인되지 않은 내용은 추측하지 않고 `확인되지 않음`으로 기록한다.

---

# 1. API 기본 정보

## 1.1 API 정보

| 항목 | 내용 |
|---|---|
| API ID | API-001 |
| 기능명 | 설비 검색 |
| HTTP Method | POST |
| Backend URL | `/api/equipment/search` |
| 화면명 | equipment-search |
| FE Action | ACT-002 |
| Controller | `EquipmentController.search()` |
| Request DTO | `EquipmentSearchRequest` |
| Response DTO | `EquipmentSearchResponse` |

HTTP Method가 분석 시작 시 `UNKNOWN`이었으나
Source 분석을 통해 확인된 경우 실제 Method를 기록한다.

끝까지 확인되지 않은 경우:

```text
HTTP Method: 확인되지 않음
```

---

# 2. API 기능 요약

이 API가 어떤 요청을 받고 어떤 응답을 반환하는지
Source에서 확인된 범위 안에서 간결하게 설명한다.

예:

```text
설비 검색 화면에서 입력된 검색 조건을 Backend로 전달하는 API이다.

Frontend에서 검색 조건을 Request Body로 생성하여 전송하고,
Backend Controller는 EquipmentSearchRequest DTO로 요청을 수신한다.

응답은 EquipmentSearchResponse 구조로 반환되며
Frontend에서는 반환된 목록을 검색 결과 데이터로 사용한다.
```

Business Logic이나 Service 내부 처리 내용을 이 영역에서 설명하지 않는다.

---

# 3. 한눈에 보는 API Contract

```text
[Frontend]
EquipmentSearch.vue
        │
        │ handleSearch()
        ▼
equipmentApi.search()
        │
        │ POST
        │ /api/equipment/search
        │
        │ Request Body
        ▼
┌───────────────────────────────┐
│ EquipmentSearchRequest        │
│                               │
│ plantCode                     │
│ equipmentName                 │
│ equipmentType                 │
│ useYn                         │
└───────────────┬───────────────┘
                │
                ▼
[Backend Controller]
EquipmentController.search()
                │
                │ Validation
                ▼
        Request Contract
                │
                ▼
════════ API Contract 경계 ════════
                │
                │ Service 내부 분석 안 함
                │
                ▼
        Response Contract
                │
                ▼
┌───────────────────────────────┐
│ EquipmentSearchResponse       │
│                               │
│ equipmentId                   │
│ equipmentName                 │
│ equipmentType                 │
│ useYn                         │
└───────────────┬───────────────┘
                │
                ▼
[Frontend Response]
검색 결과 수신
```

이 Tree는 API의 Interface 흐름을 보여주기 위한 것이다.

Service / Mapper / SQL 등의 Backend 내부 실행 흐름을 포함하지 않는다.

---

# 4. Frontend API 호출

## 4.1 호출 위치

| 항목 | 내용 |
|---|---|
| Source Path | `gipms-example/src/pages/equipment/EquipmentSearch.vue` |
| Function | `handleSearch()` |
| API Function | `equipmentApi.search()` |
| Line Range | `120-158` |
| HTTP Method | POST |
| URL | `/api/equipment/search` |

## 4.2 호출 구조

```text
handleSearch()
        ↓
검색 Parameter 생성
        ↓
equipmentApi.search(request)
        ↓
POST /api/equipment/search
```

## 4.3 Frontend Request

```text
equipmentApi.search({
    plantCode,
    equipmentName,
    equipmentType,
    useYn
})
```

위 값은 SAMPLE 표현 형식이다.

실제 분석에서는 실제 FE Source에서 확인된 구조만 기록한다.

---

# 5. Controller Mapping

## 5.1 Mapping 정보

| 항목 | 내용 |
|---|---|
| Backend Project | `gipms-api-example` |
| Source Path | `gipms-api-example/src/main/java/.../EquipmentController.java` |
| Controller | `EquipmentController` |
| Method | `search()` |
| Class Mapping | `/api/equipment` |
| Method Mapping | `/search` |
| HTTP Method | POST |
| 최종 URL | `/api/equipment/search` |
| Line Range | `40-65` |

## 5.2 Mapping 조합

```text
Class Mapping
/api/equipment

        +

Method Mapping
/search

        +

HTTP Method
POST

        ↓

POST /api/equipment/search
```

Class Mapping과 Method Mapping이 분리되어 있다면
반드시 최종 URL을 조합하여 기록한다.

---

# 6. Request Contract

## 6.1 Request 전달 구조

```text
Request
│
├─ Header
│   └─ Authorization
│
├─ Path Parameter
│   └─ 없음
│
├─ Query Parameter
│   └─ 없음
│
└─ Request Body
    └─ EquipmentSearchRequest
        ├─ plantCode
        ├─ equipmentName
        ├─ equipmentType
        └─ useYn
```

실제 API에 존재하지 않는 전달 영역은 `없음`으로 표시한다.

확인할 수 없는 경우에는 `확인되지 않음`으로 표시한다.

---

# 7. Header

| Name | Type | Required | 설명 | Evidence |
|---|---|---:|---|---|
| Authorization | String | 확인되지 않음 | 인증 Header | 실제 Source 기준 작성 |

Header가 없는 경우:

```text
Header: 없음
```

Header 존재 여부를 확인할 수 없는 경우:

```text
Header: 확인되지 않음
```

---

# 8. Path Parameter

| Name | Backend Type | Required | Validation | 설명 |
|---|---|---:|---|---|
| equipmentId | Long | YES | 확인되지 않음 | 설비 ID |

Path Parameter가 없는 경우:

```text
Path Parameter: 없음
```

---

# 9. Query Parameter

| Name | Backend Type | Required | Default | Validation | 설명 |
|---|---|---:|---|---|---|
| page | Integer | NO | `0` | 확인되지 않음 | 페이지 번호 |
| size | Integer | NO | `20` | 확인되지 않음 | 페이지 크기 |

Query Parameter가 없는 경우:

```text
Query Parameter: 없음
```

Required 여부는 FE가 항상 전달한다는 이유로 결정하지 않는다.

Backend Source Evidence를 기준으로 판단한다.

---

# 10. Request Body

## 10.1 Request DTO

```text
EquipmentSearchRequest
```

Source:

```text
gipms-api-example/src/main/java/.../dto/EquipmentSearchRequest.java
```

## 10.2 Request DTO 구조

| JSON Field | Java Field | Java Type | Required | Default | Validation | Nullable |
|---|---|---|---:|---|---|---:|
| `plantCode` | `plantCode` | String | YES | 없음 | `@NotBlank` | NO |
| `equipmentName` | `equipmentName` | String | NO | 없음 | 없음 | YES |
| `equipmentType` | `equipmentType` | String | NO | 없음 | 확인되지 않음 | YES |
| `useYn` | `useYn` | String | NO | 없음 | 확인되지 않음 | YES |

Required / Nullable은 실제 Source Evidence를 기준으로 작성한다.

Source에서 명확히 판단할 수 없는 경우:

```text
확인되지 않음
```

으로 기록한다.

---

# 11. Nested Request DTO

Nested DTO가 존재하는 경우 구조를 펼쳐서 기록한다.

예:

```text
EquipmentSearchRequest
│
├─ plantCode : String
│
├─ condition : SearchCondition
│   │
│   ├─ keyword : String
│   └─ useYn : String
│
└─ equipmentTypes : List<String>
```

Nested DTO 상세:

### SearchCondition

| JSON Field | Java Field | Java Type | Required | Validation |
|---|---|---|---:|---|
| `keyword` | `keyword` | String | NO | 확인되지 않음 |
| `useYn` | `useYn` | String | NO | 확인되지 않음 |

Collection인 경우 Collection 내부 Type까지 확인한다.

예:

```text
List<SearchCondition>
```

이라면 `SearchCondition` 구조까지 확인한다.

---

# 12. Request Validation

## 12.1 Validation 요약

```text
EquipmentSearchRequest
│
├─ plantCode
│   └─ @NotBlank
│
├─ equipmentName
│   └─ Validation 없음
│
├─ equipmentType
│   └─ 확인되지 않음
│
└─ useYn
    └─ 확인되지 않음
```

## 12.2 Validation 상세

| 대상 | Validation | 조건 | Message | Evidence |
|---|---|---|---|---|
| plantCode | `@NotBlank` | null / empty / blank 금지 | Source 확인값 | DTO Field |
| equipmentName | 없음 | - | - | DTO Field |

Custom Validator가 존재하는 경우
API 입력 Validation을 확인하는 범위까지만 분석한다.

Custom Validator에서 Service / DB / Business Logic으로 이어지면 STOP 한다.

---

# 13. Frontend ↔ Backend Request Mapping

Frontend에서 생성한 값이 Backend의 어느 항목으로 전달되는지 비교한다.

| FE Field | 전송 위치 | 전송 Name | Backend Field | Backend Type | Required | 결과 |
|---|---|---|---|---|---:|---|
| `plantCode` | Body | `plantCode` | `plantCode` | String | YES | 일치 |
| `equipmentName` | Body | `equipmentName` | `equipmentName` | String | NO | 일치 |
| `equipmentType` | Body | `equipmentType` | `equipmentType` | String | NO | 일치 |
| `useYn` | Body | `useYn` | `useYn` | String | NO | 일치 |

차이가 발견되면 숨기지 않고 기록한다.

예:

```text
[Mapping 불일치]

Frontend
equipmentTypeCode

        ↓ JSON

equipmentTypeCode

Backend DTO
equipmentType

결과
Field Name 불일치 확인
```

---

# 14. Response Contract

## 14.1 Controller 반환 타입

```text
ResponseEntity<EquipmentSearchResponse>
```

또는 실제 Source에서 확인된 반환 타입을 기록한다.

## 14.2 Response 구조

```text
EquipmentSearchResponse
│
├─ totalCount : Integer
│
└─ items : List<EquipmentItem>
    │
    └─ EquipmentItem
        ├─ equipmentId : Long
        ├─ equipmentName : String
        ├─ equipmentType : String
        └─ useYn : String
```

---

# 15. Response DTO

## 15.1 최상위 Response

| JSON Field | Java Field | Java Type | 설명 |
|---|---|---|---|
| `totalCount` | `totalCount` | Integer | 전체 건수 |
| `items` | `items` | `List<EquipmentItem>` | 검색 결과 |

## 15.2 EquipmentItem

| JSON Field | Java Field | Java Type | 설명 |
|---|---|---|---|
| `equipmentId` | `equipmentId` | Long | 설비 ID |
| `equipmentName` | `equipmentName` | String | 설비명 |
| `equipmentType` | `equipmentType` | String | 설비 유형 |
| `useYn` | `useYn` | String | 사용 여부 |

Nested DTO가 여러 단계라면
실제 Client Contract에 포함되는 범위까지 구조를 확장한다.

---

# 16. Response 확인 불가

Controller 범위에서 Response 구조를 확인할 수 없는 경우
Service 내부로 들어가서 억지로 확인하지 않는다.

예:

```text
Controller Return Type
ResponseEntity<?>

        ↓

구체적인 Response DTO
현재 Controller / DTO Source에서 확인 불가

        ↓

STOP
```

문서에는 다음과 같이 기록한다.

```text
Response DTO
확인되지 않음

사유
Controller / DTO Source 범위에서
구체적인 Response 구조를 확인할 수 없음
```

---

# 17. HTTP Status

## 17.1 확인된 Status

| 상황 | HTTP Status | Evidence |
|---|---:|---|
| 정상 응답 | 200 | `ResponseEntity.ok(...)` |
| Validation Error | 확인되지 않음 | 실제 Error Handler 확인 필요 |

실제 Source에 명시된 Status만 확정한다.

명시적 Status를 확인할 수 없는 경우:

```text
명시적 HTTP Status: 확인되지 않음
```

Framework의 일반적인 기본 동작만으로
Status를 확정하지 않는다.

---

# 18. Error Response

## 18.1 Error 처리 흐름

```text
Request
   ↓
Validation
   ↓
Validation 실패
   ↓
ExceptionHandler / ControllerAdvice
   ↓
Error Response DTO
```

## 18.2 Error Response 구조

| Field | Type | 설명 |
|---|---|---|
| `code` | String | Error Code |
| `message` | String | Error Message |
| `errors` | List | 상세 Error |

실제 현재 API와 연결되는 Error Response만 기록한다.

프로젝트 전체 Error 처리 체계를 분석하지 않는다.

---

# 19. Runtime Evidence

Chrome DevTools로 Runtime Request / Response를 확인한 경우
Source Contract와 분리해서 기록한다.

## 19.1 관찰된 Request

```text
Observed Runtime Request

Method
POST

URL
/api/equipment/search

Body
plantCode = "1000"
equipmentName = ""
useYn = "Y"
```

위 값은 **해당 실행 시점에서 관찰된 값**이다.

다음과 같이 해석하지 않는다.

```text
plantCode 허용값 = "1000"
```

Runtime Value는 API 전체 Contract를 의미하지 않는다.

---

# 20. Source Contract ↔ Runtime 비교

| 항목 | Source Contract | Runtime 관찰 | 결과 |
|---|---|---|---|
| Method | POST | POST | 일치 |
| plantCode | String / Required | `"1000"` | 일치 |
| equipmentName | String / Optional | `""` | 관찰값 |
| useYn | String / Optional | `"Y"` | 관찰값 |

Runtime을 확인하지 않은 경우:

```text
Runtime Evidence: 확인하지 않음
```

으로 기록한다.

---

# 21. Source Evidence

## 21.1 Frontend

| 구분 | Source Path | Function / Field | Line Range |
|---|---|---|---|
| API 호출 | `gipms-example/src/.../EquipmentSearch.vue` | `handleSearch()` | `120-158` |
| API 정의 | `gipms-example/src/.../equipmentApi.js` | `search()` | `20-32` |

## 21.2 Backend

| 구분 | Source Path | Class / Function / Field | Line Range |
|---|---|---|---|
| Controller | `gipms-api-example/src/.../EquipmentController.java` | `search()` | `40-65` |
| Request DTO | `gipms-api-example/src/.../EquipmentSearchRequest.java` | Class / Fields | `10-45` |
| Response DTO | `gipms-api-example/src/.../EquipmentSearchResponse.java` | Class / Fields | `10-40` |
| Error DTO | `gipms-api-example/src/.../ErrorResponse.java` | Class / Fields | `10-35` |

Line Range를 실제로 확인할 수 없는 경우:

```text
Line Range: 확인되지 않음
```

으로 기록한다.

Line 번호를 추측하거나 생성하지 않는다.

---

# 22. 확인되지 않은 항목

분석 과정에서 확인하지 못한 항목을 한곳에 정리한다.

예:

```text
[확인되지 않은 항목]

1. equipmentType 허용값
   상태:
   확인되지 않음

   사유:
   현재 Project Source에서 허용값 정의를 확인하지 못함


2. Validation Error HTTP Status
   상태:
   확인되지 않음

   사유:
   현재 API 범위의 Source에서 명시적인 Status 확인 불가


3. 특정 Response Field
   상태:
   현재 Project Source에서 확인되지 않음

   사유:
   구체적인 구조를 확인하려면 분석 경계를 넘어야 함
```

확인되지 않은 내용을 일반적인 Spring 동작이나
개발 경험으로 보완하지 않는다.

---

# 23. 분석 경계

```text
Frontend API Call
        ↓
Controller Mapping
        ↓
Request Parameter / DTO
        ↓
Validation
        ↓
Response DTO
        ↓
Error Response
        ↓
Source Evidence
        ↓

══════════════ STOP ══════════════

Service
ServiceImpl
Business Logic
Mapper
MyBatis Mapper XML
SQL
Oracle
SAP
JAR
외부 Dependency
Decompiled Class
```

API Contract 분석에서는
Backend 내부 Business Logic으로 진행하지 않는다.

---

# 24. Source 탐색 제한

분석 대상은 Code Index MCP의
현재 Project Root 아래 실제 Source이다.

허용 범위:

```text
Frontend
gipms-* 중 gipms-api-* 제외

Backend
gipms-api-*

공통 Source
Project Root 아래 실제 Source 형태로 존재하는 공통 프로젝트
예: gipms-api-common
```

자동 탐색 금지:

```text
*.jar
JAR 내부 Class
Decompiled Class
Maven Repository
.m2/
Gradle Cache
.gradle/
외부 Dependency Source
Project Root 외부 Source
```

현재 Project Source에서 확인할 수 없다면:

```text
현재 Project Source에서 확인되지 않음
```

으로 기록한다.

---

# 25. 문서 작성 원칙

실제 API 분석 문서는 다음 원칙을 따른다.

1. 실제 Source에서 확인된 내용을 우선한다.
2. Code Index 검색 결과만으로 Contract를 확정하지 않는다.
3. Runtime Evidence는 관찰 사례로 구분한다.
4. Runtime Value를 API 전체 Contract로 일반화하지 않는다.
5. FE가 값을 전송한다는 이유만으로 Backend Required라고 판단하지 않는다.
6. Controller / DTO에서 확인할 수 없는 Response를 Service까지 추적하지 않는다.
7. HTTP Status를 Framework의 일반적인 동작만으로 추측하지 않는다.
8. Line Range를 임의로 생성하지 않는다.
9. Source Path는 Project Root 기준 상대경로로 기록한다.
10. 확인되지 않은 내용은 `확인되지 않음`으로 기록한다.
11. Service / Mapper / SQL / Oracle 분석으로 넘어가지 않는다.
12. API 분석 완료 후 Backend 분석을 자동으로 시작하지 않는다.

---

# 26. 대량 Field 표시 규칙

Request / Response DTO의 Field가 많더라도
임의로 일부만 선택하여 표시하지 않는다.

예:

```text
Request DTO
총 37개 Field
```

라면 실제 Contract에 포함되는 37개 Field를 모두 기록한다.

Nested DTO 역시 동일하다.

단순히:

```text
주요 Field만 표시
...
기타 생략
```

과 같이 임의 축약하지 않는다.

Source에서 확인된 전체 Contract를 보존한다.

---

# 27. SAMPLE 데이터 사용 금지

이 Reference에 포함된 다음 값들은 모두 SAMPLE이다.

```text
EquipmentSearchRequest
EquipmentSearchResponse
EquipmentController
plantCode
equipmentName
equipmentType
useYn
/api/equipment/search
gipms-example
gipms-api-example
```

실제 분석 문서에 SAMPLE 값을 복사하지 않는다.

현재 분석 대상 Source에서 확인된 값으로 전부 대체한다.

Reference의 SAMPLE과 실제 Source가 다르면
항상 실제 Source를 따른다.

---

# 28. 최종 출력 원칙

최종 API 문서는 다음 질문에
문서 하나만 읽고 답할 수 있어야 한다.

```text
이 API는 무엇인가?

어디서 호출되는가?

어떤 HTTP Method와 URL을 사용하는가?

Controller는 어디인가?

Header는 무엇인가?

Path Parameter는 무엇인가?

Query Parameter는 무엇인가?

Request Body는 무엇인가?

Request DTO에는 어떤 Field가 있는가?

각 Field의 Type은 무엇인가?

어떤 값이 Required인가?

어떤 Validation이 적용되는가?

Frontend 값과 Backend DTO가 어떻게 연결되는가?

Response 구조는 무엇인가?

Response DTO에는 어떤 Field가 있는가?

HTTP Status는 무엇인가?

Error Response는 무엇인가?

Runtime에서 실제로 무엇이 관찰되었는가?

각 분석 결과의 Source Evidence는 어디인가?

무엇을 확인하지 못했는가?

어디에서 API 분석이 STOP 되었는가?
```

이 질문에 답하기 위해
Service / Mapper / SQL까지 내려가야 한다면
API Contract 단계의 분석 범위를 초과한 것이다.

그 내용은 이후 Backend 분석 단계에서 다룬다.
---
name: api-contract-tracing
description: 선택한 API의 Frontend 호출부터 Controller, Request/Response DTO, Validation까지 추적하여 실제 API Contract를 확인한다.
---

# API Contract Tracing Rule

## 1. 목적

선택한 Backend API의 실제 Interface Contract를
Source 기준으로 추적한다.

분석 대상:

```text
Frontend API Call
        ↓
Controller Mapping
        ↓
Request 구조
        ↓
Request Validation
        ↓
Response 구조
        ↓
Error Response
```

API가 실제로 어떤 값을 받고
어떤 값을 반환하는지 명확하게 확인하는 것이 목적이다.

Backend Business Logic은 분석하지 않는다.


---

## 2. 분석 시작점

FE 분석 문서에서 선택된 API 정보를 시작점으로 사용한다.

가능한 시작 정보:

```text
API 기능
HTTP Method
URL
Frontend 호출 Function
호출 조건
Query Parameter
Path Parameter
Request Body
Response 처리
Source Path
```

예:

```text
POST /api/equipment/search
```

Frontend Source에서 실제 호출 위치도 확인한다.

```text
equipmentService.search(params)
        ↓
POST /api/equipment/search
```


---

## 3. Controller 탐색

HTTP Method와 URL을 기준으로
실제 Backend Controller Mapping을 찾는다.

예:

```text
POST /api/equipment/search
        ↓
EquipmentController
        ↓
searchEquipment()
```

확인 항목:

```text
Controller Class
Controller Method
Class Mapping
Method Mapping
최종 URL
HTTP Method
Request 전달 방식
Request DTO
Response Type
```

Class Mapping과 Method Mapping이 분리되어 있다면
결합하여 실제 URL을 확인한다.

예:

```text
@RequestMapping("/api/equipment")

+

@PostMapping("/search")

=

POST /api/equipment/search
```


---

## 4. Controller 후보가 여러 개인 경우

동일하거나 유사한 URL Mapping이 여러 개 발견되면
Method와 전체 Mapping을 함께 비교한다.

판단 기준:

```text
HTTP Method
+
Class Mapping
+
Method Mapping
+
Frontend 호출 URL
```

이름만 비슷하다는 이유로
Controller를 확정하지 않는다.

확정할 수 없으면:

```text
Controller Mapping
→ 확인되지 않음
```

또는 후보를 구분하여 기록한다.


---

## 5. Request 전달 방식 추적

Controller Method의 Parameter를 기준으로
Request가 어떻게 전달되는지 구분한다.

```text
@RequestHeader
        ↓
Header

@PathVariable
        ↓
Path Parameter

@RequestParam
        ↓
Query Parameter

@RequestBody
        ↓
Request Body
```

예:

```text
searchEquipment(
    @RequestParam String plantCode,
    @RequestBody EquipmentSearchRequest request
)
```

이면:

```text
Request
│
├─ Query Parameter
│  └─ plantCode
│
└─ Request Body
   └─ EquipmentSearchRequest
```


---

## 6. Request DTO 추적

`@RequestBody` 또는 이에 해당하는
Request Object가 존재하면 실제 DTO Source를 확인한다.

예:

```text
EquipmentSearchRequest
│
├─ plantCode
├─ equipmentName
├─ status
├─ startDate
└─ endDate
```

각 Field에 대해 가능한 범위에서 확인한다.

```text
Field Name
Java Type
JSON Field Name
Required 여부
Default Value
Validation
Nullable 여부
허용값 / 형식
```

Source에서 확인되지 않은 속성은
추측하지 않는다.


---

## 7. JSON Field Name

Java Field 이름과 실제 JSON 이름이 다를 수 있다.

예:

```text
Java
equipmentName

@JsonProperty("equipment_name")

실제 JSON
equipment_name
```

이 경우 실제 API Contract에는:

```text
equipment_name
```

을 Request Field로 기록하고
Java Field도 Source 정보로 함께 남긴다.

Naming Strategy가 명확하게 확인되는 경우에만
자동 변환 규칙을 적용한다.

확인되지 않은 Naming Strategy를
임의로 가정하지 않는다.


---

## 8. Request Type

Request Field의 Type은
가능하면 실제 DTO 선언을 기준으로 확인한다.

예:

```text
String
Integer
Long
Boolean
LocalDate
LocalDateTime
List<String>
List<EquipmentRequest>
```

Frontend Runtime 값만 보고
Backend Type을 추측하지 않는다.

예:

```text
Runtime

"1000"
```

만 확인되었다고 해서:

```text
Type = String
```

으로 확정하지 않는다.

DTO Source에서 Type을 확인한다.


---

## 9. Required 여부

Required 여부는 실제 Source Evidence를 기준으로 판단한다.

가능한 근거:

```text
@RequestParam(required = true)

@NotNull
@NotBlank
@NotEmpty

Validation Annotation

Controller Validation
```

단순히 Frontend에서 항상 보내고 있다는 이유만으로
Backend Required Field라고 판단하지 않는다.

예:

```text
Frontend
plantCode 항상 전달

≠

Backend
plantCode 필수
```


---

## 10. Validation 추적

Request와 직접 관련된 Validation을 확인한다.

예:

```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Positive
@PositiveOrZero
@Pattern
@Email
@Past
@Future
```

Custom Validation Annotation이 존재하면
현재 Request Contract를 이해하는 데 필요한 범위까지만 확인한다.

예:

```text
@ValidPlantCode
```

필요한 경우:

```text
Annotation
        ↓
Validator
        ↓
실제 Validation 조건
```

까지만 확인할 수 있다.

단, Validator에서 Service / DB 조회로 이어지는 경우:

```text
Validator
   ↓
Service / Repository / Mapper
   ↓
STOP
```

한다.


---

## 11. Nested Request DTO

Request DTO 안에 다른 DTO가 포함될 수 있다.

예:

```text
EquipmentSearchRequest
│
├─ plantCode
├─ filter
│  ├─ equipmentType
│  └─ status
│
└─ paging
   ├─ page
   └─ size
```

현재 API Request 구조에 포함되는 Nested DTO는
필요한 깊이까지 추적한다.

단, 현재 API와 관계없는 DTO는 확장하지 않는다.


---

## 12. Collection Request

List / Array 형태의 Request도
실제 구조를 확인한다.

예:

```text
equipmentIds
│
├─ "EQ001"
├─ "EQ002"
└─ "EQ003"
```

Object List인 경우:

```text
items[]
│
├─ equipmentId
├─ quantity
└─ status
```

Collection 내부 Element Type도
Source에서 확인 가능한 경우 기록한다.


---

## 13. Response Type 추적

Controller Method의 Return Type을 기준으로
Response 구조를 확인한다.

예:

```text
ResponseEntity<ApiResponse<EquipmentSearchResponse>>
```

이면 다음과 같이 분리해서 확인한다.

```text
ResponseEntity
        ↓
ApiResponse
        ↓
EquipmentSearchResponse
```


---

## 14. Response Wrapper 추적

공통 Response Wrapper가 존재하면
현재 API Response를 이해하는 데 필요한 구조만 확인한다.

예:

```text
ApiResponse<T>
│
├─ code
├─ message
└─ data
```

그리고:

```text
T
↓
EquipmentSearchResponse
```

이면 최종 구조는:

```text
Response
│
├─ code
├─ message
└─ data
   └─ EquipmentSearchResponse
```

공통 Wrapper의 내부 Business Logic까지
분석하지 않는다.


---

## 15. Response DTO 추적

Response DTO의 실제 Field 구조를 확인한다.

예:

```text
EquipmentSearchResponse
│
├─ items
│  └─ EquipmentItem[]
│     ├─ equipmentId
│     ├─ equipmentName
│     └─ status
│
└─ totalCount
```

각 Field에서 가능한 경우:

```text
Field Name
JSON Field Name
Type
Nested Type
Nullable 여부
```

를 확인한다.

Response 값이 어떻게 계산되는지는
현재 단계에서 분석하지 않는다.


---

## 16. Generic / 상속 구조

DTO가 Generic 또는 상속 구조를 사용하는 경우
실제 API Response 구조를 이해하는 데 필요한 범위까지 해석한다.

예:

```text
PageResponse<EquipmentItem>
```

이면:

```text
PageResponse
│
├─ items
│  └─ EquipmentItem[]
├─ page
├─ size
└─ totalCount
```

형태로 실제 Contract를 표현한다.

상속이 존재하는 경우에도
최종적으로 Client가 받는 Field 구조를 기준으로 정리한다.


---

## 17. HTTP Status

Controller에서 명시적으로 확인 가능한
HTTP Status를 기록한다.

예:

```text
@ResponseStatus(HttpStatus.CREATED)

ResponseEntity.ok(...)

ResponseEntity.status(HttpStatus.BAD_REQUEST)
```

Source에서 명확하게 확인되지 않는 Status를
임의로 확정하지 않는다.

Framework 기본 동작에 의존해야 하는 경우:

```text
명시적 Status
→ 확인되지 않음
```

으로 구분한다.


---

## 18. Error Response 추적

현재 API와 직접 연결되는 Error Response를 확인한다.

가능한 경로:

```text
Request Validation
        ↓
Exception
        ↓
Exception Handler
        ↓
Error Response
```

또는:

```text
Controller
        ↓
Exception
        ↓
ControllerAdvice
        ↓
Error Response
```

확인 가능한 경우:

```text
HTTP Status
Error Code
Message
Response Structure
```

를 기록한다.

프로젝트 전체 Exception 체계를
분석하지 않는다.


---

## 19. Frontend ↔ Backend Request Mapping

가능한 경우 FE에서 생성한 Request와
Backend에서 받는 Request를 비교한다.

예:

```text
Frontend

request.plantCode
request.keyword
request.startDate

        ↓

Backend

EquipmentSearchRequest.plantCode
EquipmentSearchRequest.keyword
EquipmentSearchRequest.startDate
```

차이가 발견되면 명확하게 표시한다.

예:

```text
Frontend Field
equipmentName

        ↓ JSON

equipment_name

        ↓ Backend

EquipmentSearchRequest.equipmentName
```

Frontend에서 보내지만
Backend Contract에서 확인되지 않는 Field도 기록한다.

반대로 Backend DTO에는 존재하지만
현재 FE 호출에서 전달되지 않는 Field도 구분한다.


---

## 20. Runtime ↔ Source 비교

Chrome DevTools Runtime 정보가 존재하면
Source와 비교할 수 있다.

예:

```text
Source Contract

plantCode : String
keyword   : String
status    : String

        ↓

Runtime 관찰

plantCode = "1000"
keyword   = "PUMP"
status    = 미전송
```

이 경우:

```text
status
→ Contract에는 존재
→ 현재 Runtime Request에서는 미전송
```

으로 구분한다.

Runtime 관찰값을
Contract 전체 규칙으로 일반화하지 않는다.


---

## 21. Code Index MCP 사용

Code Index MCP는 다음 탐색에 사용할 수 있다.

```text
Controller Mapping 탐색
Controller Method 탐색
DTO Source 탐색
DTO Reference 탐색
Response Type 탐색
Exception Handler 탐색
```

Code Index 결과는 Source를 찾기 위한
탐색 수단으로 사용한다.

최종 Contract 판단은
가능한 경우 실제 Source를 확인한다.


---

## 22. Source Evidence

API Contract의 주요 판단에는
가능한 경우 Source Evidence를 남긴다.

형식:

```text
Source Path
+
Class / Function / Field
+
Line Range
```

예:

```text
Controller
src/main/java/.../EquipmentController.java
searchEquipment()
Line 45-62
```

DTO:

```text
Request DTO
src/main/java/.../EquipmentSearchRequest.java
plantCode
Line 18-20
```

Line Range를 확인할 수 없으면:

```text
Line : 확인되지 않음
```

으로 기록한다.

Line Number를 추측하지 않는다.


---

## 23. 순환 / 과도한 추적 방지

다음 조건에서는 추적을 중단한다.

```text
현재 API Contract와 관계없는 Source

이미 확인한 동일 Type

Framework 내부 구현

외부 Library 내부 구현

Service Business Logic

Mapper

MyBatis XML

SQL

Oracle
```

API Contract를 확인하는 데 필요한
최소 범위만 추적한다.


---

## 24. 최종 Contract 구조

분석 결과는 최종적으로 다음 구조를 설명할 수 있어야 한다.

```text
API
│
├─ Method / URL
│
├─ Request
│  │
│  ├─ Header
│  ├─ Path
│  ├─ Query
│  └─ Body
│     └─ Field / Type / Required / Validation
│
├─ Response
│  └─ Field / Type / Nested Structure
│
├─ HTTP Status
│
└─ Error Response
```

단순히 DTO 이름만 나열하지 않는다.

실제 Client 관점에서
어떤 JSON / Parameter 구조를 주고받는지
이해할 수 있도록 작성한다.


---

## 25. 분석 경계

```text
Frontend API Call
        ↓
Controller Mapping
        ↓
Request DTO
        ↓
Request Validation
        ↓
Response DTO
        ↓
Error Response
        ↓

══════════════ STOP ══════════════

Service
ServiceImpl
Business Logic
Mapper
MyBatis XML
SQL
Oracle
SAP
```

API Contract 분석을 완료한 후
Backend 내부 분석으로 자동 진행하지 않는다.
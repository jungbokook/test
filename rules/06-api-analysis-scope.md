---
name: api-analysis-scope
description: FE 분석에서 선택한 Backend API를 기준으로 API Interface 규격만 분석하고 Backend 내부 구현으로 진입하지 않도록 범위를 제한한다.
---

# API Analysis Scope Rule

## 1. 목적

이 Rule은 FE 분석에서 발견된 Backend API 중
사용자가 선택한 하나의 API를 대상으로
API Interface 규격을 분석하기 위한 범위를 정의한다.

API 분석의 목적은 다음 질문에 답하는 것이다.

```text
Frontend가
무엇을 보내고
        ↓
Backend가
어떤 형식으로 받고
        ↓
어떤 형식으로
응답하는가?
```

이 단계에서는 Backend 내부 Business Logic을 분석하지 않는다.


---

## 2. 분석 입력

API 분석은 사용자가 선택한
Backend API를 입력으로 사용한다.

가능한 입력 예:

```text
POST /api/equipment/search
```

또는 이후 API ID가 존재하는 경우:

```text
API-001
```

FE 분석 문서가 존재하는 경우
선택한 API와 관련된 FE 정보를 시작점으로 사용한다.

예:

```text
docs/analysis/{화면명}/frontend/
FE-{Action ID}-{기능명}.md
```

FE 문서에서 다음 정보를 확인할 수 있다.

```text
API 기능
HTTP Method
URL
호출 Function
호출 조건
Query Parameter
Path Parameter
Request Body
Runtime에서 확인된 Request
Response 처리
Source Evidence
```

FE 문서의 내용은 API 분석의 시작점이며
API 규격 전체를 의미하지 않는다.


---

## 3. API ID

API 규격 분석 단계에서
분석 대상 API에 API ID를 부여한다.

형식:

```text
API-001
API-002
API-003
...
```

API ID는 현재 화면의 API 분석 문서 내에서
식별 가능하도록 관리한다.

동일 API를 중복 분석하지 않도록
기존 API 문서가 존재하는지 먼저 확인한다.


---

## 4. 분석 범위

선택한 API에 대해 다음 항목을 분석한다.

```text
API 기능
HTTP Method
URL

Header

Query Parameter
Path Parameter
Request Body

Request Field
Request Type
필수 여부
기본값
허용값
Validation

Response
Response Field
Response Type

HTTP Status
Error Response
```

실제 Source에서 확인 가능한 경우
각 항목의 Source Evidence도 기록한다.


---

## 5. API 추적 범위

API 분석에서는 다음 범위까지만 추적한다.

```text
Frontend API Call
        ↓
HTTP Method / URL
        ↓
Controller Mapping
        ↓
Request Parameter / DTO
        ↓
Validation
        ↓
Response DTO / Response 구조
        ↓
HTTP Status / Error Response
        ↓
STOP
```

Controller는 API Interface를 확인하기 위한 범위까지만 읽는다.


---

## 6. Controller 분석 경계

Controller에서는 다음 항목을 확인할 수 있다.

```text
Controller Class
Controller Method

@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping

@RequestParam
@PathVariable
@RequestBody

Request DTO
Response DTO

Validation Annotation

HTTP Status
Exception Mapping
```

하지만 Controller 내부에서 발견되는
Service 호출을 따라가지 않는다.

예:

```text
Controller
   ↓
equipmentService.search(request)
   ↓
STOP
```

Service Method 이름은
API 규격 이해에 필요한 경우 기록할 수 있지만
Service Source 내부로 진입하지 않는다.


---

## 7. Request 분석

Request는 실제 Source를 기준으로 분석한다.

가능한 Source:

```text
Frontend API 호출 코드
Controller
Request DTO
Validation Annotation
```

확인 가능한 경우 다음을 기록한다.

```text
Field Name
전달 위치
Type
필수 여부
기본값
허용값
Validation
Frontend Source
Backend Source
```


---

## 8. Request 전달 위치 구분

Request Field가 어디로 전달되는지 구분한다.

```text
Header
Query Parameter
Path Parameter
Request Body
```

예:

```text
Request
│
├─ Header
│  └─ Authorization
│
├─ Path
│  └─ equipmentId
│
├─ Query
│  ├─ plantCode
│  └─ status
│
└─ Body
   ├─ startDate
   └─ endDate
```

확인되지 않은 전달 위치를 추측하지 않는다.


---

## 9. Validation 분석

API Interface와 직접 관련된 Validation을 확인한다.

예:

```text
@NotNull
@NotBlank
@Size
@Min
@Max
@Pattern
@Valid
```

또는 Controller에서 직접 수행하는
Request Validation이 존재할 수 있다.

예:

```text
if (request.getPlantCode() == null) {
    ...
}
```

이 경우 API Request Validation으로 기록할 수 있다.

하지만 Service 내부의 Business Validation은
현재 API 규격 분석 대상이 아니다.

```text
Request Validation
→ 분석

Service Business Validation
→ 분석하지 않음
```


---

## 10. Response 분석

API Response Interface를 분석한다.

확인 가능한 경우:

```text
Response DTO
Response Wrapper
Response Field
Field Type
Nullable 여부
HTTP Status
Error Response
```

Response 구조는 실제 Source를 기준으로 작성한다.

예:

```text
Response
│
├─ code
├─ message
└─ data
   ├─ items
   │  ├─ equipmentId
   │  ├─ equipmentName
   │  └─ status
   │
   └─ totalCount
```

Response 내부 값의 Business 생성 과정을
추적하기 위해 Service로 진입하지 않는다.


---

## 11. 공통 Response Wrapper

프로젝트에서 공통 Response Wrapper를 사용하는 경우
현재 API Response 구조를 이해하는 데 필요한 범위까지만 확인한다.

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
data
└─ EquipmentSearchResponse
```

공통 Wrapper 전체 구현이나
관련 없는 기능까지 확장 분석하지 않는다.


---

## 12. Error Response

확인 가능한 API Error Response를 기록한다.

Source 예:

```text
Controller
Validation
Exception Handler
ControllerAdvice
공통 Error Response
```

현재 API와 실제로 연결되는 범위만 확인한다.

예:

```text
Request
   ↓
Validation FAIL
   ↓
400 Bad Request
   ↓
Error Response
```

공통 Exception Handler가 존재하더라도
프로젝트 전체 예외 체계를 분석하지 않는다.


---

## 13. Runtime 정보 사용

Chrome DevTools MCP에서
현재 API의 Runtime Request / Response를
확인할 수 있는 경우 보조 Evidence로 사용할 수 있다.

Runtime에서 확인 가능한 예:

```text
실제 HTTP Method
실제 Request URL
실제 Query Parameter
실제 Request Body
실제 Response
실제 HTTP Status
```

하지만 Runtime에서 한 번 관찰된 값은
API 전체 규격을 의미하지 않는다.

예:

```text
Runtime 관찰

plantCode = "1000"
```

이것은:

```text
이번 실행에서
plantCode가 "1000"으로 전달됨
```

을 의미한다.

다음을 의미하지 않는다.

```text
plantCode의 허용값은 항상 "1000"
```

API Type, Required 여부, Validation,
허용 범위 등은 Source를 함께 확인한다.


---

## 14. Source 우선 원칙

API 규격은 가능한 경우
다음 Source를 교차 확인한다.

```text
Frontend API Call
        +
Controller
        +
Request DTO
        +
Response DTO
        +
Validation
        +
Runtime Request / Response
```

각 Source의 역할을 구분한다.

```text
Frontend
→ 실제 호출 형태 확인

Controller
→ HTTP Interface 확인

DTO
→ Field / Type 구조 확인

Validation
→ Request 제약 확인

Runtime
→ 실제 실행 사례 확인
```


---

## 15. 확인되지 않은 정보

Source 또는 Runtime에서 확인할 수 없는 내용은
추측하지 않는다.

다음과 같이 기록한다.

```text
확인되지 않음
```

예:

```text
허용값
→ 확인되지 않음

최대 길이
→ 확인되지 않음

Nullable
→ 확인되지 않음
```


---

## 16. Backend 내부 분석 금지

API 규격 분석에서는 다음 영역을 분석하지 않는다.

```text
Service
ServiceImpl
Business Logic
Mapper
MyBatis Mapper XML
Dynamic SQL
SQL
Oracle Table
Oracle View
SAP 내부 처리
외부 시스템 내부 처리
```

API 규격 확인 과정에서
Service Method가 발견되더라도:

```text
Controller
   ↓
equipmentService.search()
   ↓
STOP
```

한다.

이 영역은 이후 Backend 기능 분석 단계에서 처리한다.


---

## 17. 분석 경계

```text
Frontend API Call
        ↓
HTTP Method / URL
        ↓
Controller
        ↓
Request Parameter / DTO
        ↓
Validation
        ↓
Response DTO
        ↓
HTTP Status / Error Response
        ↓

══════════════ STOP ══════════════

Service / ServiceImpl
        ↓
Business Logic
        ↓
Mapper
        ↓
MyBatis XML
        ↓
SQL
        ↓
Oracle
```

API 분석의 목적은
Backend 기능 구현을 설명하는 것이 아니라
Frontend와 Backend 사이의
**Interface Contract를 명확하게 만드는 것**이다.


---

## 18. 분석 완료 조건

다음 항목을 확인한 후 API 분석을 종료한다.

```text
API 기능
HTTP Method
URL

Header
Query Parameter
Path Parameter
Request Body

Request Field / Type
Required 여부
Validation

Response 구조
Response Field / Type

HTTP Status
Error Response

Source Evidence
미확인 항목
```

해당 API에 존재하지 않는 항목은
임의로 생성하지 않는다.

분석이 완료되면:

```text
STOP
```

한다.

사용자가 명시적으로 선택하지 않은
다른 Backend API나
Backend Business Logic 분석으로
자동 진행하지 않는다.
---
name: api-analysis
description: FE 분석에서 선택한 Backend API 하나를 기준으로 실제 Source를 추적하여 API Contract 문서를 생성한다.
argument-hint: "<화면명> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# API Analysis Skill

## 1. 역할

이 Skill은 FE 분석 결과에서 사용자가 선택한 Backend API 하나를 기준으로
Frontend API 호출 Source와 Backend Controller / DTO / Validation을 추적하여
API Contract 문서를 생성한다.

이 단계의 목적은 Backend Business Logic 분석이 아니다.

분석 범위는 다음과 같다.

```text
Frontend API Call
        ↓
Controller Mapping
        ↓
Request Contract
        ↓
Request DTO
        ↓
Validation
        ↓
Response Contract
        ↓
Response DTO
        ↓
Error Response
        ↓
Source Evidence
        ↓
STOP
```

Service 이하의 Backend 내부 구현은 분석하지 않는다.


## 2. 입력

사용자는 다음 형식으로 Skill을 실행한다.

```text
/api-analysis <화면명> <HTTP Method|UNKNOWN> <Backend URL>
```

HTTP Method를 알고 있는 경우:

```text
/api-analysis equipment-search POST /api/equipment/search
```

HTTP Method를 모르는 경우:

```text
/api-analysis equipment-search UNKNOWN /api/equipment/search
```

입력값:

```text
화면명
HTTP Method 또는 UNKNOWN
Backend URL
```

화면명은 해당 화면의 기존 분석 문서를 찾고
API 문서 저장 위치를 결정하는 기준으로 사용한다.

Orchestrator 또는 Worker를 통해 실행되는 경우
다음 값이 추가로 전달될 수 있다.

```text
API ID
Output File
```

예:

```text
화면명:
equipment-search

HTTP Method:
POST

Backend URL:
/api/equipment/create

API ID:
API-001

Output File:
FE-ACT-010-create-API-001.md
```

`API ID`와 `Output File`은
Orchestrator / Worker 실행을 위한 선택 입력이다.

전달되지 않은 경우에는
기존 직접 실행 방식을 그대로 사용한다.

### HTTP Method UNKNOWN 처리

HTTP Method를 모르는 경우 `UNKNOWN`을 입력할 수 있다.

Method가 `UNKNOWN`인 경우
사용자에게 Method를 다시 요청하지 않는다.

Frontend API 호출 Source와
Backend Controller Mapping을 탐색하여
실제 HTTP Method를 확인한다.

실제 Source에서 Method가 확인되면
확인된 Method를 API Contract에 기록한다.

예:

```text
@GetMapping     → GET
@PostMapping    → POST
@PutMapping     → PUT
@PatchMapping   → PATCH
@DeleteMapping  → DELETE
```

`@RequestMapping`을 사용하는 경우
실제 `method` 속성을 확인한다.

URL만으로 HTTP Method를 추측하지 않는다.

동일 URL에 여러 HTTP Method가 존재하여
대상 API를 하나로 확정할 수 없는 경우
임의로 하나를 선택하지 않는다.

이 경우 확인된 후보를 출력하고 분석을 STOP 한다.

실제 Source에서도 Method를 확인할 수 없는 경우:

```text
HTTP Method: 확인되지 않음
```

으로 기록한다.


## 3. 적용 규칙

분석 시 반드시 다음 Rule을 적용한다.

```text
.claude/rules/06-api-analysis-scope.md
.claude/rules/07-api-contract-tracing.md
```

두 Rule의 분석 범위,
Evidence,
Source 탐색 기준,
외부 Dependency 제한,
STOP 조건을 준수한다.

Rule과 이 Skill의 내용이 중복되는 경우
Rule의 세부 분석 기준을 우선 적용한다.


## 4. 기존 FE 분석 결과 확인

먼저 다음 경로에서 해당 화면의 FE 분석 결과를 확인한다.

```text
docs/analysis/{화면명}/frontend/
```

전체 프로젝트를 다시 분석하지 않는다.

입력받은 Backend URL과 일치하는
Backend API 후보를 FE 분석 문서에서 찾는다.

HTTP Method가 입력된 경우:

```text
HTTP Method + Backend URL
```

을 함께 비교한다.

HTTP Method가 `UNKNOWN`인 경우:

```text
Backend URL
```

을 우선 기준으로 후보를 찾고
실제 Source에서 Method를 확인한다.

확인해야 할 정보:

```text
FE Action
Frontend Source Path
Frontend API 호출 Function
HTTP Method
Backend URL
Query Parameter
Path Parameter
Request Body
호출 조건
Response 처리 위치
```

FE 분석 문서는 탐색 시작점으로 사용한다.

FE 분석 문서의 내용만으로
API Contract를 확정하지 않는다.

최종 API Contract는 실제 Source를 확인하여 결정한다.


## 5. Source 탐색 기준

Code Index MCP를 사용하여 관련 Source를 탐색한다.

Code Index는 Source 탐색 및 범위 축소를 위한 도구이다.

최종 판단은 실제 Source 내용을 기준으로 한다.

Source 탐색은 현재 Code Index Project Root를 기준으로 수행한다.

사용자에게 절대경로를 요청하지 않는다.

절대경로를 추측하거나 생성하지 않는다.

Source Evidence에는
Project Root 기준 상대경로를 사용한다.


## 6. Frontend / Backend 프로젝트 구분

현재 Project Root 아래에는
여러 Frontend / Backend 프로젝트가 존재할 수 있다.

프로젝트 구분 기준:

```text
Backend
gipms-api-*

Frontend
gipms-* 중 gipms-api-* 제외
```

판정 우선순위:

```text
1. gipms-api-* → Backend
2. 나머지 gipms-* → Frontend
```

`gipms-api-*`는 `gipms-*` 패턴에도 포함되므로
반드시 Backend 여부를 먼저 판정한다.

프로젝트 이름만으로 실제 호출 대상을 확정하지 않는다.

Backend URL,
HTTP Method,
실제 Source Evidence를 기준으로
관련 프로젝트를 식별한다.


## 7. Frontend Source 탐색

Frontend에서는 입력받은 API를
실제로 호출하는 Source를 찾는다.

탐색 대상:

```text
gipms-*
```

단:

```text
gipms-api-*
```

는 Frontend 탐색 대상에서 제외한다.

HTTP Method를 알고 있는 경우:

```text
HTTP Method
+
Backend URL
```

을 기준으로 관련 호출 Source를 탐색한다.

HTTP Method가 `UNKNOWN`인 경우:

```text
Backend URL
```

을 기준으로 관련 호출 Source를 먼저 찾는다.

확인 항목:

```text
API 호출 Source
호출 Function
HTTP Method
URL
Header
Path Parameter
Query Parameter
Request Body
Request 생성 위치
```

Frontend Source에서 HTTP Method가 확인되면
그 값을 Method 후보로 기록한다.

FE Business Logic 전체를 다시 분석하지 않는다.

API Contract 확인에 필요한
호출 Source만 확인한다.


## 8. Backend Controller 탐색

Backend에서는 다음 범위에서
대상 API의 Controller Mapping을 찾는다.

```text
gipms-api-*
```

### HTTP Method가 확인된 경우

다음 조합을 기준으로 검색한다.

```text
HTTP Method + Backend URL
```

예:

```text
POST + /api/equipment/search
```

### HTTP Method가 UNKNOWN인 경우

먼저 다음을 기준으로 Controller Mapping 후보를 찾는다.

```text
Backend URL
```

후보 Controller의 실제 Mapping Annotation을 확인하여
HTTP Method를 식별한다.

예:

```text
@GetMapping     → GET
@PostMapping    → POST
@PutMapping     → PUT
@PatchMapping   → PATCH
@DeleteMapping  → DELETE
```

`@RequestMapping`인 경우
`method` 속성을 실제 Source에서 확인한다.

URL만으로 HTTP Method를 추측하지 않는다.

### Class / Method Mapping 조합

Class-level Mapping과
Method-level Mapping이 분리되어 있다면
실제 최종 URL을 조합하여 확인한다.

예:

```text
@RequestMapping("/api/equipment")

@PostMapping("/search")
```

최종 Mapping:

```text
POST /api/equipment/search
```

실제 Mapping이 확인된 Backend 프로젝트만
이후 분석한다.

관련 없는 `gipms-api-*` 프로젝트를
깊게 분석하지 않는다.


## 9. HTTP Method 교차 확인

HTTP Method가 `UNKNOWN`으로 입력된 경우
가능하면 다음 Evidence를 교차 확인한다.

```text
Frontend API 호출 Source
        ↕
Backend Controller Mapping
```

두 Source에서 동일한 Method가 확인되면
해당 Method를 실제 API Method로 기록한다.

예:

```text
Frontend
axios.post(...)

Backend
@PostMapping(...)

결과
HTTP Method: POST
```

Frontend와 Backend에서 확인된 Method가 다르면
임의로 하나를 선택하지 않는다.

다음과 같이 기록한다.

```text
HTTP Method 불일치

Frontend:
...

Backend:
...
```

그리고 해당 불일치를
확인 필요 항목으로 기록한다.

동일 URL에 여러 Controller Mapping이 존재하고
Method까지 하나로 특정할 수 없는 경우
후보를 출력한 후 STOP 한다.


## 10. Request 전달 방식 분석

Controller Source를 기준으로
Request 전달 방식을 구분한다.

구분:

```text
Header
Path Parameter
Query Parameter
Request Body
```

각 항목에 대해 실제 Source에서 확인 가능한 경우
다음을 분석한다.

```text
Name
JSON Name
Java Type
Required
Default
Validation
Allowed Value
Nullable
Nested Type
Collection 여부
```

Frontend에서 항상 전달한다는 이유만으로
Backend Required라고 판단하지 않는다.

Required 여부는 실제 Backend Source Evidence를 기준으로 판단한다.


## 11. Request DTO 분석

Controller에서 Request DTO가 확인되면
해당 DTO Source까지 추적한다.

다음을 확인한다.

```text
DTO Class
Field
JSON Field Name
Java Type
Required
Validation
Default
Nullable
Nested DTO
Collection
Enum
```

Nested DTO가 API Contract의 일부라면
필요한 범위까지 구조를 확장한다.

Collection 내부 DTO도
실제 Client Contract에 포함되는 경우 구조를 확인한다.

DTO Class 이름만 출력하고 종료하지 않는다.

실제 Field 구조를 확인한다.


## 12. Validation 분석

실제 Validation Evidence를 확인한다.

예:

```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Pattern
@Valid
@RequestParam(required = true)
Custom Validator
```

Custom Validator가 존재하는 경우
API 입력 검증 규칙을 확인하기 위한 범위까지만 추적한다.

Custom Validator가:

```text
Service
DB
Business Logic
```

으로 연결되면 그 경계에서 STOP 한다.

Validation을 Annotation 이름이나
Method 이름만 보고 추측하지 않는다.


## 13. Response Contract 분석

Controller의 실제 반환 선언을 확인한다.

예:

```text
ResponseEntity<SearchResponse>
SearchResponse
ApiResponse<SearchResponse>
List<SearchResponse>
Page<SearchResponse>
```

Response DTO가 Source에서 확인되면
실제 Field 구조까지 추적한다.

확인 항목:

```text
Response Wrapper
Response DTO
Field
Type
JSON Name
Nested DTO
Collection
Generic Type
```

Nested Response DTO가
실제 Client Contract의 일부라면 구조를 확장한다.

Response DTO Class 이름만 확인하고 종료하지 않는다.

실제 Client Contract를 구성하는 Field까지 확인한다.


## 14. Response 확인 불가 처리

Controller Source만으로
실제 Response DTO를 확인할 수 없는 경우가 있을 수 있다.

예:

```text
ResponseEntity<?>
Object
Map
Service 결과 직접 반환
```

구체적인 Response 구조를 확인하기 위해
Service 내부 분석이 필요한 경우
Service로 진입하지 않는다.

이 경우:

```text
Response DTO: 확인되지 않음

사유:
Controller / DTO Source 범위에서
구체적인 Response 구조 확인 불가
```

로 기록한다.

Response 구조를 추측하여 생성하지 않는다.


## 15. HTTP Status 분석

HTTP Status는
실제 Source Evidence가 있는 경우에만 확정한다.

예:

```text
@ResponseStatus
ResponseEntity.status(...)
ResponseEntity.ok(...)
ExceptionHandler
ControllerAdvice
```

명시적인 Status를 확인할 수 없는 경우:

```text
명시적 HTTP Status: 확인되지 않음
```

으로 기록한다.

Framework 기본 동작을
실제 Source-defined Contract처럼 표현하지 않는다.


## 16. Error Response 분석

현재 API와 직접 관련된
Error Response를 확인한다.

필요한 경우 다음 범위까지 추적할 수 있다.

```text
Controller
Validation
ExceptionHandler
ControllerAdvice
Error Response DTO
```

프로젝트 전체 Exception 체계를 분석하지 않는다.

현재 API Contract 확인에 필요한 범위만 추적한다.

Error Response 확인을 위해
Service 내부로 진입하지 않는다.


## 17. Frontend / Backend Mapping

Frontend에서 전송하는 Request와
Backend에서 수신하는 Request를 비교한다.

기본 흐름:

```text
Frontend Field
        ↓
전송 위치
Header / Path / Query / Body
        ↓
Backend Parameter / DTO Field
        ↓
Type
        ↓
Required / Validation
```

다음과 같은 차이가 있으면 명확히 기록한다.

```text
Field Name 차이
Type 차이
Frontend 전송하지만 Backend Source에서 확인되지 않는 값
Backend Contract에 있으나 FE 호출에서 확인되지 않는 값
Optional / Required 차이
Runtime 관찰값과 Source Contract 차이
```


## 18. Runtime Evidence

필요한 경우 Chrome DevTools MCP를 사용하여
실제 Runtime Request / Response를 교차 확인할 수 있다.

Chrome DevTools는
API Contract의 유일한 근거가 아니다.

Runtime에서 확인된 값은
해당 실행 시점에서 관찰된 사례로 취급한다.

예:

```text
plantCode = "1000"
```

가 Runtime에서 확인되었다고 해서:

```text
plantCode의 허용값은 "1000"뿐이다
```

라고 판단하지 않는다.

Runtime Evidence와 Source Contract가 다르면
둘을 구분하여 기록한다.


## 19. Source Evidence

주요 분석 결과에는 가능한 경우
다음 Evidence를 기록한다.

```text
Source Path
Class / Function / Field
Line Range
```

Source Path는
Project Root 기준 상대경로를 사용한다.

예:

```text
gipms-xxx/src/...
gipms-api-xxx/src/...
```

Windows 절대경로를
Source Evidence로 사용하지 않는다.

Line Range를 실제로 확인할 수 없는 경우
임의로 생성하지 않는다.

이 경우:

```text
Line Range: 확인되지 않음
```

으로 기록한다.

파일명,
Class명,
Method명,
Code Index 검색 결과만으로
API Contract를 확정하지 않는다.


## 20. JAR / 외부 Dependency 탐색 금지

API Contract 분석은
현재 Code Index Project Root 아래의
실제 Frontend / Backend Source를 기준으로 수행한다.

다음 영역을 자동으로 탐색하지 않는다.

```text
*.jar
JAR 내부 Class
Decompiled Class
Maven Repository
.m2/
Gradle Cache
.gradle/
외부 Dependency Source
설치된 Library Source
Project Root 외부 Source
```

다음 행위를 수행하지 않는다.

```text
JAR 파일 검색
JAR 압축 해제
JAR 내부 Class 탐색
Class Decompile
Maven Dependency Cache 탐색
Gradle Dependency Cache 탐색
외부 Library 구현체 자동 추적
Project Root 밖으로 이동하여 Source 탐색
```

Controller,
Request DTO,
Response DTO,
Validation,
Error Response가
현재 Project Source에서 확인되지 않더라도
JAR 또는 외부 Dependency로 탐색 범위를 확장하지 않는다.

확인할 수 없는 경우:

```text
현재 Project Source에서 확인되지 않음
```

으로 기록한다.

단:

```text
gipms-api-common
```

등 현재 Project Root 아래에
실제 Source 형태로 존재하는 공통 프로젝트는
외부 Dependency로 취급하지 않는다.

실제 Source Evidence가 확인되는 경우
API Contract에 필요한 범위까지 추적할 수 있다.

외부 Dependency 분석은
사용자가 명시적으로 요청한 경우에만 수행한다.


## 21. 분석 금지 영역

API 분석 단계에서는
다음 영역을 분석하지 않는다.

```text
Service 내부
ServiceImpl 내부
Business Logic
Mapper
MyBatis Mapper XML
SQL
Oracle
SAP 내부 로직
JAR 내부 구현
외부 Dependency 내부 구현
```

Controller에서 Service 호출이 발견되어도
호출 사실까지만 확인한다.

Service Method 내부로 진입하지 않는다.


## 22. API ID 결정

API 분석 문서를 생성할 때
API ID를 결정한다.

API ID 형식:

```text
API-001
API-002
API-003
...
```

API ID 결정 방식은
실행 방식에 따라 구분한다.


### 22.1 Orchestrator / Worker에서 API ID를 전달한 경우

Orchestrator 또는 Worker가
API ID를 명시적으로 전달한 경우
전달받은 API ID를 그대로 사용한다.

예:

```text
API ID:
API-001
```

이 경우:

```text
API-001
```

을 그대로 사용한다.

새로운 API ID를 생성하지 않는다.

다음 작업을 수행하지 않는다.

```text
기존 API 문서에서 최대 API ID 검색
최근 생성된 API 문서를 기준으로 ID 결정
파일 생성 순서를 기준으로 ID 변경
다른 Worker의 API ID 확인
전달받은 API ID 재배정
```

Worker 실행 순서나
분석 완료 순서에 따라
API ID를 변경하지 않는다.


### 22.2 API ID를 전달받지 않은 경우

기존 직접 실행:

```text
/api-analysis <화면명> <HTTP Method|UNKNOWN> <Backend URL>
```

에서는 기존 방식대로
현재 화면의 기존 API 문서를 확인하여
이미 사용 중인 API ID와 중복되지 않도록
API ID를 결정한다.

저장 위치:

```text
docs/analysis/{화면명}/api/
```

예:

```text
docs/analysis/equipment-search/api/
├─ API-001-equipment-search.md
├─ API-002-equipment-detail.md
└─ ...
```

동일한 Method + URL의 API 문서가 이미 존재하는 경우
새 API ID를 임의로 생성하지 않는다.

HTTP Method가 `UNKNOWN`으로 입력되었지만
분석 과정에서 Method가 확인된 경우:

```text
확인된 Method + URL
```

을 기준으로 기존 문서 중복 여부를 다시 확인한다.


### 22.3 API ID 결정 우선순위

```text
1순위
Orchestrator / Worker가 전달한 API ID

2순위
기존 API Analysis의 API ID 생성 규칙
```

명시적으로 전달된 API ID가 있으면
항상 전달받은 값을 우선한다.


## 23. 출력 파일명

출력 파일명은
실행 방식에 따라 결정한다.


### 23.1 Orchestrator / Worker에서 Output File을 전달한 경우

Orchestrator 또는 Worker가
Output File을 명시적으로 전달한 경우
전달받은 파일명을 그대로 사용한다.

예:

```text
API ID:
API-001

Output File:
FE-ACT-010-create-API-001.md
```

최종 파일명:

```text
FE-ACT-010-create-API-001.md
```

이 경우 기존 파일명 생성 규칙:

```text
{API ID}-{기능명}.md
```

을 적용하지 않는다.

API ID 또는 기능명을 이용하여
파일명을 다시 생성하지 않는다.


### 23.2 Output File을 전달받지 않은 경우

기존 직접 실행에서는
기존 파일명 규칙을 그대로 사용한다.

파일명 형식:

```text
{API ID}-{기능명}.md
```

예:

```text
API-001-equipment-search.md
```

잘못된 예:

```text
API-API-001-equipment-search.md
```

API ID 자체에 이미 `API-` Prefix가 포함되어 있으므로
Prefix를 중복하지 않는다.


### 23.3 출력 파일명 결정 우선순위

```text
1순위
Orchestrator / Worker가 전달한 Output File

2순위
기존 {API ID}-{기능명}.md 규칙
```

Output File이 명시적으로 전달된 경우
반드시 해당 파일명을 사용한다.


## 24. 출력 문서 필수 내용

API Contract 문서에는
최소한 다음 내용이 포함되어야 한다.

```text
API ID
기능명
HTTP Method
Backend URL

Frontend API 호출 Source

Controller Mapping

Request Contract
├─ Header
├─ Path Parameter
├─ Query Parameter
└─ Request Body

Request DTO
├─ Field
├─ Type
├─ Required
└─ Validation

Response Contract
├─ Response Wrapper
├─ Response DTO
├─ Field
├─ Type
└─ Nested 구조

HTTP Status

Error Response

Frontend ↔ Backend Mapping

Source Evidence

확인되지 않은 항목

분석 경계
```

HTTP Method가 처음에 `UNKNOWN`이었더라도
Source에서 확인된 경우
최종 문서에는 실제 Method를 기록한다.

끝까지 확인할 수 없는 경우:

```text
HTTP Method: 확인되지 않음
```

으로 기록한다.


## 25. Reference 적용

API Reference가 존재하는 경우:

```text
.claude/references/API-REFERENCE.md
```

를 최종 문서의
표현 형식과 상세 수준을 위한 Template으로 사용한다.

Reference는 Evidence가 아니다.

Reference의 다음 내용을
현재 분석 결과로 복사하지 않는다.

```text
API URL
HTTP Method
Parameter
DTO
Field
Validation
Response
Source Path
Class
Function
Line Range
Runtime Value
```

현재 Source에서 확인된 내용만
실제 문서에 기록한다.

API Reference가 아직 존재하지 않는 경우
이 Skill의 필수 출력 구조를 기준으로 문서를 생성한다.


## 26. 미확인 항목 처리

Source에서 확인되지 않는 내용을 추측하지 않는다.

다음 표현을 사용한다.

```text
확인되지 않음
현재 Project Source에서 확인되지 않음
명시적 HTTP Status: 확인되지 않음
```

확인되지 않은 내용을
일반적인 Spring 동작이나 경험을 근거로 채우지 않는다.


## 27. 문서 생성

분석이 완료되면
API Contract 문서를 생성한다.

기본 저장 디렉터리:

```text
docs/analysis/{화면명}/api/
```

필요한 경우:

```text
docs/analysis/{화면명}/api/
```

디렉터리를 생성한다.


### 27.1 Output File이 전달된 경우

Orchestrator 또는 Worker가
Output File을 전달한 경우:

```text
docs/analysis/{화면명}/api/{Output File}
```

에 저장한다.

예:

```text
화면명:
equipment-search

API ID:
API-001

Output File:
FE-ACT-010-create-API-001.md
```

최종 저장 위치:

```text
docs/analysis/equipment-search/api/FE-ACT-010-create-API-001.md
```

Output File이 전달된 경우
다른 이름으로 변경하지 않는다.


### 27.2 Output File이 전달되지 않은 경우

기존 직접 실행에서는
기존 파일명 규칙을 사용한다.

```text
docs/analysis/{화면명}/api/{API ID}-{기능명}.md
```

예:

```text
docs/analysis/equipment-search/api/API-001-equipment-search.md
```


### 27.3 기존 파일 보호

최종 저장 대상 파일이 이미 존재하는 경우
사용자의 명시적인 지시 없이
자동으로 덮어쓰지 않는다.

기존 문서를 삭제하거나
다른 API 문서로 대체하지 않는다.


### 27.4 문서 생성 제한

분석 중간 결과를
별도 문서로 생성하지 않는다.

최종 API Contract 문서 하나만 생성한다.


## 28. 최종 검증

문서를 저장하기 전에 다음을 확인한다.

```text
[ ] 입력한 Backend URL과 Controller Mapping이 일치하는가

[ ] Method가 입력된 경우 실제 Source와 일치하는가

[ ] Method가 UNKNOWN인 경우 실제 Source에서 Method 확인을 시도했는가

[ ] Method를 URL이나 이름만으로 추측하지 않았는가

[ ] 실제 Frontend API 호출 Source를 확인했는가

[ ] Request 전달 방식을
    Header / Path / Query / Body로 구분했는가

[ ] Request DTO의 실제 Field까지 확인했는가

[ ] Validation Evidence를 확인했는가

[ ] 확인 가능한 Response DTO의 실제 Field까지 확인했는가

[ ] Response 확인을 위해 Service 내부로 진입하지 않았는가

[ ] HTTP Status를 추측하지 않았는가

[ ] Error Response를 현재 API 범위에서만 확인했는가

[ ] Frontend ↔ Backend Mapping을 확인했는가

[ ] Source Evidence가 실제 Source에 근거하는가

[ ] Source Path가 Project Root 기준 상대경로인가

[ ] Line Range를 임의로 생성하지 않았는가

[ ] JAR을 검색하지 않았는가

[ ] JAR 내부 Class를 탐색하지 않았는가

[ ] Decompiled Class를 사용하지 않았는가

[ ] Maven Repository / .m2를 탐색하지 않았는가

[ ] Gradle Cache / .gradle을 탐색하지 않았는가

[ ] Project Root 외부 Source를 탐색하지 않았는가

[ ] Service / ServiceImpl 내부로 진입하지 않았는가

[ ] Mapper / MyBatis XML / SQL / Oracle을 분석하지 않았는가

[ ] Orchestrator / Worker에서 API ID를 전달한 경우
    전달받은 API ID를 그대로 사용했는가

[ ] Orchestrator / Worker에서 Output File을 전달한 경우
    전달받은 파일명을 그대로 사용했는가

[ ] API ID를 전달받지 않은 일반 실행인 경우
    API ID가 기존 문서와 중복되지 않는가
```


## 29. STOP

API Contract 문서 생성이 완료되면
반드시 STOP 한다.

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
API Contract Document 생성
        ↓
저장
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
JAR
외부 Dependency
```

API 분석 완료 후
BE 분석을 자동으로 시작하지 않는다.

사용자가 API 문서를 확인하고
직접 Backend 분석 대상을 선택할 때까지 대기한다.

단, Orchestrator / Worker에 의해 실행된 경우에도
이 Skill 자체가 BE 분석을 직접 시작하지 않는다.

API Analysis 완료 후 STOP하고
생성된 API Document와 API ID를
호출한 Orchestrator / Worker가 다음 단계에서 사용할 수 있도록 한다.
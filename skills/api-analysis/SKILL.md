---
name: api-analysis
description: FE 분석에서 선택한 Backend API 하나를 기준으로 실제 Source를 추적하여 API Contract 문서를 생성한다.
argument-hint: "<화면명> <HTTP Method> <Backend URL>"
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
/api-analysis <화면명> <HTTP Method> <Backend URL>
```

예:

```text
/api-analysis equipment-search POST /api/equipment/search
```

입력값:

```text
화면명
HTTP Method
Backend URL
```

화면명은 해당 화면의 기존 분석 문서를 찾고
API 문서 저장 위치를 결정하는 기준으로 사용한다.


## 3. 적용 규칙

분석 시 반드시 다음 Rule을 적용한다.

```text
.claude/rules/06-api-analysis-scope.md
.claude/rules/07-api-contract-tracing.md
```

두 Rule의 분석 범위, Evidence, 탐색 제한 및 STOP 조건을 준수한다.

Rule과 이 Skill의 내용이 중복되는 경우
Rule의 세부 분석 기준을 우선 적용한다.


## 4. 기존 FE 분석 결과 확인

먼저 다음 경로에서 해당 화면의 FE 분석 결과를 확인한다.

```text
docs/analysis/{화면명}/frontend/
```

전체 프로젝트를 다시 분석하지 않는다.

입력받은 HTTP Method + Backend URL과 일치하는
Backend API 후보를 FE 분석 문서에서 찾는다.

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

FE 분석 문서의 내용만으로 API Contract를 확정하지 않는다.

최종 API Contract는 실제 Source를 확인하여 결정한다.


## 5. Source 탐색 기준

Code Index MCP를 사용하여 관련 Source를 탐색한다.

Code Index는 Source 탐색 및 범위 축소를 위한 도구이다.

최종 판단은 실제 Source 내용을 기준으로 한다.

Source 탐색은 현재 Code Index Project Root를 기준으로 수행한다.

절대경로를 사용자에게 요청하지 않는다.

절대경로를 추측하거나 생성하지 않는다.

Source Evidence에는 Project Root 기준 상대경로를 사용한다.


## 6. Frontend / Backend 프로젝트 구분

현재 Project Root 아래에는 여러 Frontend / Backend 프로젝트가 존재할 수 있다.

프로젝트 구분 기준:

```text
Backend
gipms-api-*

Frontend
gipms-* 중 gipms-api-* 제외
```

프로젝트 이름만으로 실제 호출 대상을 확정하지 않는다.

HTTP Method + Backend URL 및 실제 Source Evidence를 기준으로
관련 프로젝트를 찾는다.


## 7. Frontend Source 탐색

Frontend에서는 입력받은 API를 실제 호출하는 Source를 찾는다.

탐색 대상:

```text
gipms-*
```

단:

```text
gipms-api-*
```

는 Frontend 탐색 대상에서 제외한다.

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

FE Business Logic 전체를 다시 분석하지 않는다.

API Contract 확인에 필요한 호출 Source만 확인한다.


## 8. Backend Controller 탐색

Backend에서는 다음 범위에서 대상 API의 Controller Mapping을 찾는다.

```text
gipms-api-*
```

우선 다음 조합을 기준으로 검색한다.

```text
HTTP Method + Backend URL
```

예:

```text
POST + /api/equipment/search
```

Class-level Mapping과 Method-level Mapping이 분리되어 있다면
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

실제 Mapping이 확인된 Backend 프로젝트만 이후 분석한다.

관련 없는 `gipms-api-*` 프로젝트를 깊게 분석하지 않는다.


## 9. Request 전달 방식 분석

Controller Source를 기준으로 Request 전달 방식을 구분한다.

구분:

```text
Header
Path Parameter
Query Parameter
Request Body
```

각 항목에 대해 실제 Source에서 확인 가능한 경우 다음을 분석한다.

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


## 10. Request DTO 분석

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

DTO Class 이름만 출력하고 종료하지 않는다.

실제 Field 구조를 확인한다.


## 11. Validation 분석

다음과 같은 실제 Validation Evidence를 확인한다.

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

Custom Validator가 Service / DB / Business Logic으로 연결되면
그 경계에서 STOP 한다.

Validation을 이름만 보고 추측하지 않는다.


## 12. Response Contract 분석

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

Response DTO Class 이름만 확인하고 종료하지 않는다.

실제 Client Contract를 구성하는 Field까지 확인한다.


## 13. Response 확인 불가 처리

다음과 같이 Controller Source만으로
실제 Response DTO를 확인할 수 없는 경우가 있을 수 있다.

예:

```text
ResponseEntity<?>
Object
Map
service.method() 결과 직접 반환
```

구체적인 Response 구조를 확인하기 위해
Service 내부 분석이 필요한 경우 Service로 진입하지 않는다.

이 경우:

```text
Response DTO: 확인되지 않음
사유: Controller/DTO Source 범위에서 구체적인 Response 구조 확인 불가
```

로 기록한다.

Response를 추측하여 생성하지 않는다.


## 14. HTTP Status 분석

HTTP Status는 실제 Source Evidence가 있는 경우에만 확정한다.

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

Framework 기본 동작을 실제 Source-defined Contract처럼 표현하지 않는다.


## 15. Error Response 분석

현재 API와 직접 관련된 Error Response를 확인한다.

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


## 16. Frontend / Backend Mapping

Frontend에서 전송하는 Request와
Backend에서 수신하는 Request를 비교한다.

확인 항목:

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


## 17. Runtime Evidence

필요한 경우 Chrome DevTools MCP를 사용하여
실제 Runtime Request / Response를 교차 확인할 수 있다.

Chrome DevTools는 API Contract의 유일한 근거가 아니다.

Runtime에서 확인된 값은
해당 실행 시점에서 관찰된 사례로 취급한다.

예:

```text
plantCode = "1000"
```

가 Runtime에서 확인되었다고 해서

```text
plantCode의 허용값은 "1000"뿐이다
```

라고 판단하지 않는다.

Contract는 실제 Source와 함께 판단한다.


## 18. Source Evidence

주요 분석 결과에는 가능한 경우 다음 Evidence를 기록한다.

```text
Source Path
Class / Function / Field
Line Range
```

Source Path는 Project Root 기준 상대경로를 사용한다.

예:

```text
gipms-xxx/src/...
gipms-api-xxx/src/...
```

Line Range를 실제로 확인할 수 없는 경우
임의로 생성하지 않는다.

이 경우:

```text
Line Range: 확인되지 않음
```

으로 기록한다.

파일명, Class명, Method명 또는 Code Index 검색 결과만으로
API Contract를 확정하지 않는다.


## 19. JAR / 외부 Dependency 탐색 금지

현재 Project Root 아래의 실제 Source를 우선한다.

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
Project Root 외부 Source
```

다음 행위를 수행하지 않는다.

```text
JAR 검색
JAR 압축 해제
Class Decompile
Maven Dependency Cache 탐색
Gradle Dependency Cache 탐색
외부 Library 구현체 자동 추적
```

현재 Project Source에서 Contract를 확인할 수 없다면
외부 Dependency로 탐색 범위를 확장하지 않는다.

다음과 같이 기록한다.

```text
현재 Project Source에서 확인되지 않음
```

단:

```text
gipms-api-common
```

등 현재 Project Root 아래에 실제 Source 형태로 존재하는 공통 프로젝트는
외부 Dependency로 취급하지 않는다.

실제 Source Evidence가 확인되는 경우
API Contract에 필요한 범위까지 추적할 수 있다.


## 20. 분석 금지 영역

API 분석 단계에서는 다음 영역을 분석하지 않는다.

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


## 21. API ID 생성

API 분석 문서를 생성할 때 API ID를 부여한다.

형식:

```text
API-001
API-002
API-003
...
```

현재 화면의 기존 API 문서를 확인하여
이미 사용 중인 API ID와 중복되지 않도록 한다.

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

기존 문서 존재 여부를 먼저 확인한다.


## 22. 출력 파일명

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


## 23. 출력 문서 필수 내용

API Contract 문서에는 최소한 다음 내용이 포함되어야 한다.

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


## 24. Reference 적용

API Reference가 존재하는 경우:

```text
.claude/references/API-REFERENCE.md
```

를 최종 문서의 표현 형식과 상세 수준을 위한 Template으로 사용한다.

Reference는 Evidence가 아니다.

Reference의 다음 내용을 현재 분석 결과로 복사하지 않는다.

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

현재 Source에서 확인된 내용만 실제 문서에 기록한다.

API Reference가 아직 존재하지 않는 경우
이 Skill의 필수 출력 구조를 기준으로 문서를 생성한다.


## 25. 미확인 항목 처리

Source에서 확인되지 않는 내용을 추측하지 않는다.

다음 표현을 사용한다.

```text
확인되지 않음
현재 Project Source에서 확인되지 않음
명시적 HTTP Status 확인되지 않음
```

확인되지 않은 내용을
일반적인 Spring 동작이나 경험을 근거로 채우지 않는다.


## 26. 문서 생성

분석이 완료되면 다음 위치에 API 문서를 생성한다.

```text
docs/analysis/{화면명}/api/{API ID}-{기능명}.md
```

필요한 경우 `api/` 디렉터리를 생성한다.

분석 중간 결과를 별도 문서로 생성하지 않는다.

최종 API Contract 문서 하나만 생성한다.


## 27. 최종 검증

문서를 저장하기 전에 다음을 확인한다.

```text
[ ] 입력한 Method + URL과 Controller Mapping이 일치하는가
[ ] 실제 Frontend API 호출 Source를 확인했는가
[ ] Request 전달 방식을 구분했는가
[ ] Request DTO의 실제 Field까지 확인했는가
[ ] Validation Evidence를 확인했는가
[ ] 확인 가능한 Response DTO의 실제 Field까지 확인했는가
[ ] HTTP Status를 추측하지 않았는가
[ ] Error Response를 현재 API 범위에서만 확인했는가
[ ] Frontend ↔ Backend Mapping을 확인했는가
[ ] Source Evidence가 실제 Source에 근거하는가
[ ] Source Path가 Project Root 기준 상대경로인가
[ ] Line Range를 임의로 생성하지 않았는가
[ ] JAR / Maven / Gradle Cache를 탐색하지 않았는가
[ ] Service 내부로 진입하지 않았는가
[ ] Mapper / MyBatis / SQL / Oracle을 분석하지 않았는가
[ ] API ID가 기존 문서와 중복되지 않는가
```


## 28. STOP

API Contract 문서 생성이 완료되면 반드시 STOP 한다.

```text
API Contract Document 생성
        ↓
저장
        ↓

══════════════ STOP ══════════════

Backend Business Logic
Service / ServiceImpl
Mapper
MyBatis XML
SQL
Oracle
SAP
```

API 분석 완료 후
BE 분석을 자동으로 시작하지 않는다.

사용자가 API 문서를 확인하고
직접 Backend 분석 대상을 선택할 때까지 대기한다.
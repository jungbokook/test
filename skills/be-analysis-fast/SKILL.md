---
name: be-analysis-fast
description: Backend URL 하나를 Strict Source Boundary 기반으로 빠르게 전체 분석하고 최종 문서 생성에 필요한 구조화된 분석 결과를 Worker에게 반환한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Analysis FAST - Common Analysis Engine

## 0. 역할

이 Skill은 Backend URL 하나를 분석하는 공통 분석 Engine이다.

담당:

Controller
→ Service
→ ServiceImpl
→ Validation
→ Branch
→ Local/private Method
→ 다른 Business Service
→ Mapper
→ MyBatis XML
→ SQL
→ External Integration
→ Post Processing
→ Response
→ Exception

이 Skill은 최종 Markdown 파일을 생성하지 않는다.

최종 문서 Format 및 Write는 Worker가 담당한다.

---

# 1. 입력

HTTP_METHOD

BACKEND_URL

예:

POST

/material/create

---

# 2. 절대 하지 않는 것

이 Skill은 다음을 하지 않는다.

- BE ID 계산
- 기존 BE 파일 검색
- 다음 BE 번호 계산
- OUTPUT_PATH 계산
- backend 디렉토리 탐색
- 다른 Backend URL 분석
- 다른 Worker 상태 확인
- 최종 Markdown Write
- BE-REFERENCE Format 임의 생성

---

# 3. 핵심 원칙

검색 범위는 좁게,
호출 깊이는 끝까지.

실제 실행 경로만 분석한다.

추측 기반 주변 Source 탐색을 하지 않는다.

---

# 4. SOURCE SEARCH BOUNDARY

Java:

gipms-api-*/src/main/java/**

Resource:

gipms-api-*/src/main/resources/**

이 범위 밖으로 확장하지 않는다.

---

# 5. HARD EXCLUDE

절대 검색하지 않는다.

- **/*.jar
- **/*.class
- **/target/**
- **/build/**
- **/.gradle/**
- **/.m2/**
- **/libs/**
- **/lib/**
- **/BOOT-INF/**
- **/WEB-INF/lib/**
- **/node_modules/**
- **/generated/**
- **/test/**
- **/tests/**
- **/sample/**
- **/samples/**
- docs/**
- .git/**

JAR fallback은 절대 금지한다.

---

# 6. SOURCE NOT FOUND

허용 Source에서 찾지 못하면:

SOURCE_NOT_FOUND

또는

EXTERNAL_OR_DEPENDENCY

또는

DEPENDENCY_BOUNDARY

로 기록한다.

검색 범위를 넓히지 않는다.

---

# 7. CURRENT_PROJECT

Controller가 발견된 gipms-api-* 프로젝트를:

CURRENT_PROJECT

로 정의한다.

기본 검색:

CURRENT_PROJECT/src/main/java/**

CURRENT_PROJECT/src/main/resources/**

---

# 8. Cross Project

다른 gipms-api-* 검색은 다음을 모두 만족할 때만 허용한다.

1. 실제 실행 Method에서 호출됨
2. 정확한 Type 이름을 알고 있음
3. Business Service임
4. CURRENT_PROJECT에서 찾지 못함
5. Common/Library/Framework 탐색이 아님

정확한 Type 이름으로 1회만 검색한다.

찾지 못하면:

EXTERNAL_OR_DEPENDENCY

로 종료한다.

---

# 9. Common Boundary

다음 계열은 광범위 탐색하지 않는다.

- Common
- Base
- Util
- Utils
- Helper
- Framework
- Core
- Shared
- Security
- Message
- Code

CURRENT_PROJECT에서 정확한 Type이 바로 발견되면
실제 호출 Method만 분석할 수 있다.

찾기 위해 범위를 확장해야 하면:

COMMON_BOUNDARY

로 처리한다.

---

# 10. FAST READ

Java:

정확한 Symbol Grep

↓

Method 시작 위치

↓

약 80줄 Read

↓

Method 종료 확인

↓

필요할 때만 다음 약 80줄

전체 Java 파일 Read를 기본적으로 하지 않는다.

XML:

Statement 위치

↓

약 40줄 Read

↓

Statement 종료 확인

↓

필요할 때만 추가 Read

전체 XML Read를 기본적으로 하지 않는다.

---

# 11. 내부 Cache

분석 중 다음을 유지한다.

CURRENT_PROJECT

KNOWN_FILES

KNOWN_SYMBOLS

KNOWN_VARIABLE_TYPES

VISITED_METHODS

VISITED_XML

VISITED_STATEMENTS

VISITED_INCLUDES

VISITED_EXTERNAL_CALLS

CALL_STACK

VISITED_METHODS Key:

ClassName#MethodName(signature)

---

# 12. Controller

BACKEND_URL에서 식별력이 높은 Path를 사용해
Controller 후보를 찾는다.

검색 범위:

gipms-api-*/src/main/java/**/*Controller.java

Controller 후보가 확정되면
불필요한 다른 Controller 검색을 중단한다.

---

# 13. Mapping

확인:

class-level mapping

+

method-level mapping

=

BACKEND_URL

HTTP Method도 함께 확인한다.

---

# 14. HTTP Method

입력이:

GET
POST
PUT
DELETE
PATCH

이면 annotation과 검증한다.

UNKNOWN이면 Controller annotation에서 확정한다.

동일 URL에 여러 Method가 있어
확정 불가능하면:

AMBIGUOUS

로 종료한다.

---

# 15. Controller Method

확인:

- Controller Class
- Controller Method
- HTTP Method
- URL
- Request
- Validation
- Service Call
- Response
- Evidence

---

# 16. Service

Controller에서 실제 호출되는 Service만 추적한다.

확인:

- Variable
- Type
- Method
- Parameter
- Return

---

# 17. Service Interface

정확한 Service Type만 찾는다.

호출된 Method 선언만 확인한다.

전체 파일 분석 금지.

---

# 18. ServiceImpl

정확한:

implements ServiceType

으로 구현체를 찾는다.

구현체 확정 후 다른 후보 검색을 중단한다.

---

# 19. ServiceImpl Method

정확한 Method 위치를 찾는다.

약 80줄 부분 Read한다.

Method 종료가 보이지 않을 때만 추가 Read한다.

---

# 20. Method 분석

실제 코드 순서대로 확인한다.

- Validation
- 조건
- Branch
- Parameter 생성
- DTO 변환
- State 변경
- Local Method
- 다른 Service
- Mapper
- External Integration
- Post Processing
- Return
- Exception

---

# 21. Branch

실제:

if
else
else if
switch
early return
throw

구조를 유지한다.

분기를 합치지 않는다.

---

# 22. Local/private Method

동일 Class 내부 Method 호출이면
현재 파일에서 정확한 Method만 찾는다.

부분 Read한다.

Local Method가 다른 Local Method를 호출하면
실제 호출 경로만 재귀 추적한다.

이미 VISITED_METHODS에 있으면 재분석하지 않는다.

---

# 23. 다른 Service

다른 Service 호출이면:

Variable

↓

Type

↓

Service

↓

ServiceImpl

↓

실제 Method

순서로 추적한다.

Method 이름만으로 전체 프로젝트를 검색하지 않는다.

---

# 24. 다른 Project Service

CURRENT_PROJECT에 없고
정확한 Business Service Type을 알고 있을 때만:

다른 gipms-api-*/src/main/java/**

에서 정확한 Type을 1회 검색한다.

없으면:

EXTERNAL_OR_DEPENDENCY

---

# 25. Service 왕복

다른 Service 분석 종료 후
Caller로 복귀한다.

예:

ServiceA.create()

→ ServiceB.check()

   → Mapper

   → SQL

   → return

→ ServiceA.create() 복귀

→ 다음 처리

이 왕복 구조를 보존한다.

---

# 26. Mapper

실제 Mapper 호출에서:

- Mapper Variable
- Mapper Type
- Mapper Method
- Parameter
- Return

을 확인한다.

---

# 27. Mapper Type

현재 읽은 범위에 Type이 없으면
현재 ServiceImpl 파일에서
Mapper Variable 이름만 정확히 Grep한다.

Type 확보 즉시 중단한다.

---

# 28. Mapper.java

FAST 기본 경로에서는 SKIP한다.

기본:

ServiceImpl

→ Mapper Type

→ Mapper Method

→ MyBatis XML

XML 연결이 모호할 때만 Mapper.java를 최소 확인한다.

---

# 29. MyBatis XML

해당 프로젝트:

src/main/resources/**

에서 Mapper Type namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

찾으면 다른 XML 검색을 중단한다.

---

# 30. XML fallback

Mapper Type으로 찾지 못했을 때만:

id="MapperMethod"

를 해당 프로젝트 resources에서 검색한다.

---

# 31. SQL Statement

실제 Mapper Method와 연결된:

select
insert
update
delete

만 분석한다.

초기 약 40줄 Read.

필요할 때만 추가 Read.

---

# 32. SQL 분석

확인:

- SQL Type
- Main Table
- Additional Table
- JOIN
- WHERE
- Parameter
- Dynamic SQL
- include
- resultMap reference

---

# 33. Parameter Mapping

가능한 범위에서:

Request

→ Service Parameter

→ Mapper Parameter

→ #{parameter}

를 연결한다.

확인되지 않는 Mapping은 추측하지 않는다.

---

# 34. Dynamic SQL

실제:

if
choose
when
otherwise
foreach

구조를 유지한다.

---

# 35. include

실제 Statement에서 사용하는 include만 추적한다.

정확한 refid만 찾는다.

전체 fragment를 검색하지 않는다.

이미 방문한 include는 다시 분석하지 않는다.

찾지 못하면:

INCLUDE_SOURCE_NOT_FOUND

---

# 36. resultMap

Response/후처리를 이해하는 데 필요한 경우만
정확한 resultMap을 최소 분석한다.

필요하지 않으면:

REFERENCE_ONLY

---

# 37. SQL 이후

SQL에서 분석을 끝내지 않는다.

SQL

↓

Mapper Return

↓

Service 복귀

↓

Result Check

↓

Post Processing

↓

다음 호출

↓

Return

까지 추적한다.

---

# 38. 외부 연동

실제 Method에서 발견된 경우만 분석한다.

대상:

- SAP
- RFC
- Feign
- RestTemplate
- WebClient
- HTTP Client
- Adapter
- Gateway
- Client

프로젝트 전체 외부 연동 선검색 금지.

---

# 39. 외부 Source

CURRENT_PROJECT에서
정확한 Type Source가 확인되면
실제 호출 Method만 분석한다.

없으면:

DEPENDENCY_BOUNDARY

JAR 검색 금지.

---

# 40. SAP/RFC

Source에서 확인 가능한 범위:

- Request 생성
- Parameter Mapping
- Function 식별자
- 호출 조건
- 실제 호출
- Response
- 성공 처리
- 실패 처리
- Exception
- Caller 복귀

Library 내부는 분석하지 않는다.

---

# 41. HTTP 외부 API

확인 가능한 경우:

- Client
- Method
- HTTP Method
- URL
- Path
- Query
- Header
- Body
- Response
- Error

Library 내부 구현은 분석하지 않는다.

---

# 42. Post Processing

DB/외부 연동 이후:

- Result Check
- DTO 변환
- State 변경
- List 가공
- 후속 Mapper
- History
- Response 생성

을 계속 추적한다.

---

# 43. Exception

실제 실행 Path에서 확인되는:

- throw
- catch
- custom exception
- validation failure
- mapper failure
- external failure
- error response

만 기록한다.

프로젝트 전체 Exception 검색 금지.

---

# 44. Response

최종:

ServiceImpl

↓

Service

↓

Controller

반환까지 연결한다.

가능하면:

- Response DTO
- HTTP Status
- Response Body

를 기록한다.

---

# 45. 종료점

실제 Path가 다음 중 하나에 도달하면 종료한다.

- 정상 Response
- 정상 void
- return
- Exception
- Error Response

외부 연동은 종료점이 아니다.

외부 호출 후 Caller로 복귀한다.

---

# 46. Boundary 표시

내부 Source를 추적하지 못해도
실제 호출은 제거하지 않는다.

예:

→ commonService.getCode()

   → [COMMON_BOUNDARY]

→ Caller 복귀

또는:

→ externalClient.send()

   → [DEPENDENCY_BOUNDARY]

→ Caller 복귀

---

# 47. Evidence

Source를 최초 확인할 때 Evidence도 같이 확보한다.

형식:

Project Root 기준 상대경로:라인범위

절대경로 금지.

Evidence를 위해 재검색하지 않는다.

---

# 48. Oracle Metadata

수행하지 않는다.

- Oracle Metadata
- DB Metadata
- Table Metadata
- Column Metadata
- DB Connection

SQL Source 분석까지만 한다.

---

# 49. 실행 흐름

단순:

Controller
→ Service
→ Mapper

로 축약하지 않는다.

실제 호출과 복귀를 유지한다.

예:

Controller

→ Service A

→ Service B

   → Mapper

   → SQL

→ Service B 복귀

→ Service A 복귀

→ IF

   ├─ YES → Mapper → SQL

   └─ NO → External

→ Service A 복귀

→ Response

---

# 50. 분석 결과 반환

최종 Markdown을 Write하지 않는다.

Worker에게 다음 구조의 분석 결과를 반환한다.

ANALYSIS_STATUS:

BACKEND_URL:

HTTP_METHOD:

CURRENT_PROJECT:

CONTROLLER:

CONTROLLER_METHOD:

ENTRY_SERVICE:

SERVICE_CHAIN:

LOCAL_METHODS:

VALIDATIONS:

BRANCHES:

MAPPER_CALLS:

SQL_STATEMENTS:

PARAMETER_MAPPINGS:

EXTERNAL_INTEGRATIONS:

POST_PROCESSING:

RESPONSES:

EXCEPTIONS:

DEPENDENCY_BOUNDARIES:

COMMON_BOUNDARIES:

EVIDENCE:

FULL_EXECUTION_FLOW:

---

# 51. 완료 Status

COMPLETED

또는

AMBIGUOUS

또는

NOT_FOUND

분석 완료 후 추가 검색하지 않는다.
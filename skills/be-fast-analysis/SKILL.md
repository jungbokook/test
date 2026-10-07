---
name: be-analysis-fast
description: Parent 또는 Worker가 전달한 Backend URL 하나를 Strict Source Boundary 기반으로 전체 분석하고 지정된 OUTPUT_PATH에 BE-REFERENCE 형식의 Markdown을 생성한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL> <OUTPUT_PATH>"
allowed-tools: Read, Grep, Write
---

# BE Analysis FAST - Common Analysis Engine

## 0. 역할

이 Skill은 Backend URL 하나를 분석하는 공통 분석 엔진이다.

단독/병렬 여부를 판단하지 않는다.

입력:

HTTP_METHOD

BACKEND_URL

OUTPUT_PATH

예:

POST
/material/create
docs/analysis/자재등록/backend/FE-ACT-010-create-BE-003.md

이 Skill은 Parent가 전달한 OUTPUT_PATH를 그대로 사용한다.

---

# 1. 절대 하지 않는 것

이 Skill은 다음을 수행하지 않는다.

- BE ID 계산
- 기존 BE 번호 검색
- 다음 BE 번호 결정
- OUTPUT_PATH 생성
- backend 디렉토리 파일 목록 확인
- 다른 BE 결과 확인
- 다른 Worker 상태 확인
- 다른 Backend URL 분석

즉:

번호/파일명 관리 = Parent

Source 분석 = 이 Skill

이다.

---

# 2. 분석 범위

전체 분석 범위:

Controller
→ Service
→ ServiceImpl
→ Validation
→ 조건/분기
→ Local/private Method
→ 다른 업무 Service
→ Mapper
→ MyBatis XML
→ SQL
→ 실제 사용 include
→ 필요한 resultMap
→ SAP/RFC/외부 연동
→ 후처리
→ Response
→ Exception
→ 실제 종료점

핵심:

검색 범위는 좁게,
호출 깊이는 끝까지.

---

# 3. SOURCE SEARCH BOUNDARY

Java Source 검색 허용:

gipms-api-*/src/main/java/**

Resource 검색 허용:

gipms-api-*/src/main/resources/**

그 밖으로 확장하지 않는다.

---

# 4. HARD EXCLUDE

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

JAR fallback은 어떤 경우에도 금지한다.

---

# 5. SOURCE NOT FOUND

Source Boundary에서 Source를 찾지 못하면:

SOURCE_NOT_FOUND

또는

EXTERNAL_OR_DEPENDENCY

또는

DEPENDENCY_BOUNDARY

로 기록한다.

다음 행동은 금지한다.

Source 없음
→ JAR 검색

Source 없음
→ lib 검색

Source 없음
→ target 검색

Source 없음
→ dependency 전체 검색

---

# 6. CURRENT_PROJECT

Controller가 발견된 gipms-api-*를:

CURRENT_PROJECT

로 정의한다.

기본 검색 범위:

CURRENT_PROJECT/src/main/java/**

CURRENT_PROJECT/src/main/resources/**

---

# 7. Cross Project

다른 gipms-api-* 검색은 다음을 모두 만족할 때만 허용한다.

1. 실제 Method에서 호출됨
2. 정확한 Type 이름을 알고 있음
3. 업무 Service임
4. CURRENT_PROJECT에서 찾지 못함
5. Common/Library/Framework 탐색 목적이 아님

정확한 Type 이름으로 1회만 검색한다.

없으면:

EXTERNAL_OR_DEPENDENCY

로 종료한다.

---

# 8. Common Boundary

다음 계열은 광범위하게 추적하지 않는다.

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

CURRENT_PROJECT Source에서
정확한 Type으로 바로 확인되면 추적 가능하다.

찾기 위해 다른 프로젝트를 광범위하게 검색해야 하면:

COMMON_BOUNDARY

로 처리한다.

이름만으로 무조건 제외하지 않는다.

---

# 9. BE-REFERENCE

최종 문서 Format을 위해
BE-REFERENCE를 분석 시작 시 정확히 1회 Read한다.

다시 읽지 않는다.

BE-REFERENCE:

Format Reference = YES

Evidence = NO

예제 데이터 복사 = NO

---

# 10. FAST READ

Java:

정확한 Symbol Grep
→ Method 시작 위치
→ 약 80줄 Read
→ Method 종료 확인
→ 필요할 때만 다음 약 80줄

전체 Java 파일 Read를 기본적으로 하지 않는다.

XML:

정확한 Statement 위치
→ 약 40줄 Read
→ 종료 확인
→ 필요할 때만 추가 약 40줄

전체 XML Read를 기본적으로 하지 않는다.

---

# 11. CACHE

분석 중 유지:

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

VISITED_METHODS:

ClassName#MethodName(signature)

형태를 사용한다.

---

# 12. Controller

BACKEND_URL의 상위 path를 우선 사용해
Controller 후보를 찾는다.

검색 허용 범위:

gipms-api-*/src/main/java/**/*Controller.java

후보가 발견되면 불필요한 Controller 검색을 중단한다.

---

# 13. Mapping

class-level mapping

+

method-level mapping

을 조합해 실제 URL을 확인한다.

HTTP Method도 함께 확인한다.

---

# 14. UNKNOWN Method

HTTP_METHOD가 UNKNOWN이면
Controller annotation에서 실제 Method를 확정한다.

동일 URL에 여러 HTTP Method가 있어
하나로 확정할 수 없으면:

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
- Response
- Validation
- Service 호출
- Evidence

Controller 확정 후 다른 Controller를 탐색하지 않는다.

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

정확한 Service Type만 검색한다.

호출된 Method 선언만 확인한다.

전체 파일 분석 금지.

---

# 18. ServiceImpl

정확한:

implements ServiceType

으로 구현체를 찾는다.

발견 즉시 불필요한 구현체 검색을 중단한다.

---

# 19. ServiceImpl Method

호출된 정확한 Method 위치를 Grep한다.

약 80줄 부분 Read한다.

Method 종료가 안 보일 때만 추가 Read한다.

---

# 20. Method 분석

실제 Method에서:

- Validation
- if/else
- switch
- early return
- Parameter 생성
- DTO 변환
- State 변경
- Local Method
- 다른 Service
- Mapper
- 외부 연동
- Return
- Exception

을 실제 순서대로 확인한다.

---

# 21. Local Method

동일 Class 내부 호출이면
현재 ServiceImpl 파일에서만
정확한 Method를 찾는다.

부분 Read한다.

Local Method 내부도 같은 규칙을 적용한다.

이미 방문한 Method는 다시 분석하지 않는다.

---

# 22. 다른 Service

다른 Service 호출이면:

Variable
→ Type
→ Interface
→ Impl
→ 실제 Method

순서로 추적한다.

Method 이름만 가지고 전체 프로젝트를 검색하지 않는다.

---

# 23. 다른 프로젝트 Service

CURRENT_PROJECT에 없고
정확한 업무 Service Type을 알고 있을 때만
다른 gipms-api-*의 src/main/java에서
정확한 Type을 1회 검색한다.

찾지 못하면:

EXTERNAL_OR_DEPENDENCY

로 처리한다.

---

# 24. Service 왕복

다른 Service 호출이 끝나면
반드시 Caller로 복귀한다.

예:

ServiceA.create()

→ ServiceB.check()

   → Mapper

   → SQL

   → return

→ ServiceA.create() 복귀

→ 다음 처리

최종 실행 흐름에 왕복을 표시한다.

---

# 25. Mapper

실제 Mapper 호출에서:

- Caller
- Mapper Variable
- Mapper Type
- Mapper Method
- Parameter
- Return

을 확인한다.

---

# 26. Mapper Type

이미 읽은 범위에서 Type이 없으면
현재 ServiceImpl 파일에서
Mapper Variable 이름만 Grep한다.

Type 확보 후 중단한다.

---

# 27. Mapper Java

FAST 기본 경로에서는 SKIP한다.

ServiceImpl
→ Mapper Type
→ Mapper Method
→ XML

XML 연결이 모호할 때만
Mapper.java를 최소 확인한다.

---

# 28. MyBatis XML

해당 프로젝트:

src/main/resources/**

에서 Mapper Type으로 namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

찾으면 다른 XML 탐색을 중단한다.

---

# 29. XML fallback

Mapper Type으로 못 찾은 경우에만:

id="MapperMethod"

를 검색한다.

해당 프로젝트 resources 내부만 검색한다.

---

# 30. Statement

실제 호출된:

select
insert
update
delete

Statement만 부분 Read한다.

초기 약 40줄.

필요할 때만 추가 Read한다.

---

# 31. SQL

확인:

- SQL Type
- Main Table
- Additional Table
- JOIN
- WHERE
- Parameter
- Dynamic SQL
- include
- resultMap

실제 Statement만 분석한다.

---

# 32. Parameter Mapping

가능한 범위에서:

Request
→ Service Parameter
→ Mapper Parameter
→ #{parameter}

연결을 확인한다.

추측하지 않는다.

---

# 33. Dynamic SQL

실제:

if
choose
when
otherwise
foreach

구조를 유지한다.

---

# 34. include

실제 Statement에서 참조되는 include만 추적한다.

정확한 refid만 찾는다.

이미 방문한 include는 다시 분석하지 않는다.

Source Boundary에서 못 찾으면:

INCLUDE_SOURCE_NOT_FOUND

로 처리한다.

---

# 35. resultMap

Response/후처리 이해에 필요한 경우만
정확한 resultMap을 최소 분석한다.

필요하지 않으면:

REFERENCE_ONLY

---

# 36. SQL 복귀

SQL에서 분석을 끝내지 않는다.

Mapper 결과
→ Caller
→ 조건
→ 후처리
→ 다음 호출
→ Return

까지 계속 추적한다.

---

# 37. 외부 연동

실제 Method에서 발견되는 경우만 추적한다.

- SAP
- RFC
- Feign
- RestTemplate
- WebClient
- HTTP Client
- Adapter
- Gateway
- Client

프로젝트 전체 선검색 금지.

---

# 38. 외부 Source

Source Boundary에서 정확한 Type을 확인할 수 있으면
실제 호출 Method만 분석한다.

없으면:

DEPENDENCY_BOUNDARY

로 기록한다.

JAR로 이동하지 않는다.

---

# 39. SAP/RFC

Source에서 확인 가능한 범위:

- Request 생성
- Parameter Mapping
- Function 식별자
- 호출 조건
- 호출
- Response
- 성공
- 실패
- Exception
- Caller 복귀

Library JAR 내부는 분석하지 않는다.

---

# 40. HTTP API

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

Library 구현은 분석하지 않는다.

---

# 41. 후처리

DB/외부 호출 이후에도:

- 결과 확인
- DTO 변환
- State 변경
- List 가공
- 후속 Mapper
- History
- Response 생성

을 계속 추적한다.

---

# 42. Exception

실제 실행 경로에서 보이는:

- throw
- catch
- custom exception
- validation failure
- mapper failure 처리
- external failure 처리
- error response

만 기록한다.

전체 프로젝트 Exception 검색 금지.

---

# 43. Response

최종 반환:

ServiceImpl
→ Service
→ Controller

를 연결한다.

가능하면:

- Response DTO
- HTTP Status
- Response Body

를 기록한다.

---

# 44. 종료

각 실제 Path는:

- 정상 Response
- void 정상 종료
- return
- Exception
- Error Response

중 하나에 도달하면 종료한다.

외부 호출 자체는 종료점이 아니다.

---

# 45. Dependency Boundary

내부 구현을 찾지 못한 호출도
실행 흐름에서 제거하지 않는다.

예:

→ commonService.getCode()
   → [COMMON_BOUNDARY]
→ Caller 복귀

→ externalClient.send()
   → [DEPENDENCY_BOUNDARY]
→ Caller 복귀

---

# 46. Evidence

최초 Source 확인 시 Evidence를 같이 확보한다.

형식:

Project Root 기준 상대경로:라인범위

절대경로 금지.

Evidence 재검색 금지.

---

# 47. Oracle Metadata

분석하지 않는다.

- Oracle Metadata
- DB Metadata
- Table Metadata
- Column Metadata
- DB Connection

SQL Source까지만 분석한다.

---

# 48. 최종 실행 흐름

단순:

Controller
→ Service
→ Mapper

로 축약하지 않는다.

실제:

Controller
→ Service A
→ Service B
→ Mapper
→ SQL
→ Service B 복귀
→ Service A 복귀
→ 조건
→ 외부 연동
→ Service A 복귀
→ Response

순서를 유지한다.

---

# 49. Markdown 생성

모든 Source 분석이 끝난 후:

BE-REFERENCE 형식 적용

↓

OUTPUT_PATH에 1회 Write

분석 중 반복 Write하지 않는다.

OUTPUT_PATH를 변경하지 않는다.

---

# 50. Output Ownership

이 Skill은 전달받은 OUTPUT_PATH 하나만 생성한다.

다른 BE 문서를 읽거나 수정하지 않는다.

---

# 51. 최종 검증

이미 확보한 정보만 사용한다.

확인:

- Controller부터 종료점까지 연결
- 실제 호출만 포함
- Local Method
- Service 왕복
- SQL 이후 복귀
- 외부 연동 이후 복귀
- 실제 분기
- Dependency Boundary
- 상대경로 Evidence
- JAR 검색 없음
- Common 광범위 검색 없음
- Oracle Metadata 없음
- OUTPUT_PATH 일치

검증을 위한 재검색은 하지 않는다.

---

# 52. 완료 응답

다음만 반환한다.

BACKEND_URL:
HTTP_METHOD:
OUTPUT_PATH:
CONTROLLER:
ENTRY_SERVICE:
SERVICE_CHAIN_COUNT:
LOCAL_METHOD_COUNT:
MAPPER_CALL_COUNT:
SQL_STATEMENT_COUNT:
EXTERNAL_INTEGRATION_COUNT:
DEPENDENCY_BOUNDARY_COUNT:
COMMON_BOUNDARY_COUNT:
STATUS:

STATUS:

COMPLETED

또는

AMBIGUOUS

또는

NOT_FOUND
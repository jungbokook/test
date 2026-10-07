---
name: be-test-controller
description: Backend URL 하나를 대상으로 허용된 Project Source 내부에서만 Controller부터 실제 종료점까지 빠르게 추적하고 BE-REFERENCE 형식의 최종 BE 분석 문서를 생성하는 전체 성능 테스트 Skill.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep, Write
---

# BE Full Fast Analysis Test - Strict Source Boundary

## 0. 목적

이 Skill은 최종 BE 전체 분석의 성능을 테스트한다.

분석 범위:

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
→ SAP/RFC/외부 연동
→ 후처리
→ Response
→ Exception
→ 최종 Markdown

핵심 원칙:

검색 범위는 좁게,
호출 깊이는 끝까지.

단:

프로젝트 Source Boundary 밖으로는 절대 확장하지 않는다.

---

# 1. 입력

사용법:

/be-test-controller <화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller 자재등록 FE-ACT-010-create POST /material/create

입력:

- 화면명
- FE Base Name
- HTTP Method 또는 UNKNOWN
- Backend URL

---

# 2. 출력 경로

출력:

docs/analysis/{화면명}/backend/

파일명:

{FE Base Name}-BE-{NNN}.md

예:

docs/analysis/자재등록/backend/FE-ACT-010-create-BE-001.md

---

# 3. BE ID

출력 디렉토리의 기존 BE 파일명만 확인한다.

예:

FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md

다음:

FE-ACT-010-create-BE-003.md

기존 분석 문서 내용은 읽지 않는다.

최대 BE 번호 + 1을 사용한다.

---

# 4. SOURCE SEARCH BOUNDARY

## 4.1 Java 검색 허용

Java Source 검색은 오직:

gipms-api-*/src/main/java/**

안에서만 수행한다.

그 밖의 Java/Class Source를 찾지 않는다.

---

## 4.2 Resource 검색 허용

MyBatis XML 및 설정 Source는 오직:

gipms-api-*/src/main/resources/**

안에서만 수행한다.

---

## 4.3 절대 검색 금지

다음 경로는 어떠한 경우에도 검색하지 않는다.

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

---

# 5. JAR HARD STOP

JAR는 절대 분석하지 않는다.

다음 상황에서도 JAR를 찾지 않는다.

- Class Source를 찾지 못함
- Service Interface를 찾지 못함
- ServiceImpl을 찾지 못함
- 외부 Client Source를 찾지 못함
- Common Class Source를 찾지 못함
- Library Method 구현을 확인하고 싶음
- Dependency 구현체를 확인하고 싶음

Source Boundary에서 Source를 찾지 못하면:

SOURCE_NOT_FOUND

또는

EXTERNAL_OR_DEPENDENCY

로 기록하고 해당 호출 내부 추적을 종료한다.

절대로 JAR fallback을 수행하지 않는다.

---

# 6. COMMON SEARCH BOUNDARY

이름에 다음과 같은 표현이 포함된 호출은 주의한다.

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

이름만 보고 무조건 제외하지는 않는다.

단:

현재 프로젝트 Source에서
정확한 Type을 즉시 찾을 수 있는 경우만 추적한다.

찾기 위해 전체 프로젝트를 확대 탐색해야 하면:

COMMON_BOUNDARY

로 처리한다.

예:

commonService.getCode()

현재 프로젝트 Source에서 CommonService를
정확한 Type 검색으로 찾음:

추적 가능

찾지 못함:

commonService.getCode()
→ COMMON_BOUNDARY
→ 내부 추적 STOP

---

# 7. 다른 프로젝트 검색 제한

현재 Controller가 속한 Backend 프로젝트를:

CURRENT_PROJECT

로 정의한다.

예:

gipms-api-material

기본 검색 범위:

CURRENT_PROJECT/src/main/java/**

CURRENT_PROJECT/src/main/resources/**

이다.

---

# 8. Cross Project 검색 허용 조건

다른 gipms-api-* 프로젝트 검색은
다음 조건을 모두 만족할 때만 허용한다.

1. 현재 Method에서 실제 호출된 Type이다.
2. 업무 Service로 판단된다.
3. 정확한 Type 이름을 알고 있다.
4. CURRENT_PROJECT에서 찾지 못했다.
5. JAR/Library/Common 탐색 목적이 아니다.

예:

orderService.createOrder()

Variable Type:

OrderService

이면:

다른 gipms-api-*에서

정확한:

OrderService

만 검색할 수 있다.

---

# 9. Cross Project 검색 금지

다음 방식은 금지한다.

모든 Service 검색

모든 common 검색

모든 Impl 검색

전체 프로젝트에서 Method 이름만 검색

관련 Class 추측 검색

JAR 검색

Dependency 검색

---

# 10. Cross Project STOP

정확한 Type으로 다른 gipms-api-* Source를
한 번 검색했는데 찾지 못하면:

EXTERNAL_OR_DEPENDENCY

로 기록한다.

더 이상 찾지 않는다.

---

# 11. BE-REFERENCE

최종 문서 형식을 위해
BE-REFERENCE를 분석 시작 시 1회만 Read한다.

재로드하지 않는다.

BE-REFERENCE는 Format Reference다.

Evidence가 아니다.

---

# 12. FAST READ 원칙

전체 Java 파일을 기본적으로 읽지 않는다.

기본:

Symbol Grep

↓

Method 위치

↓

약 80줄 Read

↓

Method 종료 확인

종료되지 않았을 때만
다음 약 80줄을 읽는다.

Method 종료 후 추가 Read 금지.

---

# 13. XML FAST READ

MyBatis XML 전체 파일을 읽지 않는다.

Statement 위치 확인

↓

약 40줄 Read

↓

Statement 종료 확인

필요한 경우에만 추가 약 40줄 Read

종료 후 추가 Read 금지.

---

# 14. 검색 캐시

내부적으로 다음 상태를 유지한다.

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

---

# 15. VISITED METHOD KEY

Method 방문 여부는:

ClassName#MethodName(signature)

로 관리한다.

예:

MaterialServiceImpl#create(MaterialRequest)

ValidationServiceImpl#check(MaterialRequest)

이미 방문한 Method는 다시 분석하지 않는다.

---

# 16. Controller 검색

Backend URL의 상위 path를 기준으로:

CURRENT_PROJECT 후보의

src/main/java/**/*Controller.java

범위에서 검색한다.

Controller 후보가 발견되면
다른 프로젝트 Controller 검색을 중단한다.

---

# 17. Controller Mapping

class-level mapping

+

method-level mapping

을 조합한다.

입력 Backend URL과 비교한다.

HTTP Method도 함께 검증한다.

---

# 18. HTTP Method UNKNOWN

Method가 UNKNOWN이면
Controller annotation에서 확정한다.

동일 URL에 여러 HTTP Method가 있고
확정할 수 없으면:

AMBIGUOUS

로 종료한다.

---

# 19. Controller 분석

확인:

- Controller
- Method
- Mapping
- HTTP Method
- Request
- Response
- Validation
- Service 호출
- Evidence

확정 후 다른 Controller를 탐색하지 않는다.

---

# 20. Service

Controller에서 실제 호출된 Service만 추적한다.

예:

materialService.create(request)

확인:

- Variable
- Type
- Method
- Parameter

---

# 21. Service Interface

CURRENT_PROJECT에서
정확한 Service Type만 검색한다.

호출된 Method 선언만 확인한다.

전체 파일 분석 금지.

---

# 22. ServiceImpl

정확한:

implements ServiceType

으로 구현체를 찾는다.

발견 즉시 검색 종료.

전체 ServiceImpl Read 금지.

---

# 23. ServiceImpl Method

정확한 Method 위치를 찾는다.

약 80줄 부분 Read한다.

종료되지 않은 경우에만 추가 Read한다.

---

# 24. Method 내부 분석

확인:

- Validation
- IF/ELSE
- switch
- Parameter 생성
- State 변경
- Local Method
- 다른 Service
- Mapper
- 외부 연동
- Return
- Exception

실제 코드 순서를 유지한다.

---

# 25. Local/private Method

동일 Class 내부 호출이면
현재 파일 하나에서만 Method를 찾는다.

예:

saveMaterial()

↓

현재 ServiceImpl 파일에서만:

saveMaterial(

검색

다른 프로젝트를 검색하지 않는다.

---

# 26. Local Method 부분 Read

Method 시작부터 약 80줄 Read.

필요할 때만 추가 Read.

Local Method 내부에서도:

- Local Method
- 업무 Service
- Mapper
- 외부 연동

을 실제 호출 기준으로 추적한다.

---

# 27. 다른 Service 판별

예:

validationService.check(request)

이면 먼저 이미 읽은 Source에서:

validationService의 Type

을 확인한다.

예:

ValidationService

Type을 확보하지 못했다고
전체 프로젝트에서 check()를 검색하지 않는다.

---

# 28. 다른 Service 현재 프로젝트 검색

정확한 Type:

ValidationService

를 CURRENT_PROJECT의:

src/main/java/**

에서 검색한다.

찾으면 실제 호출 Method만 추적한다.

---

# 29. 다른 업무 프로젝트 Service

CURRENT_PROJECT에 없고
명백한 업무 Service인 경우에만:

gipms-api-*/src/main/java/**

에서 정확한:

ValidationService

Type을 한 번 검색한다.

찾으면 해당 프로젝트를:

TARGET_PROJECT

로 사용한다.

---

# 30. 다른 Service 검색 실패

정확한 Type 검색으로 찾지 못하면:

External/Dependency Call:

- Type:
- Method:
- Status: EXTERNAL_OR_DEPENDENCY

로 기록하고 내부 추적을 중단한다.

JAR로 넘어가지 않는다.

---

# 31. Common 호출

Common/Base/Util/Helper 계열 호출은
CURRENT_PROJECT Source에서 정확히 찾을 수 있을 때만
내부 추적한다.

찾지 못하면:

Status:
COMMON_BOUNDARY

로 기록한다.

다른 gipms-api-* 전체로 확대하지 않는다.

---

# 32. Service 왕복

다른 Service 분석이 끝나면
원래 Caller로 복귀한다.

예:

MaterialServiceImpl.create()

→ ValidationServiceImpl.check()

   → Mapper

   → SQL

   → return

→ MaterialServiceImpl.create() 복귀

→ 다음 처리

실행 흐름에 복귀 지점을 유지한다.

---

# 33. Mapper

실제 호출:

materialMapper.insertMaterial(param)

에서:

- Mapper Variable
- Mapper Type
- Mapper Method

를 확보한다.

---

# 34. Mapper Type

이미 읽은 ServiceImpl 범위에 Type이 없으면
현재 ServiceImpl 파일에서:

materialMapper

만 Grep한다.

예:

private final MaterialMapper materialMapper;

Type 확보 즉시 검색 종료.

---

# 35. Mapper Java

FAST 기본 경로:

Mapper Java SKIP

ServiceImpl

→ Mapper Type

→ Mapper Method

→ XML

로 이동한다.

Mapper.java를 기본적으로 검색하지 않는다.

---

# 36. Mapper Java 예외

다음 경우에만 Mapper.java를 최소 확인할 수 있다.

- Mapper Type이 모호함
- XML namespace 연결이 모호함
- 동일 statement 후보가 여러 개

그 외에는 Mapper.java를 검색하지 않는다.

---

# 37. MyBatis XML

먼저 TARGET/CURRENT PROJECT:

src/main/resources/**

에서 Mapper Type을 검색한다.

목표:

<mapper namespace="...MaterialMapper">

발견 즉시 다른 XML 검색 중단.

---

# 38. XML fallback

Mapper Type으로 못 찾은 경우에만:

id="MapperMethod"

를 검색한다.

해당 프로젝트의:

src/main/resources/**

범위만 검색한다.

다른 프로젝트 전체 XML을 뒤지지 않는다.

---

# 39. Statement

실제 Mapper Method에 해당하는:

select
insert
update
delete

statement만 읽는다.

약 40줄 부분 Read한다.

---

# 40. SQL

분석:

- SQL Type
- Main Table
- JOIN Table
- WHERE
- Parameter
- Dynamic SQL
- include
- resultMap 참조

실제 Statement 기준으로만 분석한다.

---

# 41. Dynamic SQL

추적:

- if
- choose
- when
- otherwise
- foreach

실제 조건 구조를 유지한다.

---

# 42. include

실제 Statement가 참조하는 include만 추적한다.

예:

<include refid="Base_Column_List"/>

이면 동일 XML 또는 확인 가능한 Source에서
정확한 refid만 찾는다.

전체 XML fragment 검색 금지.

---

# 43. include 검색 실패

허용된 resources Source에서
정확한 refid를 찾지 못하면:

INCLUDE_SOURCE_NOT_FOUND

로 기록한다.

다른 JAR/resource dependency를 찾지 않는다.

---

# 44. resultMap

Response 분석에 실제 필요한 경우만
정확한 resultMap을 최소 확인한다.

필요하지 않으면:

REFERENCE_ONLY

로 기록한다.

---

# 45. SQL 이후 복귀

SQL 분석 후 Mapper에서 끝내지 않는다.

반드시 Caller로 복귀한다.

예:

SELECT

↓

Mapper 결과

↓

ServiceImpl result

↓

IF result != null

↓

후처리

↓

다음 호출

↓

Return

---

# 46. 외부 연동 탐지

현재 실제 Method에서 직접 발견되는:

- SAP
- RFC
- Feign
- RestTemplate
- WebClient
- HTTP Client
- Adapter
- Gateway
- Client

만 대상으로 한다.

프로젝트 전체 외부 연동 검색 금지.

---

# 47. 외부 연동 Source

외부 연동 Wrapper/Adapter가
CURRENT_PROJECT의 src/main/java에서
정확한 Type으로 발견되면 추적한다.

실제 호출 Method만 부분 Read한다.

---

# 48. 외부 연동 Source 없음

정확한 Type Source가
허용된 Source Boundary에서 없으면:

External Integration:

- Type:
- Method:
- Status: DEPENDENCY_BOUNDARY

로 기록한다.

JAR 내부 구현을 찾지 않는다.

---

# 49. SAP/RFC

프로젝트 Source에서 실제 구현을 확인할 수 있을 때만:

- Request 생성
- Parameter Mapping
- Function 식별자
- 호출
- Response
- 성공/실패
- Exception
- Caller 복귀

를 추적한다.

SAP 라이브러리 JAR 내부로 들어가지 않는다.

---

# 50. HTTP 외부 API

프로젝트 Source에서 확인 가능한 범위만 분석한다.

- Client Method
- HTTP Method
- URL
- Path
- Query
- Header
- Body
- Response
- Error 처리

Library 내부 구현은 분석하지 않는다.

---

# 51. 후처리

DB 또는 외부 호출 이후:

- 결과 확인
- DTO 변환
- 상태 변경
- List 가공
- 후속 Mapper
- History 저장
- Response 생성

을 실제 코드대로 계속 추적한다.

---

# 52. Exception

실제 호출 경로에서 보이는:

- throw
- catch
- custom exception
- validation failure
- external failure
- error response

만 기록한다.

프로젝트 전체 Exception 검색 금지.

---

# 53. Response

최종:

ServiceImpl

↓

Service

↓

Controller

까지 실제 반환을 연결한다.

확인 가능한 경우:

- Response DTO
- HTTP Status
- Body

를 기록한다.

---

# 54. 종료점

실행 Path가 다음에 도달하면 종료한다.

- 정상 Response
- void 정상 종료
- 명시적 return
- Exception
- Error Response

외부 호출은 종료점이 아니다.

외부 호출 후 Caller 복귀를 추적한다.

---

# 55. 모든 실제 분기

실제 Source에서 확인된 분기는
각각 종료점까지 연결한다.

추측 분기는 만들지 않는다.

---

# 56. SEARCH FAILURE POLICY

어떤 Symbol을 찾지 못했을 때:

1차:

CURRENT_PROJECT의 허용 Source 검색

↓

업무 Service이고 정확한 Type을 알고 있을 경우만:

다른 gipms-api-*의 src/main/java에서
정확한 Type 1회 검색

↓

없음:

STOP

절대로:

검색어 확대
→ common 검색
→ lib 검색
→ jar 검색
→ dependency 검색

순으로 넘어가지 않는다.

---

# 57. 탐색 금지 패턴

다음 행동을 하지 않는다.

"혹시 여기에 있을 수 있으므로..."

"관련 구현을 찾기 위해..."

"dependency에서 확인하기 위해..."

"JAR 내부를 확인하기 위해..."

"common 프로젝트를 폭넓게 확인하기 위해..."

이러한 추측 기반 탐색은 금지한다.

---

# 58. Evidence

Evidence는 최초 Source 확인 시 같이 확보한다.

형식:

Project Root 기준 상대경로:라인범위

절대경로 금지.

Evidence를 얻기 위한 재검색 금지.

---

# 59. BE-REFERENCE 적용

Source 분석이 모두 끝난 후:

이미 1회 읽은 BE-REFERENCE 형식을 사용한다.

다시 Read하지 않는다.

REFERENCE의 예제 데이터를 복사하지 않는다.

---

# 60. 최종 실행 흐름

단순 호출 목록으로 축약하지 않는다.

예:

Controller.create()

→ MaterialServiceImpl.create()

→ ValidationServiceImpl.check()

   → ValidationMapper.selectExists()

   → SELECT TB_MATERIAL

   → ValidationServiceImpl.check() 복귀

→ MaterialServiceImpl.create() 복귀

→ IF exists

   ├─ YES
   │  → MaterialMapper.update()
   │  → UPDATE TB_MATERIAL
   │
   └─ NO
      → MaterialMapper.insert()
      → INSERT TB_MATERIAL

→ SapService.send()

   → SAP Request 생성

   → SAP 호출

   → Response

→ MaterialServiceImpl.create() 복귀

→ Response DTO 생성

→ Controller 반환

---

# 61. Dependency Boundary 표시

Source Boundary 때문에 내부 구현을 추적하지 않은 호출은
실행 흐름에서 숨기지 않는다.

예:

MaterialServiceImpl.create()

→ commonService.getCode()

   → [COMMON_BOUNDARY]

→ 다음 처리

또는:

→ externalClient.send()

   → [DEPENDENCY_BOUNDARY]

→ 다음 처리

즉:

내부 구현을 못 찾았다고
호출 자체를 삭제하지 않는다.

---

# 62. 최종 Markdown

모든 Source 분석 완료 후
최종 Markdown을 한 번만 Write한다.

분석 도중 Markdown 파일을 반복 수정하지 않는다.

순서:

Source 분석

↓

호출 트리 완성

↓

분기/종료 확인

↓

BE-REFERENCE 적용

↓

Markdown Write 1회

---

# 63. Oracle / DB Metadata

수행하지 않는다.

- Oracle Metadata
- DB Metadata
- Table Metadata
- Column Metadata
- DB Connection

SQL Source 분석까지만 수행한다.

---

# 64. 최종 검증

Write 전 다음만 확인한다.

1. Controller부터 종료점까지 연결됐는가?
2. 실제 호출만 들어갔는가?
3. Service 왕복이 표시됐는가?
4. SQL 이후 복귀가 표시됐는가?
5. 외부 연동 이후 복귀가 표시됐는가?
6. Local Method를 놓치지 않았는가?
7. 실제 분기를 유지했는가?
8. Dependency Boundary 호출을 삭제하지 않았는가?
9. JAR를 검색하지 않았는가?
10. target/build/lib를 검색하지 않았는가?
11. common 때문에 광범위 검색하지 않았는가?
12. Evidence가 상대경로인가?
13. Oracle Metadata가 없는가?

검증을 위한 Source 재검색은 하지 않는다.

---

# 65. 출력

완료 후 대화에는 다음만 출력한다.

## BE Full Fast Analysis Result

- URL:
- HTTP Method:
- BE ID:
- Output:
- Controller:
- Entry Service:
- Service Chain Count:
- Local Method Count:
- Mapper Call Count:
- SQL Statement Count:
- External Integration Count:
- Dependency Boundary Count:
- Common Boundary Count:
- Final Status:

### Search Boundary

- Java: gipms-api-*/src/main/java/**
- Resources: gipms-api-*/src/main/resources/**
- JAR: FORBIDDEN
- target/build/lib: FORBIDDEN
- Common Expansion: FORBIDDEN
- Cross Project: EXACT BUSINESS TYPE ONLY

### Performance Strategy

- Java: METHOD_PARTIAL_READ
- Mapper Java: SKIPPED_UNLESS_REQUIRED
- XML: STATEMENT_PARTIAL_READ
- Evidence: COLLECTED_DURING_ANALYSIS
- Reference: READ_ONCE
- Markdown: WRITE_ONCE

### Analysis Status

COMPLETED

또는

AMBIGUOUS

또는

NOT_FOUND
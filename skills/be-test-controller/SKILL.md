---
name: be-test-controller
description: Backend URL 하나를 대상으로 Controller부터 실제 종료점까지 부분 Read와 실제 호출 기반으로 빠르게 추적하고 BE-REFERENCE 형식의 최종 BE 분석 문서를 생성하는 전체 성능 테스트 Skill.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep, Write
---

# BE Full Fast Analysis Test

## 0. 목적

이 Skill은 지금까지 검증한 FAST 탐색 규칙을 사용하여
실제 최종 BE 분석 전체 범위를 수행한다.

이번 테스트는 부분 단계 테스트가 아니다.

다음 전체 실행 흐름을 끝까지 추적한다.

Controller
→ Service
→ ServiceImpl
→ Validation
→ 조건/분기
→ Local/private Method
→ 다른 Service
→ Mapper
→ MyBatis XML
→ SQL
→ 실제 사용되는 include
→ SAP/RFC/외부 연동
→ 후처리
→ Response
→ Exception
→ 실제 종료점

그리고 BE-REFERENCE 형식을 사용하여
최종 Markdown 문서를 생성한다.

핵심 원칙:

검색 범위는 좁게,
호출 깊이는 끝까지.

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

출력 기본 경로:

docs/analysis/{화면명}/backend/

파일명:

{FE Base Name}-BE-{NNN}.md

예:

docs/analysis/자재등록/backend/FE-ACT-010-create-BE-001.md

---

# 3. BE ID 결정

출력 디렉토리에서 현재 FE Base Name에 해당하는
BE 파일명만 확인한다.

예:

FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md

가 존재하면:

다음 ID:

BE-003

기존 번호의 최대값 + 1을 사용한다.

파일 내용은 읽지 않는다.

파일명만 확인한다.

동일 Backend URL의 기존 결과가 명확히 확인되는 경우
임의 overwrite하지 않는다.

---

# 4. 검색 대상

Backend 프로젝트:

gipms-api-*

Frontend 프로젝트는 분석하지 않는다.

---

# 5. 검색 제외

다음은 검색하지 않는다.

- docs/**
- sample/**
- **/sample/**
- target/**
- build/**
- *.jar
- test/**
- **/test/**
- generated/**
- node_modules/**

절대경로 기반 탐색을 하지 않는다.

Evidence는 Project Root 기준 상대경로만 사용한다.

---

# 6. BE-REFERENCE

최종 문서 형식 확인을 위해
BE-REFERENCE를 분석 시작 시 정확히 1회만 Read한다.

중간 분석 중 다시 읽지 않는다.

BE-REFERENCE는:

- 출력 구조
- 제목
- 표
- 표현 방식
- 실행 흐름 표현

을 위한 Format Reference다.

BE-REFERENCE 자체는 Evidence가 아니다.

중요:

BE-REFERENCE의 예제 내용을
실제 분석 결과로 복사하지 않는다.

실제 Source에서 확인한 내용만 작성한다.

---

# 7. 핵심 FAST 규칙

반드시 다음 규칙을 따른다.

1. 전체 Java 파일 Read를 기본적으로 금지한다.
2. 정확한 Symbol 위치를 먼저 찾는다.
3. Method 위치 확인 후 해당 범위만 Read한다.
4. 초기 Java Method Read는 약 80줄을 기준으로 한다.
5. Method가 계속될 때만 다음 범위를 읽는다.
6. Method 종료 즉시 추가 Read를 중단한다.
7. XML도 전체 Read하지 않는다.
8. XML Statement는 약 40줄부터 시작한다.
9. Statement 종료가 안 보일 때만 다음 범위를 읽는다.
10. 동일 검색을 반복하지 않는다.
11. 동일 파일을 다시 찾지 않는다.
12. 동일 Method를 다시 분석하지 않는다.
13. 이미 확보한 정보를 다시 검색하지 않는다.
14. 현재 Backend 프로젝트를 우선한다.
15. 현재 프로젝트에서 찾지 못할 때만 다른 gipms-api-*로 확대한다.
16. Evidence는 최초 Read 시 같이 확보한다.
17. Evidence를 위한 두 번째 탐색을 하지 않는다.
18. 실제 호출된 경로만 추적한다.
19. 관련 있어 보인다는 이유로 주변 코드를 탐색하지 않는다.
20. 종료점 이후 탐색하지 않는다.

---

# 8. 탐색 상태 관리

분석 중 내부적으로 다음을 유지한다.

KNOWN_FILES

KNOWN_SYMBOLS

KNOWN_VARIABLE_TYPES

VISITED_METHODS

VISITED_XML

VISITED_STATEMENTS

VISITED_INCLUDES

VISITED_EXTERNAL_CALLS

CALL_STACK

VISITED_METHODS는 다음 형태로 관리한다.

ClassName#MethodName(signature)

예:

MaterialServiceImpl#create(MaterialRequest)

ValidationServiceImpl#check(MaterialRequest)

동일 이름 Method라도 Class 또는 signature가 다르면
별개로 처리한다.

---

# 9. URL 분리

입력 Backend URL에서:

- 상위 path
- 마지막 segment

를 확인한다.

예:

/material/create

상위:

/material

마지막:

/create

Controller 검색에서는
상위 path를 우선 사용한다.

---

# 10. Controller 검색

검색 범위:

gipms-api-*/**/*Controller.java

URL 상위 path를 먼저 Grep한다.

후보가 발견되면
전체 Backend URL 검색을 중단한다.

후보 Controller에서만:

- class-level mapping
- method-level mapping

을 확인한다.

---

# 11. HTTP Method

입력 Method가 명확하면 annotation과 비교한다.

UNKNOWN이면 Controller annotation에서 확정한다.

지원:

GET
POST
PUT
DELETE
PATCH

동일 URL에 여러 Method가 있고
UNKNOWN 상태에서 하나로 확정할 수 없으면:

AMBIGUOUS

로 종료한다.

---

# 12. Controller Method

실제 URL과 HTTP Method가 일치하는
Controller Method 하나를 확정한다.

확인:

- Controller Class
- Controller Method
- Mapping
- HTTP Method
- Request
- Response
- Evidence

다른 Controller Method는 분석하지 않는다.

---

# 13. Controller Validation

Controller Method 내부에서 직접 수행되는 Validation이 있으면 추적한다.

예:

@Valid

BindingResult

직접 조건 검사

필수값 확인

Validation 실패 시:

- 조건
- 처리
- Response 또는 Exception

을 기록한다.

실제로 없는 Validation을 추측하지 않는다.

---

# 14. Service 호출

Controller에서 실제 호출되는 Service를 확인한다.

예:

materialService.create(request);

확인:

- Service Variable
- Service Type
- Service Method
- Parameter
- Return 처리

---

# 15. Service Interface

필요한 경우 정확한 Service Type만 검색한다.

예:

interface MaterialService

호출된 Method 선언만 확인한다.

전체 Service 파일 분석은 하지 않는다.

---

# 16. ServiceImpl

implements ServiceType

으로 실제 구현체를 찾는다.

현재 Backend 프로젝트를 우선한다.

구현체가 확정되면 다른 구현체 검색을 중단한다.

ServiceImpl 전체 파일을 Read하지 않는다.

---

# 17. ServiceImpl Method 부분 Read

호출된 Method 이름을
확정된 ServiceImpl 파일 하나에서 Grep한다.

Method 시작 line 확보 후:

약 80줄 Read

↓

Method 종료 여부 확인

↓

필요한 경우에만 다음 약 80줄

Method 종료 즉시 중단한다.

---

# 18. Method 내부 분석

실제 Method 범위에서 다음을 확인한다.

- Validation
- 조건문
- 상태값 변경
- Parameter 생성
- DTO 변환
- Local/private Method
- 다른 Service 호출
- Mapper 호출
- 외부 연동
- Return
- Exception

실제 실행 흐름 순서를 유지한다.

---

# 19. Validation

다음 형태를 포함한다.

예:

if (...) {
    throw ...
}

Assert...

Objects.requireNonNull(...)

Validator 호출

Validation Service 호출

Validation마다 다음을 기록한다.

- 검사 대상
- 조건
- YES 흐름
- NO 흐름
- Exception 또는 후속 처리

---

# 20. 조건/분기

다음 분기를 실제 코드대로 유지한다.

- if
- else
- else if
- switch
- ternary
- early return

예:

IF exists

├─ YES
│  → update
└─ NO
   → insert

분기를 임의로 합치지 않는다.

---

# 21. Local/private Method

동일 Class 내부 Method 호출이 있으면 추적한다.

예:

create()
→ validate()
→ saveMaterial()

Local Method는 동일 파일에서
정확한 Method 이름을 Grep한다.

Method 위치부터 부분 Read한다.

전체 ServiceImpl 파일을 읽지 않는다.

Local Method 내부에서도 동일 분석 규칙을 적용한다.

---

# 22. Local Method 재귀

Local Method 내부에서 또 다른 Local Method가 호출되면
계속 추적한다.

예:

create()
→ save()
→ createHistory()
→ normalize()

단:

이미 VISITED_METHODS에 있는 Method는 재분석하지 않는다.

순환 호출을 방지한다.

---

# 23. 다른 Service 호출

현재 Method에서 다른 Service가 호출되면
실제 호출된 Method를 추적한다.

예:

validationService.check(request);

확인:

Service Variable

↓

Service Type

↓

Service Interface

↓

ServiceImpl

↓

check() 위치

↓

부분 Read

다른 Service 전체를 분석하지 않는다.

호출된 Method만 분석한다.

---

# 24. 다중 Service 왕복

다른 Service 호출이 끝나면
원래 Caller 흐름으로 복귀한다.

예:

MaterialServiceImpl.create()

→ ValidationServiceImpl.check()

   → ValidationMapper.select()

   → SQL

   → return

→ MaterialServiceImpl.create() 복귀

→ MaterialMapper.insert()

→ SQL

이 왕복 구조를 Execution Flow에 유지한다.

---

# 25. Service Chain 재귀

호출된 Service가 또 다른 Service를 호출하면
실제 호출 체인을 계속 추적한다.

예:

ServiceA.method()

→ ServiceB.check()

→ ServiceC.validate()

→ Mapper

실제 종료점까지 추적한다.

이미 방문한 Method는 다시 분석하지 않는다.

---

# 26. Mapper 호출

실제 Method에서 Mapper 호출을 확인한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

확인:

- Caller
- Mapper Variable
- Mapper Type
- Mapper Method
- Parameter
- Return 사용

---

# 27. Mapper Type

Mapper Variable Type이 이미 읽은 범위에 없으면
해당 ServiceImpl 파일 하나에서
Mapper Variable 이름만 Grep한다.

예:

materialMapper

↓

private final MaterialMapper materialMapper;

Type 확보 후 검색을 중단한다.

---

# 28. Mapper Java FAST 정책

기본 FAST 경로에서는 Mapper Java Interface를 중간 분석 단계로 사용하지 않는다.

즉:

ServiceImpl
→ Mapper Variable Type
→ Mapper Method
→ MyBatis XML

로 바로 이동한다.

Mapper Java 전체 Read는 하지 않는다.

Mapper Java Method를 단순 확인하기 위한 별도 탐색도 기본적으로 하지 않는다.

단:

XML 연결이 모호하거나
Mapper Type/Method 매핑을 확정할 수 없는 경우에만
Mapper Java를 보조 Evidence로 최소 확인할 수 있다.

---

# 29. MyBatis XML 검색

현재 Backend 프로젝트를 우선한다.

Mapper Type을 이용해 namespace 후보를 찾는다.

예:

MaterialMapper

↓

<mapper namespace="...MaterialMapper">

XML이 확정되면
다른 XML 검색을 중단한다.

---

# 30. XML fallback

Mapper Type으로 XML을 찾지 못한 경우에만
Mapper Method를 사용한다.

예:

id="insertMaterial"

현재 프로젝트에서 먼저 검색한다.

없을 때만 다른 gipms-api-*로 확대한다.

---

# 31. Statement

확정된 XML 파일에서
실제 Mapper Method의 statement만 찾는다.

예:

<select id="selectMaterial">

<insert id="insertMaterial">

<update id="updateMaterial">

<delete id="deleteMaterial">

Statement 시작 위치부터 약 40줄 Read한다.

종료가 안 보일 때만 추가 Read한다.

---

# 32. SQL 분석

실제 Statement에서 다음을 분석한다.

- SQL Type
- Main Table
- Additional Table
- JOIN
- WHERE
- Parameter
- Dynamic SQL
- 조건 분기
- foreach
- include
- resultMap 참조

실제 코드에 있는 것만 기록한다.

---

# 33. Parameter Mapping

Mapper 호출 Parameter와
SQL의 #{...} 사용을 연결한다.

예:

Service:

param.setMaterialId(request.getMaterialId());

↓

Mapper:

materialMapper.selectMaterial(param)

↓

SQL:

WHERE MATERIAL_ID = #{materialId}

가능한 범위에서 실제 Source State와
실제 전송값을 구분한다.

추측하지 않는다.

---

# 34. Dynamic SQL

다음을 실제 실행 구조로 분석한다.

- if
- choose
- when
- otherwise
- foreach

예:

<if test="plantCode != null">

이면:

plantCode != null
├─ YES → 조건 SQL 포함
└─ NO  → 조건 SQL 미포함

---

# 35. include

이번 전체 테스트에서는
실제 Statement가 사용하는 include를 추적한다.

예:

<include refid="Base_Column_List"/>

이면 해당 fragment를 찾는다.

단:

실제로 호출된 Statement가 참조하는 include만 찾는다.

XML 전체의 다른 sql fragment는 탐색하지 않는다.

---

# 36. include 재귀

include fragment 내부에서
다른 include를 다시 사용하는 경우
실제 참조된 것만 계속 추적한다.

VISITED_INCLUDES를 사용한다.

이미 확인한 fragment는 다시 읽지 않는다.

---

# 37. resultMap

resultMap이 실제 Response/후처리 이해에 필요한 경우에만
해당 resultMap을 최소 범위로 확인한다.

단순히 resultMap 속성이 있다는 이유만으로
모든 Mapping을 분석하지 않는다.

필요하지 않으면:

ResultMap:
REFERENCE_ONLY

로 기록할 수 있다.

---

# 38. SQL 이후 흐름

Mapper/SQL 실행 후 반드시 Caller로 복귀한다.

예:

Mapper SELECT
→ 결과 반환
→ ServiceImpl result 변수
→ IF result exists
→ DTO 변환
→ 다음 Mapper
→ return

SQL에서 분석을 끝내지 않는다.

---

# 39. 외부 연동 탐지

실제 호출 체인에서 다음과 같은 외부 연동이 발견되면 추적한다.

- SAP
- RFC
- Feign
- RestTemplate
- WebClient
- HTTP Client
- 외부 SDK
- 외부 시스템 Adapter
- Gateway
- Client Class

실제로 호출된 경우에만 추적한다.

프로젝트 전체에서 SAP/RFC를 선검색하지 않는다.

---

# 40. SAP/RFC

실제 SAP/RFC 호출이 발견되면 다음을 확인한다.

- 호출 Method
- Function/연동 식별자
- Request 생성
- Parameter Mapping
- 호출 조건
- 실제 외부 호출
- Response 수신
- 성공 처리
- 실패 처리
- Exception
- 원래 BE 흐름 복귀

외부 호출에서 분석을 끝내지 않는다.

---

# 41. 외부 HTTP/API

Feign/RestTemplate/WebClient 등이 실제 호출되면 다음을 확인한다.

- 대상 Client
- 호출 Method
- HTTP Method
- URL 또는 endpoint
- Request
- Header
- Query/Path/Body
- Response
- Error 처리
- Caller 복귀

확인 가능한 Source까지만 기록한다.

추측하지 않는다.

---

# 42. 외부 연동의 추가 내부 추적

외부 연동 전에 별도 Adapter/Service/Helper를 거치면
실제 호출된 Method만 추적한다.

예:

ServiceImpl
→ SapService
→ SapAdapter
→ RFC Call

각 Method는 부분 Read한다.

전체 Class를 읽지 않는다.

---

# 43. 후처리

DB 또는 외부 연동 이후 수행되는 후처리를 반드시 추적한다.

예:

- 결과 상태 확인
- DTO 변환
- List 가공
- 값 재설정
- 추가 Validation
- 후속 DB 저장
- History 저장
- Response 생성

DB/SAP 호출만 보고 분석을 종료하지 않는다.

---

# 44. Exception

실제 실행 경로에서 다음을 확인한다.

- throw
- catch
- finally
- custom exception
- error response
- validation failure
- mapper failure 처리
- 외부 연동 failure 처리

Exception이 발생하는 조건을 가능한 경우 분기와 연결한다.

---

# 45. Return / Response

최종적으로 Controller까지 반환되는 값을 추적한다.

확인:

ServiceImpl Return

↓

Service Return

↓

Controller Response

가능한 경우:

- Response DTO
- 상태값
- HTTP Status
- Response Body

를 기록한다.

Response가 없는 구조도 허용한다.

---

# 46. 종료점 정의

하나의 실행 Path는 다음 중 하나에 도달하면 종료된다.

1. 정상 Response
2. 정상 void 종료
3. 명시적 return
4. Exception
5. Error Response

외부 연동은 그 자체로 종료점이 아니다.

외부 연동 이후 Caller로 복귀하여
최종 처리까지 추적한다.

---

# 47. 모든 실제 분기 완료

하나의 Method에 여러 실제 분기가 존재하면
각 분기를 종료점까지 추적한다.

예:

IF exists

├─ YES
│  → UPDATE
│  → Response
│
└─ NO
   → INSERT
   → SAP
   → 후처리
   → Response

한 분기만 분석하고 종료하지 않는다.

---

# 48. 탐색 종료 조건

다음 조건이 모두 만족되면 Source 탐색을 종료한다.

- Controller 확정
- 실제 Service chain 확인
- Local Method 실제 호출 확인
- 다른 Service 실제 호출 확인
- Mapper/SQL 실제 호출 확인
- 외부 연동 실제 호출 확인
- 후처리 확인
- Exception 확인
- 모든 실제 분기가 종료점 도달

그 이후 관련 코드를 더 찾지 않는다.

---

# 49. 금지

절대 하지 않는다.

- 관련 있을 것 같은 Class 탐색
- 사용되지 않는 Mapper 탐색
- 사용되지 않는 XML Statement 탐색
- 프로젝트 전체 SAP/RFC 선검색
- 프로젝트 전체 Exception 검색
- 전체 Java 파일 반복 Read
- 전체 XML 반복 Read
- 동일 Method 재분석
- 동일 SQL 재분석
- Evidence 재수집
- Oracle Metadata 분석
- DB Metadata 분석
- JAR 탐색
- sample 탐색
- docs 탐색

---

# 50. Evidence

Evidence는 분석과 동시에 확보한다.

형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:420-468

gipms-api-material/src/main/java/.../ValidationServiceImpl.java:80-115

gipms-api-material/src/main/resources/.../MaterialMapper.xml:210-250

절대경로를 사용하지 않는다.

Evidence를 얻기 위한 두 번째 검색을 하지 않는다.

---

# 51. 실행 흐름

최종 문서에는 실제 호출 순서를 유지한
전체 실행 흐름을 작성한다.

단순:

Controller
→ Service
→ Mapper

형태로 축약하지 않는다.

예:

Controller.create()

→ MaterialServiceImpl.create()

→ ValidationServiceImpl.check()

   → ValidationMapper.selectExists()

   → SELECT TB_MATERIAL

   → 결과 반환

→ MaterialServiceImpl.create() 복귀

→ IF exists

   ├─ YES

   │  → MaterialMapper.updateMaterial()

   │  → UPDATE TB_MATERIAL

   └─ NO

      → MaterialMapper.insertMaterial()

      → INSERT TB_MATERIAL

      → SapService.send()

         → RFC Request 생성

         → RFC 호출

         → Response 수신

      → MaterialServiceImpl.create() 복귀

→ Response 생성

→ Controller 반환

---

# 52. 한눈에 보는 실행 흐름

BE-REFERENCE에서 정의한
한눈에 보는 실행 흐름 형식을 사용한다.

단:

실제 Source에서 확인된 실행 순서만 사용한다.

중간 Service 왕복,
DB,
SAP/RFC,
분기,
후처리를 생략하지 않는다.

---

# 53. 최종 문서 생성

모든 Source 분석이 완료된 후에만
Markdown 문서를 생성한다.

분석 도중 Markdown을 반복 수정하지 않는다.

즉:

Source 탐색 완료

↓

분석 결과 정리

↓

BE-REFERENCE 형식 적용

↓

최종 Markdown 1회 Write

이 순서를 사용한다.

---

# 54. 최종 문서 내용

BE-REFERENCE의 기존 형식을 그대로 유지한다.

최소한 실제 분석 결과에는 다음 정보가 반영되어야 한다.

- 대상 API
- Controller
- Request
- Validation
- Service
- 비즈니스 로직
- 조건/분기
- Local Method
- 다른 Service 호출
- Mapper
- SQL
- Parameter Mapping
- Dynamic SQL
- 외부 연동
- SAP/RFC
- 후처리
- Response
- Exception
- Evidence
- 전체 실행 흐름
- 한눈에 보는 실행 흐름

해당 항목이 실제 Source에 없으면
없는 내용을 만들어내지 않는다.

---

# 55. 최종 검증

문서 생성 전 내부적으로 확인한다.

1. 실제 호출되지 않은 내용을 넣지 않았는가?
2. Controller부터 종료점까지 연결됐는가?
3. SQL 이후 Caller 복귀를 놓치지 않았는가?
4. 외부 연동 이후 복귀를 놓치지 않았는가?
5. 모든 실제 분기를 확인했는가?
6. Local Method를 놓치지 않았는가?
7. 다른 Service 왕복을 놓치지 않았는가?
8. Evidence가 실제 Source와 연결되는가?
9. 절대경로가 없는가?
10. Oracle Metadata가 포함되지 않았는가?
11. BE-REFERENCE를 Evidence로 사용하지 않았는가?
12. 불필요한 전체 파일 Read를 하지 않았는가?

Validator를 위한 Source 재검색은 하지 않는다.

이미 확보한 분석 결과를 기준으로 검증한다.

---

# 56. 출력

최종 Markdown 파일을 생성한 후
대화에는 간단히 다음만 출력한다.

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
- External Integration:
- Final Status:

### Performance Search Strategy

- Java: METHOD_PARTIAL_READ
- Mapper Java: SKIPPED_UNLESS_REQUIRED
- MyBatis XML: STATEMENT_PARTIAL_READ
- Evidence: COLLECTED_DURING_ANALYSIS
- Reference: READ_ONCE

### Analysis Status

COMPLETED

또는

AMBIGUOUS

또는

NOT_FOUND

전체 분석이 끝난 이후
추가 Source 탐색을 수행하지 않는다.
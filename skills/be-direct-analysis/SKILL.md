---
name: be-direct-analysis
description: API 분석 문서 없이 FE Base Name, HTTP Method와 Backend URL을 직접 입력받아 Controller를 찾고 기존 Backend 분석 규칙과 BE Reference를 그대로 적용하여 실제 Backend Business Logic을 분석한다.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# Backend Direct Analysis

## 1. 목적

API 분석 문서 없이 Backend URL을 직접 입력받아
해당 Backend API의 실제 Backend Business Logic을 분석한다.

기존:

```text
API Document
 ↓
Controller 확인
 ↓
Backend Analysis
```

Direct:

```text
HTTP Method + Backend URL
 ↓
Controller 직접 탐색
 ↓
Backend Analysis
```

Controller를 확정한 이후의 분석 방식과 상세 수준은
기존 `be-analysis`와 동일한 원칙을 적용한다.

분석은 Layer 순서가 아니라
실제 Source의 실행 순서를 기준으로 수행한다.

예:

```text
Controller
 ↓
Service
 ↓
Validation
 ↓
DB #1
 ↓
조건
 ├─ YES
 │   ↓
 │  RFC #1
 │   ↓
 │  RFC Response 처리
 │
 └─ NO
     ↓
    다른 처리
 ↓
DB #2
 ↓
REST #1
 ↓
후속 Business Logic
 ↓
Response
```

DB / RFC / REST / 기타 외부 연동은
실제 호출 위치에 배치한다.

Oracle Metadata 확인은 실제 실행 단계가 아니므로
Business Logic 중간 단계로 삽입하지 않는다.


## 2. 적용 Rule

이 Skill을 실행할 때 반드시 다음 Rule을 적용한다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

두 Rule의:

```text
분석 범위
호출 추적
Source Evidence
Oracle Metadata 최적화
안전 규칙
STOP 조건
Shell 탐색 제한
```

을 준수한다.

이 Direct Skill 때문에 기존 Rule의 분석 깊이를
축소하거나 우회하지 않는다.


## 3. BE Reference 로드

Backend 분석을 시작할 때 다음 파일의 존재 여부를 확인한다.

```text
.claude/references/BE-REFERENCE.md
```

존재하면 현재 Backend 분석 시작 시
**1회만 읽는다.**

분석 도중 반복해서 다시 읽지 않는다.

다음 단계에서 재로드하지 않는다.

```text
Controller 탐색 중
Source 추적 중
Service 분석 중
Mapper / SQL 분석 중
Oracle Metadata 확인 전/후
RFC / REST 분석 중
최종 문서 작성 직전
문서 저장 후
```

처음 읽은 Reference를 현재 분석이 끝날 때까지 사용한다.

BE-REFERENCE는 다음을 결정하기 위한 Reference이다.

```text
문서 구조
표현 형식
상세 수준
ASCII Tree 형식
Excel 단일 셀 복사 구조
Block 독립성
Oracle Metadata 표현 형식
```

BE-REFERENCE는 Source Evidence가 아니다.

Reference 안의 Sample:

```text
Class
Method
Path
SQL
Table
Column
RFC Function
URL
Parameter
Response
Sample Value
```

를 현재 분석 결과로 복사하지 않는다.

실제 결과는 반드시 현재 분석 대상 Source,
MyBatis SQL 및 필요한 경우 확인한 Oracle Metadata를 기준으로 한다.


## 4. 입력

기본 입력:

```text
<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>
```

예:

```text
/be-direct-analysis equipment-search FE-ACT-010-create POST /api/equipment/create
```

Method를 모르면:

```text
/be-direct-analysis equipment-search FE-ACT-010-create UNKNOWN /api/equipment/create
```

입력:

```text
화면명
FE Base Name
HTTP Method
Backend URL
```

FE Base Name 예:

```text
FE-ACT-010-create
```

Backend URL로부터 FE Base Name을 추측하지 않는다.


## 5. Orchestrator / Worker 입력

Orchestrator 또는 Worker를 통해 실행되는 경우
다음 값이 추가로 전달될 수 있다.

```text
BE ID
Output File
```

예:

```text
화면명:
equipment-search

FE Base Name:
FE-ACT-010-create

HTTP Method:
POST

Backend URL:
/api/equipment/create

BE ID:
BE-001

Output File:
FE-ACT-010-create-BE-001.md
```

`BE ID`, `Output File`은
Orchestrator / Worker 실행을 위한 명시적 입력이다.

전달받은 값은 Direct Skill에서 다시 계산하지 않는다.


## 6. FE Base Name

FE Base Name은 Backend 문서가
어떤 FE Action에 속하는지를 나타낸다.

예:

```text
FE-ACT-010-create
```

출력:

```text
FE-ACT-010-create-BE-001.md
```

FE Base Name은 Backend URL에서 생성하지 않는다.

예를 들어:

```text
POST /api/equipment/create
```

만 보고:

```text
FE-ACT-001-create
```

를 임의 생성하지 않는다.

FE Base Name이 입력되지 않았다면 STOP 한다.


## 7. API Document 사용 금지

이 Skill은 Direct Backend 분석용이다.

따라서 다음 위치의 API 문서를
분석 시작점으로 요구하지 않는다.

```text
docs/analysis/{화면명}/api/
```

다음 파일이 없어도 분석할 수 있어야 한다.

```text
FE-ACT-010-create-API-001.md
```

API Document를 새로 생성하지 않는다.

API ID도 Direct Backend 분석의
필수 입력으로 사용하지 않는다.


## 8. Project Root 확인

Code Index MCP에 설정된 현재 Project Root를 기준으로 분석한다.

Backend Source 대상은 기본적으로:

```text
gipms-api-*
```

프로젝트이다.

현재 Backend URL과 실제 호출 관계가 있는
Backend Project만 분석한다.

모든 `gipms-api-*` 프로젝트를
처음부터 전체 탐색하지 않는다.


## 9. Source Path 기준

Source Path는 모두
Project Root 기준 상대경로로 기록한다.

예:

```text
gipms-api-equipment/src/main/java/...
gipms-api-common/src/main/java/...
```

절대경로를 사용자에게 요청하지 않는다.

절대경로를 추측하거나
최종 문서에 생성하지 않는다.


## 10. Controller 직접 탐색

API Document 대신 다음 값을 시작점으로 사용한다.

```text
HTTP Method
Backend URL
```

Code Index MCP를 우선 사용하여
Controller 후보를 탐색한다.

확인 대상:

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Class Level Mapping과
Method Level Mapping을 조합하여
최종 Backend URL을 확인한다.

예:

```java
@RequestMapping("/api/equipment")
```

+

```java
@PostMapping("/create")
```

↓

```text
POST /api/equipment/create
```

Controller 후보를 찾은 후에는
반드시 실제 Source를 읽고 Mapping을 확정한다.

Code Index 결과만으로
Controller를 확정하지 않는다.


## 11. HTTP Method 확인

입력 Method가 다음과 같이 명확한 경우:

```text
GET
POST
PUT
PATCH
DELETE
```

Method + Backend URL을 함께 사용하여
Controller를 확인한다.

실제 Source와 입력값이 다르면
차이를 기록한다.

임의로 다른 Controller로 변경하지 않는다.


## 12. UNKNOWN Method

입력:

```text
UNKNOWN /api/equipment/create
```

인 경우 Controller Source에서
실제 HTTP Method를 확인한다.

확인 대상:

```text
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@RequestMapping(method = ...)
```

확정되면:

```text
Input Method
  UNKNOWN

Resolved Method
  POST
```

형태로 기록한다.


## 13. UNKNOWN + 동일 URL 다중 Method

동일 Backend URL에 여러 HTTP Method가 존재하고
입력 Method가 UNKNOWN이라면
하나를 임의로 선택하지 않는다.

예:

```text
GET  /api/equipment
POST /api/equipment
```

결과:

```text
STATUS:
STOP

사유:
동일 Backend URL에 여러 HTTP Method 존재

확인된 Method:
GET
POST
```

사용자 또는 상위 Orchestrator가
Method를 지정해야 한다.


## 14. Controller 탐색 실패

Backend URL에 해당하는 Controller를
실제 Source에서 찾을 수 없다면
다른 유사 URL을 임의 선택하지 않는다.

```text
STATUS:
STOP

HTTP METHOD:
POST

BACKEND URL:
/api/equipment/create

사유:
Controller Mapping 확인되지 않음
```

그리고 STOP 한다.


## 15. Code Index 사용

Code Index MCP는 다음 작업에 우선 활용한다.

```text
Controller Mapping 탐색
Symbol 검색
Reference 검색
Caller 검색
Callee 검색
Call Relationship 확인
관련 Source 위치 탐색
```

Code Index는 탐색용이다.

Business Logic은 반드시 실제 Source를 읽고 확인한다.

필요한 경우 정확한:

```text
Symbol
Method
Statement ID
namespace
URL
```

위치 확인을 위해
좁은 범위의 파일 검색을 보조적으로 사용할 수 있다.

관련 없는 프로젝트 전체를
무차별적으로 검색하지 않는다.

다음은 사용하지 않는다.

```text
xargs

Project Root 전체 대상
find

grep -r
grep -R
grep -rn
```

Rule 09의 Shell 기반 재귀 탐색 제한을 준수한다.


## 16. 분석 시작점 확정

실제 Controller Source에서 다음을 확인한다.

```text
Controller Class
Controller Mapping
Controller Method
HTTP Method
Backend URL
Request 처리
호출 Service / Method
```

여기까지 실제 Source로 확인한 후
Backend Business Logic 추적을 시작한다.


## 17. Controller 분석

Controller에서 실제 실행되는 내용을 확인한다.

확인 대상:

```text
Request 처리
Validation
Parameter / DTO
Service 호출
조건 처리
Exception 관련 처리
Response 처리
```

Controller에서 여러 Method 또는 Service를 호출하면
실제 실행 순서대로 모두 추적한다.


## 18. Service / 구현체 추적

Service Interface가 존재하면
실제 구현체를 확인한다.

예:

```text
EquipmentService
 ↓
EquipmentServiceImpl
```

구현체가 여러 개이고
실제 구현체를 Source Evidence로 확정할 수 없다면
하나를 임의로 선택하지 않는다.

```text
실제 구현체:
확인되지 않음
```

으로 기록한다.


## 19. Business Logic 분석

Service Method Body를
실제 실행 순서대로 분석한다.

확인 대상:

```text
입력값 처리
Validation
조건
분기
반복
계산
데이터 변환
상태 변경
내부 Method
다른 Service
공통 Service
Mapper
DB
RFC
REST
기타 외부 연동
Exception
Return
Response 생성
```

분석 결과를 Layer별로 다시 정렬하지 않는다.


## 20. 하위 호출 추적

현재 Business Logic에서 다음 호출이 발생하면
현재 Backend URL 흐름에 직접 관련된 범위에서 추적한다.

```text
같은 Class 내부 Method
다른 Service
공통 Service
Interface Service
Adapter
Client
Mapper
```

현재 Project Root 아래 실제 Source가 존재하면
Business Logic 이해에 필요한 중간 호출 단계를 생략하지 않는다.


## 21. Caller 복귀

하위 호출 분석이 끝나면
반드시 호출한 Business Logic으로 돌아간다.

예:

```text
Service A
 │
 ├─ CommonService.call()
 │      │
 │      ├─ Request 생성
 │      ├─ RFC 호출
 │      ├─ Response 변환
 │      └─ return
 │
 ├─ CommonService 결과 확인
 ├─ DB UPDATE
 └─ Response 생성
```

하위 Service / DB / RFC / REST 분석이 끝났다는 이유로
전체 Backend 분석을 종료하지 않는다.

Oracle Metadata 확인을 위해
Caller 복귀를 지연하지 않는다.


## 22. Mapper / MyBatis 추적

Mapper 호출이 발견되면 가능한 경우 다음까지 연결한다.

```text
Service
 ↓
Mapper Interface
 ↓
Mapper Method
 ↓
MyBatis namespace
 ↓
Statement ID
 ↓
MyBatis XML 또는 Annotation SQL
 ↓
Dynamic SQL
 ↓
실제 SQL
```

Mapper Method 이름만으로 SQL을 추측하지 않는다.

MyBatis XML은:

```text
namespace + Statement ID
```

를 기준으로 정확하게 연결한다.

SQL에서 사용된 Table / View / Column은
후반 Oracle Metadata 검증을 위해 기록한다.

이 단계에서 Oracle MCP를 즉시 호출하지 않는다.


## 23. Dynamic SQL 분석

실제 SQL에 다음 요소가 존재하면
실제 조건 구조를 보존한다.

```text
if
choose
when
otherwise
foreach
where
set
trim
include
bind
```

Dynamic SQL을 하나의 고정 SQL처럼 표현하지 않는다.

`include`가 사용되면
현재 SQL 이해에 필요한 SQL Fragment를 확인한다.


## 24. SQL Parameter Mapping

가능한 경우 Parameter 전달 흐름을 연결한다.

예:

```text
Request
plantCode
 ↓
Service
plantCode
 ↓
Mapper
plantCode
 ↓
MyBatis
#{plantCode}
 ↓
Oracle
PLANT_CODE
```

Source에서 확인되지 않는 Mapping은 추측하지 않는다.

Oracle Metadata를 조회하지 않아도
SQL Source에서 확인 가능한 Parameter Mapping은 분석한다.


## 25. SQL Result Mapping

가능한 경우 SQL 결과가
Backend 객체로 Mapping되는 과정도 확인한다.

예:

```text
Oracle Column
EQUIPMENT_ID
 ↓
MyBatis
equipmentId
 ↓
DTO / VO
equipmentId
 ↓
Business Logic
```

`resultType`, `resultMap`, `association`, `collection` 등이
실제로 사용되는 경우 해당 구조를 확인한다.

Metadata 조회 여부와 관계없이
MyBatis Result Mapping 분석을 수행한다.


## 26. Oracle MCP

Oracle MCP는 필요한 DB Metadata 확인에만 사용한다.

Oracle Metadata는 Backend Business Logic을 추적하기 위한
필수 선행 단계가 아니다.

DB 호출마다 Oracle MCP를 즉시 호출하지 않는다.

우선 다음 순서로 전체 Backend 실행 흐름을 분석한다.

```text
Controller
 ↓
Service / 내부 호출
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL / Dynamic SQL
 ↓
Parameter / Result Mapping
 ↓
사용 Oracle Object / Column 기록
 ↓
Caller 복귀
 ↓
다음 Business Logic
 ↓
RFC / REST / 다음 DB
 ↓
Response
```

Response까지 전체 Call Path를 완료한 후:

```text
사용 Oracle Object / Column 수집
 ↓
동일 Object / Column 중복 제거
 ↓
Metadata 확인 필요성 판단
 ↓
필요한 Object / Column만 Oracle MCP 확인
 ↓
SQL Source ↔ Metadata 교차 검증
```

확인 가능 대상:

```text
Object 존재 여부
Table / View
Column 존재 여부
Data Type
Nullable
Primary Key
```

SQL Source만으로 충분히 확인되는 내용을
반복 검증하기 위해 Oracle MCP를 호출하지 않는다.

동일 Object / Column Metadata를
현재 Worker에서 반복 조회하지 않는다.

다음 방식으로 탐색하지 않는다.

```text
Schema 전체 Table 조회
Schema 전체 View 조회
Schema 전체 Column 조회
현재 Backend와 관련 없는 Object 조회
모든 Object의 모든 Column 무조건 조회
동일 Object 반복 조회
동일 Column 반복 조회
```

Oracle Metadata를 추가 조회하지 않았다고 해서
Source / MyBatis / SQL에서 확인된 정보를
`확인되지 않음`으로 변경하지 않는다.

Oracle MCP는 Read-Only로 사용한다.

실행 금지:

```text
INSERT
UPDATE
DELETE
MERGE
CREATE
ALTER
DROP
TRUNCATE
```

SQL Source와 실제 확인한 Oracle Metadata가 다르면
차이를 숨기지 않는다.


## 27. DB 호출 구분

현재 Backend 실행에서 DB 호출이 여러 번 존재하면
실제 실행 위치를 기준으로 각각 구분한다.

예:

```text
DB #1 - SELECT
DB #2 - UPDATE
DB #3 - SELECT
DB #4 - INSERT
```

같은 Mapper Statement가 여러 위치에서 호출되더라도
Execution Tree에서는 각각의 호출 위치를 보존한다.

DB 호출 번호와
Oracle Metadata 조회 횟수는 별개이다.


## 28. 외부 연동 탐지

Business Logic 중간 어디에서든
다음 외부 연동이 발견되면 분석한다.

```text
SAP RFC
RFC
REST / HTTP(S)
SOAP / WebService
다른 Backend API
Gateway
Interface Server
Message / Queue
File Interface
기타 외부 시스템
```

외부 연동을 마지막 단계로 재배치하지 않는다.


## 29. RFC 분석

RFC 호출이 발견되면
현재 Project Source에서 확인 가능한 범위에서 다음을 분석한다.

```text
호출 위치
호출 Method
RFC Function
호출 조건
Request 생성
Parameter Mapping
Input
Response
Response Mapping
Error 처리
후속 Business Logic
```

공통 Service / Adapter가
Project Root 아래 실제 Source로 존재하면
해당 호출 경로까지 추적한다.


## 30. REST / HTTP 분석

REST / HTTP 호출이 발견되면
현재 Project Source에서 확인 가능한 범위에서 다음을 분석한다.

```text
호출 위치
Client
HTTP Method
URL
Header
Path Parameter
Query Parameter
Request Body
Request Mapping
Response
Response Mapping
HTTP Status 처리
Error 처리
Timeout
후속 Business Logic
```

Configuration 값이 확인되지 않으면
실제 값을 임의로 생성하지 않는다.


## 31. 기타 외부 연동

SOAP, Message, File 등 다른 외부 연동도
실제 Source에서 확인되는 호출 구조를 기준으로 분석한다.

외부 Library 내부 구현으로 자동 확장하지 않는다.


## 32. 외부 연동 다중 호출

동일 Backend 흐름에서 외부 연동이 여러 번 발생하면
각 호출을 독립적으로 보존한다.

예:

```text
RFC #1
REST #1
RFC #2
SOAP #1
REST #2
```

같은 RFC Function 또는 Endpoint가 반복 호출되어도
실행 위치가 다르면 Execution Tree에서 구분한다.


## 33. 조건 / 분기

조건에 따라 호출 여부가 달라지면
조건 구조를 그대로 표현한다.

예:

```text
sapUseYn == "Y" ?
 ├─ YES
 │   ↓
 │  RFC #1
 │
 └─ NO
     ↓
    RFC 호출 없음
```

조건부 호출을 항상 실행되는 호출처럼 표현하지 않는다.


## 34. 반복 처리

반복문 안에서 DB 또는 외부 연동이 발생하면
반복 구조를 유지한다.

예:

```text
for each item
 │
 ├─ DB SELECT
 ├─ 조건 확인
 ├─ RFC 호출
 └─ DB UPDATE
```

반복 횟수를 Source에서 알 수 없으면
임의의 숫자를 만들지 않는다.


## 35. Exception

실제 Source에서 확인되는 Exception 흐름을 분석한다.

예:

```text
외부 호출
 ↓
Exception
 ↓
catch
 ↓
Error Mapping
 ↓
BusinessException
```

Framework의 일반적인 동작만으로
확인되지 않은 Exception 흐름을 추가하지 않는다.


## 36. Transaction

명시적인 Transaction 설정이 확인되면 기록한다.

예:

```text
@Transactional
```

DB 변경과 외부 연동이 섞여 있다면
실제 호출 순서를 그대로 유지한다.

Rollback 여부는 실제 Transaction 설정과
Exception 처리에서 확인되는 범위만 기록한다.


## 37. Response 추적

Backend 분석은 외부 연동이나 SQL에서 끝내지 않는다.

가능한 경우:

```text
DB / External Response
 ↓
Service 처리
 ↓
Data Transformation
 ↓
Response DTO / Object
 ↓
Controller Return
```

까지 계속 추적한다.

Oracle Metadata 검증보다
Response까지 전체 Call Path 추적을 우선한다.


## 38. Source Evidence

주요 단계에는 가능한 경우 다음을 기록한다.

```text
Source Path
Class
Method
Line Range
```

DB:

```text
Mapper Interface
Mapper Method
Mapper XML
Namespace
Statement ID
Line Range
SQL
필요한 경우 실제 확인한 Oracle Metadata
```

외부 연동:

```text
Source Path
Class
Method
Adapter / Client
RFC Function 또는 Endpoint
Line Range
```

확인할 수 없는 정보:

```text
확인되지 않음
```

Line Range를 추측하지 않는다.

Source Path는 Project Root 기준 상대경로를 사용한다.


## 39. JAR / 외부 Dependency 제한

다음은 자동 탐색하지 않는다.

```text
*.jar
JAR 내부 Class
Decompiled Class
.m2
.gradle
Maven Repository
Gradle Cache
Project Root 외부 Source
```

현재 Project Source에서 외부 Library Method를 호출한다면
호출 경계까지만 분석한다.

Project Root 아래 실제 Source로 존재하는
공통 프로젝트는 추적할 수 있다.


## 40. 분석 결과 신뢰 기준

Business Logic은 실제 Source를 기준으로 확정한다.

기본 우선순위:

```text
실제 Source
 ↓
Mapper / MyBatis XML / SQL
 ↓
필요한 경우 확인한 Oracle Metadata
 ↓
Runtime Evidence
 ↓
Code Index 탐색 결과
```

BE-REFERENCE는 이 Evidence 우선순위에 포함되지 않는다.

BE-REFERENCE는 문서 표현 형식과
상세 수준을 위한 Reference이다.


## 41. 전체 실행 Tree 검증

분석 완료 전에
Controller부터 Response까지 전체 흐름을 다시 확인한다.

예:

```text
Controller
 ↓
Service
 ↓
Validation
 ↓
DB #1
 ↓
Service 복귀
 ↓
조건
 ├─ YES
 │   ↓
 │  Common Service
 │   ↓
 │  RFC #1
 │   ↓
 │  Common Service 복귀
 │   ↓
 │  Service 복귀
 │
 └─ NO
     ↓
    Local 처리
 ↓
DB #2
 ↓
REST #1
 ↓
Service 복귀
 ↓
Response 생성
 ↓
Controller Return
```

중간 호출에서 분석이 끊기지 않았는지 확인한다.

전체 Call Path 검증이 끝난 후
필요한 Oracle Metadata를 후반 검증한다.


## 42. BE ID 결정

BE ID 형식:

```text
BE-001
BE-002
BE-003
...
```


### 42.1 Orchestrator / Worker에서 BE ID를 전달한 경우

전달받은 BE ID를 그대로 사용한다.

예:

```text
BE ID:
BE-001
```

다음을 수행하지 않는다.

```text
BE ID 재생성
기존 BE 문서 최대 번호 검색
최근 생성 파일 기준 번호 결정
Worker 완료 순서 기준 번호 결정
다른 Worker의 BE ID 확인
```


### 42.2 직접 실행

직접 실행에서도 FE Base Name이 필수이므로
출력 파일은 반드시:

```text
{FE Base Name}-{BE ID}.md
```

형식을 사용한다.

BE ID가 명시적으로 전달되지 않았다면
해당 FE Base Name에 연결된 기존 BE 문서와
충돌하지 않는 다음 BE ID를 결정할 수 있다.

예:

```text
기존:
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md

다음:
BE-003
```

기존 파일을 덮어쓰기 위해
번호를 재사용하지 않는다.


## 43. Output File

최종 Backend 분석 문서는:

```text
docs/analysis/{화면명}/backend/
```

에 저장한다.


### 43.1 Orchestrator / Worker에서 Output File 전달

Output File이 명시적으로 전달되면
**전달받은 파일명을 그대로 사용한다.**

예:

```text
FE Base Name:
FE-ACT-010-create

BE ID:
BE-001

Output File:
FE-ACT-010-create-BE-001.md
```

최종:

```text
docs/analysis/{화면명}/backend/FE-ACT-010-create-BE-001.md
```

다음 작업을 절대 하지 않는다.

```text
Output File 재생성
기능명으로 파일명 변경
Backend URL로 파일명 변경
BE ID 재할당
FE Base Name 변경
```

명시적으로 전달된 `Output File`이
최우선이다.


### 43.2 직접 실행

Output File이 별도로 전달되지 않은 경우:

```text
{FE Base Name}-{BE ID}.md
```

규칙을 사용한다.

예:

```text
FE Base Name:
FE-ACT-010-create

BE ID:
BE-001
```

↓

```text
FE-ACT-010-create-BE-001.md
```

다음 형태는 생성하지 않는다.

```text
BE-001.md
BE-001-create.md
BE-API-001-create.md
FE-ACT-010-BE-001.md
```


## 44. 기존 파일 처리

최종 저장 대상 Backend 문서가 이미 존재하면
사용자 요청 없이 임의로 덮어쓰지 않는다.

해당 Worker 또는 직접 실행을 STOP하고
기존 파일이 존재함을 알린다.


## 45. 최종 문서 구조

최종 Backend 문서는
분석 시작 시 1회 읽은:

```text
.claude/references/BE-REFERENCE.md
```

의 구조와 표현 형식을 적용한다.

기본 구조:

```text
1. 기능 정보
2. 기능 요약
3. 한눈에 보는 실행 흐름
4. 핵심 정보
5. 전체 실행 Tree
6. Business Logic 상세
7. DB / SQL 상세
8. Oracle Metadata
9. 외부 연동 상세
10. Exception / Transaction
11. Response 생성
12. Source Evidence
13. 미확인 항목
14. 분석 경계
```

현재 기능에 존재하지 않는 내용을
Reference Sample을 이용해 채우지 않는다.

Oracle Metadata 이외 영역은
기본적으로 Markdown Table을 사용하지 않는다.

Excel 단일 셀 복사가 가능한:

```text
┌─ 제목 ─────────────
│ 항목
│   값
└────────────────────
```

형식과 ASCII Tree를 우선한다.


## 46. 기능 정보의 API ID 처리

Direct Backend 분석에는 API Document와
API ID가 존재하지 않을 수 있다.

따라서 BE-REFERENCE의 기능 정보에서
API ID를 억지로 생성하지 않는다.

Direct 분석에서는 다음처럼 표현한다.

```text
┌─ 기능 정보 ─────────────────
│ 화면
│   equipment-search
│
│ FE Action
│   FE-ACT-010-create
│
│ BE ID
│   BE-001
│
│ HTTP Method
│   POST
│
│ Backend URL
│   /api/equipment/create
│
│ Backend Project
│   실제 Source에서 확인된 Project
└─────────────────────────────
```

API ID가 없는 것을
`API-001` 등으로 임의 생성하지 않는다.


## 47. 최종 검증

Backend 문서를 저장하기 전에 확인한다.

```text
[ ] 08-be-analysis-scope.md를 적용했는가

[ ] 09-be-call-tracing.md를 적용했는가

[ ] BE-REFERENCE.md가 존재하는 경우
    분석 시작 시 1회 읽었는가

[ ] BE-REFERENCE.md를 분석 도중
    반복 로드하지 않았는가

[ ] Reference Sample을 Evidence로 사용하지 않았는가

[ ] 입력된 FE Base Name을 그대로 사용했는가

[ ] 입력된 HTTP Method + Backend URL로
    Controller를 직접 탐색했는가

[ ] UNKNOWN인 경우 실제 Controller Source에서
    HTTP Method를 확인했는가

[ ] UNKNOWN + 동일 URL 다중 Method인 경우
    임의 선택하지 않았는가

[ ] Code Index 결과만으로
    Business Logic을 확정하지 않았는가

[ ] 실제 Controller Source를 확인했는가

[ ] Controller부터 Response까지
    실제 실행 순서대로 분석했는가

[ ] Service / 하위 호출 분석 후
    Caller Business Logic으로 복귀했는가

[ ] Mapper Method를 실제 MyBatis Statement와 연결했는가

[ ] MyBatis namespace + Statement ID를 확인했는가

[ ] Dynamic SQL 구조를 보존했는가

[ ] SQL Parameter Mapping을 추측하지 않았는가

[ ] SQL Result Mapping을 가능한 범위에서 확인했는가

[ ] DB 호출마다 Oracle Metadata를
    즉시 조회하지 않았는가

[ ] Response까지 전체 Call Path를 먼저 완료했는가

[ ] SQL에서 실제 사용된 Oracle Object / Column을 수집했는가

[ ] 동일 Object / Column을 중복 제거했는가

[ ] Metadata가 필요한 항목만 선별했는가

[ ] 필요한 경우에만 Oracle Metadata를
    Read-Only로 확인했는가

[ ] Schema 전체 Metadata를 탐색하지 않았는가

[ ] DB 호출을 실제 실행 위치별로 구분했는가

[ ] RFC / REST / 기타 외부 연동을
    실제 호출 위치에 배치했는가

[ ] 조건 / 분기 / 반복 구조를 보존했는가

[ ] Exception을 실제 Source 기준으로 분석했는가

[ ] Transaction 범위를 추측하지 않았는가

[ ] 최종 Response까지 복귀했는가

[ ] Source Path가 Project Root 기준 상대경로인가

[ ] Line Range를 추측하지 않았는가

[ ] JAR / Decompiled Source를 탐색하지 않았는가

[ ] xargs / Project Root 전체 Shell 재귀 검색을
    사용하지 않았는가

[ ] 전달받은 BE ID를 변경하지 않았는가

[ ] 전달받은 Output File을 변경하지 않았는가

[ ] Output File이
    {FE Base Name}-{BE ID}.md 형식인가
```


## 48. Worker 실행 결과

Orchestrator / Worker에 의해 실행된 경우
상위 Agent에 전체 Backend 문서를 다시 반환하지 않는다.

다음 정보만 반환한다.

```text
STATUS:
PASS | STOP | ERROR

FE BASE NAME:
FE-ACT-010-create

BE ID:
BE-001

INPUT METHOD:
POST

RESOLVED METHOD:
POST

BACKEND URL:
/api/equipment/create

BE DOCUMENT:
docs/analysis/{화면명}/backend/FE-ACT-010-create-BE-001.md

ERROR:
없음 또는 오류 요약
```

상세 Backend 분석 결과는
생성된 Markdown 문서에 저장한다.


## 49. 안전 규칙

분석은 Read-Only로 수행한다.

실행 금지:

```text
Oracle DML / DDL
실제 RFC 업무 Function
상태 변경 REST 요청
SOAP 업무 요청
Message Publish
업무 File 전송
실제 저장 / 수정 / 삭제 API
```

분석을 위해 실제 업무 데이터를 변경하지 않는다.

Oracle MCP 역시 Metadata 확인을 위한
Read-Only 용도로만 사용한다.


## 50. STOP

현재 입력된:

```text
FE Base Name
HTTP Method
Backend URL
```

에 대한:

```text
Controller
Business Logic
내부 호출
DB / MyBatis / SQL
필요한 경우 Oracle Metadata
RFC / 외부 연동
Exception
Response
Source Evidence
```

분석과 문서 생성이 완료되면 STOP 한다.

자동으로 다음 Backend URL을 분석하지 않는다.

자동으로 다른 FE Action을 분석하지 않는다.

자동으로 관련 없는 Service / Mapper / Table /
Oracle Object / RFC / 외부 시스템까지 확장하지 않는다.

Worker 실행인 경우에도
현재 Worker에 할당된 Backend 하나가 완료되면 STOP 한다.
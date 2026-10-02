---
name: be-analysis
description: 선택한 Backend API를 기준으로 실제 실행 순서에 따라 Controller, Service, MyBatis/Oracle, RFC 및 기타 외부 연동을 추적하여 Backend Business Logic을 분석한다.
argument-hint: "<화면명> <API ID>"
user-invocable: true
disable-model-invocation: true
---

# Backend Analysis

## 1. 목적

사용자가 선택한 Backend API 하나를 기준으로
실제 Backend Business Logic을 분석한다.

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


## 2. 적용 Rule

이 Skill을 실행할 때 반드시 다음 Rule을 적용한다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

두 Rule의 분석 범위, 호출 추적, Source Evidence,
안전 규칙 및 STOP 조건을 준수한다.


## 3. 입력

기본 입력:

```text
<화면명> <API ID>
```

예:

```text
/be-analysis equipment-search API-001
```

입력된 화면명은 해당 화면의 분석 문서를 찾는 기준으로 사용한다.

API ID는 분석할 Backend API를 선택하는 기준이다.


## 4. API 문서 확인

먼저 다음 위치에서 해당 API ID의 문서를 찾는다.

```text
docs/analysis/{화면명}/api/
```

예:

```text
docs/analysis/equipment-search/api/API-001-equipment-search.md
```

API 문서에서 가능한 경우 다음 정보를 확보한다.

```text
API ID
기능명
HTTP Method
Backend URL
Backend Project
Controller
Controller Method
Controller Source Path
Request DTO
Response DTO
```

API 문서는 Backend 분석의 시작점을 찾기 위한 자료로 사용한다.

API 문서 내용만으로 Backend Business Logic을 확정하지 않는다.


## 5. API 문서를 찾을 수 없는 경우

입력한 API ID에 해당하는 API 문서를 찾을 수 없다면
임의의 API를 선택하지 않는다.

다음과 같이 출력한다.

```text
Backend 분석 중단

화면명:
{화면명}

API ID:
{API ID}

사유:
해당 API 분석 문서를 찾을 수 없음
```

그리고 STOP 한다.


## 6. Project Root 확인

Code Index MCP에 설정된 현재 Project Root를 기준으로 분석한다.

Backend Source 대상은 기본적으로:

```text
gipms-api-*
```

프로젝트이다.

현재 API와 실제 호출 관계가 있는 Backend Project만 분석한다.

모든 `gipms-api-*` 프로젝트를 처음부터 전체 탐색하지 않는다.


## 7. Source Path 기준

Source Path는 모두
Project Root 기준 상대경로로 기록한다.

예:

```text
gipms-api-equipment/src/main/java/...
gipms-api-common/src/main/java/...
```

절대경로를 사용자에게 요청하지 않는다.

절대경로를 추측하거나 문서에 생성하지 않는다.


## 8. 분석 시작점 확인

API 문서의 Controller 정보를 시작점 후보로 사용한다.

하지만 실제 Backend 분석을 시작하기 전에
Controller Source에서 다음을 다시 확인한다.

```text
Controller Class
Controller Mapping
Controller Method
HTTP Method
Backend URL
호출 Service / Method
```

API 문서와 실제 Source가 다르면
차이를 기록하고 실제 Source를 기준으로 Backend 분석을 진행한다.

실제 Source에서 시작점을 확정할 수 없다면
추측하지 않는다.


## 9. Code Index 사용

Code Index MCP는 다음 작업에 우선 활용할 수 있다.

```text
Symbol 검색
Reference 검색
Caller 검색
Callee 검색
Call Relationship 확인
관련 Source 위치 탐색
```

Code Index 결과는 탐색을 위한 정보이다.

Business Logic은 반드시 실제 Source를 읽고 확인한다.

필요한 경우 정확한 Symbol, Method, Statement ID,
namespace, URL 등의 위치 확인을 위해
Grep 또는 파일 검색을 보조적으로 사용할 수 있다.

관련 없는 프로젝트 전체를 무차별적으로 검색하지 않는다.


## 10. Controller 분석

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


## 11. Service / 구현체 추적

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
실제 구현체: 확인되지 않음
```

으로 기록한다.


## 12. Business Logic 분석

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


## 13. 하위 호출 추적

현재 Business Logic에서 다음 호출이 발생하면
현재 API 흐름에 직접 관련된 범위에서 추적한다.

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


## 14. Caller 복귀

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


## 15. Mapper / MyBatis 추적

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


## 16. Dynamic SQL 분석

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


## 17. SQL Parameter Mapping

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


## 18. SQL Result Mapping

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


## 19. Oracle MCP

Oracle MCP는 필요한 DB Metadata 확인에 사용한다.

확인 가능 대상:

```text
Table
View
Column
Data Type
Nullable
Primary Key
Object 존재 여부
```

Oracle MCP는 Read-Only로 사용한다.

다음은 절대 실행하지 않는다.

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

SQL Source와 Oracle Metadata가 다르면
차이를 숨기지 않는다.


## 20. DB 호출 구분

현재 API에서 DB 호출이 여러 번 존재하면
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


## 21. 외부 연동 탐지

Business Logic 중간 어디에서든
다음과 같은 외부 연동이 발견되면 분석한다.

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

외부 연동을 별도의 마지막 단계로 재배치하지 않는다.


## 22. RFC 분석

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

RFC가 공통 Service 또는 Adapter 내부에 존재하면
실제 Source가 Project Root 아래 존재하는 범위에서
호출 지점까지 추적한다.


## 23. REST / HTTP 분석

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


## 24. 기타 외부 연동

SOAP, Message, File 등 다른 외부 연동도
실제 Source에서 확인되는 호출 구조를 기준으로 분석한다.

외부 Library 내부 구현으로 자동 확장하지 않는다.


## 25. 외부 연동 다중 호출

동일 API 안에서 외부 연동이 여러 번 발생하면
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


## 26. 조건 / 분기

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


## 27. 반복 처리

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


## 28. Exception

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


## 29. Transaction

명시적인 Transaction 설정이 확인되면 기록한다.

예:

```text
@Transactional
```

DB 변경과 외부 연동이 섞여 있다면
실제 호출 순서를 그대로 유지한다.

Rollback 여부는
실제 Transaction 설정과 Exception 처리에서
확인되는 범위만 기록한다.


## 30. Response 추적

Backend 분석은 외부 연동이나 SQL에서 끝내지 않는다.

가능한 경우 다음까지 계속 추적한다.

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

현재 API의 최종 Response 생성까지 확인한다.


## 31. Source Evidence

주요 단계에는 가능한 경우 다음을 기록한다.

```text
Source Path
Class
Method
Line Range
```

DB 관련:

```text
Mapper Interface
Mapper Method
Mapper XML
Namespace
Statement ID
Line Range
SQL
Oracle Metadata
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

확인할 수 없는 정보는:

```text
확인되지 않음
```

으로 기록한다.

Line Range를 추측하지 않는다.


## 32. JAR / 외부 Dependency 제한

다음은 자동으로 탐색하지 않는다.

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


## 33. 분석 결과 신뢰 기준

Business Logic은 실제 Source를 기준으로 확정한다.

기본 우선순위:

```text
실제 Source
 ↓
Mapper / MyBatis XML / SQL
 ↓
Oracle Metadata
 ↓
Runtime Evidence
 ↓
Code Index 탐색 결과
```

Code Index 검색 결과나 Method 이름만으로
Business Logic을 확정하지 않는다.


## 34. 전체 실행 Tree 검증

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


## 35. 출력 파일

최종 Backend 분석 문서는 다음 위치에 저장한다.

```text
docs/analysis/{화면명}/backend/
```

파일명:

```text
BE-{API ID}-{기능명}.md
```

예:

```text
docs/analysis/equipment-search/backend/
BE-API-001-equipment-search.md
```

기능명은 기존 API 문서와 실제 분석 대상을 기준으로
일관되게 사용한다.


## 36. 기존 파일 처리

동일한 Backend 분석 문서가 이미 존재한다면
사용자 요청 없이 기존 내용을 임의로 덮어쓰지 않는다.

기존 문서가 존재한다는 사실을 사용자에게 알리고
필요한 처리 방향을 확인한다.


## 37. BE Reference

다음 Reference가 존재하면:

```text
.claude/references/BE-REFERENCE.md
```

최종 문서 작성 시
표현 형식, 구조, 가독성, 상세 수준을 참고한다.

BE-REFERENCE는 Evidence가 아니다.

Reference 안의:

```text
Class
Method
Path
SQL
Table
RFC Function
URL
Parameter
Response
Sample Value
```

등을 실제 분석 결과로 복사하지 않는다.

현재 Source에서 확인된 정보만 사용한다.


## 38. BE Reference가 없는 경우

`BE-REFERENCE.md`가 아직 존재하지 않아도
Backend Source 분석 자체는 수행할 수 있다.

이 경우 실제 Source 분석 결과를 기준으로
임시 구조로 결과를 출력할 수 있다.

Reference가 없다는 이유로
Business Logic 분석을 중단하지 않는다.


## 39. 안전 규칙

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


## 40. STOP

선택한 API의:

```text
Controller
Business Logic
내부 호출
DB / MyBatis / Oracle
RFC / 외부 연동
Exception
Response
Source Evidence
```

분석과 문서 생성이 완료되면 STOP 한다.

자동으로 다음 API를 분석하지 않는다.

자동으로 다른 화면을 분석하지 않는다.

자동으로 관련 없는 Service / Mapper / Table / RFC /
외부 시스템까지 범위를 확장하지 않는다.
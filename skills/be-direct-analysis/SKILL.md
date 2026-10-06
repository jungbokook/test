---
name: be-direct-analysis
description: API 분석 문서 없이 FE Base Name, HTTP Method와 Backend URL을 직접 입력받아 Controller를 찾고, BE Reference와 Backend Rule을 실제 파일에서 로드한 후 기존 Backend 분석 수준으로 분석하여 지정된 OUTPUT_PATH에 저장한다.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# Backend Direct Analysis

## 1. 목적

API 분석 문서 없이 다음 입력으로 Backend를 직접 분석한다.

```text
화면명
FE Base Name
HTTP Method
Backend URL
```

분석 시작점만 기존 `be-analysis`와 다르다.

기존:

```text
API Document
 ↓
Controller
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

Controller가 확정된 이후에는 기존 Backend 분석과 동일하게:

```text
Controller
 ↓
Service / ServiceImpl
 ↓
실제 Business Logic
 ↓
내부 Method / 다른 Service / 공통 Service
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL / Dynamic SQL
 ↓
Caller 복귀
 ↓
RFC / REST / 기타 외부 연동
 ↓
Caller 복귀
 ↓
후속 Business Logic
 ↓
Response
```

를 실제 Source 실행 순서대로 추적한다.

Layer별로 재정렬하지 않는다.


# 2. 필수 실행 계약

이 Skill에서 다음 세 항목은 선택 사항이 아니다.

```text
REFERENCE LOAD
RULE LOAD
OUTPUT PATH
```

분석을 시작하기 전에 반드시 순서대로 확인한다.

```text
1. Rule 파일 실제 Read
2. BE-REFERENCE.md 실제 Read
3. OUTPUT_PATH 확정
4. Backend Source 분석
5. 최종 문서 작성
6. OUTPUT_PATH 저장
7. 저장 결과 검증
```

1~3이 완료되기 전에 Backend 상세 분석을 시작하지 않는다.


# 3. 필수 Rule Load

다음 두 파일을 실제로 읽는다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

파일명을 알고 있다는 이유로
내용을 읽지 않고 진행하지 않는다.

이 Skill 안에 비슷한 규칙이 있다는 이유로
Rule Read를 생략하지 않는다.

Rule 파일이 존재하는데 읽을 수 없다면:

```text
STATUS:
STOP

ERROR:
필수 Backend Rule을 읽을 수 없음
```

으로 종료한다.


# 4. BE Reference 강제 Load

다음 파일을 실제로 읽는다.

```text
.claude/references/BE-REFERENCE.md
```

Backend Source 상세 분석을 시작하기 전에 읽는다.

Reference가 존재하는데 실제 Read 없이
Backend 문서를 작성하면 안 된다.

이 Skill 안에 Reference 구조를 복사해서
대체하지 않는다.

Parent Agent가 Reference를 요약해서 전달했더라도
그 요약으로 실제 Reference Read를 대체하지 않는다.

반드시 현재 Project의:

```text
.claude/references/BE-REFERENCE.md
```

를 직접 읽는다.


## 4.1 Reference Load 상태

Reference를 실제 읽은 후 내부적으로 다음 상태를 확정한다.

```text
REFERENCE PATH:
.claude/references/BE-REFERENCE.md

REFERENCE LOADED:
YES
```

`REFERENCE LOADED = YES`가 되기 전에는
최종 Backend 문서를 작성하지 않는다.


## 4.2 Reference 적용 범위

Reference에서 다음을 적용한다.

```text
문서 구조
Section 배치
표현 형식
상세 수준
ASCII Tree
Business Logic Block
조건 / 분기 Block
DB / SQL Block
Dynamic SQL Block
Parameter Mapping
Result Mapping
Oracle Metadata 표현
RFC / REST / 외부 연동 Block
Exception / Transaction
Response
Source Evidence
미확인 항목
분석 경계
Excel 단일 셀 복사 구조
Block 독립성
```

Reference의 Sample 데이터는 사용하지 않는다.

다음은 Evidence가 아니다.

```text
Sample Class
Sample Method
Sample Path
Sample SQL
Sample Table
Sample Column
Sample RFC
Sample URL
Sample Parameter
Sample Response
```

실제 분석값은 현재 Source에서만 가져온다.


## 4.3 Reference 재로드 금지

현재 Worker의 분석 시작 시 1회 읽는다.

이후 다음 단계에서는 다시 읽지 않는다.

```text
Controller 탐색
Service 분석
Mapper 분석
SQL 분석
Oracle Metadata 확인
RFC 분석
REST 분석
최종 Rendering
파일 저장
저장 검증
```


# 5. 입력

직접 실행:

```text
/be-direct-analysis <화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>
```

예:

```text
/be-direct-analysis equipment-search FE-ACT-010-create UNKNOWN /api/equipment/create
```

필수 값:

```text
화면명
FE Base Name
HTTP Method
Backend URL
```

하나라도 없으면 추측하지 않고 STOP 한다.


# 6. Worker 입력 계약

Parallel Agent의 Worker로 실행되는 경우
다음 값을 전달받는다.

```text
SCREEN_NAME
FE_BASE_NAME
BE_ID
HTTP_METHOD
BACKEND_URL
OUTPUT_PATH
```

예:

```text
SCREEN_NAME:
equipment-search

FE_BASE_NAME:
FE-ACT-010-create

BE_ID:
BE-001

HTTP_METHOD:
UNKNOWN

BACKEND_URL:
/api/equipment/create

OUTPUT_PATH:
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-001.md
```

Worker 실행에서는 이 값들을
**불변값(immutable execution contract)** 으로 취급한다.


# 7. FE Base Name

FE Base Name 예:

```text
FE-ACT-010-create
```

Backend URL 또는 Controller 이름으로
FE Base Name을 새로 생성하지 않는다.

다음과 같이 변경하지 않는다.

```text
FE-ACT-010
ACT-010-create
create
equipment-create
```

전달받은 문자열을 그대로 사용한다.


# 8. BE ID

Worker에서 BE ID가 전달되면 그대로 사용한다.

예:

```text
BE_ID:
BE-001
```

다음을 수행하지 않는다.

```text
BE ID 재계산
기존 최대 BE 번호 검색
완료 순서에 따른 재할당
Controller 발견 순서에 따른 재할당
다른 Worker BE ID 조회
```

Worker가 받은:

```text
BE-001
```

은 분석 종료까지:

```text
BE-001
```

이다.


# 9. OUTPUT_PATH 강제 규칙

Worker 실행에서 가장 중요한 저장 계약이다.

Worker가 다음을 전달받았다고 가정한다.

```text
OUTPUT_PATH:
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-001.md
```

최종 Backend 문서는 반드시 정확히 이 경로에 저장한다.

`OUTPUT_PATH`를 보고 새로운 파일명을 생성하지 않는다.

다음 작업을 금지한다.

```text
파일명 재생성
파일명 축약
기능명 재계산
URL 기반 파일명 생성
Controller 기반 파일명 생성
BE ID 재계산
FE Base Name 변경
다른 backend 폴더 선택
```

즉:

```text
INPUT OUTPUT_PATH
        ↓
그대로 Write
```

이다.


# 10. Worker 파일명 검증

Worker 실행에서는 OUTPUT_PATH의 마지막 파일명이:

```text
{FE_BASE_NAME}-{BE_ID}.md
```

와 정확히 일치해야 한다.

예:

```text
FE_BASE_NAME:
FE-ACT-010-create

BE_ID:
BE-001

Expected Filename:
FE-ACT-010-create-BE-001.md
```

OUTPUT_PATH:

```text
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md
```

이면 PASS.

다음은 FAIL:

```text
BE-001.md
BE-001-create.md
FE-ACT-010-BE-001.md
BE-API-001-create.md
FE-ACT-010-create.md
```

불일치하면 분석을 시작하지 않고 STOP 한다.


# 11. 직접 실행의 OUTPUT_PATH

직접 `/be-direct-analysis`를 실행하여
Parent Agent가 OUTPUT_PATH를 전달하지 않은 경우에만
Skill이 BE ID를 결정한다.

대상 폴더:

```text
docs/analysis/{화면명}/backend/
```

현재 FE Base Name에 해당하는 파일만 확인한다.

예:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
```

가 존재하면:

```text
BE_ID:
BE-003
```

OUTPUT_PATH:

```text
docs/analysis/{화면명}/backend/
FE-ACT-010-create-BE-003.md
```

로 결정한다.

다른 FE Action의 BE 번호는
현재 FE Base Name의 번호 결정에 사용하지 않는다.


# 12. 기존 파일 보호

OUTPUT_PATH가 이미 존재하면
사용자가 명시적으로 overwrite를 요청하지 않은 한
덮어쓰지 않는다.

```text
STATUS:
STOP

EXPECTED OUTPUT:
{OUTPUT_PATH}

ERROR:
Output file already exists
```


# 13. API Document 사용 금지

Direct Backend 분석에서는:

```text
docs/analysis/{화면명}/api/
```

문서를 분석 시작점으로 요구하지 않는다.

API Document를 생성하지 않는다.

API ID를 새로 만들지 않는다.

시작점은 오직:

```text
HTTP Method
Backend URL
```

이다.


# 14. Project Root

Code Index MCP에 설정된
현재 Project Root를 기준으로 분석한다.

Backend Source 기본 대상:

```text
gipms-api-*
```

현재 Backend URL과 연결된 Project만 탐색한다.

모든 `gipms-api-*`를
처음부터 전체 순회하지 않는다.


# 15. Source Path

최종 문서의 Source Path는
Project Root 기준 상대경로를 사용한다.

예:

```text
gipms-api-equipment/src/main/java/...
gipms-api-common/src/main/java/...
```

절대경로를 문서에 기록하지 않는다.


# 16. Code Index 우선

Controller 및 Backend Source 탐색은
Code Index MCP를 우선 사용한다.

사용 목적:

```text
Controller Mapping 후보 탐색
Symbol 검색
Reference 검색
Caller 검색
Callee 검색
관련 Source 위치 확인
```

Code Index 결과는 탐색용이다.

Business Logic 확정은 반드시
실제 Source를 읽은 결과를 기준으로 한다.


# 17. Shell 재귀 탐색 제한

사용하지 않는다.

```text
xargs

Project Root 전체 대상 find

grep -r
grep -R
grep -rn
```

탐색:

```text
Code Index
 ↓
후보 Source
 ↓
실제 Source Read
 ↓
필요 Symbol / Reference
 ↓
관련 Source만 추가 Read
```

Rule 09의 제한을 그대로 적용한다.


# 18. Controller 직접 탐색

입력:

```text
HTTP_METHOD
BACKEND_URL
```

을 이용해 Controller Mapping을 찾는다.

확인:

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Class Mapping + Method Mapping을 조합하여
실제 URL을 확인한다.

Code Index에서 후보를 찾은 후
Controller Source를 반드시 직접 읽는다.


# 19. UNKNOWN Method

HTTP_METHOD가:

```text
UNKNOWN
```

이면 실제 Controller Source에서 Method를 결정한다.

예:

```text
Input:
UNKNOWN /api/equipment/create

Source:
@PostMapping("/create")

Resolved:
POST
```

동일 URL에 여러 Method가 존재하면
하나를 임의 선택하지 않는다.

```text
STATUS:
STOP

ERROR:
동일 Backend URL에 여러 HTTP Method 존재
```


# 20. Controller 탐색 실패

실제 Source에서 Controller를 확정하지 못하면:

```text
STATUS:
STOP

ERROR:
Controller Mapping 확인되지 않음
```

으로 종료한다.

유사한 URL이나 Method를 대신 사용하지 않는다.


# 21. Controller 분석

실제 Controller에서 확인한다.

```text
Request 처리
Validation
Parameter / DTO
조건
Service 호출
Exception
Response 처리
```

여러 호출이 있으면 실제 순서대로 추적한다.


# 22. Service / 구현체

Interface가 있으면 실제 구현체를 찾는다.

```text
Service
 ↓
ServiceImpl
```

구현체를 확정할 Evidence가 없으면:

```text
확인되지 않음
```

으로 기록한다.

임의 선택하지 않는다.


# 23. Business Logic

실제 Method Body 실행 순서대로 분석한다.

```text
입력값 처리
Validation
조건
분기
반복
계산
변환
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
Response
```

Layer 기준으로 재정렬하지 않는다.


# 24. 하위 호출 및 Caller 복귀

하위 호출:

```text
Internal Method
Other Service
Common Service
Interface Service
Adapter
Client
Mapper
```

를 실제 Call Path에 필요한 범위까지 추적한다.

하위 호출이 끝나면 반드시 Caller로 돌아온다.

```text
Service A
 │
 ├─ Common Service
 │    │
 │    ├─ RFC
 │    ├─ Result Mapping
 │    └─ return
 │
 ├─ Service A 복귀
 ├─ RFC 결과 처리
 ├─ DB #2
 └─ Response
```

하위 호출에서 전체 분석을 종료하지 않는다.


# 25. Mapper / MyBatis / SQL

Mapper 호출:

```text
Service
 ↓
Mapper Interface
 ↓
Mapper Method
 ↓
namespace
 ↓
Statement ID
 ↓
MyBatis XML / Annotation SQL
 ↓
Dynamic SQL
 ↓
실제 SQL
```

까지 연결한다.

Method 이름으로 SQL을 추측하지 않는다.

`namespace + Statement ID`를 실제 Source에서 확인한다.


# 26. Dynamic SQL

존재하는 경우 다음 구조를 보존한다.

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

Dynamic SQL을 하나의 고정 SQL로 바꾸지 않는다.


# 27. SQL Parameter / Result Mapping

가능한 범위에서 연결한다.

```text
Request
 ↓
Service
 ↓
Mapper
 ↓
MyBatis Parameter
 ↓
SQL Column
```

Result:

```text
SQL Column
 ↓
MyBatis resultType / resultMap
 ↓
DTO / VO
 ↓
Business Logic
```

Source에서 확인되지 않는 Mapping을
추측하지 않는다.


# 28. DB 호출 번호

DB 호출은 실제 실행 위치마다 구분한다.

```text
DB #1
DB #2
DB #3
...
```

같은 Statement가 반복 호출되어도
실행 위치가 다르면 별도 호출이다.


# 29. 외부 연동

다음을 실제 실행 위치에서 분석한다.

```text
SAP RFC
RFC
REST / HTTP
SOAP
WebService
다른 Backend API
Gateway
Interface Server
Message / Queue
File Interface
기타 외부 시스템
```

마지막 Section으로 이동시키기 위해
실제 실행 Tree 순서를 변경하지 않는다.


# 30. RFC

확인:

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
Error
후속 Business Logic
```


# 31. REST / HTTP

확인:

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
HTTP Status
Error
Timeout
후속 Business Logic
```

확인되지 않은 Configuration 값은 만들지 않는다.


# 32. 조건 / 반복

조건부 호출은 실제 분기를 유지한다.

```text
condition
 ├─ YES
 │   └─ RFC #1
 │
 └─ NO
     └─ RFC 미호출
```

반복 내부 호출도 반복 구조를 유지한다.

Runtime 반복 횟수를 알 수 없으면
숫자를 만들지 않는다.


# 33. Exception / Transaction

실제 Source에서 확인된 Exception 흐름만 기록한다.

Transaction 역시 실제 설정을 확인한다.

```text
@Transactional
```

등이 확인되지 않았다면
Rollback 동작을 추측하지 않는다.


# 34. Response까지 추적

SQL이나 외부 연동에서 분석을 종료하지 않는다.

```text
DB / External Result
 ↓
Caller 복귀
 ↓
후속 Business Logic
 ↓
Data Transformation
 ↓
Response Object / DTO
 ↓
Controller Return
```

까지 추적한다.


# 35. Oracle Metadata

Oracle Metadata는
Backend Call Path의 선행 단계가 아니다.

먼저:

```text
Controller
 ↓
...
 ↓
Response
```

까지 완료한다.

분석 중 SQL에서 사용된:

```text
Table
View
Column
```

을 수집한다.

전체 Call Path 완료 후:

```text
Oracle Object / Column 수집
 ↓
중복 제거
 ↓
Metadata 필요성 판단
 ↓
필요한 항목만 Oracle MCP
 ↓
SQL Source ↔ Metadata 교차 검증
```

한다.

금지:

```text
Schema 전체 Table 조회
Schema 전체 View 조회
Schema 전체 Column 조회
관련 없는 Object 조회
모든 Column 무조건 조회
동일 Object 반복 조회
동일 Column 반복 조회
```

Oracle MCP는 Read-Only로 사용한다.

DML / DDL을 실행하지 않는다.

Metadata를 조회하지 않았다는 이유로
MyBatis SQL에서 확인된 사실을
`확인되지 않음`으로 변경하지 않는다.


# 36. Source Evidence

가능한 경우 기록한다.

```text
Source Path
Class
Method
Line Range
```

DB:

```text
Mapper
Mapper Method
Mapper XML
Namespace
Statement ID
Line Range
SQL
```

외부 연동:

```text
Source Path
Class
Method
Adapter / Client
RFC Function / Endpoint
Line Range
```

Line Range를 확인할 수 없으면:

```text
확인되지 않음
```

으로 기록한다.

추측하지 않는다.


# 37. JAR / 외부 Dependency 경계

자동 탐색하지 않는다.

```text
*.jar
Decompiled Class
.m2
.gradle
Maven Repository
Gradle Cache
Project Root 외부 Source
```

Project Root 아래 실제 Source Project는
현재 Call Path와 연결되는 경우 추적할 수 있다.


# 38. 전체 Call Path 검증

문서 작성 전에:

```text
Controller
 ↓
Service
 ↓
Business Logic
 ↓
하위 호출
 ↓
Caller 복귀
 ↓
DB / External
 ↓
Caller 복귀
 ↓
후속 처리
 ↓
Response
 ↓
Controller Return
```

이 끊기지 않았는지 확인한다.


# 39. Reference 기반 최종 Rendering

최종 문서 Rendering은
분석 시작 시 실제 읽은:

```text
.claude/references/BE-REFERENCE.md
```

의 표현 형식을 따른다.

Skill 내부의 임의 양식으로
Reference를 대체하지 않는다.

Reference에서 요구하는:

```text
전체 실행 Tree
독립 Business Logic Block
DB별 독립 Block
SQL 상세
Dynamic SQL
Parameter Mapping
Result Mapping
필요한 Oracle Metadata
외부 연동별 독립 Block
Exception
Transaction
Response
Source Evidence
미확인 항목
분석 경계
Excel 단일 셀 복사 구조
```

를 현재 Source에 존재하는 범위에서 적용한다.


# 40. Direct 기능 정보

API ID가 없는 Direct 분석에서는
API ID를 임의 생성하지 않는다.

Reference의 기능 정보 Block에서
현재 Direct 분석에 맞는 식별값을 사용한다.

```text
화면
FE Action
BE ID
HTTP Method
Backend URL
Backend Project
```

실제 Source에서 확인되지 않은 기능명은
추측하지 않는다.


# 41. OUTPUT_PATH 저장

Rendering이 끝나면 최종 문서를:

```text
OUTPUT_PATH
```

에 저장한다.

Worker 실행에서는 다른 파일명을 만들지 않는다.

예:

```text
OUTPUT_PATH:

docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md
```

이면 최종 문서는 정확히:

```text
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md
```

에 존재해야 한다.


# 42. 저장 후 검증

Write가 끝났다는 이유로 PASS 하지 않는다.

실제 저장 대상 파일을 다시 확인한다.

검증:

```text
REFERENCE LOADED = YES

EXPECTED OUTPUT PATH
  =
실제 전달받은 OUTPUT_PATH

ACTUAL OUTPUT PATH
  =
실제로 생성된 문서 경로

EXPECTED FILENAME
  =
{FE_BASE_NAME}-{BE_ID}.md

ACTUAL FILENAME
  =
실제 생성 파일명
```

모두 일치해야 한다.


# 43. PASS 조건

다음을 모두 만족해야 PASS이다.

```text
REFERENCE LOADED:
YES

RULE 08 LOADED:
YES

RULE 09 LOADED:
YES

CONTROLLER CONFIRMED:
YES

CALL PATH COMPLETE:
YES

OUTPUT WRITTEN:
YES

OUTPUT PATH MATCH:
YES

OUTPUT FILENAME MATCH:
YES
```

하나라도 NO이면 PASS를 반환하지 않는다.


# 44. Worker 결과

Parent Agent에는 상세 문서를 복사하지 않는다.

다음만 반환한다.

```text
STATUS:
PASS | STOP | ERROR

REFERENCE LOADED:
YES | NO

RULE 08 LOADED:
YES | NO

RULE 09 LOADED:
YES | NO

FE BASE NAME:
...

BE ID:
...

INPUT METHOD:
...

RESOLVED METHOD:
...

BACKEND URL:
...

EXPECTED OUTPUT:
...

ACTUAL OUTPUT:
...

OUTPUT PATH MATCH:
YES | NO

OUTPUT FILENAME MATCH:
YES | NO

ERROR:
없음 또는 오류 내용
```


# 45. 안전 규칙

Read-Only 분석이다.

실행 금지:

```text
Oracle DML / DDL
실제 RFC 업무 Function
상태 변경 REST 요청
SOAP 업무 요청
Message Publish
업무 File 전송
저장 / 수정 / 삭제 API
```


# 46. STOP

현재 Backend 하나의:

```text
Reference Load
Rule Load
Controller
Business Logic
DB / MyBatis / SQL
필요한 Oracle Metadata
RFC / REST / 외부 연동
Exception / Transaction
Response
Source Evidence
Reference 기반 Rendering
OUTPUT_PATH 저장
저장 결과 검증
```

이 완료되면 STOP 한다.

다른 Backend URL을 자동 분석하지 않는다.
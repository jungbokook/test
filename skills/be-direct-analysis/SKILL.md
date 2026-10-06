---
name: be-direct-analysis
description: API 분석 문서 없이 HTTP Method와 Backend URL을 직접 입력받아 Controller를 찾고 실제 실행 순서에 따라 Backend Business Logic, MyBatis/SQL, 필요한 Oracle Metadata, RFC 및 기타 외부 연동을 분석한다.
argument-hint: "<화면명> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# Backend Direct Analysis

## 1. 목적

API 분석 문서를 먼저 생성하지 않고
사용자가 입력한 Backend URL을 직접 시작점으로 하여
Backend Business Logic을 분석한다.

기본 실행:

```text
/be-direct-analysis <화면명> <HTTP Method|UNKNOWN> <Backend URL>
```

예:

```text
/be-direct-analysis equipment-search POST /api/equipment/search
```

HTTP Method를 모르는 경우:

```text
/be-direct-analysis equipment-search UNKNOWN /api/equipment/search
```

분석 흐름:

```text
HTTP Method + Backend URL
 ↓
Controller Mapping 탐색
 ↓
Controller Method 확정
 ↓
Service / ServiceImpl
 ↓
Business Logic
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL
 ↓
Caller 복귀
 ↓
RFC / REST / 기타 외부 연동
 ↓
후속 Business Logic
 ↓
Response
 ↓
필요한 경우 Oracle Metadata 검증
 ↓
BE 문서 생성
 ↓
STOP
```

API Contract 문서를 생성하지 않는다.

---

# 2. 적용 Rule

이 Skill을 실행할 때 반드시 다음 Rule을 적용한다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

최종 문서 작성 시 다음 Reference가 존재하면 사용한다.

```text
.claude/references/BE-REFERENCE.md
```

Reference는 표현 형식과 상세 수준을 위한 자료이며
Evidence가 아니다.

---

# 3. 입력

입력:

```text
<화면명> <HTTP Method|UNKNOWN> <Backend URL>
```

예:

```text
/be-direct-analysis equipment-search POST /api/equipment/search
```

또는:

```text
/be-direct-analysis equipment-search UNKNOWN /api/equipment/search
```

입력 의미:

```text
화면명
  BE 문서 저장 위치를 결정한다.

HTTP Method
  GET
  POST
  PUT
  PATCH
  DELETE
  UNKNOWN

Backend URL
  분석을 시작할 Backend Endpoint
```

화면명은 Source 탐색 조건으로 사용하지 않는다.

화면명은 기본적으로:

```text
docs/analysis/{화면명}/
```

아래에 결과를 저장하기 위한 식별자로 사용한다.

---

# 4. API 문서 사용 금지

이 Skill은 API 분석 문서를 필요로 하지 않는다.

다음 문서를 분석 시작점으로 요구하지 않는다.

```text
docs/analysis/{화면명}/api/*.md
```

기존 API 문서가 존재하더라도
Controller를 결정하기 위한 필수 입력으로 사용하지 않는다.

분석 시작점은 항상:

```text
HTTP Method
+
Backend URL
```

이다.

API 문서가 없다는 이유로 STOP 하지 않는다.

API 문서를 새로 생성하지 않는다.

---

# 5. Project Root

Code Index MCP에 설정된 현재 Project Root를 기준으로 분석한다.

Backend 프로젝트 우선 범위:

```text
gipms-api-*
```

Backend URL과 Controller Mapping을 기준으로
관련 Backend Project를 찾는다.

모든 `gipms-api-*` 프로젝트를
처음부터 Deep Read 하지 않는다.

기본 탐색 순서:

```text
Backend URL
 ↓
Code Index
 ↓
Controller Mapping 후보
 ↓
관련 Backend Project
 ↓
Controller Source
```

---

# 6. Controller 탐색

입력된 Backend URL을 기준으로
Controller Mapping을 찾는다.

확인 대상:

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
기타 실제 Spring Mapping
```

Class Level Mapping과
Method Level Mapping을 함께 확인한다.

예:

```java
@RequestMapping("/api/equipment")
```

+

```java
@PostMapping("/search")
```

이면 실제 Endpoint 후보:

```text
POST /api/equipment/search
```

이다.

URL 문자열 하나만 발견했다고
Controller를 확정하지 않는다.

실제 Controller Source에서 Mapping을 확인한다.

---

# 7. HTTP Method 확인

입력 HTTP Method가:

```text
GET
POST
PUT
PATCH
DELETE
```

중 하나라면 URL과 Method를 함께 사용하여
Controller Method를 식별한다.

예:

```text
입력

POST
/api/equipment/search
```

Controller:

```text
GET  /api/equipment/search
POST /api/equipment/search
```

가 모두 존재한다면:

```text
POST
```

Mapping을 선택한다.

---

# 8. HTTP Method UNKNOWN

입력이:

```text
UNKNOWN
```

이면 Controller Source에서 HTTP Method를 확인한다.

예:

```text
/be-direct-analysis equipment-search UNKNOWN /api/equipment/search
```

탐색 결과:

```text
@PostMapping("/search")
```

만 존재한다면:

```text
HTTP Method
  POST
```

로 확정하고 분석을 계속한다.

---

# 9. 동일 URL 다중 Method

동일 Backend URL에 여러 HTTP Method가 존재할 수 있다.

예:

```text
GET  /api/equipment
POST /api/equipment
DELETE /api/equipment
```

사용자 입력이:

```text
UNKNOWN /api/equipment
```

이고 Source만으로 하나를 선택할 근거가 없다면
임의로 하나를 선택하지 않는다.

다음과 같이 출력한다.

```text
Backend Direct Analysis 중단

Backend URL
  /api/equipment

HTTP Method
  UNKNOWN

확인된 Mapping 후보
  GET /api/equipment
  POST /api/equipment
  DELETE /api/equipment

사유
  동일 URL에 여러 HTTP Method가 존재하여
  분석 대상을 하나로 확정할 수 없음
```

그리고 STOP 한다.

사용자가 Method를 선택한 후 다시 실행한다.

---

# 10. Controller 후보가 여러 개인 경우

동일 Method + URL에 여러 Controller 후보가 발견되면
다음 Evidence를 추가 확인한다.

```text
Class Level Mapping
Method Level Mapping
Spring Annotation
Project
Package
Controller Source
실제 Mapping 조합
```

하나의 Controller를 확정할 수 있다면 계속 진행한다.

확정할 수 없다면
후보를 사용자에게 보여주고 STOP 한다.

추측해서 하나를 선택하지 않는다.

---

# 11. Controller를 찾을 수 없는 경우

입력 URL에 대응하는 Controller를
현재 Project Source에서 찾을 수 없다면
다음과 같이 출력한다.

```text
Backend Direct Analysis 중단

HTTP Method
  {입력 Method}

Backend URL
  {입력 URL}

Controller
  확인되지 않음

사유
  현재 Project Source에서
  해당 Backend Mapping을 확인할 수 없음
```

그리고 STOP 한다.

유사한 URL의 Controller를 임의로 선택하지 않는다.

---

# 12. Controller 확정

Controller가 확인되면 다음 정보를 기록한다.

```text
HTTP Method
Backend URL
Backend Project
Controller Class
Controller Method
Controller Source Path
Line Range
```

예:

```text
HTTP Method
  POST

Backend URL
  /api/equipment/search

Backend Project
  gipms-api-equipment

Controller
  EquipmentController

Method
  search()

Source
  gipms-api-equipment/src/main/java/.../
  EquipmentController.java

Line Range
  40-65
```

Line Range를 확인할 수 없으면:

```text
Line Range
  확인되지 않음
```

으로 기록한다.

---

# 13. Controller부터 BE 분석 시작

Controller가 확정되면
기존 Backend 분석 규칙에 따라 실제 Source를 추적한다.

```text
Controller
 ↓
Controller Method
 ↓
Service
 ↓
ServiceImpl
 ↓
Business Logic
```

Controller에서 여러 Service / Method를 호출하면
실제 실행 순서를 유지한다.

대표 Service 하나만 선택하지 않는다.

---

# 14. Service 구현체 확인

Service가 Interface인 경우
실제 구현체를 확인한다.

예:

```text
EquipmentService
 ↓
EquipmentServiceImpl
```

확인 기준:

```text
implements
Spring Bean
Injection
Qualifier
Reference
실제 호출 Source
```

구현체가 여러 개이고
실제 구현체를 확정할 수 없다면:

```text
실제 구현체
  확인되지 않음
```

으로 기록한다.

임의로 구현체를 선택하지 않는다.

---

# 15. Business Logic 분석

Service / ServiceImpl에서는
실제 Method Body 실행 순서대로 분석한다.

확인 대상:

```text
입력값 처리
Validation
조건
분기
반복
데이터 변환
계산
상태 변경
내부 Method
다른 Service
공통 Service
Mapper
DB
RFC
REST
SOAP
Message
File
Exception
Transaction
Response
```

Layer별로 다시 정렬하지 않는다.

---

# 16. 실제 실행 순서 유지

예를 들어 Source가:

```text
Service
 ↓
Validation
 ↓
DB SELECT
 ↓
조건
 ↓
RFC
 ↓
RFC 결과 처리
 ↓
DB UPDATE
 ↓
REST
 ↓
DB INSERT
 ↓
Response
```

순서라면 최종 문서도 동일한 순서를 유지한다.

다음처럼 재배열하지 않는다.

```text
모든 DB
 ↓
모든 RFC
 ↓
모든 REST
```

---

# 17. 하위 호출 후 Caller 복귀

하위 Method 또는 Service를 분석한 뒤
반드시 Caller의 다음 실행 위치로 복귀한다.

예:

```text
Service A
 │
 ├─ Service B
 │    │
 │    ├─ DB #1
 │    └─ return
 │
 ├─ Service B 결과 처리
 │
 ├─ RFC #1
 │
 ├─ RFC 결과 처리
 │
 └─ Response
```

하위 호출을 분석했다고
현재 API 분석을 종료하지 않는다.

---

# 18. Mapper / MyBatis 분석

Mapper 호출이 발견되면:

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
Mapper XML 또는 Annotation
 ↓
Dynamic SQL
 ↓
실제 SQL
```

까지 연결한다.

Statement는 반드시:

```text
namespace + Statement ID
```

를 기준으로 식별한다.

Method명이나 Statement ID만으로
SQL을 추측하지 않는다.

---

# 19. SQL 분석

실제 SQL에서 가능한 경우 다음을 확인한다.

```text
SQL Type
Table / View
Column
JOIN
WHERE
Subquery
Dynamic SQL
Parameter Mapping
Result Mapping
INSERT 값
UPDATE 값
DELETE 조건
MERGE 조건
```

Method 이름보다 실제 SQL을 우선한다.

예:

```text
deleteEquipment()
```

가 호출되더라도 SQL이:

```sql
UPDATE TB_EQUIPMENT
SET STATUS = 'D'
```

라면 실제 DB 동작은:

```text
UPDATE 기반 상태 변경
```

으로 분석한다.

---

# 20. Oracle Metadata 최적화

Oracle Metadata는
DB 호출을 발견할 때마다 조회하지 않는다.

우선 전체 Backend Call Path를 추적한다.

```text
Source
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL
 ↓
사용 Object / Column 기록
 ↓
Caller 복귀
 ↓
다음 Business Logic
```

Controller부터 Response까지 추적이 끝나면:

```text
사용 Oracle Object / Column 수집
 ↓
중복 제거
 ↓
Metadata 필요성 판단
 ↓
필요한 Metadata만 Oracle MCP 확인
```

Metadata 확인 대상 예:

```text
Object 존재 여부
Table / View 구분
Column 존재 여부
Data Type
Nullable
Primary Key
SQL ↔ DB 구조 불일치
Parameter / Result Mapping 검증
```

다음은 수행하지 않는다.

```text
Schema 전체 Metadata 조회
관련 없는 Table 조회
관련 없는 View 조회
모든 Column 무조건 조회
동일 Object 반복 조회
동일 Column 반복 조회
```

Metadata가 필요하지 않으면
Oracle MCP 호출 없이 BE 분석을 완료할 수 있다.

---

# 21. Oracle 안전 규칙

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
기타 DML
기타 DDL
```

Source에서 DML SQL이 발견되더라도
실제 DB에는 실행하지 않는다.

SQL Source를 읽고 동작만 분석한다.

---

# 22. RFC / 외부 연동

Business Logic 중 발견되는 외부 연동은
실제 위치에서 분석한다.

예:

```text
SAP RFC
REST / HTTP
SOAP
다른 Backend API
Gateway
Message
File
기타 외부 시스템
```

RFC가 중간에 있다면:

```text
DB #1
 ↓
RFC #1
 ↓
DB #2
```

순서를 유지한다.

RFC를 문서 마지막으로 이동시키지 않는다.

---

# 23. 다중 외부 연동

외부 연동이 여러 번 발생하면
각 호출을 구분한다.

예:

```text
RFC #1
REST #1
RFC #2
REST #2
```

같은 RFC Function이나 Endpoint가
반복 호출되더라도 실행 위치가 다르면 구분한다.

---

# 24. 조건 / 반복 / Exception

실제 Source의:

```text
if
else
switch
반복문
try
catch
throw
```

구조를 Business Logic에 영향을 주는 범위에서 보존한다.

예:

```text
sapUseYn == "Y"
 │
 ├─ YES
 │   ↓
 │  RFC #1
 │
 └─ NO
     ↓
    RFC 미호출
```

실행되지 않는 Branch라고 해서
Source에 존재하는 Business Logic을 삭제하지 않는다.

---

# 25. Transaction

실제 Source에서:

```text
@Transactional
```

또는 명시적인 Transaction 설정이 확인되면 기록한다.

Rollback 범위는
실제 Source에서 확인되는 범위만 설명한다.

일반적인 Spring 동작만으로
확인되지 않은 Rollback을 단정하지 않는다.

---

# 26. Response까지 추적

분석은 DB 또는 외부 연동에서 끝나지 않는다.

가능한 경우:

```text
DB / RFC / REST 결과
 ↓
Business Logic
 ↓
데이터 변환
 ↓
Response Object
 ↓
Controller Return
```

까지 추적한다.

최종 Response 생성이
현재 API 분석의 끝이다.

---

# 27. Source Evidence

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

Oracle:

```text
실제 Metadata를 확인한 경우에만
Object
Column
Data Type
Nullable
PK
```

Source Path는 반드시
Project Root 기준 상대경로를 사용한다.

---

# 28. Shell 기반 전체 검색 제한

Backend Source 탐색에서 `xargs`를 사용하지 않는다.

Project Root 전체 대상의 다음 검색을 사용하지 않는다.

```text
find ... | xargs ...
grep ... | xargs ...
grep -r ...
grep -R ...
grep -rn ...
```

탐색 순서:

```text
Code Index
 ↓
관련 Project 확인
 ↓
Symbol / Reference 확인
 ↓
관련 Source 범위 축소
 ↓
필요 Source Read
 ↓
실제 Source 확인
```

정확한 문자열 검색이 필요한 경우에도
관련 Source 범위를 먼저 좁힌 후 보조적으로 사용한다.

---

# 29. JAR / 외부 Dependency 제한

다음으로 자동 확장하지 않는다.

```text
*.jar
JAR 내부
Decompiled Source
.m2/
.gradle/
Maven Repository
Gradle Cache
Project Root 외부 Source
```

Project Root 아래 실제 Source Project로 존재하는
공통 모듈은 현재 Call Path와 연결되는 경우 분석할 수 있다.

---

# 30. BE ID

Direct Analysis에서는
API ID를 전제로 하지 않는다.

BE ID는 현재 화면의 기존 Direct BE 문서를 기준으로
사용 가능한 다음 번호를 결정한다.

형식:

```text
BE-001
BE-002
BE-003
...
```

단, Orchestrator 또는 호출자가
BE ID를 명시적으로 전달한 경우
전달받은 BE ID를 그대로 사용한다.

전달받은 BE ID를 재배정하지 않는다.

---

# 31. 출력 파일명

기본 출력 파일명:

```text
BE-{nnn}-{기능명}.md
```

예:

```text
BE-001-equipment-search.md
```

저장 위치:

```text
docs/analysis/{화면명}/backend/
```

예:

```text
docs/analysis/equipment-search/backend/
BE-001-equipment-search.md
```

기능명은 실제 Controller / Backend 기능을 기준으로
간결하게 결정한다.

확인되지 않은 업무 의미를
파일명에 추측해서 넣지 않는다.

---

# 32. Output File 지정

Orchestrator 또는 호출자가
Output File을 명시적으로 전달한 경우
전달받은 파일명을 그대로 사용한다.

예:

```text
Output File:
FE-ACT-010-create-BE-001.md
```

저장:

```text
docs/analysis/{화면명}/backend/
FE-ACT-010-create-BE-001.md
```

이 경우 Skill이 파일명을 다시 생성하지 않는다.

---

# 33. 기존 파일 보호

저장 대상 파일이 이미 존재하면
사용자 요청 없이 자동 덮어쓰기하지 않는다.

기존 파일 존재를 알리고 STOP 한다.

---

# 34. BE Reference 적용

다음 파일이 존재하면:

```text
.claude/references/BE-REFERENCE.md
```

최종 문서의:

```text
구조
표현 방식
상세 수준
ASCII Execution Tree
Excel 단일 셀 복사 구조
Oracle Metadata 표현
```

에 적용한다.

Reference 내용은 Evidence가 아니다.

Reference의 Sample Class / SQL / Table / RFC / URL 등을
실제 결과에 복사하지 않는다.

---

# 35. 최종 검증

문서 저장 전에 다음을 확인한다.

```text
[ ] API 문서 없이 Backend URL에서 직접 시작했는가

[ ] HTTP Method + URL을 실제 Controller Mapping과 확인했는가

[ ] UNKNOWN인 경우 Source에서 Method를 확정했는가

[ ] 동일 URL 다중 Method가 모호하면 STOP 했는가

[ ] Controller를 실제 Source에서 확인했는가

[ ] Controller부터 Response까지 추적했는가

[ ] Service / ServiceImpl을 실제 Source로 확인했는가

[ ] 내부 Method / 다른 Service 호출을 필요한 범위까지 추적했는가

[ ] 하위 호출 후 Caller로 복귀했는가

[ ] Mapper → namespace + Statement ID → MyBatis를 연결했는가

[ ] 실제 SQL을 확인했는가

[ ] Dynamic SQL을 보존했는가

[ ] Parameter Mapping을 확인했는가

[ ] Result Mapping을 가능한 범위에서 확인했는가

[ ] DB마다 Oracle Metadata를 즉시 조회하지 않았는가

[ ] 전체 Call Path를 먼저 완료했는가

[ ] Oracle Object / Column을 수집하고 중복 제거했는가

[ ] 필요한 Metadata만 Oracle MCP로 확인했는가

[ ] 동일 Metadata를 반복 조회하지 않았는가

[ ] Schema 전체 Metadata를 탐색하지 않았는가

[ ] RFC / REST / 기타 외부 연동을 실제 위치에 배치했는가

[ ] 외부 연동 후 Caller로 복귀했는가

[ ] 조건 / 분기 / 반복을 보존했는가

[ ] Exception을 실제 Source 기준으로 확인했는가

[ ] Transaction을 추측하지 않았는가

[ ] 최종 Response까지 추적했는가

[ ] Source Path가 Project Root 기준 상대경로인가

[ ] Line Range를 추측하지 않았는가

[ ] JAR / 외부 Dependency 내부로 확장하지 않았는가

[ ] xargs / Project Root 전체 재귀 Shell 검색을 사용하지 않았는가

[ ] 기존 파일을 임의로 덮어쓰지 않았는가
```

---

# 36. 최종 결과

성공 시:

```text
Backend Direct Analysis 완료

STATUS:
PASS

화면명:
{화면명}

HTTP METHOD:
{확정 Method}

BACKEND URL:
{Backend URL}

CONTROLLER:
{Controller}

BE ID:
{BE ID}

BE DOCUMENT:
docs/analysis/{화면명}/backend/{파일명}
```

상세 Backend 분석 내용은
생성된 Markdown 문서에 저장한다.

터미널/Claude 응답에 전체 BE 문서를
다시 출력하지 않는다.

---

# 37. STOP

현재 Backend URL의:

```text
Controller
Service / ServiceImpl
Business Logic
Internal Method
Other Service
Mapper
MyBatis
SQL
필요한 Oracle Metadata
RFC / REST / 기타 외부 연동
Exception
Transaction
Response
Source Evidence
```

분석과 문서 생성이 완료되면 STOP 한다.

API 문서를 생성하지 않는다.

다음 Backend URL을 자동 분석하지 않는다.

다른 API를 자동 분석하지 않는다.

다른 화면으로 확장하지 않는다.

현재 Backend URL의 실제 Call Path와 연결된 범위까지만 분석한다.
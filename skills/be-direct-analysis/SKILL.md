---
name: be-direct-analysis
description: API 분석 문서 없이 HTTP Method와 Backend URL을 직접 입력받아 Controller를 찾고 실제 실행 순서에 따라 Backend Business Logic, MyBatis/SQL, 필요한 Oracle Metadata, RFC 및 기타 외부 연동을 분석한다.
argument-hint: "<화면명> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# Backend Direct Analysis

## 1. 목적

API 분석 문서 없이 Backend URL을 직접 입력받아
해당 Backend API의 실제 실행 흐름을 분석한다.

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
 │  내부 Service
 │   ↓
 │  RFC 호출
 │   ↓
 │  DB #2
 │
 └─ NO
     ↓
    예외 처리
 ↓
Response 생성
 ↓
Controller Response
```

다음과 같은 단순 Layer 나열은 금지한다.

```text
Controller
 ↓
Service
 ↓
Mapper
 ↓
DB
```

실제 Business Logic의 실행 순서와
조건 분기, 반복 호출, DB/RFC/REST 호출 위치를 추적해야 한다.

---

# 2. 호출 방식

사용자 직접 실행:

```text
/be-direct-analysis <화면명> <HTTP Method|UNKNOWN> <Backend URL>
```

예:

```text
/be-direct-analysis equipment POST /api/equipment/create
```

또는:

```text
/be-direct-analysis equipment UNKNOWN /api/equipment/create
```

---

# 3. Parallel Agent 호출 지원

이 Skill은 단독 실행뿐 아니라
`be-direct-parallel` Agent의 Worker에서도 실행할 수 있다.

Agent에서 호출할 경우 다음 값을 추가로 전달받을 수 있다.

```text
Screen Name
FE Base Name
BE ID
HTTP Method
Backend URL
Output File
```

예:

```text
Screen Name: equipment
FE Base Name: FE-ACT-010-create
BE ID: BE-001
HTTP Method: POST
Backend URL: /api/equipment/create
Output File: docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
```

Agent가 다음 값을 명시적으로 전달한 경우:

```text
BE ID
Output File
```

Skill 내부에서 새 번호를 생성하거나
파일명을 다시 계산하지 않는다.

전달받은 값을 그대로 사용한다.

---

# 4. 필수 규칙

반드시 다음 규칙을 따른다.

1. 실제 Source를 최종 Evidence로 사용한다.
2. Code Index MCP는 Source 탐색에 사용한다.
3. 추측하지 않는다.
4. 확인할 수 없는 내용은 `확인되지 않음`으로 기록한다.
5. Layer 순서가 아닌 실제 실행 순서를 분석한다.
6. Controller에서 시작하여 Response까지 추적한다.
7. Service 내부 메서드 호출도 실제 순서대로 추적한다.
8. DB 호출이 여러 번 발생하면 모두 기록한다.
9. RFC 호출이 여러 번 발생하면 모두 기록한다.
10. REST/외부 시스템 호출이 여러 번 발생하면 모두 기록한다.
11. DB → RFC → DB와 같은 교차 흐름을 그대로 표현한다.
12. 조건 분기를 생략하지 않는다.
13. 예외 흐름을 생략하지 않는다.
14. API 분석 문서를 생성하지 않는다.
15. FE 분석 문서를 생성하지 않는다.
16. Backend 분석 문서만 생성한다.

---

# 5. 분석 범위

분석 시작점:

```text
HTTP Method
+
Backend URL
```

분석 종료점:

```text
Controller Response
```

전체 범위:

```text
Backend URL
 ↓
Controller Mapping
 ↓
Controller Method
 ↓
Service
 ↓
Business Logic
 ↓
Internal Service
 ↓
Mapper
 ↓
MyBatis XML
 ↓
SQL
 ↓
Oracle
 ↓
RFC
 ↓
REST
 ↓
기타 외부 연동
 ↓
Response 생성
 ↓
Controller Response
```

위 Layer가 항상 모두 존재하는 것은 아니다.

실제 Source에 존재하는 흐름만 기록한다.

---

# 6. 분석 금지 사항

다음은 금지한다.

```text
Source 확인 없이 흐름 추정

메서드 이름만 보고 Business Logic 추정

URL 이름만 보고 기능 추정

DTO 이름만 보고 데이터 의미 추정

SQL 확인 없이 DB 동작 추정

RFC 이름만 보고 SAP 동작 추정

REST URL만 보고 외부 시스템 동작 추정
```

확인되지 않은 경우:

```text
확인되지 않음
```

으로 기록한다.

---

# 7. 프로젝트 탐색 원칙

프로젝트 전체를 무차별 탐색하지 않는다.

우선순위:

```text
1. Code Index MCP
2. Backend URL 검색
3. Controller Mapping 검색
4. Controller Method 확인
5. 호출 Symbol 추적
6. 실제 Source 확인
```

Backend 프로젝트 우선순위:

```text
gipms-api-*
```

필요한 프로젝트만 탐색한다.

---

# 8. Shell 기반 재귀 탐색 제한

프로젝트 Root 전체를 대상으로 하는
광범위 Shell 재귀 탐색을 금지한다.

금지 예:

```bash
find . -type f
find . | xargs grep ...
grep -r "keyword" .
grep -R "keyword" .
grep -rn "keyword" .
```

특히 다음은 사용하지 않는다.

```text
xargs
```

탐색 순서:

```text
Code Index
 ↓
후보 Source 특정
 ↓
필요한 파일만 Read
 ↓
실제 Source 확인
```

---

# 9. 제외 경로

불필요한 영역은 분석 대상에서 제외한다.

예:

```text
docs/**
sample/**
samples/**
target/**
build/**
dist/**
node_modules/**
*.jar
```

JAR 내부 탐색을 시도하지 않는다.

---

# 10. HTTP Method 처리

입력 Method가 명확한 경우:

```text
GET
POST
PUT
PATCH
DELETE
```

해당 Method와 URL을 함께 사용하여
Controller Mapping을 찾는다.

---

# 11. Method UNKNOWN 처리

입력:

```text
UNKNOWN /api/equipment/create
```

인 경우 실제 Controller Source에서 Method를 확인한다.

확인 대상:

```java
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@RequestMapping(method = ...)
```

Method를 확인하면 문서에 기록한다.

예:

```text
Input Method : UNKNOWN
Resolved Method : POST
```

---

# 12. 동일 URL 다중 Method

동일 URL에 여러 HTTP Method가 존재하고
입력 Method가 UNKNOWN인 경우 임의 선택하지 않는다.

예:

```text
GET  /api/equipment
POST /api/equipment
```

이 경우 분석을 중단하고 사용자에게 선택을 요청한다.

```text
STOP

동일 Backend URL에 여러 HTTP Method가 존재합니다.

GET  /api/equipment
POST /api/equipment

분석할 Method를 지정해 주세요.
```

---

# 13. Controller 탐색

다음을 모두 고려한다.

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Class Level Mapping과
Method Level Mapping을 조합한다.

예:

```java
@RequestMapping("/api/equipment")
```

+

```java
@PostMapping("/create")
```

=

```text
POST /api/equipment/create
```

---

# 14. Controller 분석

Controller에서 확인한다.

```text
HTTP Method
URL
Controller Class
Controller Method
Request Parameter
Path Variable
Request Body
Request DTO
Validation
Service 호출
Response 생성
Exception 처리
```

Evidence:

```text
Project Root 기준 상대경로
파일명
Line Range
```

절대경로는 기록하지 않는다.

---

# 15. Service 분석

Controller에서 호출되는 Service부터 추적한다.

분석 대상:

```text
Service
ServiceImpl
내부 Method
다른 Service
Helper
Validator
Utility
Domain Logic
```

중요:

```text
Service → Mapper
```

만 기록하지 않는다.

실제 실행 순서를 추적한다.

예:

```text
Service.create()
 ↓
validate()
 ↓
checkDuplicate()
 ↓
Mapper.selectCount()
 ↓
if duplicate
 ├─ YES → Exception
 └─ NO
      ↓
     makeParameter()
      ↓
     Mapper.insert()
      ↓
     callSap()
      ↓
     Mapper.updateStatus()
```

---

# 16. 조건 분기

조건은 반드시 기록한다.

예:

```text
status == "A"
```

문서:

```text
조건: status == "A"

├─ YES
│   ↓
│  RFC 호출
│
└─ NO
    ↓
   RFC 호출 생략
```

조건에 사용되는 값의 출처도 가능한 경우 기록한다.

예:

```text
Source State:
request.status
```

또는:

```text
Source State:
DB #1 조회 결과.status
```

---

# 17. 내부 Backend 왕복

Backend 내부에서 여러 Service/Component를
오가는 경우 모두 추적한다.

예:

```text
Controller
 ↓
Service A
 ↓
Service B
 ↓
Mapper #1
 ↓
Service A
 ↓
RFC
 ↓
Service C
 ↓
Mapper #2
 ↓
Service A
 ↓
Response
```

이를 단순히:

```text
Controller → Service → DB → Response
```

로 축약하지 않는다.

---

# 18. MyBatis 분석

Mapper 호출을 발견하면
실제 MyBatis XML까지 추적한다.

확인 대상:

```text
Mapper Interface
Mapper Method
Namespace
Statement ID
MyBatis XML
SQL
Parameter
Result Mapping
```

예:

```text
EquipmentMapper.selectEquipment
 ↓
namespace
 ↓
selectEquipment
 ↓
SELECT ...
```

---

# 19. SQL 분석

SQL은 가능한 경우 실제 SQL 기준으로 기록한다.

확인 대상:

```text
SELECT
INSERT
UPDATE
DELETE
MERGE
Procedure
Function
Sequence
Table
View
Column
Join
WHERE 조건
ORDER BY
동적 SQL
```

MyBatis 동적 SQL:

```xml
<if>
<choose>
<when>
<otherwise>
<foreach>
```

도 실제 조건 흐름에 반영한다.

---

# 20. DB 호출 번호

DB 호출은 실제 실행 순서 기준으로 번호를 부여한다.

예:

```text
DB #1
DB #2
DB #3
```

예:

```text
Service
 ↓
DB #1 - 중복 확인
 ↓
조건
 ↓
DB #2 - 데이터 INSERT
 ↓
RFC #1
 ↓
DB #3 - 결과 상태 UPDATE
```

---

# 21. RFC 분석

RFC 호출을 발견하면 실제 Source 기준으로 분석한다.

확인 대상:

```text
호출 위치
호출 조건
RFC Function
Input Mapping
Output Mapping
Return Code
Return Message
Exception
후속 처리
```

SAP 내부 동작은 Source 또는 연결된 Evidence에서
확인할 수 없는 경우 추정하지 않는다.

```text
SAP 내부 처리: 확인되지 않음
```

---

# 22. RFC 호출 번호

RFC 호출은 실행 순서 기준으로 번호를 부여한다.

```text
RFC #1
RFC #2
RFC #3
```

DB와 RFC 번호는 서로 독립적이다.

예:

```text
DB #1
 ↓
RFC #1
 ↓
DB #2
 ↓
RFC #2
 ↓
DB #3
```

---

# 23. REST / 외부 연동

다음 호출도 추적한다.

```text
RestTemplate
WebClient
HTTP Client
Feign
외부 SDK
사내 공통 Client
기타 External Interface
```

확인 대상:

```text
호출 위치
호출 조건
Method
URL
Request Mapping
Response Mapping
Timeout
Exception
후속 처리
```

확인 가능한 범위까지만 기록한다.

---

# 24. 외부 호출 번호

필요한 경우 호출 유형별 번호를 사용한다.

예:

```text
REST #1
REST #2

RFC #1
RFC #2

DB #1
DB #2
```

실행 트리에서는 실제 순서대로 섞어서 표현한다.

---

# 25. Oracle Metadata 조회 원칙

Oracle MCP는 Source 분석을 대체하지 않는다.

Oracle Metadata는
MyBatis/SQL 분석 후 필요한 대상만 확인한다.

순서:

```text
Controller
 ↓
Service
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL 분석 완료
 ↓
Oracle 대상 수집
 ↓
중복 제거
 ↓
필요한 Metadata만 조회
```

---

# 26. Oracle Metadata 조회 대상

SQL에서 실제 사용된 대상만 수집한다.

예:

```text
TABLE
VIEW
COLUMN
PROCEDURE
FUNCTION
SEQUENCE
```

예:

```text
TB_EQUIPMENT
TB_EQUIPMENT_STATUS
FN_GET_STATUS
SEQ_EQUIPMENT
```

---

# 27. Oracle Metadata 중복 제거

같은 Worker 내부에서 동일 Object를
반복 조회하지 않는다.

예:

```text
DB #1 → TB_EQUIPMENT
DB #2 → TB_EQUIPMENT
DB #3 → TB_EQUIPMENT
```

Oracle Metadata 조회:

```text
TB_EQUIPMENT → 1회
```

DB 호출마다 Metadata를 다시 조회하지 않는다.

---

# 28. Oracle Metadata 금지 사항

금지:

```text
Schema 전체 조회
전체 Table 탐색
전체 Column 탐색
DB 호출마다 Metadata 조회
같은 Object 반복 조회
Source 분석 전에 Oracle 탐색
```

Oracle MCP가 느리거나 조회되지 않는 경우:

```text
Oracle Metadata: 확인되지 않음
```

으로 기록하고 분석을 계속한다.

핵심 실행 흐름 자체를 확인할 수 없는 경우에만
분석 불가 상태를 명확하게 기록한다.

---

# 29. 예외 처리

다음을 확인한다.

```text
try/catch
throw
Custom Exception
Validation Exception
DB Exception
RFC Exception
REST Exception
Global Exception Handler
Error Response
```

실제 실행 흐름에서
어디에서 예외가 발생하고
어디에서 처리되는지 연결한다.

예:

```text
RFC #1
 ↓
Exception
 ↓
catch
 ↓
DB #3 상태 UPDATE
 ↓
throw BusinessException
 ↓
GlobalExceptionHandler
 ↓
Error Response
```

---

# 30. Response 분석

최종 Response까지 추적한다.

확인 대상:

```text
Response DTO
Response Entity
Status
Result Code
Result Message
Data
Error Response
```

Service 반환값이 Controller에서 어떻게 변환되는지도 기록한다.

---

# 31. BE ID

BE ID 형식:

```text
BE-001
BE-002
BE-003
...
```

Parallel Agent에서 BE ID가 전달된 경우
전달받은 ID를 그대로 사용한다.

단독 실행에서 BE ID가 전달되지 않은 경우
해당 FE Base Name에 이미 존재하는 BE 문서와 충돌하지 않는
다음 번호를 사용한다.

단, FE Base Name을 확인할 수 없는 경우
임의로 FE Action 이름을 생성하지 않는다.

---

# 32. 출력 파일명 규칙

표준 Backend 문서 파일명:

```text
{FE Base Name}-{BE ID}.md
```

예:

```text
FE Base Name = FE-ACT-010-create
BE ID       = BE-001
```

결과:

```text
FE-ACT-010-create-BE-001.md
```

전체 경로:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

예:

```text
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
```

다음과 같은 파일명은 사용하지 않는다.

```text
BE-001.md
BE-001-create.md
FE-ACT-010-BE-001.md
```

---

# 33. FE Base Name 결정

Parallel Agent가 호출한 경우:

```text
FE Base Name
```

을 반드시 전달받는다.

전달받은 값을 그대로 사용한다.

예:

```text
FE-ACT-010-create
```

단독 실행에서 FE Base Name이 명확하지 않은 경우
Backend URL로부터 FE Action 이름을 추정하지 않는다.

필요한 경우 사용자에게 FE Base Name을 요청한다.

---

# 34. Output File 우선 규칙

Parallel Agent 또는 상위 Orchestrator가
다음 값을 전달한 경우:

```text
Output File
```

해당 경로를 최우선으로 사용한다.

예:

```text
Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
```

Skill은 다음을 수행하지 않는다.

```text
파일명 재생성
BE ID 재할당
FE Base Name 변경
Output File 변경
```

---

# 35. 기존 파일 보호

Output File이 이미 존재하는 경우
임의로 덮어쓰지 않는다.

```text
STOP

Output File already exists:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
```

사용자가 명시적으로 덮어쓰기를 요청한 경우에만
기존 파일을 갱신한다.

---

# 36. 문서 기본 구조

생성 문서는 최소 다음 구조를 가진다.

```markdown
# Backend Analysis

## 1. 분석 대상

## 2. Controller Mapping

## 3. 전체 실행 흐름

## 4. Controller

## 5. Business Logic

## 6. 조건 분기

## 7. DB / MyBatis / SQL

## 8. Oracle Metadata

## 9. RFC / 외부 연동

## 10. Response

## 11. 예외 처리

## 12. Evidence

## 13. 전체 흐름도
```

실제 Source에 없는 영역은 임의로 만들지 않는다.

필요한 경우:

```text
해당 없음
```

또는:

```text
확인되지 않음
```

으로 기록한다.

---

# 37. 전체 실행 흐름 작성 규칙

가장 중요한 결과는
실제 실행 순서를 보여주는 전체 실행 트리이다.

예:

```text
POST /api/equipment/create
 ↓
EquipmentController.create()
 ↓
EquipmentService.create()
 ↓
validateRequest()
 ↓
DB #1
 EquipmentMapper.selectDuplicate()
 ↓
중복 존재?
 ├─ YES
 │   ↓
 │  BusinessException
 │   ↓
 │  Error Response
 │
 └─ NO
     ↓
    DB #2
     EquipmentMapper.insert()
     ↓
    RFC #1
     Z_EQUIPMENT_CREATE
     ↓
    RFC 성공?
     ├─ YES
     │   ↓
     │  DB #3
     │   상태 UPDATE
     │   ↓
     │  Success Response
     │
     └─ NO
         ↓
        DB #4
         오류 상태 UPDATE
         ↓
        BusinessException
         ↓
        Error Response
```

---

# 38. Evidence 규칙

Evidence는 반드시 실제 Source 기준으로 기록한다.

형식:

```text
프로젝트 상대경로
파일명
Line Range
Symbol
```

예:

```text
gipms-api-equipment/src/main/java/.../EquipmentController.java
Lines 42-58
EquipmentController.create()
```

절대경로는 기록하지 않는다.

REFERENCE 문서는 Evidence가 아니다.

---

# 39. 분석 완료 조건

다음이 확인되어야 완료로 판단한다.

```text
Controller Mapping 확인
Controller Method 확인
실제 Service 흐름 확인
조건 분기 확인
Mapper/MyBatis 확인
SQL 확인
DB 호출 순서 확인
RFC/REST/외부 호출 확인
Response 확인
예외 처리 확인
Evidence 기록
전체 실행 트리 작성
```

해당 요소가 Source에 존재하지 않는 경우에는
존재하지 않음을 명확히 기록한다.

---

# 40. STOP 조건

다음 경우 임의 진행하지 않는다.

```text
Controller를 찾을 수 없음

UNKNOWN Method인데 동일 URL에 여러 Method 존재

Backend URL이 여러 Controller에 모호하게 Mapping됨

필수 Source 접근 불가

FE Base Name이 필요한데 전달되지 않음

Output File이 이미 존재하고 덮어쓰기 지시 없음
```

STOP 시 이유와
사용자가 선택하거나 제공해야 할 값을 명확하게 출력한다.

---

# 41. 최종 원칙

이 Skill의 핵심은 다음이다.

```text
Backend URL
 ↓
Controller 찾기
 ↓
실제 Source 실행 경로 추적
 ↓
Business Logic
 ↓
DB / RFC / REST / External 호출을
실제 발생 순서대로 연결
 ↓
SQL 기반 Oracle 대상 수집
 ↓
중복 제거 후 필요한 Metadata만 확인
 ↓
Response까지 연결
 ↓
Evidence 포함 문서 생성
```

API 문서가 없어도 분석할 수 있어야 한다.

하지만 API 문서가 없다는 이유로
Source에서 확인되지 않은 내용을 추측해서는 안 된다.
---
name: be-direct-analysis
description: API 분석 문서 없이 화면명, Action ID, HTTP Method와 Backend URL을 입력받아 실제 FE 문서에서 FE Base Name을 확정하고 Controller부터 Response까지 Backend를 분석한다. BE-REFERENCE.md를 최종 출력 Template으로 사용하며 FE Base Name 기반의 정확한 BE 파일명으로 저장한다.
argument-hint: "<화면명> <Action ID> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# Backend Direct Analysis

## 1. 목적

API 분석 문서 없이 Backend를 직접 분석한다.

입력:

```text
화면명
Action ID
HTTP Method
Backend URL
```

예:

```text
/be-direct-analysis equipment-search ACT-010 UNKNOWN /api/equipment/create
```

Direct 분석의 시작점:

```text
화면명 + Action ID
 ↓
실제 FE 문서 탐색
 ↓
FE Base Name 확정
 ↓
HTTP Method + Backend URL
 ↓
Controller 직접 탐색
 ↓
Backend 실제 실행 흐름 분석
 ↓
BE-REFERENCE 기반 최종 문서 생성
 ↓
FE Base Name 기반 파일명으로 저장
 ↓
결과 문서 재검증
```

API 분석 문서는 필요하지 않다.


# 2. 적용 파일

분석 시작 시 다음 파일을 실제로 읽는다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
.claude/references/BE-REFERENCE.md
```

이 세 파일은 이름만 알고 진행하면 안 된다.

반드시 실제 파일 내용을 읽는다.


# 3. 실행 순서 강제

다음 순서를 변경하지 않는다.

```text
STEP 1
FE 문서 확인

STEP 2
FE Base Name 확정

STEP 3
BE ID / OUTPUT_PATH 확정

STEP 4
08-be-analysis-scope.md Read

STEP 5
09-be-call-tracing.md Read

STEP 6
BE-REFERENCE.md Read

STEP 7
Controller 탐색

STEP 8
Backend 전체 Call Path 분석

STEP 9
필요한 Oracle Metadata 확인

STEP 10
BE-REFERENCE 기반 Rendering

STEP 11
OUTPUT_PATH Write

STEP 12
생성된 문서 다시 Read

STEP 13
파일명 + Reference 형식 검증

STEP 14
PASS / STOP / ERROR
```

STEP 1~6이 완료되기 전에
Backend 상세 분석을 시작하지 않는다.


# 4. FE 문서 탐색

입력받은:

```text
화면명
Action ID
```

를 이용한다.

FE 문서 위치:

```text
docs/analysis/{화면명}/frontend/
```

Action ID에 해당하는 FE 문서를 찾는다.

예:

```text
Action ID:
ACT-010

검색 대상:
docs/analysis/equipment-search/frontend/

발견:
FE-ACT-010-create.md
```

검색은 해당 frontend 디렉토리로 제한한다.

Project 전체 재귀 검색을 하지 않는다.


# 5. FE 문서 선택 규칙

Action ID와 일치하는 FE 문서가 정확히 하나이면 사용한다.

예:

```text
ACT-010

→ FE-ACT-010-create.md
```

일치하는 FE 문서가 없으면:

```text
STATUS:
STOP

ERROR:
Action ID에 해당하는 FE 문서를 찾을 수 없음
```

여러 개가 발견되어 하나를 확정할 수 없으면:

```text
STATUS:
STOP

ERROR:
Action ID에 해당하는 FE 문서가 여러 개 존재
```

임의 선택하지 않는다.


# 6. FE Base Name 확정

FE Base Name은 사용자 입력이나 Backend URL에서 만들지 않는다.

반드시 실제 FE 문서 파일명에서 확정한다.

예:

```text
FE Document:
FE-ACT-010-create.md

.md 제거
 ↓

FE_BASE_NAME:
FE-ACT-010-create
```

즉:

```text
FE-ACT-010-create.md
        ↓
FE-ACT-010-create
```

이다.

다음으로 변경하지 않는다.

```text
FE-ACT-010
ACT-010-create
create
equipment-create
```


# 7. BE ID

Standalone 실행에서는 현재 FE Base Name에 연결된
Backend 문서만 확인하여 다음 BE ID를 결정한다.

Backend 문서 위치:

```text
docs/analysis/{화면명}/backend/
```

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

으로 결정한다.

다른 FE Action의 BE 번호는 사용하지 않는다.

Parallel Worker에서 BE_ID를 전달받은 경우에는
새로 계산하지 않는다.

전달받은 BE_ID를 그대로 사용한다.


# 8. 공식 파일명

Backend 문서의 공식 파일명은 오직 다음 형식이다.

```text
{FE_BASE_NAME}-{BE_ID}.md
```

예:

```text
FE_BASE_NAME:
FE-ACT-010-create

BE_ID:
BE-001
```

이면:

```text
FE-ACT-010-create-BE-001.md
```

이다.

다음 형식은 허용하지 않는다.

```text
BE-001.md
BE-001-create.md
BE-ACT-010-create.md
FE-ACT-010-BE-001.md
BE-API-001-create.md
API-001-BE-001.md
```


# 9. OUTPUT_PATH

Standalone 실행:

```text
docs/analysis/{화면명}/backend/{FE_BASE_NAME}-{BE_ID}.md
```

예:

```text
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-001.md
```

Parallel Worker에서 OUTPUT_PATH를 전달받은 경우
전달받은 값을 그대로 사용한다.

Worker는 OUTPUT_PATH를 다시 생성하지 않는다.


# 10. Worker 불변값

Parallel Worker 실행 시 다음 값은 불변이다.

```text
SCREEN_NAME
ACTION_ID
FE_DOCUMENT
FE_BASE_NAME
BE_ID
HTTP_METHOD
BACKEND_URL
OUTPUT_PATH
```

Worker가 다시 계산하거나 변경하지 않는다.


# 11. OUTPUT_PATH 사전 검증

Backend 분석 시작 전에 검증한다.

```text
EXPECTED_FILENAME
  =
{FE_BASE_NAME}-{BE_ID}.md

OUTPUT_PATH filename
  =
EXPECTED_FILENAME
```

다르면 STOP 한다.

예:

```text
FE_BASE_NAME:
FE-ACT-010-create

BE_ID:
BE-001

EXPECTED_FILENAME:
FE-ACT-010-create-BE-001.md
```

OUTPUT_PATH가:

```text
docs/analysis/equipment-search/backend/BE-001.md
```

이면 분석을 시작하지 않는다.


# 12. 기존 파일 보호

OUTPUT_PATH가 이미 존재하면
사용자가 명시적으로 overwrite를 요청하지 않은 한
덮어쓰지 않는다.

```text
STATUS:
STOP

ERROR:
Output file already exists
```


# 13. Rule Load

다음 두 파일을 실제 Read 한다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

Skill에 유사한 내용이 있다는 이유로
Read를 생략하지 않는다.

Rule을 읽을 수 없으면 STOP 한다.


# 14. BE-REFERENCE Load

다음 파일을 실제 Read 한다.

```text
.claude/references/BE-REFERENCE.md
```

이 파일은 단순 참고자료가 아니다.

현재 Backend 문서의:

```text
OUTPUT TEMPLATE
```

이다.

Reference를 읽을 수 없으면
Backend 분석을 계속하지 않는다.

```text
STATUS:
STOP

ERROR:
BE-REFERENCE.md를 읽을 수 없음
```


# 15. Reference 역할

BE-REFERENCE.md는 다음을 결정한다.

```text
문서 구조
Section 순서
Section 제목
표현 형식
상세 수준
ASCII Tree
ASCII Block
Excel 복사 구조
Block 독립성
DB 상세 표현
SQL 상세 표현
Dynamic SQL 표현
Parameter Mapping 표현
Result Mapping 표현
Oracle Metadata 표현
외부 연동 표현
Exception / Transaction 표현
Response 표현
Source Evidence 표현
미확인 항목 표현
분석 경계 표현
```

실제 값은 Reference에서 가져오지 않는다.

실제 값은:

```text
실제 Source
MyBatis SQL
필요한 경우 Oracle Metadata
확인된 Runtime Evidence
```

에서 가져온다.


# 16. Reference Sample 사용 금지

BE-REFERENCE.md의 Sample은 Evidence가 아니다.

예:

```text
EquipmentController
EquipmentServiceImpl
EquipmentMapper
TB_EQUIPMENT
TB_PLANT
Z_PM_EQUIPMENT_SEARCH
/api/equipment/search
plantCode
```

실제 Source에서 확인되지 않았다면
최종 문서에 사용하지 않는다.


# 17. Reference 재로드 금지

BE-REFERENCE.md는 STEP 6에서 1회 읽는다.

이후:

```text
Controller 탐색
Service 분석
Mapper 분석
SQL 분석
Oracle Metadata
RFC
REST
Rendering
Write
검증
```

중간에 반복 Read하지 않는다.


# 18. Reference Template 우선 원칙

최종 문서를 작성할 때
Claude가 별도의 문서 디자인을 만들지 않는다.

Reference의 형식을 우선한다.

금지:

```text
Reference Section을 임의로 합치기

Section 순서를 임의 변경

ASCII Block을 Markdown Bullet로 대체

ASCII 실행 Tree를 일반 번호 목록으로 대체

ASCII Tree를 Mermaid로 대체

DB 상세 Block을 Markdown Table로 변경

Business Logic Block을 Markdown Table로 변경

RFC / REST Block을 Markdown Table로 변경

Source Evidence를 Markdown Table로 변경

Reference보다 간단한 요약형 문서 생성

Claude가 더 보기 좋다고 판단한
새로운 문서 형식 사용
```

Oracle Metadata는 Reference에서 허용한 경우에만
Markdown Table을 사용할 수 있다.


# 19. Reference 기본 Section

최종 문서는 BE-REFERENCE.md의 기본 구조를 따른다.

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

실제 기능에 존재하지 않는 상세 항목을
억지로 생성하지 않는다.

단, 문서 전체 구조를 Claude 임의 형식으로
재설계하지 않는다.


# 20. Direct 기능 정보 적용

Direct 분석에는 API Document가 없으므로
API ID를 임의 생성하지 않는다.

Reference의 기능 정보 형식을 유지하면서
Direct 식별값을 사용한다.

예:

```text
┌─ 기능 정보 ──────────────────────────────────
│ 화면
│   {화면명}
│
│ FE Action
│   {FE_BASE_NAME}
│
│ BE ID
│   {BE_ID}
│
│ 기능명
│   {실제 Source에서 확인된 경우}
│
│ HTTP Method
│   {RESOLVED_METHOD}
│
│ Backend URL
│   {BACKEND_URL}
│
│ Backend Project
│   {실제 Project}
└──────────────────────────────────────────────
```

기능명을 Source에서 확정할 수 없으면
추측하지 않는다.


# 21. Project Root

Code Index MCP에 설정된
현재 Project Root를 기준으로 분석한다.

Backend 기본 대상:

```text
gipms-api-*
```

현재 Backend URL과 실제 Call Path에 연결된
Project만 분석한다.


# 22. Source Path

Source Evidence는 Project Root 기준 상대경로를 사용한다.

절대경로를 최종 문서에 기록하지 않는다.


# 23. Controller 탐색

입력:

```text
HTTP_METHOD
BACKEND_URL
```

을 기준으로 Controller Mapping을 찾는다.

Code Index MCP를 우선 사용한다.

확인 대상:

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Class Mapping과 Method Mapping을 조합한다.

Code Index 결과만으로 확정하지 않는다.

실제 Controller Source를 읽어서 확정한다.


# 24. UNKNOWN Method

HTTP_METHOD가 UNKNOWN이면
실제 Controller Source에서 Method를 확정한다.

동일 URL에 여러 Method가 존재하고
하나로 확정할 수 없으면 STOP 한다.

임의 선택하지 않는다.


# 25. Controller 탐색 실패

실제 Source에서 Controller Mapping을
확정하지 못하면 STOP 한다.

유사 URL을 대신 선택하지 않는다.


# 26. Backend 분석

Controller 확정 이후에는
08-be-analysis-scope.md와
09-be-call-tracing.md를 기준으로 분석한다.

실제 Source 실행 순서를 보존한다.

기본 흐름:

```text
Controller
 ↓
Service / ServiceImpl
 ↓
Business Logic
 ↓
Internal Method
 ↓
Other Service / Common Service
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
 ↓
Controller Return
```

이 흐름은 예시이며
실제 Source 순서가 우선이다.


# 27. Code Index 사용

Code Index는 Discovery 용도이다.

```text
Controller 후보
Symbol
Reference
Caller
Callee
관련 Source
```

를 찾는 데 사용한다.

Business Logic 확정은
실제 Source를 읽은 결과를 기준으로 한다.


# 28. Shell 탐색 제한

다음을 사용하지 않는다.

```text
xargs
Project Root 전체 find
grep -r
grep -R
grep -rn
```

Rule 09의 탐색 제한을 그대로 적용한다.


# 29. 실제 실행 순서

Layer 기준으로 재정렬하지 않는다.

예를 들어 실제 Source가:

```text
DB #1
 ↓
조건
 ↓
RFC #1
 ↓
RFC 결과 처리
 ↓
DB #2
 ↓
REST #1
 ↓
REST 결과 처리
 ↓
DB #3
 ↓
Response
```

이면 그대로 유지한다.


# 30. Caller 복귀

하위 호출 이후 Caller의 후속 처리를
반드시 계속 추적한다.

```text
Service A
 │
 ├─ Common Service
 │    │
 │    ├─ RFC #1
 │    └─ return
 │
 ├─ Service A 복귀
 ├─ RFC 결과 처리
 ├─ DB #2
 └─ Response
```

하위 호출에서 분석을 종료하지 않는다.


# 31. Mapper / MyBatis / SQL

실제 연결을 확인한다.

```text
Mapper Interface
 ↓
Mapper Method
 ↓
namespace
 ↓
Statement ID
 ↓
MyBatis XML / Annotation
 ↓
Dynamic SQL
 ↓
SQL
```

Method 이름으로 SQL을 추측하지 않는다.


# 32. DB 호출

실제 실행 위치별로 번호를 부여한다.

```text
DB #1
DB #2
DB #3
...
```

동일 Mapper/Statement가 반복되어도
실행 위치가 다르면 별도 호출이다.


# 33. 외부 연동

다음을 실제 실행 위치에서 추적한다.

```text
RFC
REST / HTTP
SOAP
WebService
다른 Backend
Gateway
Interface Server
Message / Queue
File
기타 외부 시스템
```

종류별로 재정렬하지 않는다.


# 34. 조건 / 반복 / Exception / Transaction

실제 Source에서 확인되는 구조만 기록한다.

```text
조건
분기
반복
Exception
Transaction
```

Framework 일반 동작으로 추측하지 않는다.


# 35. Response까지 추적

DB나 외부 연동에서 종료하지 않는다.

```text
DB / External Result
 ↓
Caller 복귀
 ↓
후속 Business Logic
 ↓
변환
 ↓
Response
 ↓
Controller Return
```

까지 확인한다.


# 36. Oracle Metadata

먼저 Controller부터 Response까지
전체 Call Path를 완료한다.

그 과정에서 실제 SQL에 사용된:

```text
Table
View
Column
```

을 수집한다.

그 후:

```text
중복 제거
 ↓
Metadata 필요성 판단
 ↓
필요한 Object / Column만 Oracle MCP 확인
 ↓
SQL ↔ Metadata 교차 검증
```

한다.

금지:

```text
Schema 전체 탐색
관련 없는 Object 조회
모든 Column 무조건 조회
동일 Object 반복 조회
동일 Column 반복 조회
DB 호출마다 Metadata 조회
```

Metadata가 필요하지 않으면:

```text
Metadata 추가 조회하지 않음
```

으로 구분한다.


# 37. Source Evidence

중요한 판단은 가능한 경우 다음과 연결한다.

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
MyBatis XML
Namespace
Statement ID
SQL
```

외부 연동:

```text
Source Path
Class
Method
Adapter / Client
RFC Function / Endpoint
```

Line Range를 확인할 수 없으면 추측하지 않는다.


# 38. JAR / 외부 Dependency

자동 확장하지 않는다.

```text
*.jar
Decompiled Source
.m2
.gradle
Maven Repository
Gradle Cache
Project Root 외부 Source
```

Project Root 아래 실제 Source Project이며
현재 Call Path와 연결된 경우에는 추적할 수 있다.


# 39. 전체 Call Path 검증

Rendering 전에 확인한다.

```text
Controller
 ↓
Service / 구현체
 ↓
Business Logic
 ↓
Internal / Other Service
 ↓
Caller 복귀
 ↓
DB / External
 ↓
Caller 복귀
 ↓
후속 Business Logic
 ↓
Response
 ↓
Controller Return
```

끊긴 부분이 있으면
확인 가능한 Source를 추가 추적한다.


# 40. Reference 기반 Rendering

분석이 완료되면
STEP 6에서 읽은 BE-REFERENCE.md를
최종 문서의 Template으로 사용한다.

중요:

```text
분석 결과를 먼저 임의 Markdown으로 작성한 뒤
Reference를 참고해 조금 수정하는 방식이 아니다.
```

반드시:

```text
BE-REFERENCE Template
        +
실제 분석 결과
        ↓
최종 문서
```

방식으로 생성한다.


# 41. OUTPUT_PATH Write

최종 문서를 오직:

```text
OUTPUT_PATH
```

에 저장한다.

Worker에서는 별도 파일명을 생성하지 않는다.

다른 이름의 Backend 문서를 추가 생성하지 않는다.


# 42. 생성 문서 재검증

Write 직후 OUTPUT_PATH의 문서를
실제로 다시 읽는다.

자기 직전 출력 내용을 기억해서
검증했다고 처리하지 않는다.

실제 저장된 파일을 대상으로 검증한다.


# 43. 파일명 검증

다시 읽은 실제 파일의 경로가:

```text
OUTPUT_PATH
```

와 정확히 일치하는지 확인한다.

파일명:

```text
{FE_BASE_NAME}-{BE_ID}.md
```

와 정확히 일치해야 한다.

다르면 PASS 금지.


# 44. Reference 결과 검증

`REFERENCE LOADED = YES`만으로
Reference 적용 성공으로 판단하지 않는다.

실제 생성된 문서를 대상으로 검증한다.

다음을 확인한다.

```text
[ ] Reference 기본 Section 구조 사용

[ ] 기능 정보가 Reference의 ASCII Block 형태

[ ] 기능 요약이 Reference의 ASCII Block 형태

[ ] 한눈에 보는 실행 흐름이 ASCII Block 형태

[ ] 핵심 정보가 ASCII Block 형태

[ ] 전체 실행 Tree가 ASCII Tree 형태

[ ] Business Logic 상세가 독립 ASCII Block

[ ] DB 호출이 DB #n별 독립 Block

[ ] SQL 상세가 Reference 형식

[ ] Dynamic SQL이 존재하면 Reference 형식

[ ] Parameter Mapping이 존재하면 Reference 형식

[ ] Result Mapping이 존재하면 Reference 형식

[ ] 외부 연동이 존재하면 각각 독립 Block

[ ] Exception이 존재하면 독립 Block

[ ] Transaction 표현이 Reference 형식

[ ] Response 생성이 독립 Block

[ ] Source Evidence가 독립 Block

[ ] 미확인 항목이 Reference 형식

[ ] 분석 경계가 Reference 형식

[ ] Oracle Metadata 이외 일반 영역에
    Markdown Table을 임의 사용하지 않음

[ ] "위와 동일", "앞에서 설명" 등
    Block 독립성을 깨는 표현 사용하지 않음
```

실제 기능에 존재하지 않는 세부 Block은
없다는 이유만으로 실패 처리하지 않는다.


# 45. Reference 검증 실패

문서 구조가 Reference와 다르면
PASS 하지 않는다.

가능한 경우 OUTPUT_PATH의 문서를
Reference Template에 맞게 수정한 뒤
다시 Read하여 재검증한다.

무한 반복하지 않는다.

최대 1회 수정한다.

1회 수정 후에도 Reference 형식이 맞지 않으면:

```text
STATUS:
ERROR

REFERENCE FORMAT MATCH:
NO
```

로 종료한다.


# 46. PASS 조건

다음을 모두 만족해야 한다.

```text
FE DOCUMENT FOUND:
YES

FE BASE NAME FROM FE DOCUMENT:
YES

RULE 08 LOADED:
YES

RULE 09 LOADED:
YES

REFERENCE LOADED:
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

REFERENCE FORMAT MATCH:
YES
```

하나라도 필수 조건이 NO이면
PASS를 반환하지 않는다.


# 47. Worker 결과

Parent에는 상세 Backend 문서 본문을 반환하지 않는다.

다음만 반환한다.

```text
STATUS:
PASS | STOP | ERROR

SCREEN:
...

ACTION ID:
...

FE DOCUMENT:
...

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

REFERENCE LOADED:
YES | NO

REFERENCE FORMAT MATCH:
YES | NO

EXPECTED OUTPUT:
...

ACTUAL OUTPUT:
...

OUTPUT PATH MATCH:
YES | NO

OUTPUT FILENAME MATCH:
YES | NO

ERROR:
없음 또는 오류
```


# 48. 안전 원칙

Read-Only 분석이다.

실행하지 않는다.

```text
Oracle DML / DDL
실제 RFC 업무 Function
상태 변경 REST
SOAP 업무 요청
Message Publish
업무 File 전송
실제 저장 / 수정 / 삭제 API
```


# 49. STOP

현재 Backend 하나의 분석과:

```text
Reference 기반 Rendering
OUTPUT_PATH 저장
저장 파일 재검증
Reference 형식 검증
파일명 검증
```

까지 완료하면 STOP 한다.

다른 Backend를 자동 분석하지 않는다.
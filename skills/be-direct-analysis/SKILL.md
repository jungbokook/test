---
name: be-direct-analysis
description: API 분석 문서 없이 FE Base Name, HTTP Method와 Backend URL을 직접 입력받아 Controller를 찾고, Backend Rule과 BE-REFERENCE.md를 적용하여 기존 Backend 분석 수준의 문서를 정확한 FE Base Name 기반 파일명으로 생성한다.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
user-invocable: true
disable-model-invocation: true
---

# Backend Direct Analysis

## 1. 목적

API 분석 문서 없이 Backend를 직접 분석한다.

입력:

```text
화면명
FE Base Name
HTTP Method
Backend URL
```

예:

```text
/be-direct-analysis equipment-search FE-ACT-010-create UNKNOWN /api/equipment/create
```

분석 흐름:

```text
입력 검증
 ↓
FE Base Name 고정
 ↓
BE ID 확정
 ↓
OUTPUT_PATH 확정
 ↓
Backend Rule Read
 ↓
BE-REFERENCE.md Read
 ↓
Controller 확인
 ↓
Backend 전체 Call Path 분석
 ↓
Oracle Metadata 필요성 판단
 ↓
BE-REFERENCE 기반 Rendering
 ↓
OUTPUT_PATH 저장
 ↓
생성 파일 재검증
 ↓
PASS / STOP / ERROR
```

API Document는 필요하지 않다.


# 2. 입력

필수 입력:

```text
SCREEN_NAME
FE_BASE_NAME
HTTP_METHOD
BACKEND_URL
```

예:

```text
SCREEN_NAME:
equipment-search

FE_BASE_NAME:
FE-ACT-010-create

HTTP_METHOD:
UNKNOWN

BACKEND_URL:
/api/equipment/create
```

FE_BASE_NAME은 사용자가 전달한 값을 그대로 사용한다.


# 3. FE Base Name 불변 계약

FE_BASE_NAME은 파일명 생성의 기준값이다.

입력받은 FE_BASE_NAME을:

```text
재계산하지 않는다.
추측하지 않는다.
축약하지 않는다.
변환하지 않는다.
다른 문서에서 다시 추출하지 않는다.
```

예:

```text
INPUT:

FE_BASE_NAME:
FE-ACT-010-create
```

이면 분석 종료까지:

```text
FE_BASE_NAME:
FE-ACT-010-create
```

이다.

다음 값으로 변경하면 안 된다.

```text
ACT-010
ACT-010-create
create
equipment-create
Controller 이름
Service 이름
URL 일부
문서 Section 번호
문서 Heading 번호
36
36.2
```

특히 Markdown 문서의:

```text
# 36.
## 36.2
```

같은 Section 번호를
FE_BASE_NAME 또는 파일명으로 사용하지 않는다.


# 4. 적용 Rule / Reference

Backend 분석 시 다음 실제 파일을 사용한다.

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
.claude/references/BE-REFERENCE.md
```

Skill 내부 설명으로 위 파일을 대체하지 않는다.


# 5. 실행 순서

다음 순서를 따른다.

```text
STEP 1
입력 검증

STEP 2
FE_BASE_NAME 불변값 확정

STEP 3
BE_ID 확정

STEP 4
OUTPUT_FILENAME 계산

STEP 5
OUTPUT_PATH 계산 및 검증

STEP 6
08-be-analysis-scope.md Read

STEP 7
09-be-call-tracing.md Read

STEP 8
BE-REFERENCE.md Read

STEP 9
Controller Mapping 확인

STEP 10
Controller → Response 전체 Call Path 분석

STEP 11
Oracle Metadata 필요성 판단 및 필요한 경우 최소 조회

STEP 12
BE-REFERENCE 기반 Rendering

STEP 13
OUTPUT_PATH Write

STEP 14
실제 저장 파일 Read

STEP 15
Filename 검증

STEP 16
Reference Format 검증

STEP 17
PASS / STOP / ERROR
```


# 6. BE ID

## 6.1 Parallel Worker

Parent가 다음 값을 전달한 경우:

```text
BE_ID
OUTPUT_PATH
```

Worker는 전달받은 값을 그대로 사용한다.

재계산하지 않는다.


## 6.2 Standalone 실행

Parent가 없는 `/be-direct-analysis` 직접 실행에서는
다음 디렉토리만 확인한다.

```text
docs/analysis/{SCREEN_NAME}/backend/
```

현재 FE_BASE_NAME으로 시작하는 Backend 문서만 확인한다.

예:

```text
FE_BASE_NAME:
FE-ACT-010-create
```

확인 대상:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
FE-ACT-010-create-BE-003.md
```

다른 FE Action의 Backend 문서는
BE ID 계산에 사용하지 않는다.

기존 번호 다음의 충돌 없는 BE ID를 사용한다.

예:

```text
BE-001
BE-002
```

가 존재하면:

```text
BE_ID:
BE-003
```

이다.


# 7. 파일명 생성 규칙

파일명 생성 공식은 하나뿐이다.

```text
OUTPUT_FILENAME
=
FE_BASE_NAME + "-" + BE_ID + ".md"
```

예:

```text
FE_BASE_NAME:
FE-ACT-010-create

BE_ID:
BE-001
```

계산:

```text
FE-ACT-010-create
+
-
+
BE-001
+
.md
```

결과:

```text
FE-ACT-010-create-BE-001.md
```

파일명을 생성할 때 사용할 수 있는 값은 오직:

```text
FE_BASE_NAME
BE_ID
```

두 개뿐이다.


# 8. 파일명 생성 금지 Source

다음 값으로 파일명을 생성하지 않는다.

```text
Backend URL
HTTP Method
Controller
Controller Method
Service
Service Method
Mapper
SQL ID
기능명
Reference Heading
Skill Heading
Section 번호
문서 번호
분석 단계 번호
현재 Markdown Heading
36
36.1
36.2
```

예를 들어 분석 중:

```text
## 36.2 Standalone 실행
```

을 읽었더라도:

```text
36.2.md
36.2-BE-001.md
```

같은 파일을 생성해서는 안 된다.


# 9. OUTPUT_PATH

Standalone 공식:

```text
OUTPUT_PATH
=
docs/analysis/{SCREEN_NAME}/backend/{OUTPUT_FILENAME}
```

예:

```text
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-001.md
```

Parallel Worker에서는 Parent가 전달한 OUTPUT_PATH를 사용한다.


# 10. OUTPUT_PATH 불변 계약

OUTPUT_PATH가 확정된 이후에는:

```text
재계산 금지
변경 금지
축약 금지
Rename 금지
다른 이름 생성 금지
```

이다.

Worker가 분석 결과를 보고
더 적절한 파일명을 판단해서는 안 된다.


# 11. 분석 전 파일명 검증

Backend Source 분석 전에 먼저 계산한다.

```text
EXPECTED_FILENAME
=
{FE_BASE_NAME}-{BE_ID}.md
```

그 후:

```text
basename(OUTPUT_PATH)
=
EXPECTED_FILENAME
```

인지 확인한다.

다르면 Backend 분석을 시작하지 않는다.

```text
STATUS:
STOP

ERROR:
OUTPUT_PATH filename does not match FE_BASE_NAME + BE_ID
```


# 12. 기존 파일 보호

OUTPUT_PATH가 이미 존재하면
사용자가 명시적으로 overwrite를 요청하지 않는 한
덮어쓰지 않는다.

```text
STATUS:
STOP

ERROR:
Output file already exists
```


# 13. Rule 08 Read

실제 파일:

```text
.claude/rules/08-be-analysis-scope.md
```

을 읽는다.

읽을 수 없으면 STOP 한다.


# 14. Rule 09 Read

실제 파일:

```text
.claude/rules/09-be-call-tracing.md
```

을 읽는다.

읽을 수 없으면 STOP 한다.

특히 기존 Rule 09의 탐색 제한을 그대로 적용한다.

```text
xargs 금지
Project Root 전체 find 금지
grep -r/-R/-rn 금지
```


# 15. BE-REFERENCE Read

실제 파일:

```text
.claude/references/BE-REFERENCE.md
```

을 읽는다.

Reference는 Evidence가 아니다.

Reference의 Sample 값도 Evidence가 아니다.

Reference는 최종 Backend 문서의:

```text
구조
표현 형식
상세 수준
ASCII Block
ASCII Tree
Section 구성
```

을 결정하는 Rendering Template이다.

읽을 수 없으면 STOP 한다.


# 16. Reference 적용 대상

BE-REFERENCE.md의 다음 특성을 유지한다.

```text
Section 순서
Section 제목
기능 정보 Block
기능 요약 Block
한눈에 보는 실행 흐름 Block
핵심 정보 Block
전체 실행 Tree
Business Logic Block
DB Block
SQL Block
Dynamic SQL Block
Parameter Mapping
Result Mapping
Oracle Metadata
RFC Block
REST Block
Exception Block
Transaction Block
Response Block
Source Evidence Block
미확인 항목 Block
분석 경계 Block
```


# 17. Reference Sample 금지

Reference 안의 예제 값은
실제 분석 결과로 사용하지 않는다.

최종 값은 다음에서만 가져온다.

```text
실제 Backend Source
실제 MyBatis
실제 SQL
필요한 경우 Oracle Metadata
확인된 외부 연동 Source
```


# 18. Reference 재로드 금지

BE-REFERENCE.md는 STEP 8에서 1회 읽는다.

이후 분석 중 반복해서 다시 읽지 않는다.

처음 읽은 Reference의 구조를
최종 Rendering까지 유지한다.


# 19. Project 범위

Project Root 아래 Backend 프로젝트:

```text
gipms-api-*
```

중 현재 Backend URL과 실제 Call Path에
연결된 프로젝트를 분석한다.

관련 없는 프로젝트로 확장하지 않는다.


# 20. Source Evidence Path

Source Evidence에는
Project Root 기준 상대경로를 사용한다.

절대경로를 최종 문서에 기록하지 않는다.


# 21. Controller 탐색

입력:

```text
HTTP_METHOD
BACKEND_URL
```

을 기준으로 Controller 후보를 찾는다.

Code Index MCP를 Discovery 용도로 사용한다.

후보를 찾은 후 반드시 실제 Controller Source를 읽는다.

확인:

```text
Class Mapping
Method Mapping
HTTP Method
Backend URL
Controller Method
Request
Response
```


# 22. UNKNOWN Method

HTTP_METHOD가:

```text
UNKNOWN
```

이면 실제 Controller Source에서 확정한다.

동일 URL에 여러 HTTP Method가 존재하여
하나로 확정할 수 없으면 STOP 한다.

임의 선택하지 않는다.


# 23. Controller 확정 실패

실제 Source에서 Controller를 확정하지 못하면:

```text
STATUS:
STOP
```

한다.

유사 URL을 대신 선택하지 않는다.


# 24. Code Index 원칙

Code Index는 Discovery이다.

다음 탐색에 사용한다.

```text
Controller 후보
Symbol
Reference
Caller
Callee
관련 Source
```

Business Logic 확정은
실제 Source를 읽어서 수행한다.


# 25. Backend Call Path

Controller 이후 실제 실행 순서를 추적한다.

예:

```text
Controller
 ↓
Service
 ↓
ServiceImpl
 ↓
Business Logic
 ↓
Internal Method
 ↓
Other Service
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL
 ↓
Caller 복귀
 ↓
후속 Business Logic
 ↓
RFC / REST
 ↓
Caller 복귀
 ↓
Response 생성
 ↓
Controller Return
```

실제 Source 순서가 우선이다.


# 26. Layer 재정렬 금지

실제 실행 순서를
Layer 종류별로 재정렬하지 않는다.

실제 코드가:

```text
DB #1
 ↓
조건
 ↓
RFC #1
 ↓
결과 가공
 ↓
DB #2
 ↓
REST #1
 ↓
결과 가공
 ↓
DB #3
 ↓
Response
```

이면 그대로 기록한다.


# 27. Caller 복귀

하위 호출에서 분석을 종료하지 않는다.

예:

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

Caller로 돌아온 후
후속 처리를 계속 추적한다.


# 28. Mapper / MyBatis / SQL

다음 연결을 실제 Source로 확인한다.

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
실제 SQL
```

Mapper Method 이름만 보고
SQL을 추측하지 않는다.


# 29. DB 번호

DB 호출은 실제 실행 위치 기준으로:

```text
DB #1
DB #2
DB #3
...
```

번호를 부여한다.

동일 Mapper Statement가 여러 번 호출되어도
실행 위치가 다르면 별도 DB 호출이다.


# 30. 외부 연동

실제 Source에서 확인되는:

```text
RFC
REST
HTTP
SOAP
WebService
다른 Backend
Gateway
Interface Server
Message / Queue
File
기타 외부 시스템
```

을 실제 실행 위치에서 기록한다.

RFC/REST를 문서 뒤로 임의 이동하지 않는다.


# 31. 조건 / 반복

실제 Source에서 확인되는:

```text
if
else
switch
loop
stream 조건
filter
분기
```

를 실행 흐름에 보존한다.

추측하지 않는다.


# 32. Exception / Transaction

실제 Source에서 확인되는 내용만 기록한다.

Framework 일반 동작을
실제 동작처럼 추측하지 않는다.


# 33. Response

DB나 외부 연동에서 분석을 끝내지 않는다.

반드시 가능한 범위에서:

```text
하위 호출 결과
 ↓
Caller 복귀
 ↓
후속 Business Logic
 ↓
변환
 ↓
Response 생성
 ↓
Controller Return
```

까지 추적한다.


# 34. Oracle Metadata

먼저 Controller → Response 전체 Call Path를 완료한다.

그 과정에서 실제 SQL에 사용된:

```text
Table
View
Column
```

을 수집한다.

전체 Call Path 완료 후 중복 제거한다.

그 다음 Metadata 필요성을 판단한다.

필요한 경우에만
Oracle MCP로 최소 조회한다.

금지:

```text
Schema 전체 탐색
관련 없는 Object 조회
모든 Column 무조건 조회
동일 Object 반복 조회
DB 호출마다 Metadata 조회
```


# 35. Metadata 상태 표현

Metadata 추가 조회가 필요하지 않았다면:

```text
Metadata 추가 조회하지 않음
```

으로 기록한다.

Metadata 확인이 필요했지만
실제로 확인하지 못했다면:

```text
확인되지 않음
```

으로 기록한다.

두 상태를 혼동하지 않는다.


# 36. 전체 Call Path 검증

Rendering 전에 실제 Call Path를 확인한다.

```text
Controller
 ↓
Service / 구현체
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
후속 Business Logic
 ↓
Response
 ↓
Controller Return
```

끊긴 구간이 있는지 확인한다.


# 37. BE-REFERENCE 기반 Rendering

분석 완료 후
BE-REFERENCE.md의 구조로 최종 문서를 작성한다.

먼저 Claude 자체 문서를 만든 후
Reference 스타일을 일부 덧씌우는 방식은 사용하지 않는다.

정상 방식:

```text
BE-REFERENCE 구조
        +
실제 분석 결과
        ↓
최종 Backend 문서
```


# 38. 최종 Section 순서

BE-REFERENCE.md의 기본 순서를 따른다.

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

실제 기능에 존재하지 않는 세부 Block을
억지로 만들지 않는다.


# 39. Markdown Table 제한

일반 분석 영역은
Reference의 ASCII/Text Block 형식을 사용한다.

Markdown Table은
BE-REFERENCE.md에서 허용하는
실제 Oracle Metadata 비교 영역에만 사용한다.

다음을 임의 Table로 변경하지 않는다.

```text
Business Logic
DB 호출
SQL
RFC
REST
Exception
Response
Source Evidence
```


# 40. 한눈에 보는 실행 흐름 강제 형식

`한눈에 보는 실행 흐름`은
BE-REFERENCE.md의 해당 Block 형식을 그대로 사용한다.

이 Section만 Claude가
다른 표현 방식으로 변경하면 안 된다.

반드시 다음 형태를 사용한다.

```text
┌─ 한눈에 보는 실행 흐름 ─────────────────────
│ {실제 시작 처리}
│   ↓
│ {실제 다음 처리}
│   ↓
│ {실제 다음 처리}
│   ↓
│ {실제 Response 처리}
└──────────────────────────────────────────────
```

조건이나 분기가 존재하면
ASCII Tree 표현을 사용한다.

예시 형식:

```text
│ {실제 조건}
│   ├─ {조건 1} → {실제 처리}
│   └─ {조건 2} → {실제 처리}
```

반복이 중요한 실행 흐름이면
실제 Source에서 확인된 반복 구조를
ASCII 형태로 표현한다.

이 Block은:

```text
전체 실행 Tree
```

를 그대로 복사하는 영역이 아니다.

주요 실행 흐름을 한눈에 볼 수 있도록
핵심 단계만 표시한다.

하지만 실제 실행 순서는 보존한다.


# 41. 한눈에 보는 실행 흐름 금지 형식

다음 형식을 사용하지 않는다.

```text
Mermaid

Markdown Bullet List

Markdown Number List

Markdown Table

Controller → Service → DB → Response

한 줄 Arrow Flow

Reference에 없는 Flowchart 문법
```

반드시 Reference의:

```text
┌─
│
│   ↓
│
└─
```

형식을 사용한다.


# 42. 전체 실행 Tree

전체 실행 Tree는
Backend 분석의 핵심 Section이다.

실제 실행 순서를 보존한다.

다음을 포함한다.

```text
Controller
Business Logic
조건
분기
반복
DB #n
RFC #n
REST #n
Caller 복귀
후속 처리
Exception
Response
```

실제로 존재하는 항목만 표현한다.


# 43. 독립 Block

각 상세 Block은
해당 Block만 복사해도 이해할 수 있게 작성한다.

다음 표현을 피한다.

```text
위와 동일
앞에서 설명
상동
이전 내용 참고
```

필요한 식별 정보를 Block 안에 포함한다.


# 44. Source Evidence

가능한 경우:

```text
Source Path
Class
Method
Line Range
```

를 기록한다.

Line Range를 실제로 확인하지 못하면
만들어내지 않는다.

MyBatis는 가능한 경우:

```text
Mapper
Mapper Method
XML
namespace
Statement ID
```

를 포함한다.


# 45. JAR / 외부 Source 제한

자동 탐색하지 않는다.

```text
*.jar
Decompiled Source
.m2
.gradle
Maven Repository
Gradle Cache
Project Root 외부 Source
```

현재 Project Root 아래의 실제 Source이며
Call Path에 연결된 경우만 분석한다.


# 46. 안전 원칙

Read-Only 분석이다.

실행하지 않는다.

```text
Oracle DML
Oracle DDL
실제 RFC 업무 Function
상태 변경 REST
SOAP 업무 요청
Message Publish
업무 File 전송
실제 저장/수정/삭제 API
```


# 47. OUTPUT Write

최종 문서는 오직:

```text
OUTPUT_PATH
```

에 저장한다.

새로운 파일명을 만들지 않는다.

OUTPUT_FILENAME을 다시 계산하지 않는다.

Section 번호를 파일명으로 사용하지 않는다.


# 48. Write 직전 파일명 재확인

파일을 Write하기 **직전** 다시 확인한다.

```text
FE_BASE_NAME:
{처음 입력받은 값}

BE_ID:
{확정된 값}

EXPECTED_FILENAME:
{FE_BASE_NAME}-{BE_ID}.md

OUTPUT_PATH:
{확정된 경로}
```

그리고:

```text
basename(OUTPUT_PATH)
=
EXPECTED_FILENAME
```

이어야 한다.

아니면 Write하지 않고 ERROR로 종료한다.


# 49. Write 후 실제 파일 Read

Write가 완료되면
OUTPUT_PATH의 실제 파일을 다시 읽는다.

기억하고 있는 생성 결과만으로
검증했다고 판단하지 않는다.


# 50. 파일명 최종 검증

다음을 다시 검증한다.

```text
EXPECTED_FILENAME
=
FE_BASE_NAME + "-" + BE_ID + ".md"

ACTUAL_FILENAME
=
실제 생성된 파일의 basename
```

반드시:

```text
EXPECTED_FILENAME
=
ACTUAL_FILENAME
```

이어야 한다.

다르면 PASS 금지.


# 51. Reference 결과 검증

실제 생성된 파일을 기준으로 확인한다.

```text
[ ] 기능 정보가 Reference 형식

[ ] 기능 요약이 Reference 형식

[ ] 한눈에 보는 실행 흐름이
    `┌─ 한눈에 보는 실행 흐름 ─`
    ASCII Block 형식

[ ] 한눈에 보는 실행 흐름의 각 단계가
    `│` 내부에 존재

[ ] 순차 흐름에 `↓` 사용

[ ] 조건/분기가 존재하면
    `├─`, `└─` 형식 사용

[ ] 한눈에 보는 실행 흐름에
    Mermaid / Markdown List / Table /
    한 줄 Arrow Flow를 사용하지 않음

[ ] 핵심 정보가 Reference 형식

[ ] 전체 실행 Tree가 ASCII Tree 형식

[ ] Business Logic이 독립 Block

[ ] DB 호출이 존재하면 DB #n별 독립 Block

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

[ ] Oracle Metadata 이외 영역에서
    Markdown Table을 임의 사용하지 않음
```


# 52. Reference 형식 수정

Reference 검증에서 실패한 경우
OUTPUT_PATH의 같은 파일을
Reference 형식에 맞게 최대 1회 수정할 수 있다.

새 파일을 만들지 않는다.

특히:

```text
한눈에 보는 실행 흐름
```

만 실패했다면
다른 정상 Section을 다시 설계하지 않는다.

해당 Section만 수정한다.

수정 후 실제 파일을 다시 읽어 검증한다.


# 53. Reference 검증 실패

1회 수정 후에도 실패하면:

```text
STATUS:
ERROR

REFERENCE FORMAT MATCH:
NO
```

로 종료한다.

문서가 존재한다는 이유만으로
PASS하지 않는다.


# 54. PASS 조건

다음을 모두 만족해야 한다.

```text
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

필수 조건 하나라도 NO이면
PASS하지 않는다.


# 55. Worker 결과 Contract

Parent에는 상세 Backend 문서 본문을 반환하지 않는다.

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

EXPECTED FILENAME:
...

ACTUAL FILENAME:
...

EXPECTED OUTPUT:
...

ACTUAL OUTPUT:
...

OUTPUT PATH MATCH:
YES | NO

OUTPUT FILENAME MATCH:
YES | NO

REFERENCE FORMAT MATCH:
YES | NO

ERROR:
...
```


# 56. STOP

현재 Backend 하나에 대해:

```text
Controller 확정
Backend Call Path 분석
Response 추적
필요한 Oracle Metadata 확인
BE-REFERENCE Rendering
OUTPUT_PATH Write
실제 파일 재검증
파일명 검증
Reference 형식 검증
```

이 완료되면 STOP 한다.

다른 Backend를 자동 분석하지 않는다.
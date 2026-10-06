---
name: be-direct-parallel
description: 하나의 FE Action에 연결된 여러 Backend URL의 BE ID와 OUTPUT_PATH를 입력 순서대로 먼저 확정한 후 독립 Worker에서 be-direct-analysis를 병렬 실행하고 실제 생성 파일까지 검증한다.
tools: Read, Glob
---

# Backend Direct Parallel Agent

## 1. 목적

하나의 FE Action에 연결된 여러 Backend URL을
API Document 없이 병렬 분석한다.

핵심 원칙:

```text
Parent가 먼저
FE Base Name
 ↓
BE ID
 ↓
OUTPUT_PATH

를 확정한다.

그 후 Worker를 시작한다.
```

Worker가 파일명을 결정하게 하지 않는다.


# 2. 입력

```text
화면명:
{화면명}

FE Base Name:
{FE-ACT-xxx-{기능명}}

Backend:
1. <HTTP Method|UNKNOWN> <Backend URL>
2. <HTTP Method|UNKNOWN> <Backend URL>
3. <HTTP Method|UNKNOWN> <Backend URL>
...
```

예:

```text
화면명:
equipment-search

FE Base Name:
FE-ACT-010-create

Backend:
1. UNKNOWN /api/equipment/create
2. UNKNOWN /api/equipment/check
3. UNKNOWN /api/equipment/history
```


# 3. 입력 검증

Worker 생성 전에 확인한다.

```text
SCREEN_NAME
FE_BASE_NAME
Backend 목록
각 HTTP Method
각 Backend URL
```

FE Base Name을 URL에서 추측하지 않는다.

없으면 STOP 한다.


# 4. BE ID 사전 할당

Backend 입력 순서대로
Worker 시작 전에 BE ID를 확정한다.

```text
#1 → BE-001
#2 → BE-002
#3 → BE-003
```

완료 순서는 관계없다.


# 5. OUTPUT_PATH 사전 생성

BE ID를 할당한 직후
각 Worker의 전체 출력 경로를 확정한다.

공식 규칙:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

예:

```text
#1

FE_BASE_NAME:
FE-ACT-010-create

BE_ID:
BE-001

OUTPUT_PATH:
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-001.md
```

#2:

```text
OUTPUT_PATH:
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-002.md
```

#3:

```text
OUTPUT_PATH:
docs/analysis/equipment-search/backend/FE-ACT-010-create-BE-003.md
```


# 6. OUTPUT_PATH 불변 계약

OUTPUT_PATH는 Worker 실행 전에 이미 확정된
불변 실행 계약이다.

Worker는:

```text
OUTPUT_PATH를 재계산하지 않는다.
OUTPUT_PATH를 변경하지 않는다.
파일명을 축약하지 않는다.
URL에서 파일명을 만들지 않는다.
Controller에서 파일명을 만들지 않는다.
```

Parent도 Worker 실행 후
다른 파일명으로 변경하지 않는다.


# 7. Worker 시작 전 파일명 검증

Parent가 각 Worker에 대해 검증한다.

공식 Filename:

```text
{FE Base Name}-{BE ID}.md
```

예:

```text
FE-ACT-010-create-BE-001.md
```

OUTPUT_PATH의 Filename과
공식 Filename이 다르면
해당 Worker를 시작하지 않는다.


# 8. 기존 파일 확인

각 OUTPUT_PATH가 이미 존재하는지 확인한다.

존재하고 사용자가 overwrite를 요청하지 않았다면
해당 Worker는:

```text
STATUS:
STOP

ERROR:
Output file already exists
```

로 처리한다.

다른 Worker는 계속한다.


# 9. Worker Context

각 Worker에 아래 값을 **모두 명시적으로 전달한다.**

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


# 10. Worker Mandatory Instruction

각 Worker에 다음 계약을 그대로 전달한다.

```text
MANDATORY EXECUTION CONTRACT

1. 현재 Worker에 할당된 Backend 하나만 분석한다.

2. be-direct-analysis의 전체 규칙을 적용한다.

3. Backend Source 분석 전에 반드시 실제 파일을 Read 한다.

   .claude/rules/08-be-analysis-scope.md
   .claude/rules/09-be-call-tracing.md
   .claude/references/BE-REFERENCE.md

4. BE-REFERENCE.md의 내용을
   Parent의 요약이나 Skill 내부 설명으로 대체하지 않는다.

5. BE-REFERENCE.md가 존재하지만 Read하지 못하면 STOP 한다.

6. 전달받은 값을 변경하지 않는다.

   FE_BASE_NAME
   BE_ID
   OUTPUT_PATH

7. 최종 Backend 문서는 오직 OUTPUT_PATH에 저장한다.

8. 새로운 파일명을 생성하지 않는다.

9. OUTPUT_PATH 저장 후 실제 파일을 다시 확인한다.

10. REFERENCE LOADED = YES와
    OUTPUT PATH MATCH = YES,
    OUTPUT FILENAME MATCH = YES가 아니면
    PASS를 반환하지 않는다.
```


# 11. Reference 적용

각 Worker는 자신의 분석 시작 시:

```text
.claude/references/BE-REFERENCE.md
```

를 직접 1회 읽는다.

Parent가 한 번 읽고
모든 Worker에게 요약 전달하는 것으로 대체하지 않는다.

Reference는:

```text
문서 구조
표현 형식
상세 수준
ASCII Tree
독립 Block
Excel 복사 구조
Oracle Metadata 표현
```

에 적용한다.

Sample은 Evidence로 사용하지 않는다.


# 12. Rule 적용

각 Worker는 직접:

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

를 읽고 적용한다.

특히 Rule 09의:

```text
xargs 금지
Project Root 전체 find 금지
grep -r/-R/-rn 금지
```

를 유지한다.


# 13. Worker 분석

Worker는:

```text
HTTP_METHOD + BACKEND_URL
 ↓
Code Index
 ↓
Controller 후보
 ↓
실제 Controller Source
 ↓
Controller Mapping 확정
 ↓
Service / ServiceImpl
 ↓
Business Logic
 ↓
하위 호출
 ↓
DB / MyBatis / SQL
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
전체 Call Path 검증
 ↓
Oracle Object / Column 수집
 ↓
중복 제거
 ↓
필요한 Metadata만 확인
 ↓
BE-REFERENCE 기반 Rendering
 ↓
OUTPUT_PATH Write
 ↓
실제 생성 파일 검증
```

순서로 처리한다.


# 14. UNKNOWN Method

Method가 UNKNOWN이면
실제 Controller Source에서 해결한다.

동일 URL에 여러 Method가 존재하면
해당 Worker만 STOP 한다.

다른 Worker는 계속한다.


# 15. Worker 독립성

Backend 하나당 Worker 하나이다.

```text
Worker #1 → Backend #1
Worker #2 → Backend #2
Worker #3 → Backend #3
```

Worker끼리 Source Evidence를
확인 없이 공유하지 않는다.


# 16. 병렬 수

동시 Worker 최대:

```text
3
```

Backend가 5개라면:

```text
Worker #1 ─┐
Worker #2 ─┼─ 동시
Worker #3 ─┘

하나 완료
 ↓
Worker #4 시작

하나 완료
 ↓
Worker #5 시작
```

으로 처리한다.


# 17. Worker 실패 격리

하나가 STOP/ERROR여도
다른 Worker는 계속한다.

```text
BE-001 PASS
BE-002 STOP
BE-003 PASS
```

가능하다.


# 18. Oracle Metadata

각 Worker 내부에서:

```text
Call Path 완료
 ↓
Oracle Object / Column 수집
 ↓
중복 제거
 ↓
필요성 판단
 ↓
필요한 Metadata만 Oracle MCP
```

를 수행한다.

DB 호출마다 Metadata를 조회하지 않는다.

Schema 전체 Metadata를 탐색하지 않는다.


# 19. API Document 금지

Direct 분석에서는 다음을 생성하지 않는다.

```text
API-001
FE-ACT-010-create-API-001.md
```

API Document가 없어도 정상 동작해야 한다.


# 20. Worker 결과 Contract

Worker는 Parent에게 다음만 반환한다.

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
...
```

상세 Backend 문서 본문은 반환하지 않는다.


# 21. Parent 완료 검증

Worker가 `PASS`라고 반환했다고
그대로 PASS 처리하지 않는다.

Parent는 각 Worker에 대해
최소한 다음을 다시 확인한다.

```text
EXPECTED OUTPUT 존재 여부

EXPECTED OUTPUT Filename
  =
{FE Base Name}-{BE ID}.md
```

예:

```text
EXPECTED:

docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md
```

실제로 이 파일이 존재해야 한다.


# 22. 잘못된 파일 발견

Worker가 예를 들어:

```text
BE-001.md
```

를 생성했지만 Expected Output:

```text
FE-ACT-010-create-BE-001.md
```

가 없다면 PASS가 아니다.

```text
STATUS:
ERROR

EXPECTED OUTPUT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md

ACTUAL OUTPUT:
docs/analysis/equipment-search/backend/BE-001.md

OUTPUT PATH MATCH:
NO

OUTPUT FILENAME MATCH:
NO
```

잘못 생성된 파일을
정상 결과로 인정하지 않는다.


# 23. Reference 검증

Worker가:

```text
REFERENCE LOADED:
NO
```

이면 문서가 생성됐더라도 PASS로 인정하지 않는다.

```text
STATUS:
ERROR

ERROR:
BE-REFERENCE.md not loaded
```

로 처리한다.


# 24. Parent 최종 정렬

Worker 완료 순서가 아니라
입력 순서로 결과를 표시한다.

```text
BE-001
BE-002
BE-003
...
```


# 25. 최종 Summary

예:

```text
Backend Direct Parallel Analysis 완료

화면명:
equipment-search

FE Base Name:
FE-ACT-010-create


BE-001

STATUS:
PASS

REFERENCE LOADED:
YES

METHOD:
POST

BACKEND URL:
/api/equipment/create

EXPECTED OUTPUT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md

ACTUAL OUTPUT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md

OUTPUT PATH MATCH:
YES


BE-002

STATUS:
PASS

REFERENCE LOADED:
YES

METHOD:
POST

BACKEND URL:
/api/equipment/check

EXPECTED OUTPUT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-002.md

ACTUAL OUTPUT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-002.md

OUTPUT PATH MATCH:
YES
```


# 26. 실행 예제

사용자:

```text
be-direct-parallel agent를 사용해서
다음 Backend를 분석해줘.

화면명:
equipment-search

FE Base Name:
FE-ACT-010-create

Backend:
1. UNKNOWN /api/equipment/create
2. UNKNOWN /api/equipment/check
3. UNKNOWN /api/equipment/history
```

Parent는 Worker 실행 전에:

```text
#1
BE_ID:
BE-001

OUTPUT_PATH:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md


#2
BE_ID:
BE-002

OUTPUT_PATH:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-002.md


#3
BE_ID:
BE-003

OUTPUT_PATH:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-003.md
```

를 확정한다.


# 27. 최종 검증

Parent 종료 전에 확인한다.

```text
[ ] 모든 Worker에 SCREEN_NAME 전달

[ ] 모든 Worker에 FE_BASE_NAME 전달

[ ] BE ID를 입력 순서대로 사전 할당

[ ] Worker 시작 전에 OUTPUT_PATH 확정

[ ] OUTPUT_PATH가
    docs/analysis/{화면명}/backend/
    {FE Base Name}-{BE ID}.md
    형식인지 확인

[ ] Worker가 BE ID를 변경하지 않음

[ ] Worker가 FE Base Name을 변경하지 않음

[ ] Worker가 OUTPUT_PATH를 변경하지 않음

[ ] 각 Worker가 Rule 08 실제 Read

[ ] 각 Worker가 Rule 09 실제 Read

[ ] 각 Worker가 BE-REFERENCE 실제 Read

[ ] Reference Sample을 Evidence로 사용하지 않음

[ ] Method + URL로 Controller 직접 확인

[ ] 실제 Source 기반 Business Logic 분석

[ ] Controller → Response Call Path 완료

[ ] Caller 복귀 보존

[ ] DB / RFC / REST 실제 위치 보존

[ ] Oracle Metadata 후반 검증

[ ] EXPECTED OUTPUT 실제 존재

[ ] EXPECTED FILENAME과 ACTUAL FILENAME 일치

[ ] REFERENCE LOADED = YES

[ ] OUTPUT PATH MATCH = YES

[ ] OUTPUT FILENAME MATCH = YES

[ ] 위 조건 미충족 Worker를 PASS 처리하지 않음
```


# 28. STOP

모든 Worker가:

```text
PASS
STOP
ERROR
```

중 하나로 종료되고,

Parent가:

```text
Reference Load 상태
Expected Output
Actual Output
Filename 일치
```

를 검증한 후 STOP 한다.

자동으로 다른 FE Action이나
다른 Backend를 추가 분석하지 않는다.
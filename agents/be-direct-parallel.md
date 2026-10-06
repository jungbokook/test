---
name: be-direct-parallel
description: 하나의 FE Action에 연결된 여러 Backend URL에 BE ID와 Output File을 입력 순서대로 사전 할당하고, 독립 Worker에서 be-direct-analysis를 병렬 실행한다.
tools: Read, Glob
---

# Backend Direct Parallel Agent

## 1. 목적

하나의 FE Action에 연결된 여러 Backend URL을
API 분석 문서 없이 병렬 분석한다.

각 Backend URL은 독립 Worker가 담당한다.

```text
FE Action
 │
 ├─ Backend #1 → Worker #1
 ├─ Backend #2 → Worker #2
 └─ Backend #3 → Worker #3
```

Backend 간 분석은 병렬로 수행할 수 있다.

각 Worker 내부에서는:

```text
be-direct-analysis
```

Skill을 사용하여
Controller부터 Response까지 실제 실행 순서대로 분석한다.


## 2. 입력

필수 입력:

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
1. POST /api/equipment/create
2. POST /api/equipment/check
3. UNKNOWN /api/equipment/history
```


## 3. 필수 입력 검증

Worker를 시작하기 전에 다음을 확인한다.

```text
화면명 존재
FE Base Name 존재
Backend 목록 존재
각 Backend URL 존재
각 Backend Method 존재
```

FE Base Name이 없으면 추측하지 않는다.

Backend URL에서 FE Base Name을 생성하지 않는다.

입력이 부족하면 Worker를 시작하지 않고 STOP 한다.


## 4. BE Reference

각 Worker는 `be-direct-analysis` Skill을 사용한다.

따라서 각 Worker의 Backend 분석에는:

```text
.claude/references/BE-REFERENCE.md
```

가 적용되어야 한다.

각 Worker는 자신의 Backend 분석 시작 시
BE-REFERENCE가 존재하면 1회 읽는다.

Reference는:

```text
문서 구조
표현 형식
상세 수준
ASCII Tree
Excel Block
Oracle Metadata 표현
```

을 위한 것이다.

Reference Sample은 Evidence가 아니다.

Parent Agent가 Reference 내용을 대신 요약하여
Worker에게 전달하는 방식으로 대체하지 않는다.

각 Worker는 `be-direct-analysis`의
Reference 적용 규칙을 그대로 따른다.


## 5. Rule 적용

각 Worker는 `be-direct-analysis`를 통해:

```text
.claude/rules/08-be-analysis-scope.md
.claude/rules/09-be-call-tracing.md
```

를 적용한다.

Parent Agent가 이 Rule을 생략하거나
더 단순한 자체 Backend 분석 방식으로 대체하지 않는다.


## 6. BE ID 사전 할당

Worker 실행 전에
Backend 입력 순서대로 BE ID를 확정한다.

예:

```text
Backend #1 → BE-001
Backend #2 → BE-002
Backend #3 → BE-003
Backend #4 → BE-004
```

BE ID는 Worker 완료 순서와 관계없다.

다음 방식으로 결정하지 않는다.

```text
Worker 완료 순서
Controller 발견 순서
파일 생성 순서
Oracle 응답 순서
최근 생성된 BE 문서
```

입력 순서만 사용한다.


## 7. Output File 사전 할당

Worker를 시작하기 전에
모든 Output File을 확정한다.

파일명 규칙:

```text
{FE Base Name}-{BE ID}.md
```

예:

```text
FE Base Name:
FE-ACT-010-create
```

이면:

```text
Backend #1
BE-001
FE-ACT-010-create-BE-001.md

Backend #2
BE-002
FE-ACT-010-create-BE-002.md

Backend #3
BE-003
FE-ACT-010-create-BE-003.md
```

전체 경로:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```


## 8. 파일명 고정

Worker에게 전달할 Output File은
Parent Agent가 미리 완성한다.

예:

```text
Output File:
FE-ACT-010-create-BE-001.md
```

Worker는 이 파일명을 변경하지 않는다.

다음 형태로 변경하면 안 된다.

```text
BE-001.md
BE-001-create.md
BE-API-001-create.md
FE-ACT-010-BE-001.md
```

반드시:

```text
FE-ACT-010-create-BE-001.md
```

형식을 유지한다.


## 9. Worker Context

각 Worker에는 다음 값을 명시적으로 전달한다.

```text
화면명
FE Base Name
BE ID
HTTP Method
Backend URL
Output File
```

Worker #1 예:

```text
화면명:
equipment-search

FE Base Name:
FE-ACT-010-create

BE ID:
BE-001

HTTP Method:
POST

Backend URL:
/api/equipment/create

Output File:
FE-ACT-010-create-BE-001.md
```

Worker #2:

```text
화면명:
equipment-search

FE Base Name:
FE-ACT-010-create

BE ID:
BE-002

HTTP Method:
POST

Backend URL:
/api/equipment/check

Output File:
FE-ACT-010-create-BE-002.md
```


## 10. Worker 지시

각 Worker에는 다음 작업을 지시한다.

```text
현재 Worker에 할당된 Backend 하나만 분석한다.

반드시 be-direct-analysis Skill의 규칙을 적용한다.

BE-REFERENCE.md가 존재하면
현재 Worker 분석 시작 시 1회 읽고
문서 구조/표현 형식/상세 수준에 적용한다.

08-be-analysis-scope.md를 적용한다.

09-be-call-tracing.md를 적용한다.

전달받은 FE Base Name을 변경하지 않는다.

전달받은 BE ID를 변경하지 않는다.

전달받은 Output File을 변경하지 않는다.

API Document를 생성하거나 요구하지 않는다.

HTTP Method + Backend URL로
Controller를 직접 탐색한다.

Code Index는 탐색에 사용하고
실제 Business Logic은 Source를 읽고 확정한다.

Controller부터 Response까지
전체 Call Path를 완료한다.

상세 결과는 지정된 Output File에 저장한다.

완료 후 Parent Agent에는
상태와 파일 정보만 반환한다.
```


## 11. Worker 내부 실행 흐름

각 Worker:

```text
Worker 시작
 ↓
be-direct-analysis 규칙 적용
 ↓
08-be-analysis-scope.md 적용
 ↓
09-be-call-tracing.md 적용
 ↓
BE-REFERENCE.md 1회 로드
 ↓
HTTP Method + Backend URL
 ↓
Code Index로 Controller 후보 탐색
 ↓
실제 Controller Source 확인
 ↓
Controller Mapping 확정
 ↓
Service / 구현체
 ↓
Business Logic
 ↓
내부 Method / 다른 Service
 ↓
DB / MyBatis / SQL
 ↓
Caller 복귀
 ↓
RFC / REST / 외부 연동
 ↓
Caller 복귀
 ↓
다음 Business Logic
 ↓
Response
 ↓
전체 Call Path 검증
 ↓
Oracle Object / Column 중복 제거
 ↓
필요한 Metadata만 Oracle MCP 확인
 ↓
BE-REFERENCE 형식으로 문서 작성
 ↓
지정된 Output File 저장
 ↓
Worker STOP
```


## 12. Worker 독립성

각 Backend는 독립 Worker에서 분석한다.

```text
Worker #1
Backend #1

Worker #2
Backend #2

Worker #3
Backend #3
```

Worker 간 상세 분석 결과를 공유하지 않는다.

Worker #1의 Source 분석 결과를
Worker #2가 Evidence 없이 그대로 사용하지 않는다.


## 13. 최대 병렬 Worker

동시에 실행하는 Worker는 최대:

```text
3
```

개로 제한한다.

예:

```text
Backend 5개

처음:
Worker #1
Worker #2
Worker #3

동시 실행 수 = 3
```

하나가 완료되면:

```text
Worker #4
```

를 시작할 수 있다.

또 하나가 완료되면:

```text
Worker #5
```

를 시작할 수 있다.

모든 Worker가 끝날 때까지
항상 동시 실행 수를 최대 3 이하로 유지한다.


## 14. UNKNOWN Method

Worker Method가:

```text
UNKNOWN
```

이면 `be-direct-analysis` 규칙에 따라
실제 Controller Source에서 Method를 확인한다.

예:

```text
Input Method:
UNKNOWN

Backend URL:
/api/equipment/history
```

Source 확인:

```text
@GetMapping("/history")
```

결과:

```text
Resolved Method:
GET
```


## 15. 동일 URL 다중 Method

UNKNOWN Backend URL에
여러 HTTP Method가 실제로 존재하면
Worker가 하나를 임의 선택하지 않는다.

해당 Worker:

```text
STATUS:
STOP
```

다른 Worker는 계속 실행한다.

예:

```text
BE-001 → PASS
BE-002 → STOP
BE-003 → PASS
```


## 16. Worker 실패 격리

하나의 Worker가 실패하거나 STOP되어도
다른 Worker를 중단하지 않는다.

예:

```text
Worker #1 → PASS
Worker #2 → ERROR
Worker #3 → PASS
```

Parent Agent는
각 결과를 독립적으로 수집한다.


## 17. 기존 파일 확인

Worker 시작 전에 지정된 Output File이
이미 존재하는지 확인한다.

예:

```text
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md
```

이미 존재하고
사용자가 덮어쓰기를 요청하지 않았다면:

```text
Worker #1
STATUS:
STOP

ERROR:
Output File already exists
```

다른 Worker는 계속 실행한다.


## 18. 탐색 제한

각 Worker는 현재 Backend 분석에 필요한
Source만 탐색한다.

Code Index를 우선 사용한다.

금지:

```text
xargs

Project Root 전체 대상
find

grep -r
grep -R
grep -rn

JAR 탐색
Decompiled Source 탐색
```

탐색 흐름:

```text
Code Index
 ↓
Controller 후보
 ↓
실제 Source
 ↓
Symbol / Reference
 ↓
필요 Source만 추가 확인
```


## 19. Oracle Metadata

각 Worker는 먼저
Controller부터 Response까지 Call Path를 완료한다.

그 과정에서:

```text
Table
View
Column
```

후보를 수집한다.

Call Path 완료 후:

```text
Oracle Object / Column 수집
 ↓
Worker 내부 중복 제거
 ↓
Metadata 필요성 판단
 ↓
필요한 항목만 Oracle MCP
```

순서로 처리한다.

DB 호출마다 Oracle Metadata를 조회하지 않는다.

Schema 전체 Metadata를 탐색하지 않는다.


## 20. Worker 간 Oracle Metadata

각 Worker는 독립적이다.

따라서 Worker 간 Metadata Cache 공유를
필수 조건으로 만들지 않는다.

중요한 것은 각 Worker 내부에서:

```text
동일 Object 반복 조회 금지
동일 Column 반복 조회 금지
```

이다.


## 21. API 문서 생성 금지

이 Agent는 Direct Backend 분석용이다.

다음 문서를 생성하지 않는다.

```text
FE-ACT-010-create-API-001.md
FE-ACT-010-create-API-002.md
```

API ID를 새로 생성하지 않는다.

Backend 문서만 생성한다.


## 22. Worker 결과

Worker는 상세 Backend 문서 전체를
Parent Agent에게 다시 반환하지 않는다.

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
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md

ERROR:
없음 또는 오류 요약
```


## 23. Parent 결과 정렬

Worker 완료 순서와 관계없이
최종 결과는 사용자 입력 순서대로 정렬한다.

예:

실제 완료:

```text
BE-003
BE-001
BE-002
```

최종 표시:

```text
BE-001
BE-002
BE-003
```


## 24. 최종 Summary

Parent Agent는 모든 Worker 완료 후
상세 분석 내용을 다시 출력하지 않는다.

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

METHOD:
POST

BACKEND URL:
/api/equipment/create

BE DOCUMENT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-001.md


BE-002
STATUS:
PASS

METHOD:
POST

BACKEND URL:
/api/equipment/check

BE DOCUMENT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-002.md


BE-003
STATUS:
PASS

INPUT METHOD:
UNKNOWN

RESOLVED METHOD:
GET

BACKEND URL:
/api/equipment/history

BE DOCUMENT:
docs/analysis/equipment-search/backend/
FE-ACT-010-create-BE-003.md
```


## 25. 실행 예제

사용자 입력:

```text
be-direct-parallel agent를 사용해서
다음 Backend들을 분석해줘.

화면명:
equipment-search

FE Base Name:
FE-ACT-010-create

Backend:
1. POST /api/equipment/create
2. POST /api/equipment/check
3. UNKNOWN /api/equipment/history
```

Parent Agent 사전 할당:

```text
#1

BE ID:
BE-001

Output File:
FE-ACT-010-create-BE-001.md


#2

BE ID:
BE-002

Output File:
FE-ACT-010-create-BE-002.md


#3

BE ID:
BE-003

Output File:
FE-ACT-010-create-BE-003.md
```

병렬 실행:

```text
FE-ACT-010-create
 │
 ├─ Worker #1
 │    ├─ BE-001
 │    ├─ POST /api/equipment/create
 │    └─ FE-ACT-010-create-BE-001.md
 │
 ├─ Worker #2
 │    ├─ BE-002
 │    ├─ POST /api/equipment/check
 │    └─ FE-ACT-010-create-BE-002.md
 │
 └─ Worker #3
      ├─ BE-003
      ├─ UNKNOWN /api/equipment/history
      └─ FE-ACT-010-create-BE-003.md
```


## 26. 최종 검증

Parent Agent 종료 전에 확인한다.

```text
[ ] 화면명이 모든 Worker에 전달됐는가

[ ] FE Base Name이 모든 Worker에 전달됐는가

[ ] BE ID가 입력 순서대로 사전 할당됐는가

[ ] Output File이 Worker 시작 전에 사전 할당됐는가

[ ] Output File이
    {FE Base Name}-{BE ID}.md 형식인가

[ ] Worker가 전달받은 BE ID를 변경하지 않았는가

[ ] Worker가 전달받은 Output File을 변경하지 않았는가

[ ] 각 Worker가 be-direct-analysis 규칙을 적용했는가

[ ] 각 Worker가 08-be-analysis-scope.md를 적용했는가

[ ] 각 Worker가 09-be-call-tracing.md를 적용했는가

[ ] 각 Worker가 BE-REFERENCE.md가 존재하는 경우
    분석 시작 시 1회 읽었는가

[ ] Reference Sample을 Evidence로 사용하지 않았는가

[ ] API Document를 생성하지 않았는가

[ ] Controller를 Method + Backend URL에서
    실제 Source로 확인했는가

[ ] UNKNOWN Method를 실제 Source에서 해결했는가

[ ] Code Index 결과만으로
    Business Logic을 확정하지 않았는가

[ ] Controller부터 Response까지
    전체 Call Path를 완료했는가

[ ] Caller 복귀를 보존했는가

[ ] DB / RFC / REST를 실제 실행 위치에 보존했는가

[ ] Oracle Metadata를 Call Path 완료 후
    필요한 경우에만 확인했는가

[ ] Worker 실패가 다른 Worker를 중단시키지 않았는가

[ ] 최종 결과를 입력 순서대로 정렬했는가
```


## 27. STOP

모든 Worker가:

```text
PASS
STOP
ERROR
```

중 하나의 상태로 종료되고
Parent Agent가 결과를 수집하면 STOP 한다.

자동으로 다른 FE Action을 분석하지 않는다.

자동으로 다른 화면을 분석하지 않는다.

자동으로 새로운 Backend URL을 발견하여
추가 Worker를 만들지 않는다.
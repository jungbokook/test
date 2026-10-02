---
name: fe-parallel-orchestrator
description: 여러 화면명과 Action ID를 입력받아 각 화면/Action 조합별 독립 Worker를 구성하고 기존 FE Analysis를 병렬 실행한다.
tools: Read, Glob
model: inherit
---

# FE Parallel Orchestrator

## 1. 목적

여러 화면의 Action을 한 번에 입력받아
각 `(화면명, Action ID)` 조합을 독립 Worker로 구성하고
기존 FE Analysis를 병렬 실행한다.

기존 FE Analysis 구조:

```text
/fe-analysis <화면명> <Action ID>
```

는 변경하지 않는다.

이 Agent는 기존 FE Analysis를
여러 Worker에서 독립적으로 실행하기 위한
Orchestrator 역할만 수행한다.


## 2. 기본 구조

입력:

```text
화면명 + Action ID
```

Worker 단위:

```text
1 Worker
=
1 화면명
+
1 Action ID
```

예:

```text
equipment-search ACT-001
equipment-search ACT-003
workorder-list ACT-002
```

실행 구조:

```text
Orchestrator
 │
 ├─ WORKER-001
 │    └─ /fe-analysis equipment-search ACT-001
 │
 ├─ WORKER-002
 │    └─ /fe-analysis equipment-search ACT-003
 │
 └─ WORKER-003
      └─ /fe-analysis workorder-list ACT-002
```


## 3. 입력 형식

기본 입력 형식:

```text
TARGETS:
<화면명> <Action ID>
<화면명> <Action ID>
...
```

예:

```text
TARGETS:
equipment-search ACT-001
equipment-search ACT-003
workorder-list ACT-002
tasklist-detail ACT-005
```

병렬 Worker 수를 지정할 수도 있다.

```text
MAX PARALLEL WORKERS:
5

TARGETS:
equipment-search ACT-001
equipment-search ACT-003
workorder-list ACT-002
tasklist-detail ACT-005
pm-sheet ACT-001
```


## 4. MAX PARALLEL WORKERS

동시에 실행할 Worker 수는:

```text
MAX PARALLEL WORKERS
```

값으로 결정한다.

사용자가 값을 지정하지 않으면 기본값:

```text
3
```

을 사용한다.

예:

```text
MAX PARALLEL WORKERS:
5
```

이면 동시에 최대 5개의 FE Worker를 실행한다.

이 값은:

```text
TARGET 순서
Worker ID
화면명
Action ID
FE Document 이름
```

을 변경하지 않는다.

오직 동시 실행 Worker 수와
Batch 구성에만 영향을 준다.


## 5. TARGET 순서

사용자가 입력한 TARGET 순서를 유지한다.

예:

```text
TARGETS:

equipment-search ACT-001
workorder-list ACT-002
pm-sheet ACT-003
```

이면:

```text
WORKER-001
equipment-search ACT-001

WORKER-002
workorder-list ACT-002

WORKER-003
pm-sheet ACT-003
```

으로 배정한다.

Worker 완료 순서는
Worker ID에 영향을 주지 않는다.


## 6. 동일 화면의 여러 Action

같은 화면에서 여러 Action을 입력할 수 있다.

예:

```text
TARGETS:

equipment-search ACT-001
equipment-search ACT-002
equipment-search ACT-005
```

각 Action은 별도 Worker로 실행한다.

```text
WORKER-001
equipment-search ACT-001

WORKER-002
equipment-search ACT-002

WORKER-003
equipment-search ACT-005
```

같은 SCREEN Document를 사용하더라도
각 Worker의 FE Analysis Context는 독립적으로 유지한다.


## 7. 여러 화면

서로 다른 화면도 동시에 입력할 수 있다.

예:

```text
TARGETS:

equipment-search ACT-001
workorder-list ACT-002
tasklist-detail ACT-005
pm-sheet ACT-001
```

각 Worker는 자신에게 배정된
화면만 분석한다.

다른 화면의 SCREEN Document 또는
Action을 분석하지 않는다.


## 8. 중복 TARGET

완전히 동일한:

```text
화면명 + Action ID
```

조합이 여러 번 입력되면
하나의 TARGET으로 정리한다.

예:

```text
equipment-search ACT-001
equipment-search ACT-001
```

결과:

```text
equipment-search ACT-001
```

중복 제거 후 최종 TARGET 순서를 기준으로
Worker ID를 연속 배정한다.


## 9. SCREEN Document

각 Worker는 자신의 화면에 해당하는
SCREEN Document를 사용한다.

기본 경로:

```text
docs/analysis/{화면명}/SCREEN-{화면명}.md
```

예:

```text
화면명:
equipment-search
```

이면:

```text
docs/analysis/equipment-search/SCREEN-equipment-search.md
```

을 사용한다.

다른 화면의 SCREEN Document를
대체 자료로 사용하지 않는다.


## 10. SCREEN Document 확인

Worker 실행 전에 해당 화면의
SCREEN Document 존재 여부를 확인한다.

예:

```text
WORKER-001

SCREEN:
equipment-search

SCREEN DOCUMENT:
docs/analysis/equipment-search/SCREEN-equipment-search.md
```

문서가 존재하지 않으면 해당 Worker는:

```text
STOP
```

한다.

다른 화면의 SCREEN Document를 검색하여
임의 대체하지 않는다.


## 11. Action ID 확인

각 Worker는 SCREEN Document에서
자신에게 배정된 Action ID가 존재하는지 확인한다.

예:

```text
ACT-003
```

이 SCREEN Document에 존재해야 한다.

존재하지 않으면:

```text
STATUS:
STOP

ERROR:
Action ID 확인되지 않음
```

으로 종료한다.

비슷한 Action을 임의로 선택하지 않는다.


## 12. FE Analysis 시작점

SCREEN Document에서 선택된 Action의:

```text
Action ID
Event
Handler
Source Path
```

를 FE Analysis 시작점으로 사용한다.

FE Analysis를 위해 다른 Action으로
분석 범위를 확장하지 않는다.

필요한 호출 관계는 기존 FE Analysis 규칙에 따라
선택된 Action에서 시작하여 추적한다.


## 13. 기존 FE Analysis 사용

각 Worker는 기존 FE 분석 자산을 그대로 사용한다.

```text
.claude/rules/04-fe-analysis-scope.md
.claude/rules/05-fe-call-tracing.md
.claude/skills/fe-analysis/SKILL.md
.claude/references/FE-REFERENCE.md
```

Orchestrator가 별도의 FE 분석 방법을
새로 정의하지 않는다.


## 14. Worker 실행

각 Worker는 기존 FE Analysis와 동일하게:

```text
/fe-analysis <화면명> <Action ID>
```

에 해당하는 분석을 수행한다.

예:

```text
WORKER-001

화면명:
equipment-search

Action ID:
ACT-001
```

이면 기존 FE Analysis 기준으로:

```text
/fe-analysis equipment-search ACT-001
```

에 해당하는 작업을 수행한다.


## 15. FE 분석 범위

FE Analysis의 상세 범위는
기존 FE Rules와 Skill을 따른다.

대표 흐름:

```text
Selected Action
 ↓
Event
 ↓
Handler
 ↓
Input
 ↓
Validation
 ↓
Condition / Branch
 ↓
Parameter Construction
 ↓
Data Transformation
 ↓
State Change
 ↓
Function Call
 ↓
Backend API Call
 ↓
Response Handling
 ↓
Exception Handling
 ↓
FE Document
```

Orchestrator가 이 범위를 임의로 확대하지 않는다.


## 16. Chrome DevTools 사용 금지

FE Analysis에서는
Chrome DevTools를 사용하지 않는다.

SCREEN Runtime 분석은 이미 완료된 것으로 간주한다.

FE Worker는:

```text
SCREEN Document
+
Code Index
+
실제 FE Source
```

를 기준으로 분석한다.


## 17. Backend 분석 금지

FE Worker는 Backend 내부 구현을 분석하지 않는다.

다음 범위로 진입하지 않는다.

```text
Controller 내부 Business Logic
Service
ServiceImpl
Mapper
MyBatis XML
SQL
Oracle
RFC 내부 구현
Backend 내부 호출 흐름
```

FE에서는 Backend API 호출 정보까지만 확인한다.

예:

```text
HTTP Method
Backend URL
Query Parameter
Path Parameter
Request Body
호출 조건
호출 순서
Response 처리
```

까지 분석한다.


## 18. Backend API 여러 개

하나의 FE Action에서
Backend API가 여러 개 호출될 수 있다.

예:

```text
ACT-001
 ↓
FE Business Logic
 ├─ API #1
 ├─ API #2
 └─ API #3
```

이 경우 해당 FE Worker가
모든 실제 Backend API 호출을 기록한다.

API가 여러 개라는 이유로
FE Worker를 다시 분할하지 않는다.


## 19. Source Evidence

FE 분석 결과는 실제 Source를 기준으로 한다.

가능한 경우:

```text
Source:
{Project Root 상대 경로}:{시작 Line}-{종료 Line}
```

을 사용한다.

Line Range를 확인할 수 없으면
임의 생성하지 않는다.

```text
Source:
{Project Root 상대 경로}

Line Range:
확인되지 않음
```

으로 기록한다.

Code Index 검색 결과만으로
Business Logic을 확정하지 않는다.


## 20. FE Document

각 Worker는 기존 FE Skill의
파일명 규칙을 그대로 사용한다.

예:

```text
FE-ACT-001-initial-search.md
FE-ACT-002-search.md
FE-ACT-005-save.md
```

저장 위치:

```text
docs/analysis/{화면명}/frontend/
```

예:

```text
docs/analysis/equipment-search/frontend/
```

Orchestrator가 FE Document 파일명을
임의로 다시 정의하지 않는다.


## 21. 기존 FE Document 보호

동일 화면 + 동일 Action의
FE Document가 이미 존재하는 경우
자동 삭제하거나 덮어쓰지 않는다.

기존 FE Skill의 문서 보호 규칙을 따른다.

Worker 완료 순서를 기준으로
기존 문서를 변경하지 않는다.


## 22. Worker 독립성

각 Worker의 Context는 독립적으로 유지한다.

다음과 같은 공유를 하지 않는다.

```text
WORKER-001 FE 분석 본문
        ↓
WORKER-002 Context
```

또는:

```text
WORKER-002 Source 분석 내용
        ↓
WORKER-003 Context
```

공통으로 사용할 수 있는 것은:

```text
Project Root
공통 Rules
공통 Skill
공통 Reference
```

이다.

분석 결과 Context는 Worker 간 공유하지 않는다.


## 23. 병렬 실행

동일 Batch에 포함된 Worker는 병렬 실행한다.

예:

```text
MAX PARALLEL WORKERS:
3
```

```text
Batch #1

WORKER-001
equipment-search ACT-001

WORKER-002
workorder-list ACT-002

WORKER-003
pm-sheet ACT-003
```

세 Worker는 서로의 완료를 기다리지 않는다.


## 24. Batch 구성

전체 TARGET 수가
MAX PARALLEL WORKERS보다 많으면
Batch로 나눈다.

예:

```text
TARGET COUNT:
7

MAX PARALLEL WORKERS:
3
```

이면:

```text
Batch #1

WORKER-001
WORKER-002
WORKER-003


Batch #2

WORKER-004
WORKER-005
WORKER-006


Batch #3

WORKER-007
```


## 25. Batch 실행

동일 Batch의 Worker는 병렬 실행한다.

서로 다른 Batch는 순차 실행한다.

```text
Batch #1
 │
 ├─ WORKER-001
 ├─ WORKER-002
 └─ WORKER-003
       ↓
모든 Worker 종료
       ↓
Batch #2
 │
 ├─ WORKER-004
 ├─ WORKER-005
 └─ WORKER-006
```

현재 Batch의 모든 Worker가:

```text
PASS
STOP
ERROR
```

중 하나의 상태에 도달한 후
다음 Batch를 실행한다.


## 26. Worker 실패 격리

하나의 Worker가 실패해도
같은 Batch의 다른 Worker 결과를 취소하지 않는다.

예:

```text
WORKER-001 → PASS
WORKER-002 → ERROR
WORKER-003 → PASS
```

이면:

```text
WORKER-001 결과 유지
WORKER-003 결과 유지
```

한다.

WORKER-002 실패 때문에
다른 FE Document를 삭제하지 않는다.


## 27. 다음 Batch 처리

현재 Batch에 실패 Worker가 있더라도
나머지 Worker가 모두 종료되면
다음 Batch를 계속 실행한다.

실패한 Worker를 자동으로 재시도하지 않는다.

Worker ID를 다시 배정하지 않는다.


## 28. 완료 순서

Worker 완료 순서는
입력 순서와 다를 수 있다.

예:

```text
완료 순서:

WORKER-003
WORKER-001
WORKER-002
```

이어도:

```text
WORKER-001
WORKER-002
WORKER-003
```

ID 관계를 그대로 유지한다.


## 29. Context 보호

각 Worker의 전체 FE 분석 결과를
Orchestrator Context로 반환하지 않는다.

상세 결과는 FE Markdown Document에 저장한다.

Worker는 최소 결과만 반환한다.

```text
STATUS
WORKER ID
SCREEN
ACTION ID
FE DOCUMENT
ERROR
```


## 30. Worker 결과 형식

정상 완료:

```text
STATUS:
PASS

WORKER ID:
WORKER-001

SCREEN:
equipment-search

ACTION ID:
ACT-001

FE DOCUMENT:
FE-ACT-001-initial-search.md

ERROR:
없음
```

실패:

```text
STATUS:
ERROR

WORKER ID:
WORKER-002

SCREEN:
workorder-list

ACTION ID:
ACT-002

FE DOCUMENT:
생성되지 않음

ERROR:
FE Analysis 실패
```


## 31. Tool 사용 제한

Orchestrator는 FE Source Business Logic을
직접 분석하지 않는다.

Orchestrator가 TARGET 및 SCREEN Document를
확인할 때는:

```text
Read
Glob
```

을 사용한다.

다음과 같은 Shell 기반 재귀 검색으로
Read / Glob 제한을 우회하지 않는다.

```text
grep
find
xargs
rg
sed
awk
cat
PowerShell 기반 재귀 파일 검색
```

실제 FE Source 탐색은
각 FE Worker가 기존 FE Rules와 Skill에 따라 수행한다.


## 32. Sample 제외 정책

기존 프로젝트에서:

```text
/sample/**
```

또는 Sample Source가 분석 제외 대상으로
설정되어 있다면 해당 정책을 그대로 따른다.

Orchestrator와 Worker 모두
Sample 제외 정책을 우회하지 않는다.


## 33. 전체 실행 상태

모든 Worker가 PASS이면:

```text
STATUS:
PASS
```

하나 이상의 Worker가:

```text
STOP
ERROR
```

이면:

```text
STATUS:
PARTIAL
```

로 표시한다.

어떤 Worker가 실패했는지
Worker ID와 화면명/Action ID로 표시한다.


## 34. 최종 출력

Orchestrator는 FE 분석 상세 내용을
다시 출력하지 않는다.

예:

```text
FE PARALLEL RESULT

STATUS:
PASS

INPUT TARGET COUNT:
5

UNIQUE TARGET COUNT:
5

MAX PARALLEL WORKERS:
3

BATCH COUNT:
2


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BATCH #1

WORKER-001

STATUS:
PASS

SCREEN:
equipment-search

ACTION ID:
ACT-001

FE DOCUMENT:
FE-ACT-001-initial-search.md

ERROR:
없음


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WORKER-002

STATUS:
PASS

SCREEN:
equipment-search

ACTION ID:
ACT-003

FE DOCUMENT:
FE-ACT-003-search.md

ERROR:
없음


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WORKER-003

STATUS:
PASS

SCREEN:
workorder-list

ACTION ID:
ACT-002

FE DOCUMENT:
FE-ACT-002-search.md

ERROR:
없음


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BATCH #2

WORKER-004

STATUS:
PASS

SCREEN:
tasklist-detail

ACTION ID:
ACT-005

FE DOCUMENT:
FE-ACT-005-save.md

ERROR:
없음


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WORKER-005

STATUS:
PASS

SCREEN:
pm-sheet

ACTION ID:
ACT-001

FE DOCUMENT:
FE-ACT-001-initial-search.md

ERROR:
없음
```


## 35. 금지 사항

다음은 금지한다.

```text
하나의 Worker에서 여러 TARGET 분석

다른 화면의 SCREEN Document 사용

다른 Action으로 임의 변경

Worker 간 FE 분석 Context 공유

FE Worker에서 Chrome DevTools 사용

FE Worker에서 Backend 내부 분석

Orchestrator가 직접 FE Business Logic 분석

Worker 완료 순서 기준 Worker ID 재배정

기존 FE Document 자동 삭제

기존 FE Document 자동 덮어쓰기

Worker 전체 FE 분석 내용을 Orchestrator에 반환

Shell 기반 재귀 검색으로 Read / Glob 제한 우회
```


## 36. PASS 조건

다음을 확인한다.

```text
TARGET 입력 정상 확인

화면명 정상 확인

Action ID 정상 확인

동일 TARGET 중복 제거

입력 순서 유지

TARGET 하나당 Worker 하나

Worker ID 사전 배정

MAX PARALLEL WORKERS 정상 결정

사용자 지정값이 없으면 기본값 3

SCREEN Document 정상 확인

선택 Action 정상 확인

기존 FE Rules / Skill / Reference 사용

동일 Batch Worker 병렬 실행

Worker Context 독립

Worker 완료 순서에 따른 ID 변경 없음

한 Worker 실패가 다른 Worker 결과에 영향 없음

FE Document 정상 저장

기존 FE Document 자동 덮어쓰기 없음

Worker 최소 결과만 반환

Batch 완료 후 다음 Batch 실행
```


## 37. STOP

모든 Batch의 Worker가:

```text
PASS
STOP
ERROR
```

중 하나의 종료 상태에 도달하면
Worker별 최소 결과를 취합한다.

최종 구조:

```text
TARGETS
 ↓
중복 제거
 ↓
Worker ID 사전 배정
 ↓
MAX PARALLEL WORKERS 결정
 ↓
Batch 구성
 ↓

┌────────────────────────────────────────────┐
│ Batch #1                                   │
│                                            │
│ WORKER-001  WORKER-002  WORKER-003        │
│     │           │           │              │
│ 화면A/ACT-1   화면B/ACT-2   화면C/ACT-3    │
│     ↓           ↓           ↓              │
│ FE Analysis   FE Analysis   FE Analysis    │
│     ↓           ↓           ↓              │
│ FE Document   FE Document   FE Document    │
└────────────────────────────────────────────┘

 ↓
Batch 완료
 ↓
다음 Batch
 ↓
모든 Worker 결과 취합
 ↓
PASS 또는 PARTIAL
 ↓
STOP
```

FE 완료 후 API Analysis 또는
BE Analysis를 자동으로 시작하지 않는다.

API/BE 처리는 별도의:

```text
api-be-parallel-orchestrator
```

에서 수행한다.

다른 화면이나 Action을 임의로 추가 분석하지 않는다.
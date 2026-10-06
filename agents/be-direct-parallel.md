---
name: be-direct-parallel
description: 하나의 FE Action에 연결된 여러 Backend URL을 독립 Worker로 병렬 분석하여 API 문서 없이 FE Action 기준 Backend 문서를 생성한다.
tools: Read, Glob
---

# Backend Direct Parallel Agent

## 1. 목적

하나의 FE Action에서 호출되는
여러 Backend URL을 병렬로 분석한다.

API 분석 문서는 필요하지 않다.

각 Backend URL마다 독립 Worker를 생성하고
각 Worker는 `be-direct-analysis` Skill의 규칙을 따라
Backend Business Logic을 분석한다.

핵심 구조:

```text
FE Action
   │
   ├─ Backend #1 → Worker #1
   │
   ├─ Backend #2 → Worker #2
   │
   └─ Backend #3 → Worker #3
```

Worker 내부 분석은 순차적으로 수행한다.

```text
Controller
 ↓
Service
 ↓
Business Logic
 ↓
DB
 ↓
RFC
 ↓
DB
 ↓
REST
 ↓
Response
```

즉:

```text
Backend 간 분석 = 병렬

Worker 내부 실행 흐름 분석 = 실제 Source 순서
```

---

# 2. 입력 형식

입력은 다음 형식을 사용한다.

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
equipment

FE Base Name:
FE-ACT-010-create

Backend:
1. POST /api/equipment/create
2. POST /api/equipment/check
3. UNKNOWN /api/equipment/history
```

---

# 3. FE Base Name 필수

`FE Base Name`은 필수 입력이다.

예:

```text
FE-ACT-010-create
```

Agent는 Backend URL을 이용하여
FE Base Name을 추정하지 않는다.

다음과 같이 임의 생성하면 안 된다.

```text
/api/equipment/create
→ FE-ACT-001-create
```

금지한다.

FE Base Name이 없는 경우:

```text
STOP

FE Base Name이 필요합니다.

예:
FE-ACT-010-create
```

---

# 4. FE Base Name 역할

FE Base Name은 모든 Backend 문서의
부모 Action을 나타낸다.

예:

```text
FE-ACT-010-create
```

Backend가 3개라면:

```text
FE-ACT-010-create
 ├─ BE-001
 ├─ BE-002
 └─ BE-003
```

파일:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
FE-ACT-010-create-BE-003.md
```

---

# 5. BE ID 사전 할당

병렬 Worker를 시작하기 전에
입력 순서대로 BE ID를 확정한다.

예:

```text
1 → BE-001
2 → BE-002
3 → BE-003
4 → BE-004
```

Worker 완료 순서로 번호를 정하지 않는다.

잘못된 예:

```text
Worker #3이 먼저 끝남
→ BE-001
```

금지한다.

항상 입력 순서 기준이다.

---

# 6. Output File 사전 할당

Worker를 시작하기 전에
각 Worker의 Output File도 확정한다.

규칙:

```text
{FE Base Name}-{BE ID}.md
```

예:

```text
FE Base Name:
FE-ACT-010-create
```

결과:

```text
BE-001
→ FE-ACT-010-create-BE-001.md

BE-002
→ FE-ACT-010-create-BE-002.md

BE-003
→ FE-ACT-010-create-BE-003.md
```

---

# 7. 전체 Output Path

출력 디렉터리:

```text
docs/analysis/{화면명}/backend/
```

전체 파일:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

예:

```text
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
docs/analysis/equipment/backend/FE-ACT-010-create-BE-002.md
docs/analysis/equipment/backend/FE-ACT-010-create-BE-003.md
```

---

# 8. 잘못된 파일명

다음 파일명은 생성하지 않는다.

```text
BE-001.md
BE-002.md

BE-001-create.md

FE-ACT-010-BE-001.md

create-BE-001.md
```

반드시:

```text
FE-ACT-010-create-BE-001.md
```

형식을 사용한다.

---

# 9. Worker 생성

각 Backend마다 독립 Worker를 생성한다.

예:

```text
Backend #1
POST /api/equipment/create

Backend #2
POST /api/equipment/check

Backend #3
UNKNOWN /api/equipment/history
```

Worker:

```text
Worker #1
Worker #2
Worker #3
```

---

# 10. Worker Context

각 Worker에는 필요한 정보만 전달한다.

Worker #1 예:

```text
Screen Name:
equipment

FE Base Name:
FE-ACT-010-create

BE ID:
BE-001

HTTP Method:
POST

Backend URL:
/api/equipment/create

Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
```

Worker #2:

```text
Screen Name:
equipment

FE Base Name:
FE-ACT-010-create

BE ID:
BE-002

HTTP Method:
POST

Backend URL:
/api/equipment/check

Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-002.md
```

Worker #3:

```text
Screen Name:
equipment

FE Base Name:
FE-ACT-010-create

BE ID:
BE-003

HTTP Method:
UNKNOWN

Backend URL:
/api/equipment/history

Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-003.md
```

---

# 11. Worker 실행 규칙

각 Worker는:

```text
be-direct-analysis
```

Skill의 규칙을 따라 분석한다.

Worker는 전달받은 다음 값을 변경하면 안 된다.

```text
FE Base Name
BE ID
Output File
```

즉:

```text
Agent
 ↓
ID 확정
 ↓
Output File 확정
 ↓
Worker
 ↓
be-direct-analysis
```

구조이다.

---

# 12. Worker 독립성

각 Worker는 완전히 독립적으로 분석한다.

Worker #1의 분석 결과를
Worker #2가 기다릴 필요가 없다.

예:

```text
Worker #1 ───────────────→ 완료
Worker #2 ───────→ 완료
Worker #3 ─────────────────────→ 완료
```

완료 순서는 결과 ID에 영향을 주지 않는다.

---

# 13. 최대 병렬 수

동시에 실행하는 Worker는 최대:

```text
3
```

개로 제한한다.

Backend가 3개 이하:

```text
모두 동시에 실행
```

Backend가 3개 초과:

```text
최대 3개 실행
 ↓
Worker 완료
 ↓
다음 Worker 시작
```

예:

```text
Backend 7개

Batch #1
 ├─ Worker #1
 ├─ Worker #2
 └─ Worker #3

Batch #2
 ├─ Worker #4
 ├─ Worker #5
 └─ Worker #6

Batch #3
 └─ Worker #7
```

단, Worker 완료 즉시 다음 대기 Worker를 시작할 수 있는
실행 환경이라면 반드시 고정 Batch 완료를 기다릴 필요는 없다.

항상 동시에 실행 중인 Worker 수만 3 이하로 유지한다.

---

# 14. Worker 내부 분석

각 Worker는 해당 Backend 하나만 분석한다.

예:

```text
Worker #1

POST /api/equipment/create
 ↓
Controller
 ↓
Service
 ↓
Validation
 ↓
DB #1
 ↓
조건
 ↓
RFC #1
 ↓
DB #2
 ↓
Response
```

다른 Backend URL까지 확장하지 않는다.

단, 해당 Backend의 실제 Business Logic에서
내부적으로 다른 Service/RFC/REST/DB를 호출하는 경우는
동일 Worker의 실행 흐름으로 계속 추적한다.

---

# 15. Backend 간 병렬 / 내부 순차

다음 구조를 유지한다.

```text
                  FE-ACT-010-create
                         │
        ┌────────────────┼────────────────┐
        │                │                │
     BE-001           BE-002           BE-003
        │                │                │
    Worker #1         Worker #2         Worker #3
        │                │                │
   Controller        Controller        Controller
        ↓                ↓                ↓
     Service           Service           Service
        ↓                ↓                ↓
      DB #1            RFC #1            DB #1
        ↓                ↓                ↓
      RFC #1            DB #1            REST #1
        ↓                ↓                ↓
      DB #2           Response          Response
        ↓
    Response
```

---

# 16. Method UNKNOWN

Worker 입력 Method가:

```text
UNKNOWN
```

인 경우 Worker가 실제 Controller Source에서
HTTP Method를 확인한다.

예:

```text
Input:
UNKNOWN /api/equipment/history
```

Source:

```java
@GetMapping("/history")
```

결과:

```text
Input Method    : UNKNOWN
Resolved Method : GET
```

---

# 17. 동일 URL 다중 Method

UNKNOWN URL에 여러 Method가 존재하면
해당 Worker는 임의 선택하지 않는다.

예:

```text
GET  /api/equipment/history
POST /api/equipment/history
```

해당 Worker 상태:

```text
STOPPED
```

다른 Worker는 계속 실행한다.

즉:

```text
Worker #1 → SUCCESS
Worker #2 → STOPPED
Worker #3 → SUCCESS
```

Worker #2 때문에
전체 병렬 작업을 중단하지 않는다.

---

# 18. Worker 실패 격리

하나의 Worker 실패가
다른 Worker 분석을 중단시키면 안 된다.

예:

```text
Worker #1 → SUCCESS
Worker #2 → FAILED
Worker #3 → SUCCESS
```

최종 결과:

```text
BE-001 → SUCCESS
BE-002 → FAILED
BE-003 → SUCCESS
```

실패 Worker의 원인을 최종 Summary에 기록한다.

---

# 19. 기존 파일 보호

Worker 실행 전에
Output File 존재 여부를 확인한다.

예:

```text
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
```

이미 존재하고
사용자가 덮어쓰기를 명시하지 않은 경우:

```text
Worker #1 → STOPPED
Reason: Output File already exists
```

다른 Worker는 계속 진행한다.

---

# 20. Worker 탐색 범위

각 Worker는 자신의 Backend 분석에 필요한
Source만 탐색한다.

금지:

```text
프로젝트 전체 find

프로젝트 전체 grep -r

프로젝트 전체 grep -R

프로젝트 전체 grep -rn

xargs
```

탐색 순서:

```text
Code Index
 ↓
Controller 후보
 ↓
실제 Source
 ↓
호출 Symbol
 ↓
필요 Source만 추가 확인
```

---

# 21. Worker 간 Source 공유

Worker 간 분석 결과를
의존성으로 사용하지 않는다.

예:

```text
Worker #1이 찾은 Service를
Worker #2가 Evidence 없이 그대로 사용
```

금지한다.

각 Worker는 자신의 Source Evidence를
독립적으로 확인한다.

---

# 22. Oracle Metadata 처리

Oracle Metadata는
각 Worker 내부에서 필요한 경우에만 조회한다.

순서:

```text
Worker
 ↓
Controller → Response 추적
 ↓
MyBatis / SQL 확인
 ↓
Oracle 대상 수집
 ↓
Worker 내부 중복 제거
 ↓
필요한 Metadata만 조회
```

---

# 23. Oracle Metadata 중복 제거

Worker 내부에서는
동일 Object를 반복 조회하지 않는다.

예:

```text
Worker #1

DB #1 → TB_EQUIPMENT
DB #2 → TB_EQUIPMENT
DB #3 → TB_EQUIPMENT
```

Oracle Metadata:

```text
TB_EQUIPMENT → 1회
```

---

# 24. Worker 간 Oracle Metadata

Worker는 서로 독립적이므로
Oracle Metadata Cache 공유를 필수로 요구하지 않는다.

즉:

```text
Worker #1 → TB_EQUIPMENT 조회
Worker #2 → TB_EQUIPMENT 조회
```

가 발생할 수 있다.

병렬 Worker 간 강제 공유 상태를 만들지 않는다.

각 Worker 내부 중복 제거를 우선한다.

---

# 25. Oracle 탐색 제한

금지:

```text
Schema 전체 조회
전체 Table 조회
전체 Column 조회
DB 호출마다 Metadata 조회
Source 분석 전에 Metadata 조회
```

Oracle MCP가 느리거나 응답하지 않는 경우:

```text
Oracle Metadata: 확인되지 않음
```

으로 기록하고 가능한 분석을 계속한다.

---

# 26. Worker 결과 저장

Worker는 상세 분석 결과를
부모 Agent 응답으로 전부 반환하지 않는다.

상세 결과는 지정된 Output File에 저장한다.

Worker가 부모 Agent에 반환할 정보:

```text
Status
FE Base Name
BE ID
HTTP Method
Resolved Method
Backend URL
Output File
Error
```

---

# 27. Worker Return 예

성공:

```text
Status: SUCCESS
FE Base Name: FE-ACT-010-create
BE ID: BE-001
HTTP Method: POST
Resolved Method: POST
Backend URL: /api/equipment/create
Output File: docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md
Error: NONE
```

UNKNOWN 해결:

```text
Status: SUCCESS
FE Base Name: FE-ACT-010-create
BE ID: BE-003
HTTP Method: UNKNOWN
Resolved Method: GET
Backend URL: /api/equipment/history
Output File: docs/analysis/equipment/backend/FE-ACT-010-create-BE-003.md
Error: NONE
```

실패:

```text
Status: FAILED
FE Base Name: FE-ACT-010-create
BE ID: BE-002
HTTP Method: POST
Resolved Method: POST
Backend URL: /api/equipment/check
Output File: docs/analysis/equipment/backend/FE-ACT-010-create-BE-002.md
Error: Controller Mapping 확인 실패
```

---

# 28. 부모 Agent 결과 수집

모든 Worker가 완료되면
입력 순서 기준으로 결과를 정렬한다.

Worker 완료 순서로 정렬하지 않는다.

예:

```text
실제 완료 순서:

BE-003
BE-001
BE-002
```

최종 출력 순서:

```text
BE-001
BE-002
BE-003
```

---

# 29. 최종 Summary

최종 응답은 간단하게 출력한다.

예:

```text
Backend Direct Parallel Analysis 완료

화면명:
equipment

FE Base Name:
FE-ACT-010-create

결과:

BE-001
- Status: SUCCESS
- Method: POST
- URL: /api/equipment/create
- Output:
  docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md

BE-002
- Status: SUCCESS
- Method: POST
- URL: /api/equipment/check
- Output:
  docs/analysis/equipment/backend/FE-ACT-010-create-BE-002.md

BE-003
- Status: SUCCESS
- Input Method: UNKNOWN
- Resolved Method: GET
- URL: /api/equipment/history
- Output:
  docs/analysis/equipment/backend/FE-ACT-010-create-BE-003.md
```

상세 Backend 분석 내용을
부모 Agent 응답에 다시 출력하지 않는다.

---

# 30. 실행 예제

사용자:

```text
be-direct-parallel agent를 사용해서 아래 Backend를 분석해줘.

화면명:
equipment

FE Base Name:
FE-ACT-010-create

Backend:
1. POST /api/equipment/create
2. POST /api/equipment/check
3. UNKNOWN /api/equipment/history
```

Agent 사전 할당:

```text
Worker #1
BE ID:
BE-001

Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-001.md


Worker #2
BE ID:
BE-002

Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-002.md


Worker #3
BE ID:
BE-003

Output File:
docs/analysis/equipment/backend/FE-ACT-010-create-BE-003.md
```

병렬 실행:

```text
                    FE-ACT-010-create
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
     Worker #1          Worker #2          Worker #3
      BE-001             BE-002             BE-003
        │                  │                  │
       분석                분석                분석
        │                  │                  │
        ↓                  ↓                  ↓
FE-ACT-010-create-   FE-ACT-010-create-   FE-ACT-010-create-
BE-001.md            BE-002.md            BE-003.md
```

---

# 31. API 문서 생성 금지

이 Agent의 목적은
API 분석 단계를 건너뛰고 Backend를 직접 분석하는 것이다.

따라서 다음 파일을 생성하지 않는다.

```text
FE-ACT-010-create-API-001.md
FE-ACT-010-create-API-002.md
```

생성 대상은 Backend 문서뿐이다.

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
```

---

# 32. ID 안정성

BE ID는 다음 요소에 영향을 받지 않는다.

```text
Worker 시작 순서
Worker 완료 순서
Controller 발견 순서
Oracle 응답 순서
파일 작성 완료 순서
```

오직:

```text
사용자 Backend 입력 순서
```

로 결정한다.

---

# 33. 분석 완료 조건

Agent 완료 조건:

```text
모든 Backend에 BE ID 할당 완료

모든 Output File 사전 결정 완료

모든 Worker 실행 완료 또는 STOP/FAILED 상태 확정

SUCCESS Worker 문서 저장 완료

최종 결과 입력 순서로 정렬 완료
```

---

# 34. 핵심 원칙

전체 구조:

```text
사용자
 ↓
화면명
FE Base Name
Backend 목록
 ↓
BE ID 사전 할당
 ↓
Output File 사전 할당
 ↓
최대 3개 Worker 병렬 실행
 ↓
각 Worker
  └─ be-direct-analysis
       ↓
      Controller
       ↓
      Service
       ↓
      Business Logic
       ↓
      DB / RFC / REST / External
       ↓
      Response
       ↓
      Backend MD 저장
 ↓
모든 Worker 결과 수집
 ↓
입력 순서 정렬
 ↓
Summary
```

가장 중요한 파일명 규칙:

```text
{FE Base Name}-{BE ID}.md
```

예:

```text
FE-ACT-010-create-BE-001.md
```

이 규칙을 변경하거나
Backend URL을 기반으로 파일명을 다시 생성하지 않는다.
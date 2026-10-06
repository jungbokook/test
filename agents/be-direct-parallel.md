---
name: be-direct-parallel
description: 여러 Backend URL을 독립 Worker로 병렬 분석하여 API 문서 없이 각각의 Backend Business Logic 문서를 생성한다.
tools: Read, Glob
---

# Backend Direct Parallel Agent

## 1. 목적

여러 Backend URL을 입력받아
각 Backend URL을 독립적인 Worker에 할당하고
Backend Direct Analysis를 병렬 실행한다.

각 Worker는 다음 Skill의 분석 규칙을 따른다.

```text
.claude/skills/be-direct-analysis/SKILL.md
```

기본 구조:

```text
Backend URL List
        │
        ├─ Worker #1
        │    ↓
        │   BE-001
        │    ↓
        │   Backend URL #1
        │    ↓
        │   be-direct-analysis
        │
        ├─ Worker #2
        │    ↓
        │   BE-002
        │    ↓
        │   Backend URL #2
        │    ↓
        │   be-direct-analysis
        │
        └─ Worker #3
             ↓
            BE-003
             ↓
            Backend URL #3
             ↓
            be-direct-analysis
```

Worker 간 Backend 분석은 독립적으로 수행한다.

---

# 2. 입력

사용자로부터 다음 정보를 받는다.

```text
화면명

Backend 목록
- HTTP Method | Backend URL
- HTTP Method | Backend URL
- HTTP Method | Backend URL
...
```

예:

```text
화면명:
equipment-search

Backend:
1. POST /api/equipment/search
2. POST /api/equipment/detail
3. UNKNOWN /api/equipment/status
4. GET /api/plant/list
```

HTTP Method는 다음 값을 허용한다.

```text
GET
POST
PUT
PATCH
DELETE
UNKNOWN
```

---

# 3. API 문서 사용 금지

이 Agent는 API 분석 문서를 필요로 하지 않는다.

다음 흐름을 사용하지 않는다.

```text
Backend URL
 ↓
API Analysis
 ↓
API Document
 ↓
BE Analysis
```

대신:

```text
Backend URL
 ↓
Controller 탐색
 ↓
BE Direct Analysis
```

를 사용한다.

각 Worker는 API 문서를 생성하지 않는다.

기존 API 문서가 존재하더라도
Direct Backend 분석의 필수 시작점으로 사용하지 않는다.

---

# 4. 입력 순서 고정

Backend 목록의 입력 순서를
분석 대상의 고정 순서로 사용한다.

예:

```text
입력

1. POST /api/a
2. POST /api/b
3. GET  /api/c
```

사전 할당:

```text
Backend #1
  BE-001
  POST /api/a

Backend #2
  BE-002
  POST /api/b

Backend #3
  BE-003
  GET /api/c
```

Worker 실행 순서 또는 완료 순서와 관계없이
이 번호는 변경하지 않는다.

---

# 5. BE ID 사전 할당

Worker를 시작하기 전에
모든 Backend 대상에 BE ID를 미리 할당한다.

형식:

```text
BE-001
BE-002
BE-003
...
```

예:

```text
Input #1 → BE-001
Input #2 → BE-002
Input #3 → BE-003
Input #4 → BE-004
```

다음 기준으로 BE ID를 변경하지 않는다.

```text
Worker 시작 순서
Worker 완료 순서
Controller 발견 순서
파일 생성 순서
최근 수정 시간
기존 파일의 최대 ID
다른 Worker의 결과
```

병렬 실행 중에도 BE ID는 고정이다.

---

# 6. Output File 사전 할당

Worker 시작 전에
각 Worker의 Output File도 고정한다.

기본 형식:

```text
BE-{nnn}-{기능명}.md
```

하지만 Controller 분석 전에는
기능명을 정확히 알 수 없을 수 있다.

따라서 병렬 Direct Analysis에서는
충돌 방지를 위해 기본적으로 다음 형식을 사용한다.

```text
BE-{nnn}.md
```

예:

```text
BE-001.md
BE-002.md
BE-003.md
```

저장 위치:

```text
docs/analysis/{화면명}/backend/
```

예:

```text
docs/analysis/equipment-search/backend/BE-001.md
docs/analysis/equipment-search/backend/BE-002.md
docs/analysis/equipment-search/backend/BE-003.md
```

Worker가 기능명을 발견하더라도
사전 할당된 Output File을 임의로 변경하지 않는다.

문서 내부에는 실제 기능명을 기록한다.

---

# 7. Worker 독립성

각 Worker는 Backend URL 하나만 분석한다.

예:

```text
Worker #1

화면명
  equipment-search

HTTP Method
  POST

Backend URL
  /api/equipment/search

BE ID
  BE-001

Output File
  BE-001.md
```

Worker #1은 다른 Worker의:

```text
Backend URL
Controller
Service
Mapper
SQL
Oracle Metadata
RFC
REST
BE ID
Output File
진행 상태
```

를 분석에 사용하지 않는다.

---

# 8. Worker 실행

각 Worker는 다음 Skill을 기준으로 분석한다.

```text
be-direct-analysis
```

논리적인 실행 입력:

```text
화면명
HTTP Method
Backend URL
BE ID
Output File
```

예:

```text
화면명:
equipment-search

HTTP Method:
POST

Backend URL:
/api/equipment/search

BE ID:
BE-001

Output File:
BE-001.md
```

Worker는 전달받은:

```text
BE ID
Output File
```

을 그대로 사용한다.

---

# 9. Worker 분석 흐름

각 Worker 내부 흐름:

```text
HTTP Method + Backend URL
 ↓
Controller Mapping 탐색
 ↓
Controller 확정
 ↓
Service
 ↓
ServiceImpl
 ↓
Business Logic
 ↓
Internal Method / Other Service
 ↓
Mapper
 ↓
MyBatis
 ↓
SQL
 ↓
Caller 복귀
 ↓
다음 Business Logic
 ↓
RFC / REST / 기타 외부 연동
 ↓
Caller 복귀
 ↓
Response
 ↓
전체 Call Path 완료
 ↓
Oracle Metadata 후보 수집
 ↓
중복 제거
 ↓
필요한 Metadata만 Oracle MCP 확인
 ↓
BE Document
 ↓
STOP
```

API Analysis 단계는 실행하지 않는다.

---

# 10. HTTP Method UNKNOWN

Worker 입력이:

```text
UNKNOWN
```

이면 `be-direct-analysis` 규칙에 따라
Controller Source에서 Method를 확인한다.

예:

```text
UNKNOWN /api/equipment/status
```

Source에서 하나만 확인:

```text
@GetMapping("/status")
```

이면:

```text
GET
```

으로 확정하고 계속한다.

---

# 11. UNKNOWN + 동일 URL 다중 Method

예:

```text
UNKNOWN /api/equipment
```

Source:

```text
GET    /api/equipment
POST   /api/equipment
DELETE /api/equipment
```

하나로 확정할 수 없다면
해당 Worker만 STOP 한다.

다른 Worker는 계속 실행한다.

예:

```text
Worker #1
  PASS

Worker #2
  STOP
  동일 URL 다중 Method

Worker #3
  PASS
```

Worker 하나의 STOP 때문에
전체 병렬 분석을 중단하지 않는다.

---

# 12. Worker 실패 격리

각 Worker는 독립적으로 처리한다.

한 Worker가:

```text
STOP
ERROR
Controller 미발견
Method 모호
Source 미확인
```

상태가 되더라도
다른 Worker를 중단하지 않는다.

예:

```text
Worker #1 ─ PASS
Worker #2 ─ ERROR
Worker #3 ─ PASS
Worker #4 ─ PASS
```

전체 Agent는 모든 Worker 결과를 수집한다.

---

# 13. 최대 병렬 Worker

기본 최대 병렬 Worker 수:

```text
MAX PARALLEL WORKERS = 3
```

Backend 대상이 3개 이하이면
가능한 Worker를 동시에 실행한다.

예:

```text
Backend 3개

Worker #1 ─┐
Worker #2 ─┼─ Parallel
Worker #3 ─┘
```

Backend 대상이 3개보다 많으면
최대 3개씩 처리한다.

예:

```text
Backend 7개

Batch #1
  Worker #1
  Worker #2
  Worker #3

완료 후

Batch #2
  Worker #4
  Worker #5
  Worker #6

완료 후

Batch #3
  Worker #7
```

MAX PARALLEL WORKERS를 초과하여
동시에 Worker를 생성하지 않는다.

---

# 14. Worker Context 최소화

각 Worker에는
자신의 분석에 필요한 정보만 전달한다.

전달:

```text
화면명
HTTP Method
Backend URL
BE ID
Output File
be-direct-analysis Skill
필요한 Rule / Reference
```

전달하지 않음:

```text
다른 Backend URL의 분석 결과
다른 Worker의 Source 내용
다른 Worker의 SQL
다른 Worker의 Oracle Metadata
다른 Worker의 RFC 결과
다른 Worker의 전체 문서
```

Worker Context를 불필요하게 공유하지 않는다.

---

# 15. Code Index 사용

각 Worker는 Source 탐색 시
Code Index를 우선 사용한다.

```text
Backend URL
 ↓
Code Index
 ↓
관련 Backend Project
 ↓
Controller 후보
 ↓
실제 Source 확인
```

모든 Worker가 Project Root 전체를
Shell 재귀 검색하지 않는다.

`xargs`를 사용하지 않는다.

---

# 16. Worker Source 범위

각 Worker는 자신의 Backend URL과
실제 Call Path에 연결되는 Source만 분석한다.

예:

```text
Worker #1
POST /api/equipment/search

분석 가능
  Controller
  Service
  Common Service
  Mapper
  MyBatis
  SQL
  RFC
  REST
  Response

단,
현재 URL Call Path와 실제 연결되는 범위
```

관련 없는 프로젝트나 기능으로
자동 확장하지 않는다.

---

# 17. Oracle Metadata 병렬 처리 원칙

각 Worker의 Oracle Metadata 분석은
독립적으로 수행한다.

Worker 내부에서는:

```text
전체 Call Path 추적
 ↓
사용 Object / Column 수집
 ↓
중복 제거
 ↓
필요성 판단
 ↓
필요한 Metadata만 Oracle MCP 확인
```

순서를 사용한다.

DB 호출마다 Oracle MCP를 호출하지 않는다.

---

# 18. Oracle Metadata 중복 제거 범위

Metadata 중복 제거 범위는:

```text
현재 Worker
```

이다.

예:

```text
Worker #1

DB #1 → TB_EQUIPMENT
DB #2 → TB_EQUIPMENT
DB #3 → TB_PLANT

Metadata 후보

TB_EQUIPMENT
TB_PLANT
```

`TB_EQUIPMENT`를 Worker #1 내부에서
반복 조회하지 않는다.

---

# 19. Worker 간 Metadata 공유 금지

Worker 간에는
Oracle Metadata Cache를 공유하지 않는다.

예:

```text
Worker #1
  TB_EQUIPMENT Metadata 확인

Worker #2
  TB_EQUIPMENT 사용
```

Worker #2가 Metadata 확인이 필요하다면
Worker #2 자신의 분석 Context에서 확인한다.

다음과 같은 공유 상태를 만들지 않는다.

```text
Global Metadata Cache
Shared Mutable Metadata File
Worker 간 임시 Metadata 상태 공유
```

병렬 Worker 간 분석 결과가 서로 영향을 주지 않도록 한다.

---

# 20. Oracle 전체 탐색 금지

각 Worker는 다음을 수행하지 않는다.

```text
Schema 전체 Metadata 조회
Schema 전체 Table 조회
Schema 전체 View 조회
Schema 전체 Column 조회
관련 없는 Object 조회
모든 Object의 전체 Column 무조건 조회
동일 Object 반복 조회
```

현재 Backend URL의 실제 SQL에서 사용되는 범위를 기준으로 한다.

---

# 21. Source / SQL 분석 우선

Oracle Metadata 조회가 느리거나
일부 Metadata를 확인할 수 없어도
Source / MyBatis / SQL 분석을 중단하지 않는다.

예:

```text
Source
  확인 완료

MyBatis
  확인 완료

SQL
  확인 완료

Oracle Metadata
  일부 확인되지 않음
```

이라면 Backend Call Path 자체는
계속 완성한다.

Metadata가 전체 Business Logic 추적을
불필요하게 Blocking하지 않도록 한다.

---

# 22. Worker 파일 독립성

각 Worker는 자신의 Output File만 작성한다.

예:

```text
Worker #1
  backend/BE-001.md

Worker #2
  backend/BE-002.md

Worker #3
  backend/BE-003.md
```

Worker #1이:

```text
BE-002.md
BE-003.md
```

를 수정하지 않는다.

---

# 23. 기존 파일 보호

사전 할당된 Output File이 이미 존재하면
해당 Worker는 기존 파일을 자동 덮어쓰지 않는다.

예:

```text
BE-002.md
이미 존재
```

해당 Worker:

```text
STATUS
  STOP

ERROR
  Output File already exists
```

다른 Worker는 계속 실행한다.

---

# 24. Worker 간 ID 탐색 금지

Worker는 자신의 BE ID를 결정하기 위해
다음 작업을 하지 않는다.

```text
backend 폴더의 최대 BE ID 검색
최근 생성 파일 확인
최근 수정 파일 확인
다른 Worker Output 확인
파일 생성 순서 확인
```

BE ID는 Agent가 실행 전에 할당한다.

---

# 25. Worker 완료 결과

각 Worker는 완료 후
상위 Agent에 상세 문서 전체를 반환하지 않는다.

다음 정보만 반환한다.

```text
WORKER:
1

STATUS:
PASS | STOP | ERROR

HTTP METHOD:
POST

BACKEND URL:
/api/equipment/search

BE ID:
BE-001

BE DOCUMENT:
docs/analysis/{화면명}/backend/BE-001.md

CONTROLLER:
확인된 Controller 또는 확인되지 않음

ERROR:
없음 또는 오류 요약
```

상세 분석 내용은
BE Markdown 문서에 저장한다.

---

# 26. Parent Agent 결과 수집

Parent Agent는
각 Worker의 결과만 수집한다.

예:

```text
Worker #1
  PASS
  BE-001

Worker #2
  STOP
  BE-002

Worker #3
  PASS
  BE-003
```

Parent Agent가 Worker의 상세 Backend 분석을
다시 분석하지 않는다.

---

# 27. Parent Agent 재분석 금지

Worker가 완료한 후
Parent Agent는 다음을 다시 수행하지 않는다.

```text
Controller 재탐색
Service 재분석
Mapper 재분석
SQL 재분석
Oracle Metadata 재조회
RFC 재분석
REST 재분석
```

Parent Agent의 역할은:

```text
입력 정리
ID 사전 할당
Worker 병렬 실행
Worker 상태 수집
최종 결과 요약
```

까지이다.

---

# 28. Worker 자동 재시도 금지

Worker가:

```text
STOP
ERROR
```

상태가 되었다고 해서
Parent Agent가 자동으로 같은 분석을 반복 실행하지 않는다.

특히:

```text
Controller 미발견
UNKNOWN 다중 Method
Output File 존재
Source 불일치
```

등은 자동 재시도로 해결하려 하지 않는다.

오류 내용을 최종 결과에 기록한다.

---

# 29. 실행 순서와 ID 분리

병렬 실행에서 다음 둘을 혼동하지 않는다.

```text
입력 순서
  → BE ID 결정

실행 / 완료 순서
  → BE ID에 영향 없음
```

예:

```text
입력

#1 → BE-001
#2 → BE-002
#3 → BE-003
```

완료 순서가:

```text
BE-003
BE-001
BE-002
```

여도 ID는 그대로 유지한다.

---

# 30. 전체 실행 예시

입력:

```text
화면명:
equipment

Backend:

1. POST /api/equipment/search
2. UNKNOWN /api/equipment/status
3. POST /api/equipment/save
4. GET /api/plant/list
```

사전 할당:

```text
#1
BE-001
POST /api/equipment/search
Output: BE-001.md

#2
BE-002
UNKNOWN /api/equipment/status
Output: BE-002.md

#3
BE-003
POST /api/equipment/save
Output: BE-003.md

#4
BE-004
GET /api/plant/list
Output: BE-004.md
```

실행:

```text
Batch #1

Worker #1 ─┐
Worker #2 ─┼─ Parallel
Worker #3 ─┘

완료
 ↓

Batch #2

Worker #4
```

각 Worker:

```text
Backend URL
 ↓
Controller
 ↓
BE Direct Analysis
 ↓
BE Document
```

---

# 31. 최종 결과

모든 Worker 실행이 완료되면
Parent Agent는 간단한 결과만 출력한다.

예:

```text
Backend Direct Parallel Analysis 완료

화면명:
equipment

전체:
4

PASS:
3

STOP:
1

ERROR:
0

────────────────────────────────

#1

STATUS:
PASS

HTTP METHOD:
POST

BACKEND URL:
/api/equipment/search

BE ID:
BE-001

BE DOCUMENT:
docs/analysis/equipment/backend/BE-001.md

────────────────────────────────

#2

STATUS:
STOP

HTTP METHOD:
UNKNOWN

BACKEND URL:
/api/equipment/status

BE ID:
BE-002

BE DOCUMENT:
생성되지 않음

ERROR:
동일 URL에 여러 HTTP Method 존재

────────────────────────────────

#3

STATUS:
PASS

HTTP METHOD:
POST

BACKEND URL:
/api/equipment/save

BE ID:
BE-003

BE DOCUMENT:
docs/analysis/equipment/backend/BE-003.md

────────────────────────────────

#4

STATUS:
PASS

HTTP METHOD:
GET

BACKEND URL:
/api/plant/list

BE ID:
BE-004

BE DOCUMENT:
docs/analysis/equipment/backend/BE-004.md
```

상세 Backend 분석 내용을
Parent Agent 응답에 다시 출력하지 않는다.

---

# 32. 최종 검증

완료 전에 다음을 확인한다.

```text
[ ] Backend 입력 순서대로 BE ID를 사전 할당했는가

[ ] Worker 완료 순서로 BE ID를 변경하지 않았는가

[ ] Worker마다 Backend URL 하나만 분석했는가

[ ] API Analysis를 실행하지 않았는가

[ ] API Document를 요구하지 않았는가

[ ] 각 Worker가 be-direct-analysis 규칙을 따랐는가

[ ] 각 Worker의 Output File을 사전 고정했는가

[ ] Worker가 다른 Worker의 Output을 수정하지 않았는가

[ ] Worker 하나의 STOP / ERROR가 다른 Worker를 중단시키지 않았는가

[ ] MAX PARALLEL WORKERS를 초과하지 않았는가

[ ] DB 호출마다 Oracle Metadata를 조회하지 않았는가

[ ] Worker 내부에서 Oracle Object / Column을 중복 제거했는가

[ ] Worker 간 Mutable Metadata Cache를 공유하지 않았는가

[ ] Schema 전체 Metadata 탐색을 하지 않았는가

[ ] Parent Agent가 Worker 결과를 재분석하지 않았는가

[ ] Parent Agent가 Oracle Metadata를 다시 조회하지 않았는가

[ ] 기존 Output File을 임의로 덮어쓰지 않았는가

[ ] 최종 응답에는 Worker 상태와 문서 위치만 요약했는가
```

---

# 33. STOP

모든 Worker가 다음 중 하나의 상태에 도달하면:

```text
PASS
STOP
ERROR
```

Parent Agent는 결과를 수집하고 STOP 한다.

자동으로 실패 Worker를 다시 실행하지 않는다.

자동으로 다른 Backend URL을 찾지 않는다.

자동으로 API 분석을 시작하지 않는다.

자동으로 다음 화면을 분석하지 않는다.
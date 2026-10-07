---
name: be-parallel-test
description: 하나의 FE Action에 연결된 여러 Backend URL의 BE ID와 OUTPUT_PATH를 모두 선할당한 뒤 최대 2개 Worker씩 Batch 병렬 실행하여 BE 분석 문서를 생성한다.
argument-hint: "<화면명> <FE Base Name> <METHOD URL>..."
allowed-tools: Glob, Agent
---

# BE Parallel Batch Analysis

## 0. 역할

이 Skill은 Backend 병렬 분석 Parent다.

Parent 역할:

1. 입력 파싱
2. 중복 제거
3. 기존 BE 번호 확인
4. 모든 BE ID 선할당
5. 모든 OUTPUT_PATH 선할당
6. Batch 생성
7. Batch별 최대 2 Worker 병렬 실행
8. 결과 수집
9. 다음 Batch 실행
10. 최종 결과 출력

Parent는 Backend Source를 분석하지 않는다.

---

# 1. 입력

형식:

/be-parallel-test <화면명> <FE Base Name> <METHOD1> <URL1> <METHOD2> <URL2> ...

Backend 개수 제한 없음.

예:

/be-parallel-test 자재등록 FE-ACT-010-create POST /material/create GET /material/check POST /material/history GET /material/detail POST /material/approve

---

# 2. Method

각 URL 앞에는 반드시 Method가 존재해야 한다.

허용:

GET

POST

PUT

DELETE

PATCH

UNKNOWN

---

# 3. Pair Parsing

FE Base Name 이후 인자를:

METHOD + URL

두 개씩 Pair로 파싱한다.

예:

POST /api/a

GET /api/b

UNKNOWN /api/c

---

# 4. 입력 오류

Method만 있고 URL이 없거나
URL만 존재하면 실행하지 않는다.

Status:

INVALID_METHOD_URL_PAIR

---

# 5. 중복 제거

동일한:

HTTP Method + Backend URL

Pair가 중복되면 하나만 유지한다.

예:

POST /api/a

POST /api/a

→ 1개

하지만:

GET /api/a

POST /api/a

→ 2개

---

# 6. 출력 Directory

docs/analysis/{화면명}/backend/

---

# 7. 기존 BE 번호

Parent만:

{FE Base Name}-BE-*.md

파일명을 확인한다.

파일 내용은 읽지 않는다.

가장 큰 번호를:

MAX_BE_ID

로 결정한다.

파일이 없으면:

MAX_BE_ID = 0

---

# 8. 전체 BE ID 선할당

중요:

Batch를 만들기 전에
모든 Backend의 BE ID를 먼저 결정한다.

예:

기존:

BE-001
BE-002

신규 URL 5개:

URL A → BE-003

URL B → BE-004

URL C → BE-005

URL D → BE-006

URL E → BE-007

---

# 9. 전체 OUTPUT_PATH 선할당

BE ID 결정 직후
모든 OUTPUT_PATH도 확정한다.

예:

BE-003

docs/analysis/{화면명}/backend/{FE Base Name}-BE-003.md

BE-004

docs/analysis/{화면명}/backend/{FE Base Name}-BE-004.md

이후 변경하지 않는다.

---

# 10. Preallocation 완료 조건

다음이 모든 URL에 대해 존재해야 한다.

INDEX

HTTP_METHOD

BACKEND_URL

BE_ID

OUTPUT_PATH

하나라도 없으면 Worker 실행 금지.

---

# 11. Batch 생성

전체 작업을 입력 순서대로
최대 2개씩 묶는다.

MAX_CONCURRENCY = 2

예:

5개:

Batch 1

- Task 1
- Task 2

Batch 2

- Task 3
- Task 4

Batch 3

- Task 5

---

# 12. Batch 예

7개:

Batch 1:

BE-001
BE-002

Batch 2:

BE-003
BE-004

Batch 3:

BE-005
BE-006

Batch 4:

BE-007

---

# 13. Batch 실행

각 Batch 안의 Worker는 반드시 병렬 실행한다.

잘못된 방식:

Worker A

↓

완료

↓

Worker B

올바른 방식:

Worker A START

+

Worker B START

↓

동시 실행

↓

둘 다 완료

---

# 14. 다음 Batch

현재 Batch의 모든 Worker가 종료된 이후에만
다음 Batch를 시작한다.

예:

Batch 1

Worker A ─┐

Worker B ─┘

↓

A/B 모두 종료

↓

Batch 2

Worker C ─┐

Worker D ─┘

---

# 15. Worker Agent

각 Task마다:

be-fast-worker

Agent를 하나 생성한다.

Worker에게 전달:

BE_ID

HTTP_METHOD

BACKEND_URL

OUTPUT_PATH

---

# 16. Worker Prompt

각 Worker에 다음을 명확히 전달한다.

너는 be-fast-worker다.

다음 Backend 하나만 처리한다.

BE_ID:
{BE_ID}

HTTP_METHOD:
{HTTP_METHOD}

BACKEND_URL:
{BACKEND_URL}

OUTPUT_PATH:
{OUTPUT_PATH}

Parent가 BE ID와 OUTPUT_PATH를 이미 확정했다.

BE ID를 다시 계산하지 않는다.

OUTPUT_PATH를 변경하지 않는다.

기존 backend 파일 목록을 확인하지 않는다.

다른 Backend URL을 분석하지 않는다.

반드시 BE-REFERENCE를 먼저 1회 Load한다.

REFERENCE_LOADED 확인 후
be-analysis-fast Skill을 사용한다.

분석 완료 후 BE-REFERENCE Format을 적용한다.

Reference Format Validation 통과 후에만
OUTPUT_PATH를 생성한다.

---

# 17. Parent Source 분석 금지

Parent는 다음을 하지 않는다.

- Controller 검색
- Service 검색
- ServiceImpl 검색
- Mapper 검색
- XML 검색
- SQL 분석
- SAP/RFC 분석
- BE-REFERENCE Read
- 결과 Markdown 전체 Read

모든 분석은 Worker가 담당한다.

---

# 18. Parent Reference 책임

Parent는 BE-REFERENCE를 읽지 않는다.

대신 Worker 반환값의:

REFERENCE_LOADED

REFERENCE_FORMAT_VALID

상태만 확인한다.

---

# 19. Worker 완료 조건

정상 Worker는:

REFERENCE_LOADED = YES

REFERENCE_FORMAT_VALID = YES

STATUS = COMPLETED

이어야 한다.

---

# 20. Reference 실패

다음은 실패로 처리한다.

REFERENCE_LOADED = NO

또는

REFERENCE_FORMAT_VALID = NO

Parent가 Reference를 대신 적용하지 않는다.

Parent가 해당 Markdown을 수정하지 않는다.

---

# 21. Worker 실패

하나의 Worker가 실패해도
같은 Batch의 다른 Worker 결과를 유지한다.

예:

Worker A:

COMPLETED

Worker B:

REFERENCE_FORMAT_FAILED

이면:

A 결과 유지

B 실패 기록

---

# 22. Batch 실패

Worker 하나가 실패했다고
전체 Batch를 재실행하지 않는다.

자동 Retry 하지 않는다.

이번 버전에서는 성능 측정을 우선한다.

---

# 23. 다음 Batch 진행

현재 Batch의 Worker가 모두 종료 상태에 도달하면
성공/실패와 관계없이 다음 Batch로 진행한다.

예:

Batch 1

A COMPLETED

B NOT_FOUND

↓

Batch 2 계속 실행

---

# 24. Worker 간 상태 공유 금지

Worker들은 다음을 공유하지 않는다.

- Analysis Cache
- Reference State
- Source 결과
- Known Files
- Visited Methods
- Visited XML
- Context

각 Worker는 독립적이다.

---

# 25. 파일 충돌 방지

Worker 시작 전에
모든 OUTPUT_PATH가 서로 다른지 확인한다.

중복이 하나라도 있으면:

OUTPUT_COLLISION

으로 실행을 중단한다.

---

# 26. Batch 병렬 수

고정:

MAX_CONCURRENCY = 2

URL 개수가 몇 개든
동시에 3개 이상 실행하지 않는다.

---

# 27. 예: 3개

입력:

A
B
C

실행:

Batch 1

A + B

↓

완료

↓

Batch 2

C

---

# 28. 예: 5개

입력:

A
B
C
D
E

실행:

Batch 1

A + B

↓

Batch 2

C + D

↓

Batch 3

E

---

# 29. 예: 10개

실행:

Batch 1 → 1 + 2

Batch 2 → 3 + 4

Batch 3 → 5 + 6

Batch 4 → 7 + 8

Batch 5 → 9 + 10

동시에 최대 2개다.

---

# 30. BE ID 순서

BE ID는 Worker 완료 순서가 아니라
입력 순서를 따른다.

예:

입력:

A
B
C

할당:

A → BE-003

B → BE-004

C → BE-005

B가 A보다 먼저 완료해도
번호를 변경하지 않는다.

---

# 31. 결과 순서

최종 결과도 Worker 완료 순서가 아니라
원래 입력 순서대로 출력한다.

---

# 32. 성능 측정 기준

측정:

Parent 시작

↓

Preallocation

↓

Batch 1 시작

↓

Batch 1 완료

↓

Batch 2 시작

↓

...

↓

마지막 Batch 완료

↓

모든 Markdown 생성 완료

전체 Wall Clock Time을 측정한다.

---

# 33. Parent 완료 출력

최종:

## BE Batch Parallel Result

### Configuration

MAX_CONCURRENCY:

2

TOTAL_TASKS:

{count}

TOTAL_BATCHES:

{ceil(count / 2)}

---

### Batch 1

#### BE-xxx

Method:

URL:

Output:

Reference Loaded:

Reference Format:

Status:

#### BE-xxx

Method:

URL:

Output:

Reference Loaded:

Reference Format:

Status:

---

### Batch 2

동일 형식.

---

### Summary

Requested:

Deduplicated:

Total Workers:

Total Batches:

Completed:

Reference Failed:

Not Found:

Ambiguous:

Failed:

Output Files:

---

### Architecture

BE ID:

PARENT_PREALLOCATED

OUTPUT PATH:

PARENT_PREALLOCATED

MAX CONCURRENCY:

2

EXECUTION:

BATCH_PARALLEL

WORKER:

be-fast-worker

ANALYSIS ENGINE:

be-analysis-fast

REFERENCE OWNER:

WORKER

REFERENCE LOAD:

ONCE_BEFORE_ANALYSIS

REFERENCE VALIDATION:

REQUIRED_BEFORE_WRITE

PARENT SOURCE ANALYSIS:

DISABLED

JAR SEARCH:

FORBIDDEN

COMMON EXPANSION:

RESTRICTED

---

### Final Status

모두 성공:

COMPLETED

일부 실패:

PARTIAL

모두 실패:

FAILED

완료 후 추가 Source 분석을 수행하지 않는다.
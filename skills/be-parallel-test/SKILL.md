---
name: be-parallel-test
description: 여러 Backend URL의 BE ID와 OUTPUT_PATH를 Parent에서 먼저 확정하고 최대 3개의 be-fast-worker Agent를 동시에 실행하여 동일 be-analysis-fast Skill로 병렬 분석한다.
argument-hint: "<화면명> <FE Base Name> <METHOD1> <URL1> [<METHOD2> <URL2>] [<METHOD3> <URL3>]"
allowed-tools: Glob, Agent
---

# BE Parallel Test

## 0. 역할

이 Skill은 병렬 분석 Parent다.

Parent 역할:

1. 입력 파싱
2. 기존 BE 번호 확인
3. 모든 BE ID 선할당
4. 모든 OUTPUT_PATH 선할당
5. Worker Agent 병렬 실행
6. 결과 수집

Parent는 Backend Source를 분석하지 않는다.

---

# 1. 구조

실행 구조:

Parent

├─ Worker 1
│   └─ be-analysis-fast
│
├─ Worker 2
│   └─ be-analysis-fast
│
└─ Worker 3
    └─ be-analysis-fast

Worker들은 동시에 실행한다.

---

# 2. 사용법

/be-parallel-test <화면명> <FE Base Name> <METHOD1> <URL1> [<METHOD2> <URL2>] [<METHOD3> <URL3>]

예:

/be-parallel-test 자재등록 FE-ACT-010-create POST /material/create GET /material/check POST /material/history

최대 URL:

3

---

# 3. Method

허용:

GET
POST
PUT
DELETE
PATCH
UNKNOWN

각 URL마다 Method를 별도로 가진다.

---

# 4. 중복 제거

동일한:

HTTP Method + Backend URL

조합이 여러 번 입력되면
Worker 하나만 생성한다.

단:

GET /material/check

POST /material/check

는 서로 다른 대상이다.

---

# 5. 출력 디렉토리

docs/analysis/{화면명}/backend/

---

# 6. 기존 BE 파일 확인

Parent만 수행한다.

Pattern:

{FE Base Name}-BE-*.md

파일명만 확인한다.

내용은 읽지 않는다.

예:

FE-ACT-010-create-BE-001.md

FE-ACT-010-create-BE-002.md

이면:

MAX_BE_ID = 2

---

# 7. BE ID 선할당

Worker 시작 전에
모든 번호를 결정한다.

예:

MAX_BE_ID = 2

입력 URL 3개:

URL 1 → BE-003

URL 2 → BE-004

URL 3 → BE-005

Worker 실행 후 번호를 변경하지 않는다.

---

# 8. OUTPUT_PATH 선할당

각 Worker의 OUTPUT_PATH도
Worker 실행 전에 확정한다.

예:

Worker 1:

docs/analysis/자재등록/backend/FE-ACT-010-create-BE-003.md

Worker 2:

docs/analysis/자재등록/backend/FE-ACT-010-create-BE-004.md

Worker 3:

docs/analysis/자재등록/backend/FE-ACT-010-create-BE-005.md

---

# 9. 충돌 검증

모든 Worker의:

BE_ID

OUTPUT_PATH

가 서로 달라야 한다.

중복이 있으면 Worker를 실행하지 않는다.

Status:

OUTPUT_COLLISION

---

# 10. Worker Prompt

각 Worker에 다음만 전달한다.

HTTP_METHOD:
{Method}

BACKEND_URL:
{URL}

OUTPUT_PATH:
{Output Path}

그리고 다음을 명시한다.

be-fast-worker로 실행한다.

Parent가 이미 OUTPUT_PATH를 확정했다.

BE ID를 다시 계산하지 않는다.

기존 BE 파일을 확인하지 않는다.

다른 Backend URL을 분석하지 않는다.

be-analysis-fast Skill을 사용하여
할당된 URL 하나만 분석한다.

---

# 11. 병렬 실행

가장 중요한 규칙이다.

모든 be-fast-worker Agent를
가능한 한 동일 단계에서 동시에 시작한다.

금지:

Worker 1
→ 완료
→ Worker 2
→ 완료
→ Worker 3

반드시:

Worker 1 START
Worker 2 START
Worker 3 START

↓

동시 실행

↓

모든 Worker 완료

형태로 실행한다.

---

# 12. Fan-Out

입력 URL 수만큼 Worker를 만든다.

1 URL:

1 Worker

2 URL:

2 Workers 동시

3 URL:

3 Workers 동시

MAX:

3

---

# 13. Fan-In

모든 Worker가 완료될 때까지 기다린다.

각 Worker의 완료 결과만 수집한다.

Parent가 Backend Source를 다시 분석하지 않는다.

---

# 14. Parent 금지

Parent는 하지 않는다.

- Controller Grep
- Service Grep
- Mapper Grep
- XML Grep
- SQL 분석
- BE-REFERENCE Read
- 결과 Markdown 전체 Read
- Worker 결과 재검증
- 실패 Worker 자동 재분석

---

# 15. Worker 실패

Worker 하나가 실패해도
다른 Worker 결과를 유지한다.

예:

BE-003 COMPLETED

BE-004 NOT_FOUND

BE-005 COMPLETED

Parent가 BE-004를 자동 재실행하지 않는다.

---

# 16. 분석 Engine 통일

모든 Worker는 반드시:

be-analysis-fast

하나만 사용한다.

Worker별로 분석 Prompt를 다르게 만들지 않는다.

따라서:

단독 분석 Engine

=

병렬 분석 Engine

이 되도록 유지한다.

---

# 17. Source Cache 공유

이번 테스트에서는 Worker 간 Cache를 공유하지 않는다.

각 Worker가 독립적으로:

- Controller
- Service
- Mapper
- XML

을 탐색한다.

먼저 순수 병렬 성능을 측정한다.

---

# 18. MAX WORKERS

MAX_PARALLEL_WORKERS = 3

3개 초과 입력 시:

MAX_PARALLEL_3

으로 종료한다.

이번 단계에서는 4개 이상 실행하지 않는다.

---

# 19. 성능 측정

측정 기준:

명령 실행 시작

↓

모든 Worker Agent 시작

↓

모든 Worker 완료

↓

모든 Markdown 생성 완료

까지의 전체 Wall Clock Time.

Worker 개별 시간보다
전체 완료 시간을 우선한다.

---

# 20. 완료 출력

## BE Parallel Test Result

### Worker Results

Worker 1

- BE ID:
- Method:
- URL:
- Output:
- Status:

Worker 2

- BE ID:
- Method:
- URL:
- Output:
- Status:

Worker 3

- BE ID:
- Method:
- URL:
- Output:
- Status:

입력되지 않은 Worker는 표시하지 않는다.

### Summary

- Requested URLs:
- Started Workers:
- Completed:
- Failed:
- Output Files:
- Parallel Workers:

### Architecture

Parent:
ID_AND_PATH_ONLY

Worker:
be-fast-worker

Analysis Engine:
be-analysis-fast

Execution:
PARALLEL

MAX WORKERS:
3

Worker Cache:
ISOLATED

Parent Source Analysis:
DISABLED

### Status

COMPLETED

또는

PARTIAL

또는

FAILED
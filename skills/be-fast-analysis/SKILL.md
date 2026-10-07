---
name: be-fast-analysis
description: 화면명, FE Base Name, HTTP Method, Backend URL을 입력받아 BE ID와 OUTPUT_PATH를 빠르게 확정한 뒤 be-analysis-worker를 실행한다.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Glob, Grep, Agent
---
# BE Fast Analysis Skill
## 1. 목적
하나의 Backend URL에 대한 BE 분석을 실행한다.
이 Skill은 Backend 소스를 직접 분석하지 않는다.
역할은 다음으로 제한한다.
```text
입력 파싱
 ↓
FE Base Name 검증
 ↓
기존 BE 파일명 확인
 ↓
BE ID 결정
 ↓
OUTPUT_PATH 결정
 ↓
be-analysis-worker 실행
 ↓
생성 파일 존재 확인

실제 Backend 분석은 반드시 be-analysis-worker에게 위임한다.

2. 사용법

/be-fast-analysis <화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-fast-analysis 자재등록 FE-ACT-010-create POST /material/create

Method를 모르면:

/be-fast-analysis 자재등록 FE-ACT-010-create UNKNOWN /material/create

3. 입력 파싱

입력을 다음 4개 값으로 해석한다.

SCREEN_NAME
FE_BASE_NAME
HTTP_METHOD
BACKEND_URL

인자 순서를 변경하지 않는다.

4. FE Base Name 규칙

FE Base Name은 입력값을 그대로 사용한다.

예:

FE-ACT-010-create

다음을 금지한다.

기능명 재생성
Action ID 재계산
URL 기반 이름 생성
문서 내부 번호 사용

FE Base Name이 명백하게 잘못된 경우 Worker를 실행하지 않는다.

5. 출력 디렉토리

출력 디렉토리는 다음으로 고정한다.

docs/analysis/{SCREEN_NAME}/backend/

6. 기존 BE 파일 확인

다음 패턴의 파일명만 확인한다.

docs/analysis/{SCREEN_NAME}/backend/{FE_BASE_NAME}-BE-*.md

기존 BE 문서의 내용은 읽지 않는다.

7. BE ID 결정

정상적인 파일명:

{FE_BASE_NAME}-BE-NNN.md

여기서 NNN은 정확히 3자리 숫자이다.

예:

FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md

위 두 파일이 존재하면:

BE-003

을 사용한다.

기존 파일이 없으면:

BE-001

부터 시작한다.

8. BE ID 추출 제한

BE ID는 파일명의:

-BE-NNN.md

부분에서만 추출한다.

허용:

BE-001
BE-002
BE-036
BE-105

금지:

36.2
2.1
POST
ACT-010
문서 섹션 번호
URL 일부

정상적인 BE-NNN 값 중 가장 큰 번호에 +1 한다.

9. OUTPUT_PATH

다음 공식만 사용한다.

docs/analysis/{SCREEN_NAME}/backend/{FE_BASE_NAME}-{BE_ID}.md

예:

docs/analysis/자재등록/backend/FE-ACT-010-create-BE-003.md

10. 파일명 보호

다음 파일명은 생성하면 안 된다.

FE-ACT-010-create-BE-36.2.md
FE-ACT-010-create-POST-BE-001.md
FE-ACT-010-create-material-create.md
FE-ACT-010-create-BE-1.md

정상 형식은 반드시:

{FE_BASE_NAME}-BE-{3자리 숫자}.md

이다.

11. Worker 실행 전 값 고정

Worker 실행 전에 다음 값을 모두 확정한다.

SCREEN_NAME
FE_BASE_NAME
BE_ID
HTTP_METHOD
BACKEND_URL
OUTPUT_PATH

Worker 실행 후 변경하지 않는다.

12. Worker 호출

be-analysis-worker를 정확히 1회 호출한다.

다음 정보를 전달한다.

화면명:
{SCREEN_NAME}
FE Base Name:
{FE_BASE_NAME}
BE ID:
{BE_ID}
HTTP Method:
{HTTP_METHOD}
Backend URL:
{BACKEND_URL}
OUTPUT_PATH:
{OUTPUT_PATH}

Worker에게 불필요한 추가 분석 지시를 덧붙이지 않는다.

13. Skill 직접 분석 금지

이 Skill은 다음 작업을 하지 않는다.

Controller 검색
Service 분석
ServiceImpl 분석
Mapper 분석
MyBatis XML 분석
SQL 분석
SAP 분석
RFC 분석
외부 API 분석
Evidence 수집
BE-REFERENCE 분석

모든 실제 BE 분석은 be-analysis-worker가 담당한다.

14. 중복 실행 금지

하나의 요청에서 Worker 하나만 실행한다.

분석이 오래 걸린다는 이유로 동일 URL에 Worker를 추가 실행하지 않는다.

Worker 완료 여부가 확인되기 전에 재실행하지 않는다.

15. 결과 검증

Worker가 완료되면 다음만 확인한다.

OUTPUT_PATH 파일 존재 여부

생성된 BE 문서 전체를 다시 읽지 않는다.

Worker 결과를 다시 분석하지 않는다.

16. 실패 처리

OUTPUT_PATH가 생성되지 않았으면 실패로 처리한다.

Skill이 직접 Backend 분석을 다시 수행하지 않는다.

다음 정보를 보고한다.

BE 분석 실패
예정 BE ID:
{BE_ID}
예정 OUTPUT_PATH:
{OUTPUT_PATH}

Worker가 실패 원인을 반환했다면 함께 표시한다.

17. 성공 처리

파일이 정상적으로 생성됐으면 간단하게 보고한다.

BE 분석 완료
BE ID:
{BE_ID}
OUTPUT_PATH:
{OUTPUT_PATH}

생성된 BE 문서 내용을 응답에 다시 출력하지 않는다.

18. 성능 규칙

속도를 위해 다음을 반드시 지킨다.

기존 BE 문서 내용 Read 금지
기존 BE 파일명만 확인
BE 소스 직접 분석 금지
BE-REFERENCE Read 금지
Worker 결과 전체 재검증 금지
동일 Worker 중복 실행 금지

Skill은 가능한 짧게 실행한다.

19. Parallel 확장 규칙

이 Skill은 현재 단일 Backend URL 분석용이다.

향후 Parallel Parent에서는 같은 be-analysis-worker를 사용한다.

Parallel Parent가 먼저 모든 URL에 대해:

BE ID
OUTPUT_PATH

를 확정한다.

예:

URL-1 → BE-001 → ...-BE-001.md
URL-2 → BE-002 → ...-BE-002.md
URL-3 → BE-003 → ...-BE-003.md
URL-4 → BE-004 → ...-BE-004.md

모든 번호와 OUTPUT_PATH가 확정된 이후에만 Worker들을 병렬 실행한다.

Worker는 BE ID나 OUTPUT_PATH를 결정하지 않는다.

20. 완료 조건

다음을 모두 만족하면 완료이다.

[ ] 입력 4개 파싱
[ ] FE Base Name 검증
[ ] 기존 BE 파일명만 확인
[ ] 다음 BE ID 결정
[ ] BE ID 3자리 확인
[ ] OUTPUT_PATH 결정
[ ] Worker 실행 전 값 고정
[ ] be-analysis-worker 1회 실행
[ ] OUTPUT_PATH 존재 확인
이제 기존 `/be-analysis`는 **그대로 놔두고**, 새 버전은:
```text
/be-fast-analysis 화면명 FE-ACT-xxx-기능명 UNKNOWN /backend/url

로 테스트하면 돼.

아직 기존 /be-analysis를 지우지 말자. 새 버전의 속도와 결과 품질을 비교할 기준점으로 남겨두는 게 좋다.
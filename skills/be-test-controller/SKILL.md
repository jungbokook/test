---
name: be-test-controller
description: Backend URL에서 ServiceImpl의 실제 Mapper 호출을 확인한 뒤 Mapper Java Interface를 건너뛰고 MyBatis XML statement까지 부분 탐색하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test - Direct MyBatis XML

## 0. 목적

이 Skill은 최종 BE 분석 문서를 생성하지 않는다.

이번 테스트 범위:

Controller
→ Service
→ ServiceImpl
→ ServiceImpl 대상 Method 부분 Read
→ Mapper 호출 확인
→ Mapper Type 확인
→ Mapper Java Interface SKIP
→ MyBatis XML 찾기
→ 실제 Statement 부분 Read
→ STOP

이번 테스트에서는 Mapper Java Interface를 검색하지 않는다.

목적은:

Mapper Interface 탐색을 제거하고
ServiceImpl에서 확인한 Mapper 정보로
MyBatis XML까지 직접 연결했을 때의 속도를 측정하는 것이다.

SQL의 상세 비즈니스 분석은 아직 하지 않는다.

---

# 1. 입력

사용법:

/be-test-controller <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller POST /material/create

/be-test-controller UNKNOWN /material/create

---

# 2. 검색 범위

Backend 프로젝트:

gipms-api-*

제외:

- docs/**
- sample/**
- **/sample/**
- target/**
- build/**
- *.jar
- test/**
- **/test/**
- generated/**
- node_modules/**

Frontend 프로젝트는 검색하지 않는다.

절대경로를 출력하지 않는다.

---

# 3. 허용 도구

사용:

- Grep
- Read

사용하지 않는다:

- Glob
- Code Index
- Agent
- BE-REFERENCE

---

# 4. 핵심 성능 원칙

가장 중요한 원칙:

전체 파일 Read를 기본적으로 하지 않는다.

다음 방식으로 탐색한다.

정확한 Symbol
→ 제한된 Grep
→ 위치 확보
→ 필요한 범위만 Read
→ 다음 단계

반드시 다음 규칙을 따른다.

1. 동일 검색을 반복하지 않는다.
2. 이미 발견한 파일을 다시 찾지 않는다.
3. 이미 확보한 정보를 다시 검색하지 않는다.
4. 현재 Backend 프로젝트를 우선한다.
5. 현재 프로젝트에서 찾지 못한 경우에만 다른 gipms-api-*로 확대한다.
6. Controller 확정 후 URL 검색을 중단한다.
7. Service 확정 후 Service 검색을 중단한다.
8. ServiceImpl 확정 후 구현체 검색을 중단한다.
9. ServiceImpl 전체 Read를 하지 않는다.
10. Mapper Java Interface를 검색하지 않는다.
11. Mapper Java Method 선언을 검색하지 않는다.
12. XML 전체 Read를 하지 않는다.
13. 실제 statement 위치만 부분 Read한다.
14. Evidence를 위한 재검색을 하지 않는다.

내부적으로 다음 정보를 재사용한다.

KNOWN_FILES

KNOWN_SYMBOLS

VISITED_FILES

VISITED_METHODS

VISITED_XML

VISITED_STATEMENTS

---

# STEP 1. URL 분리

입력 Backend URL을 path 단위로 확인한다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

Controller 검색에서는 가능한 경우
식별력이 높은 상위 path를 먼저 사용한다.

---

# STEP 2. Controller 찾기

검색 범위:

gipms-api-*/**/*Controller.java

URL 상위 path를 우선 Grep한다.

예:

/material

후보 Controller가 발견되면
Backend 전체 URL 검색을 중단한다.

후보 Controller만 확인한다.

상위 path로 확정할 수 없을 경우에만
마지막 segment를 보조 검색어로 사용한다.

---

# STEP 3. Controller Mapping 검증

Controller에서:

class-level mapping

+

method-level mapping

을 조합한다.

예:

@RequestMapping("/material")

+

@PostMapping("/create")

=

POST /material/create

전체 URL과 HTTP Method가 일치하는
Controller Method를 확정한다.

---

# STEP 4. HTTP Method 확정

입력이:

GET
POST
PUT
DELETE
PATCH

이면 Controller annotation과 일치해야 한다.

UNKNOWN이면 Controller annotation에서 실제 Method를 확정한다.

동일 URL에 여러 HTTP Method가 존재하고
입력 Method가 UNKNOWN이면 임의 선택하지 않는다.

AMBIGUOUS로 종료한다.

---

# STEP 5. Controller Method 확인

확인:

- Controller Class
- Controller Method
- HTTP Method
- Evidence

Controller Method가 확정되면
다른 Controller를 탐색하지 않는다.

---

# STEP 6. Service 호출 확인

Controller Method 내부에서
실제로 호출되는 Service를 확인한다.

예:

materialService.create(request);

기록:

Service Variable:
materialService

Service Method:
create

---

# STEP 7. Service Type 확인

이미 확인한 Controller 내용에서
Service Variable Type을 확인한다.

예:

private final MaterialService materialService;

결과:

Service Type:
MaterialService

Service Variable:
materialService

Controller를 다시 검색하지 않는다.

---

# STEP 8. Service 찾기

Service 파일 경로를 모르면
현재 Backend 프로젝트에서 정확한 Type만 Grep한다.

예:

interface MaterialService

현재 프로젝트에서 발견되면
다른 프로젝트를 검색하지 않는다.

현재 프로젝트에 없을 때만
다른 gipms-api-*로 범위를 확대한다.

---

# STEP 9. Service Method 확인

Controller가 실제 호출한 Method 선언만 확인한다.

확인:

- Service
- Service Method
- Parameter
- Return Type
- Evidence

다른 Service Method는 분석하지 않는다.

---

# STEP 10. ServiceImpl 찾기

Service가 interface이면 구현체를 찾는다.

검색:

implements MaterialService

먼저 현재 Backend 프로젝트에서만 검색한다.

구현체가 발견되면 즉시 검색을 중단한다.

ServiceImpl 전체 파일은 Read하지 않는다.

파일 경로만 확보한다.

---

# STEP 11. ServiceImpl Method 위치

확정된 ServiceImpl 파일 하나에서만
Controller가 호출한 정확한 Method 이름을 Grep한다.

예:

create(

Service Interface signature와 비교하여
실제 구현 Method를 확정한다.

---

# STEP 12. ServiceImpl 부분 Read

ServiceImpl Method 시작 위치부터
약 80줄을 우선 Read한다.

예:

Method 시작:
420

초기 Read:
420 ~ 500

Method 종료가 확인되면 추가 Read하지 않는다.

Method가 계속되는 경우에만
다음 약 80줄을 추가한다.

예:

501 ~ 580

Method 종료 즉시 Read를 중단한다.

ServiceImpl 전체 파일을 읽지 않는다.

---

# STEP 13. Mapper 호출 확인

ServiceImpl Method에서 직접 실행되는
Mapper 호출을 확인한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

기록:

Mapper Variable:
materialMapper

Mapper Methods:
- selectMaterial
- insertMaterial

조건문 안의 직접 Mapper 호출도 기록한다.

Local/private Method 내부는 현재 테스트에서 추적하지 않는다.

다른 Service 내부도 추적하지 않는다.

---

# STEP 14. Mapper Type 확인

Mapper Java Interface를 찾지 않는다.

Mapper Variable Type만 확인한다.

우선 이미 확보한 ServiceImpl 범위에서 확인한다.

예:

private final MaterialMapper materialMapper;

이면:

Mapper Variable:
materialMapper

Mapper Type:
MaterialMapper

---

# STEP 15. Mapper Type이 보이지 않는 경우

현재 부분 Read 범위에 Mapper 선언이 없을 경우에만
확정된 ServiceImpl 파일 하나에서
Mapper Variable 이름을 Grep한다.

예:

materialMapper

field 또는 constructor declaration을 확인한다.

예:

private final MaterialMapper materialMapper;

Mapper Type을 확보하면 즉시 검색을 종료한다.

ServiceImpl 전체 파일을 읽지 않는다.

---

# STEP 16. Mapper Java 강제 SKIP

이번 테스트에서는 다음을 절대 수행하지 않는다.

- interface MaterialMapper 검색
- MaterialMapper.java 검색
- Mapper Java Method 검색
- Mapper annotation 검색
- Mapper Java Evidence 수집

ServiceImpl에서 확보한:

Mapper Type

+

Mapper Method

정보를 사용해서 바로 MyBatis XML을 찾는다.

예:

Mapper Type:
MaterialMapper

Mapper Method:
insertMaterial

↓

MyBatis XML 탐색

---

# STEP 17. MyBatis XML 검색 전략

먼저 현재 Backend 프로젝트 내부에서만 검색한다.

처음부터 모든 gipms-api-*를 검색하지 않는다.

검색 대상:

*.xml

우선순위:

1. namespace와 Mapper Type 연결
2. statement id

가능하면 Mapper Type 이름을 먼저 사용한다.

예:

MaterialMapper

MyBatis XML에서 다음과 같은 namespace 후보를 찾는다.

예:

<mapper namespace="...MaterialMapper">

후보 XML이 발견되면
다른 XML 검색을 중단한다.

---

# STEP 18. XML namespace 확인

후보 XML에서 namespace가
Mapper Type과 대응되는지 확인한다.

예:

Mapper Type:

MaterialMapper

XML:

<mapper namespace="com.xxx.material.MaterialMapper">

이면 일치 후보로 판단한다.

namespace가 다른 Mapper를 가리키면 제외한다.

---

# STEP 19. namespace 검색 실패 시 fallback

Mapper Type으로 XML을 찾지 못한 경우에만
실제 Mapper Method 이름을 사용한다.

예:

insertMaterial

현재 Backend 프로젝트의 XML에서만 Grep한다.

목표:

id="insertMaterial"

후보 XML 찾기.

후보가 발견되면 namespace를 확인한다.

처음부터 Mapper Type과 Method를 동시에 여러 번 검색하지 않는다.

---

# STEP 20. Statement 위치 찾기

XML 파일이 확정되면
해당 파일 하나에서만 실제 Mapper Method에 대응하는
statement id를 Grep한다.

예:

id="insertMaterial"

가능한 statement:

<select>
<insert>
<update>
<delete>

예:

<insert id="insertMaterial">

statement 시작 line을 확보한다.

---

# STEP 21. XML 전체 Read 금지

MyBatis XML 전체 파일을 Read하지 않는다.

statement 시작 line부터 필요한 범위만 Read한다.

초기 범위:

statement 시작부터 약 40줄

예:

statement 시작:
210

초기 Read:
210 ~ 250

statement 종료 tag가 확인되면
추가 Read하지 않는다.

---

# STEP 22. Statement가 긴 경우

40줄 안에서 statement 종료가 확인되지 않은 경우에만
다음 범위를 추가 Read한다.

예:

210 ~ 250
↓
251 ~ 290

다음 중 해당되는 종료 tag가 확인되면 즉시 중단한다.

</select>
</insert>
</update>
</delete>

XML 전체를 읽지 않는다.

---

# STEP 23. 여러 Mapper Method

ServiceImpl에서 같은 Mapper의 여러 Method를 호출하면:

예:

materialMapper.selectMaterial()

materialMapper.insertMaterial()

XML 파일은 한 번만 찾는다.

그 다음 동일 XML에서:

id="selectMaterial"

id="insertMaterial"

위치만 각각 찾는다.

XML 파일 경로를 다시 검색하지 않는다.

---

# STEP 24. 여러 Mapper Type

예:

materialMapper.selectMaterial()

historyMapper.insertHistory()

이면 각각의 Mapper Type에 대해
필요한 XML을 한 번씩만 찾는다.

동일 Mapper Type에 대한 XML 검색을 반복하지 않는다.

---

# STEP 25. Dynamic SQL

현재 테스트에서는 Dynamic SQL의 상세 흐름을 분석하지 않는다.

statement 부분에서 다음이 보일 수 있다.

<if>
<choose>
<when>
<otherwise>
<foreach>
<include>

이번 단계에서는 존재 여부만 기록한다.

예:

Dynamic SQL:
YES

Elements:
- if
- foreach

하지만:

- 조건 의미 분석
- include fragment 추적
- SQL 분기 분석

은 하지 않는다.

---

# STEP 26. SQL 분석 금지

이번 테스트에서는 SQL 의미를 분석하지 않는다.

예:

SELECT ...
FROM ...
WHERE ...

가 보여도:

- 테이블 의미 분석
- JOIN 분석
- WHERE 조건 분석
- 컬럼 Mapping 분석
- DB Metadata 확인

을 하지 않는다.

목표는 정확한 MyBatis statement를 찾는 것까지다.

---

# STEP 27. Evidence

Evidence는 다음에 대해 확보한다.

- Controller Method
- Service Method
- ServiceImpl Method
- MyBatis XML Statement

Mapper Java Evidence는 현재 테스트에서 제외한다.

형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:420-468

gipms-api-material/src/main/resources/.../MaterialMapper.xml:210-235

절대경로를 출력하지 않는다.

Evidence 확보를 위해 재검색하지 않는다.

---

# STEP 28. 강제 STOP

실제로 호출된 Mapper Method에 대응하는
MyBatis XML statement가 모두 확인되면 즉시 종료한다.

이후 Grep/Read를 수행하지 않는다.

절대 수행하지 않는다:

- Mapper Java Interface 검색
- Mapper Java Method 검색
- SQL 상세 분석
- SQL 의미 해석
- include fragment 추적
- resultMap 추적
- DB Metadata
- Oracle Metadata
- Local/private Method 추적
- 다른 Service 내부 추적
- SAP 검색
- RFC 검색
- 외부 API 검색
- BE-REFERENCE 읽기
- Markdown 문서 생성

---

# STEP 29. 출력

아래 형식으로만 출력한다.

## BE Fast Search Result

- URL:
- HTTP Method:

### Controller

- Controller:
- Controller Method:
- Evidence:

### Service

- Service Type:
- Service Method:
- Service Interface:
- Service Implementation:
- Evidence:

### ServiceImpl Method

- Method:
- Read Range:
- Evidence:

### Mapper Call

각 호출에 대해:

- Mapper Variable:
- Mapper Type:
- Mapper Method:

Mapper Java Interface:

SKIPPED

### MyBatis

각 실제 Mapper Method에 대해:

- XML:
- Namespace:
- Statement Type:
- Statement ID:
- Dynamic SQL:
- Evidence:

### Execution Flow

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ MaterialMapper.insertMaterial()
→ Mapper Java SKIP
→ MaterialMapper.xml
→ <insert id="insertMaterial">
→ STOP

여러 호출이면 실제 순서를 유지한다.

### Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.
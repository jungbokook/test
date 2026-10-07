---
name: be-test-controller
description: Backend URL에서 Controller부터 ServiceImpl, 동일 Class Local/private Method, Mapper 호출, MyBatis XML과 직접 SQL까지 부분 Read 방식으로 추적하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test - Local Method + SQL

## 0. 목적

이 Skill은 최종 BE 분석 문서를 생성하지 않는다.

이번 테스트 범위:

Controller
→ Service
→ ServiceImpl 대상 Method
→ 동일 ServiceImpl 내부 Local/private Method
→ Mapper 호출
→ Mapper Type
→ Mapper Java SKIP
→ MyBatis XML
→ Statement 부분 Read
→ 직접 SQL 분석
→ STOP

이번 테스트에서 새로 추가되는 것은:

동일 ServiceImpl Class 내부의
Local/private Method 호출 추적

이다.

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

전체 파일 Read를 기본적으로 하지 않는다.

기본 탐색 방식:

정확한 Symbol
→ 제한 Grep
→ 위치 확보
→ 필요한 범위만 Read
→ 다음 호출
→ STOP 조건 확인

반드시 다음 규칙을 따른다.

1. 동일 검색을 반복하지 않는다.
2. 이미 발견한 파일을 다시 찾지 않는다.
3. 이미 확보한 정보를 다시 검색하지 않는다.
4. 이미 읽은 Method를 다시 읽지 않는다.
5. 현재 Backend 프로젝트부터 검색한다.
6. 찾지 못할 때만 다른 gipms-api-*로 확대한다.
7. ServiceImpl 전체 Read를 하지 않는다.
8. Local Method도 전체 파일 Read 없이 Method 위치부터 부분 Read한다.
9. Mapper Java Interface를 검색하지 않는다.
10. XML 전체 Read를 하지 않는다.
11. SQL은 이미 읽은 Statement 범위에서 분석한다.
12. Evidence를 위한 재검색을 하지 않는다.

내부적으로 유지:

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

Controller 검색에서는
식별력이 높은 상위 path를 우선 사용한다.

---

# STEP 2. Controller 찾기

검색 범위:

gipms-api-*/**/*Controller.java

URL 상위 path를 먼저 Grep한다.

후보 Controller가 발견되면
Backend 전체 URL 검색을 중단한다.

필요한 경우에만 마지막 segment를 사용한다.

---

# STEP 3. Controller Mapping 검증

class-level mapping과
method-level mapping을 조합한다.

예:

@RequestMapping("/material")

+

@PostMapping("/create")

=

POST /material/create

입력 URL과 일치하는 Method를 확정한다.

---

# STEP 4. HTTP Method

입력이:

GET
POST
PUT
DELETE
PATCH

이면 annotation과 일치해야 한다.

UNKNOWN이면 annotation에서 실제 Method를 확정한다.

동일 URL에 여러 HTTP Method가 존재하면:

AMBIGUOUS

로 종료한다.

---

# STEP 5. Controller Method

확인:

- Controller Class
- Controller Method
- HTTP Method
- Evidence

확정 후 다른 Controller를 탐색하지 않는다.

---

# STEP 6. Service 호출

Controller Method에서
실제로 호출되는 Service를 확인한다.

예:

materialService.create(request);

확인:

Service Variable:
materialService

Service Method:
create

---

# STEP 7. Service Type

이미 확보한 Controller 내용에서
Service Variable Type을 확인한다.

예:

private final MaterialService materialService;

결과:

Service Type:
MaterialService

Controller를 재검색하지 않는다.

---

# STEP 8. Service 찾기

현재 Backend 프로젝트에서
정확한 Service Type을 검색한다.

예:

interface MaterialService

발견되면 다른 프로젝트를 검색하지 않는다.

없을 때만 다른 gipms-api-*로 확대한다.

---

# STEP 9. Service Method

Controller가 실제 호출한
Service Method 선언만 확인한다.

확인:

- Service Method
- Parameter
- Return Type
- Evidence

---

# STEP 10. ServiceImpl 찾기

Service가 interface이면:

implements MaterialService

를 검색한다.

현재 Backend 프로젝트를 우선한다.

구현체 발견 즉시 검색을 중단한다.

ServiceImpl 전체 파일을 읽지 않는다.

---

# STEP 11. ServiceImpl Entry Method

확정된 ServiceImpl 파일 하나에서
실제 호출된 Method 이름을 Grep한다.

예:

create(

Service Interface signature와 비교해
정확한 구현 Method를 확정한다.

---

# STEP 12. Method 부분 Read

Method 시작부터 약 80줄을 Read한다.

예:

420 ~ 500

Method 종료가 확인되면 중단한다.

종료되지 않은 경우에만
다음 약 80줄을 추가한다.

전체 파일을 읽지 않는다.

---

# STEP 13. Entry Method 호출 추출

ServiceImpl Entry Method에서
직접 호출되는 항목을 확인한다.

이번 테스트에서 추적 대상:

1. Mapper 호출
2. 동일 Class Local/private Method 호출

현재 추적하지 않는 것:

- 다른 Service
- 외부 Component
- Client
- SAP
- RFC
- HTTP API

---

# STEP 14. Local/private Method 판별

Entry Method에서 다음과 같은 호출이 보이면:

validate(request);

saveMaterial(request);

buildResult(request);

동일 ServiceImpl Class 내부 Method일 가능성이 있는 호출을 기록한다.

하지만 모든 Method 호출을 무조건 Local Method라고 판단하지 않는다.

다음은 Local Method 후보에서 제외한다.

예:

request.getId()

list.add(...)

String.valueOf(...)

Objects.requireNonNull(...)

Collections.emptyList()

mapper.xxx(...)

다른 객체의 명시적 Method 호출

---

# STEP 15. Local Method 위치 찾기

Local Method 후보:

saveMaterial

가 발견되면

이미 확정된 동일 ServiceImpl 파일 하나에서만:

saveMaterial(

을 Grep한다.

다른 파일을 검색하지 않는다.

Method 선언 위치를 찾는다.

예:

private void saveMaterial(

protected void saveMaterial(

public void saveMaterial(

Method 선언이 존재하면
Local Method로 확정한다.

---

# STEP 16. Local Method 부분 Read

Local Method 시작 위치부터
약 80줄만 Read한다.

예:

610 ~ 690

Method 종료가 확인되면 중단한다.

종료되지 않은 경우에만
다음 약 80줄을 추가한다.

ServiceImpl 전체 파일을 읽지 않는다.

---

# STEP 17. Local Method 재귀 추적

Local Method 내부에서
또 다른 Local Method가 호출될 수 있다.

예:

create()
→ saveMaterial()
→ createHistory()

이 경우 동일 방식으로 추적한다.

createHistory(

Grep

↓

Method 위치

↓

부분 Read

↓

내부 호출 확인

단:

VISITED_METHODS에 이미 존재하는 Method는
다시 추적하지 않는다.

---

# STEP 18. 순환 호출 방지

예:

methodA()
→ methodB()
→ methodA()

와 같은 구조가 있을 수 있다.

이미 VISITED_METHODS에 등록된 Method가 다시 호출되면:

ALREADY_VISITED

로 처리하고 재탐색하지 않는다.

무한 추적하지 않는다.

---

# STEP 19. Local Method Overload

동일 이름 Method가 여러 개 존재하면
호출부 argument와 Method signature를 비교한다.

예:

saveMaterial(request)

saveMaterial(request, user)

호출과 일치하는 Method만 추적한다.

확정할 수 없으면:

AMBIGUOUS_LOCAL_METHOD

로 기록한다.

임의 선택하지 않는다.

---

# STEP 20. Mapper 호출

Entry Method 또는 추적된 Local Method에서
직접 Mapper 호출이 발견되면 기록한다.

예:

materialMapper.insertMaterial(param);

기록:

- 호출 Method
- Mapper Variable
- Mapper Method
- 호출 순서

예:

create()
→ saveMaterial()
→ materialMapper.insertMaterial()

---

# STEP 21. Mapper Type

Mapper Java Interface는 찾지 않는다.

Mapper Variable Type만 확인한다.

이미 확보한 ServiceImpl 정보에 없으면
동일 ServiceImpl 파일에서
Mapper Variable 이름만 Grep한다.

예:

materialMapper

↓

private final MaterialMapper materialMapper;

결과:

Mapper Type:
MaterialMapper

Type 확보 후 추가 검색하지 않는다.

---

# STEP 22. Mapper Java SKIP

절대 수행하지 않는다.

- MaterialMapper.java 검색
- interface MaterialMapper 검색
- Mapper Java Method 검색
- @Mapper 검색
- Mapper Java Evidence 검색

Mapper Type + Mapper Method로
MyBatis XML을 직접 찾는다.

---

# STEP 23. MyBatis XML 찾기

현재 Backend 프로젝트 XML에서
Mapper Type을 먼저 검색한다.

예:

MaterialMapper

목표:

<mapper namespace="...MaterialMapper">

XML이 확정되면
다른 XML 검색을 중단한다.

---

# STEP 24. XML fallback

Mapper Type으로 찾지 못한 경우에만
Mapper Method로 검색한다.

예:

id="insertMaterial"

현재 Backend 프로젝트를 우선한다.

현재 프로젝트에서 없을 때만
다른 gipms-api-*로 확대한다.

---

# STEP 25. Statement 위치

확정된 XML 파일 하나에서
실제 Mapper Method에 대응하는
statement id를 찾는다.

예:

id="insertMaterial"

가능:

<select>
<insert>
<update>
<delete>

---

# STEP 26. Statement 부분 Read

Statement 시작부터 약 40줄을 Read한다.

종료 tag가 확인되면 중단한다.

종료되지 않은 경우에만
다음 약 40줄을 추가한다.

XML 전체 파일을 읽지 않는다.

---

# STEP 27. SQL 분석

이미 읽은 Statement 범위에서만 분석한다.

확인:

- SQL Type
- Main Table
- 직접 보이는 추가 Table
- Parameter
- Dynamic SQL
- Include 존재
- ResultMap 존재
- SQL 구조

SQL 분석을 위해 추가 검색하지 않는다.

---

# STEP 28. SQL Type

Statement tag 기준:

<select> → SELECT

<insert> → INSERT

<update> → UPDATE

<delete> → DELETE

---

# STEP 29. Table

직접 보이는 SQL에서 Table을 추출한다.

예:

FROM TB_MATERIAL

INSERT INTO TB_MATERIAL

UPDATE TB_MATERIAL

DELETE FROM TB_MATERIAL

JOIN TB_PLANT

결과:

Main Table:
TB_MATERIAL

Additional Tables:
- TB_PLANT

다른 SQL을 검색하지 않는다.

---

# STEP 30. Parameter

직접 보이는:

#{...}

Parameter만 추출한다.

예:

#{materialId}

#{plantCode}

결과:

- materialId
- plantCode

중복은 제거한다.

Parameter 원본 추적은 하지 않는다.

---

# STEP 31. Dynamic SQL

직접 보이는 다음 Element만 기록한다.

- if
- choose
- when
- otherwise
- foreach

조건이 직접 보이면 함께 기록한다.

예:

<if test="plantCode != null">

↓

IF plantCode != null

---

# STEP 32. include

다음이 발견되어도:

<include refid="Base_Column_List"/>

추적하지 않는다.

기록만 한다.

Include:
Base_Column_List

Status:
NOT_TRACED

---

# STEP 33. resultMap

resultMap이 보여도
정의를 추적하지 않는다.

기록:

ResultMap:
MaterialResultMap

Status:
NOT_TRACED

---

# STEP 34. 호출 순서 유지

Entry Method와 Local Method의
실제 호출 순서를 유지한다.

예:

create()

→ validate()

→ saveMaterial()

→ materialMapper.insertMaterial()

→ createHistory()

→ historyMapper.insertHistory()

순서를 임의로 재배치하지 않는다.

---

# STEP 35. 분기 유지

Local Method 내부에 조건 분기가 있으면
직접 보이는 구조를 유지한다.

예:

saveMaterial()

→ IF exists

   ├─ YES
   │   → materialMapper.updateMaterial()

   └─ NO
       → materialMapper.insertMaterial()

조건값의 출처는 아직 추적하지 않는다.

---

# STEP 36. 현재 테스트에서 다른 Service는 STOP

예:

commonService.validate(request);

가 발견되어도
CommonService 내부로 들어가지 않는다.

기록만 가능하다.

External Service Call:

commonService.validate()

Status:
NOT_TRACED

---

# STEP 37. Evidence

Evidence 대상:

- Controller Method
- Service Method
- ServiceImpl Entry Method
- Local/private Method
- MyBatis Statement

형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:420-468

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:610-645

gipms-api-material/src/main/resources/.../MaterialMapper.xml:210-235

Evidence 확보를 위해 재검색하지 않는다.

---

# STEP 38. 강제 STOP

다음이 완료되면 종료한다.

- Entry Method 확인
- 동일 Class Local/private Method 추적
- 직접 Mapper 호출 확인
- MyBatis Statement 확인
- 직접 SQL 분석

이후 추가 Grep/Read를 수행하지 않는다.

현재 절대 수행하지 않는다:

- Mapper Java Interface
- Mapper Java Method
- include fragment 추적
- resultMap 추적
- 다른 Service 내부 추적
- Component 내부 추적
- Client 내부 추적
- SAP
- RFC
- 외부 API
- DB Metadata
- Oracle Metadata
- BE-REFERENCE
- Markdown 문서 생성

---

# STEP 39. 출력

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

### ServiceImpl Entry Method

- Method:
- Read Range:
- Evidence:

### Local Methods

각 Local Method:

- Method:
- Caller:
- Read Range:
- Evidence:

Local Method가 없으면:

- Local Method: NONE

### Mapper Calls

각 Mapper 호출:

- Caller:
- Mapper Variable:
- Mapper Type:
- Mapper Method:

Mapper Java Interface:

SKIPPED

### MyBatis

각 Mapper Method:

- XML:
- Namespace:
- Statement Type:
- Statement ID:
- Evidence:

### SQL

각 Statement:

- SQL Type:
- Main Table:
- Additional Tables:
- Parameters:
- Dynamic SQL:
- Include:
- ResultMap:

### Execution Flow

실제 호출 순서를 유지한다.

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ validate()
→ saveMaterial()
   → MaterialMapper.insertMaterial()
   → MaterialMapper.xml
   → <insert id="insertMaterial">
   → INSERT TB_MATERIAL
→ createHistory()
   → HistoryMapper.insertHistory()
   → HistoryMapper.xml
   → <insert id="insertHistory">
   → INSERT TB_HISTORY
→ STOP

분기가 있으면 실제 구조를 유지한다.

### Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.
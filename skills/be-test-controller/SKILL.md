---
name: be-test-controller
description: Backend URL에서 Controller부터 ServiceImpl 실제 Mapper 호출과 MyBatis XML Statement를 찾고 해당 Statement의 직접 SQL 구조까지만 분석하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test - Direct SQL Analysis

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
→ Statement 내부 SQL 분석
→ STOP

목적:

MyBatis Statement까지 찾은 상태에서
SQL 자체를 분석하는 비용이 얼마나 추가되는지 측정한다.

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

탐색 순서:

정확한 Symbol
→ 제한 Grep
→ 위치 확보
→ 필요한 범위만 Read
→ 정보 확보
→ 다음 단계

규칙:

1. 동일 검색을 반복하지 않는다.
2. 이미 발견한 파일을 다시 찾지 않는다.
3. 이미 확보한 정보를 다시 검색하지 않는다.
4. 현재 Backend 프로젝트부터 검색한다.
5. 찾지 못할 경우에만 다른 gipms-api-*로 확대한다.
6. Controller 확정 후 URL 검색을 중단한다.
7. Service 확정 후 Service 검색을 중단한다.
8. ServiceImpl 확정 후 구현체 검색을 중단한다.
9. ServiceImpl 전체 Read를 하지 않는다.
10. Mapper Java Interface를 검색하지 않는다.
11. Mapper Java Method를 검색하지 않는다.
12. XML 전체 Read를 하지 않는다.
13. 실제 Statement만 부분 Read한다.
14. SQL 분석은 이미 읽은 Statement 범위에서 수행한다.
15. SQL 분석을 위해 추가 검색하지 않는다.
16. Evidence를 위해 재검색하지 않는다.

내부적으로 재사용:

KNOWN_FILES

KNOWN_SYMBOLS

VISITED_FILES

VISITED_METHODS

VISITED_XML

VISITED_STATEMENTS

---

# STEP 1. URL 분리

입력 URL을 path 단위로 확인한다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

Controller 검색에서는
식별력이 높은 상위 path를 먼저 사용한다.

---

# STEP 2. Controller 찾기

검색 범위:

gipms-api-*/**/*Controller.java

URL 상위 path를 먼저 Grep한다.

후보 Controller가 발견되면
Backend 전체 URL 검색을 중단한다.

상위 path로 확정할 수 없을 경우에만
마지막 segment를 사용한다.

---

# STEP 3. Controller Mapping 검증

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

입력 URL과 일치하는 Method를 확정한다.

---

# STEP 4. HTTP Method 확정

입력이:

GET
POST
PUT
DELETE
PATCH

이면 annotation과 일치해야 한다.

UNKNOWN이면 annotation에서 실제 Method를 확정한다.

동일 URL에 여러 Method가 존재하면:

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

Controller Method 내부의
실제 Service 호출만 확인한다.

예:

materialService.create(request);

확인:

Service Variable:
materialService

Service Method:
create

---

# STEP 7. Service Type

이미 확보한 Controller 내용에서 Type을 확인한다.

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

Controller가 호출한 Service Method만 확인한다.

확인:

- Method
- Parameter
- Return Type
- Evidence

다른 Method는 분석하지 않는다.

---

# STEP 10. ServiceImpl 찾기

Service가 interface이면:

implements MaterialService

를 현재 Backend 프로젝트에서 먼저 검색한다.

발견 즉시 구현체 검색을 중단한다.

ServiceImpl 전체 파일은 읽지 않는다.

---

# STEP 11. ServiceImpl Method 위치

확정된 ServiceImpl 파일 하나에서
실제 호출된 Method 이름만 Grep한다.

예:

create(

Service Interface signature와 비교해
정확한 구현 Method를 확정한다.

---

# STEP 12. ServiceImpl 부분 Read

Method 시작부터 약 80줄을 Read한다.

예:

420 ~ 500

Method 종료가 확인되면 중단한다.

종료되지 않은 경우에만:

501 ~ 580

처럼 필요한 만큼 추가한다.

전체 ServiceImpl 파일을 읽지 않는다.

---

# STEP 13. Mapper 호출 확인

ServiceImpl 대상 Method에서
직접 실행되는 Mapper 호출을 확인한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

기록:

- Mapper Variable
- Mapper Method
- 호출 순서
- 직접 보이는 조건 분기

Local/private Method 내부는 추적하지 않는다.

다른 Service 내부도 추적하지 않는다.

---

# STEP 14. Mapper Type 확인

Mapper Java Interface는 찾지 않는다.

Mapper Variable Type만 확인한다.

예:

private final MaterialMapper materialMapper;

결과:

Mapper Variable:
materialMapper

Mapper Type:
MaterialMapper

현재 범위에 선언이 없을 때만
확정된 ServiceImpl 파일 하나에서
Mapper Variable 이름을 Grep한다.

Type 확보 후 즉시 중단한다.

---

# STEP 15. Mapper Java SKIP

절대 수행하지 않는다:

- MaterialMapper.java 검색
- interface MaterialMapper 검색
- Mapper Java Method 검색
- @Mapper 검색
- Mapper Java Evidence 검색

다음 정보만 사용한다.

Mapper Type

+

Mapper Method

↓

MyBatis XML

---

# STEP 16. MyBatis XML 찾기

현재 Backend 프로젝트의 XML에서
Mapper Type을 우선 검색한다.

예:

MaterialMapper

목표:

<mapper namespace="...MaterialMapper">

후보 XML이 확정되면
다른 XML을 찾지 않는다.

---

# STEP 17. XML fallback

Mapper Type으로 XML을 찾지 못한 경우에만
Mapper Method 이름으로 검색한다.

예:

id="insertMaterial"

현재 Backend 프로젝트에서 먼저 검색한다.

발견한 XML의 namespace를 확인한다.

현재 프로젝트에서 찾지 못할 때만
다른 gipms-api-*로 확대한다.

---

# STEP 18. Statement 위치

확정된 XML 하나에서
실제 Mapper Method의 statement id를 찾는다.

예:

id="insertMaterial"

가능한 타입:

<select>
<insert>
<update>
<delete>

Statement 시작 line을 확보한다.

---

# STEP 19. Statement 부분 Read

Statement 시작부터 약 40줄만 먼저 Read한다.

예:

210 ~ 250

다음 종료 tag가 확인되면 중단한다.

</select>
</insert>
</update>
</delete>

40줄 안에서 종료되지 않으면
다음 약 40줄만 추가한다.

XML 전체 파일을 읽지 않는다.

---

# STEP 20. SQL 분석

이 단계부터 이번 테스트에서 새로 추가되는 범위다.

이미 읽은 Statement 내용만 사용한다.

추가 Grep이나 Read를 하지 않는다.

확인:

- SQL Type
- Main Table
- 직접 보이는 Sub Table
- SQL Parameter
- Dynamic SQL Element
- SQL 실행 구조

---

# STEP 21. SQL Type

Statement tag 기준으로 SQL Type을 기록한다.

예:

<select>

→ SELECT

<insert>

→ INSERT

<update>

→ UPDATE

<delete>

→ DELETE

---

# STEP 22. Main Table

Statement에 직접 작성된 SQL에서
주요 대상 Table을 확인한다.

예:

SELECT
FROM TB_MATERIAL

결과:

Main Table:
TB_MATERIAL

예:

INSERT INTO TB_MATERIAL

결과:

Main Table:
TB_MATERIAL

예:

UPDATE TB_MATERIAL

결과:

Main Table:
TB_MATERIAL

예:

DELETE FROM TB_MATERIAL

결과:

Main Table:
TB_MATERIAL

---

# STEP 23. 직접 보이는 추가 Table

JOIN 또는 Subquery에 다른 Table이 직접 보이면 기록한다.

예:

FROM TB_MATERIAL A
LEFT JOIN TB_PLANT B
    ON ...

결과:

Tables:

- TB_MATERIAL
- TB_PLANT

추가 Table을 찾기 위해
다른 파일이나 Statement를 검색하지 않는다.

---

# STEP 24. SQL Parameter

Statement에서 직접 사용하는 MyBatis Parameter를 기록한다.

예:

#{materialId}

#{plantCode}

#{userId}

결과:

Parameters:

- materialId
- plantCode
- userId

동일 Parameter는 중복 출력하지 않는다.

Parameter 값의 원본까지 역추적하지 않는다.

---

# STEP 25. Dynamic SQL

Statement 내부에 다음이 직접 존재하면 기록한다.

- <if>
- <choose>
- <when>
- <otherwise>
- <foreach>

예:

<if test="plantCode != null">
    AND PLANT_CODE = #{plantCode}
</if>

결과:

Dynamic SQL:
YES

Condition:

plantCode != null

SQL:

AND PLANT_CODE = #{plantCode}

직접 보이는 조건까지만 기록한다.

---

# STEP 26. choose

예:

<choose>

    <when test="type == 'A'">
        ...
    </when>

    <otherwise>
        ...
    </otherwise>

</choose>

이면 직접 보이는 분기만 기록한다.

예:

CHOOSE

├─ type == 'A'
└─ otherwise

분기의 상세 비즈니스 의미는 분석하지 않는다.

---

# STEP 27. foreach

예:

<foreach
    collection="items"
    item="item"
>

이면 다음만 기록한다.

Collection:
items

Item:
item

반복 생성되는 SQL 구조가 직접 보이면 간단히 기록한다.

Java Collection 생성 위치는 추적하지 않는다.

---

# STEP 28. include 처리

이번 테스트에서 매우 중요하다.

다음이 발견되어도:

<include refid="Base_Column_List"/>

include 대상 fragment를 찾지 않는다.

기록만 한다.

Include:
Base_Column_List

Status:
NOT_TRACED

추가 Grep을 수행하지 않는다.

---

# STEP 29. resultMap

다음이 보여도:

resultMap="MaterialResultMap"

resultMap 정의를 찾지 않는다.

기록:

ResultMap:
MaterialResultMap

Status:
NOT_TRACED

---

# STEP 30. SQL 실행 구조

이미 읽은 Statement 기준으로
간단한 실행 구조만 만든다.

예:

INSERT
→ TB_MATERIAL
→ Parameters
   - materialId
   - plantCode
   - userId

또는:

SELECT
→ TB_MATERIAL
→ LEFT JOIN TB_PLANT
→ IF plantCode != null
   → PLANT_CODE = #{plantCode}

새로운 검색 없이 작성한다.

---

# STEP 31. SQL 분석 금지 범위

현재 테스트에서는 하지 않는다.

- include fragment 추적
- resultMap 추적
- 다른 Statement 추적
- DB Metadata
- Oracle Metadata
- 컬럼 Metadata
- FK/PK 분석
- Index 분석
- 실행계획 분석
- SQL 성능 분석
- 실제 DB 조회

---

# STEP 32. Evidence

Evidence:

- Controller Method
- Service Method
- ServiceImpl Method
- MyBatis Statement

형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:420-468

gipms-api-material/src/main/resources/.../MaterialMapper.xml:210-235

Mapper Java Evidence는 만들지 않는다.

SQL Evidence는 MyBatis Statement Evidence와 동일하게 사용한다.

Evidence를 위해 재검색하지 않는다.

---

# STEP 33. 강제 STOP

실제 MyBatis Statement 내부의 직접 SQL 분석이 완료되면
즉시 종료한다.

이후 Grep/Read를 수행하지 않는다.

절대 수행하지 않는다:

- Mapper Java 검색
- include fragment 추적
- resultMap 추적
- Local/private Method 추적
- 다른 Service 추적
- 다른 Mapper Statement 추적
- DB Metadata
- Oracle Metadata
- SAP
- RFC
- 외부 API
- BE-REFERENCE
- Markdown 문서 생성

---

# STEP 34. 출력

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

- Mapper Variable:
- Mapper Type:
- Mapper Method:

Mapper Java Interface:

SKIPPED

### MyBatis

- XML:
- Namespace:
- Statement Type:
- Statement ID:
- Evidence:

### SQL

- SQL Type:
- Main Table:
- Tables:
- Parameters:
- Dynamic SQL:
- Include:
- ResultMap:

### SQL Flow

실제 Statement에서 확인된 구조만 출력한다.

예:

INSERT
→ TB_MATERIAL
→ Parameters
   ├─ materialId
   ├─ plantCode
   └─ userId

또는:

SELECT
→ TB_MATERIAL
→ LEFT JOIN TB_PLANT
→ IF plantCode != null
   └─ AND PLANT_CODE = #{plantCode}

### Execution Flow

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ MaterialMapper.insertMaterial()
→ Mapper Java SKIP
→ MaterialMapper.xml
→ <insert id="insertMaterial">
→ INSERT TB_MATERIAL
→ STOP

### Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.
# BE ANALYSIS REFERENCE

> 이 문서는 Backend 분석 결과의 표현 형식, 문서 구조,
> 가독성 및 상세 수준을 정의하기 위한 Reference이다.
>
> 이 문서에 포함된 Class, Method, Path, SQL, Table,
> RFC Function, URL, Parameter, Response 및 Sample 값은
> 실제 분석 Evidence가 아니다.
>
> 최종 Backend 분석 문서는 반드시 실제 Source,
> MyBatis SQL, Oracle Metadata 및 확인된 실행 흐름을
> 기준으로 작성한다.

---

# 1. 문서 목적

Backend 분석 문서는 선택한 API 하나의 실제 Backend 실행 흐름을
개발자가 Source를 따라갈 수 있는 수준으로 설명한다.

단순한 Layer 목록이 아니라 실제 실행 순서를 표현한다.

예:

```text
Controller
 ↓
Service
 ↓
Validation
 ↓
DB #1
 ↓
조건
 ├─ YES
 │   ↓
 │  RFC #1
 │   ↓
 │  RFC 결과 처리
 │
 └─ NO
     ↓
    Local 처리
 ↓
DB #2
 ↓
REST #1
 ↓
REST 결과 처리
 ↓
DB #3
 ↓
Response 생성
```

DB / RFC / REST / 기타 외부 연동은
종류별로 재배치하지 않는다.

실제 Business Logic에서 호출되는 위치를 유지한다.

---

# 2. 기본 문서 구조

최종 Backend 문서는 기본적으로 다음 순서로 작성한다.

```text
1. 기능 정보
2. 기능 요약
3. 한눈에 보는 실행 흐름
4. 핵심 정보
5. 전체 실행 Tree
6. Business Logic 상세
7. DB / SQL 상세
8. 외부 연동 상세
9. Exception / Transaction
10. Response 생성
11. Source Evidence
12. 미확인 항목
13. 분석 경계
```

실제 기능에 존재하지 않는 항목을
억지로 생성하지 않는다.

---

# 3. 기능 정보

```text
┌─ 기능 정보 ──────────────────────────────────
│ 화면
│   equipment-search
│
│ API ID
│   API-001
│
│ 기능명
│   설비 목록 조회
│
│ HTTP Method
│   POST
│
│ Backend URL
│   /api/equipment/search
│
│ Backend Project
│   gipms-api-equipment
└──────────────────────────────────────────────
```

기능 정보 Block 하나만 복사해도
어떤 API를 분석한 것인지 알 수 있어야 한다.

---

# 4. 기능 요약

긴 Source 분석 내용을 그대로 반복하지 않고
해당 Backend 기능이 무엇을 수행하는지 설명한다.

```text
┌─ 기능 요약 ──────────────────────────────────
│ 설비 검색 요청을 받아 입력값을 검증한 후
│ 설비 기본정보를 조회한다.
│
│ 조회 결과와 요청 조건에 따라 SAP RFC를 호출하고,
│ RFC 결과를 Backend 데이터 구조로 변환한다.
│
│ 이후 상태 정보를 갱신하고 최종 조회 결과를
│ Response 객체로 생성하여 Controller에 반환한다.
└──────────────────────────────────────────────
```

실제 Source에서 확인되지 않은 업무 목적을
추측해서 추가하지 않는다.

---

# 5. 한눈에 보는 실행 흐름

복잡한 Backend 로직을 먼저 짧게 이해할 수 있도록
주요 실행 흐름을 요약한다.

```text
┌─ 한눈에 보는 실행 흐름 ─────────────────────
│ Controller
│   ↓
│ Service
│   ↓
│ Validation
│   ↓
│ DB #1 - 설비 조회
│   ↓
│ 조건 판단
│   ├─ YES → RFC #1
│   └─ NO  → RFC 미호출
│   ↓
│ RFC 결과 가공
│   ↓
│ DB #2 - 상태 갱신
│   ↓
│ Response 생성
└──────────────────────────────────────────────
```

이 Block은 전체 흐름의 요약이다.

상세 호출 관계는 전체 실행 Tree에서 표현한다.

---

# 6. 핵심 정보

```text
┌─ 핵심 정보 ──────────────────────────────────
│ API
│   POST /api/equipment/search
│
│ Controller
│   EquipmentController.search()
│   Source:
│   gipms-api-equipment/src/main/java/.../EquipmentController.java
│
│ Main Service
│   EquipmentServiceImpl.searchEquipment()
│   Source:
│   gipms-api-equipment/src/main/java/.../EquipmentServiceImpl.java
│
│ DB 호출
│   총 3개
│
│   DB #1 SELECT
│   DB #2 UPDATE
│   DB #3 INSERT
│
│ 외부 연동
│   총 3개
│
│   RFC #1
│   RFC #2
│   REST #1
│
│ 주요 Oracle Object
│   TB_EQUIPMENT
│   TB_PLANT
│
│ 최종 Response
│   EquipmentSearchResponse
└──────────────────────────────────────────────
```

호출 수가 많은 경우에도
임의의 일부만 선택하지 않는다.

---

# 7. 전체 실행 Tree

Backend 문서에서 가장 중요한 영역이다.

실제 Source 실행 순서를 기준으로 작성한다.

```text
Controller
│
├─ EquipmentController.search()
│
└─ EquipmentService.searchEquipment()
    ↓
    EquipmentServiceImpl.searchEquipment()
    │
    ├─ [1] Request 값 확인
    │
    ├─ [2] validateRequest()
    │      │
    │      ├─ plantCode 확인
    │      └─ searchType 확인
    │
    ├─ [3] DB #1 - 설비 기본정보 조회
    │      │
    │      ├─ EquipmentMapper.selectEquipment()
    │      ├─ namespace
    │      │    com.example.EquipmentMapper
    │      ├─ Statement ID
    │      │    selectEquipment
    │      └─ SELECT ...
    │
    ├─ [4] 조회 결과 확인
    │      │
    │      ├─ 결과 있음
    │      │    ↓
    │      │   다음 처리
    │      │
    │      └─ 결과 없음
    │           ↓
    │          Empty Response
    │
    ├─ [5] SAP 호출 조건 판단
    │      │
    │      ├─ sapUseYn == "Y"
    │      │    ↓
    │      │   SapService.callEquipment()
    │      │    ↓
    │      │   SapAdapter.call()
    │      │    ↓
    │      │   RFC #1
    │      │
    │      └─ 그 외
    │           ↓
    │          RFC 호출 없음
    │
    ├─ [6] RFC #1 Response 처리
    │
    ├─ [7] DB #2 - 상태 갱신
    │
    ├─ [8] REST #1
    │
    ├─ [9] REST Response 처리
    │
    ├─ [10] RFC #2
    │
    ├─ [11] DB #3
    │
    ├─ [12] Response 생성
    │
    └─ return
```

호출 순서를 보기 좋게 만들기 위해
DB와 외부 연동을 별도 단계로 이동시키지 않는다.

---

# 8. Business Logic 상세

Business Logic은 실행 순서에 따라
독립적인 Block으로 작성한다.

예:

```text
┌─ Business Logic #1 : 요청값 검증 ────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 처리 내용
│   검색 요청값을 확인한다.
│
│ 입력
│   plantCode
│   searchType
│
│ 조건
│   plantCode가 비어 있음
│     → Validation Error
│
│   searchType이 허용값이 아님
│     → Validation Error
│
│ 정상 처리
│   DB #1 실행
│
│ Source
│   gipms-api-equipment/src/main/java/.../
│   EquipmentServiceImpl.java
│
│ Method
│   searchEquipment()
│
│ Line Range
│   55-72
└──────────────────────────────────────────────
```

각 Block은 다른 Block을 보지 않아도
무슨 처리를 하는지 이해할 수 있어야 한다.

---

# 9. 조건 / 분기 상세

중요 조건은 독립 Block으로 표현할 수 있다.

```text
┌─ 조건 #1 : SAP 호출 여부 ───────────────────
│ 판단 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 조건
│   sapUseYn == "Y"
│
│ YES
│   RFC #1 호출
│
│ NO
│   RFC 호출 없이 다음 Business Logic 수행
│
│ 후속 처리
│   RFC 호출 여부와 관계없이 이후 상태 처리 진행
│
│ Source
│   gipms-api-equipment/src/main/java/.../
│   EquipmentServiceImpl.java
└──────────────────────────────────────────────
```

Source에 없는 YES/NO 결과를 만들지 않는다.

---

# 10. DB 상세 기본 원칙

각 DB 호출은 별도의 독립 Block으로 작성한다.

```text
DB #1
DB #2
DB #3
...
```

같은 Mapper가 반복 호출되어도
실행 위치가 다르면 별도 호출로 표현한다.

---

# 11. DB 상세 Block

```text
┌─ DB #1 : 설비 기본정보 조회 ────────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 호출 목적
│   검색 조건에 해당하는 설비 기본정보 조회
│
│ Mapper
│   EquipmentMapper.selectEquipment()
│
│ Mapper Source
│   gipms-api-equipment/src/main/java/.../
│   EquipmentMapper.java
│
│ MyBatis XML
│   gipms-api-equipment/src/main/resources/.../
│   EquipmentMapper.xml
│
│ Namespace
│   com.example.EquipmentMapper
│
│ Statement ID
│   selectEquipment
│
│ SQL Type
│   SELECT
│
│ 입력 Parameter
│   plantCode
│   equipmentName
│   useYn
│
│ 주요 Oracle Object
│   TB_EQUIPMENT
│   TB_PLANT
│
│ Result
│   EquipmentVO List
│
│ 호출 후 처리
│   조회 결과 존재 여부를 판단한 후
│   SAP RFC 호출 여부를 결정한다.
└──────────────────────────────────────────────
```

---

# 12. SQL 상세

SQL이 중요하거나 복잡하면
DB Block과 별도로 SQL 상세 Block을 작성한다.

```text
┌─ DB #1 SQL ──────────────────────────────────
│ SQL Type
│   SELECT
│
│ FROM
│   TB_EQUIPMENT E
│
│ JOIN
│   TB_PLANT P
│
│ Join Condition
│   E.PLANT_CODE = P.PLANT_CODE
│
│ 기본 조건
│   E.USE_YN = 'Y'
│
│ Dynamic 조건
│
│   plantCode 존재
│     → E.PLANT_CODE = #{plantCode}
│
│   equipmentName 존재
│     → E.EQUIPMENT_NAME LIKE ...
│
│   equipmentIds 존재
│     → foreach를 사용하여 IN 조건 생성
│
│ 정렬
│   실제 SQL 기준으로 기록
│
│ Result Mapping
│   실제 resultType / resultMap 기준으로 기록
└──────────────────────────────────────────────
```

SQL 전체를 무조건 복사하는 것이 목적이 아니다.

개발자가 실제 처리 내용을 이해하는 데 필요한
SQL 구조와 조건을 설명한다.

단, 중요한 계산식이나 업무 조건은 생략하지 않는다.

---

# 13. Dynamic SQL 상세

```text
┌─ DB #1 Dynamic SQL ──────────────────────────
│ plantCode
│   값 있음
│     → PLANT_CODE 조건 추가
│
│   값 없음
│     → PLANT_CODE 조건 추가 안 함
│
│ equipmentName
│   값 있음
│     → EQUIPMENT_NAME 검색 조건 추가
│
│ equipmentIds
│   Collection 존재
│     → foreach
│     → 각 equipmentId를 Parameter Mapping
│     → IN (...) 생성
│
│ useYn
│   값 있음
│     → 전달된 값 사용
│
│   값 없음
│     → 기본 조건 적용
└──────────────────────────────────────────────
```

실제 MyBatis XML의 분기 구조를 보존한다.

---

# 14. SQL Parameter Mapping

```text
┌─ DB #1 Parameter Mapping ────────────────────
│ plantCode
│
│ API Request
│   plantCode
│     ↓
│ Service
│   request.getPlantCode()
│     ↓
│ Mapper
│   plantCode
│     ↓
│ MyBatis
│   #{plantCode}
│     ↓
│ Oracle
│   TB_EQUIPMENT.PLANT_CODE
│
│ equipmentName
│
│ API Request
│   equipmentName
│     ↓
│ Mapper
│   equipmentName
│     ↓
│ MyBatis
│   #{equipmentName}
│     ↓
│ Oracle
│   TB_EQUIPMENT.EQUIPMENT_NAME
└──────────────────────────────────────────────
```

Mapping을 Source에서 확인할 수 없는 단계는
억지로 연결하지 않는다.

---

# 15. SQL Result Mapping

```text
┌─ DB #1 Result Mapping ───────────────────────
│ Oracle
│   EQUIPMENT_ID
│     ↓
│ MyBatis
│   equipmentId
│     ↓
│ VO
│   EquipmentVO.equipmentId
│
│ Oracle
│   EQUIPMENT_NAME
│     ↓
│ MyBatis
│   equipmentName
│     ↓
│ VO
│   EquipmentVO.equipmentName
└──────────────────────────────────────────────
```

필드가 매우 많은 경우 대량 항목 규칙을 적용한다.

---

# 16. Oracle Metadata

```text
┌─ Oracle Object #1 ───────────────────────────
│ Object
│   TB_EQUIPMENT
│
│ Object Type
│   TABLE
│
│ 분석 API에서 사용되는 주요 Column
│
│ EQUIPMENT_ID
│   Type: VARCHAR2(...)
│   Nullable: NO
│   PK: YES
│
│ PLANT_CODE
│   Type: VARCHAR2(...)
│   Nullable: NO
│
│ EQUIPMENT_NAME
│   Type: VARCHAR2(...)
│   Nullable: YES
│
│ Metadata Source
│   Oracle MCP
└──────────────────────────────────────────────
```

Oracle Metadata는 실제 확인 결과만 기록한다.

---

# 17. 외부 연동 기본 원칙

외부 연동은 실제 실행 위치에 따라 번호를 부여한다.

예:

```text
RFC #1
REST #1
RFC #2
SOAP #1
```

종류별로 다시 정렬하지 않는다.

전체 실행 Tree에서는 실제 순서를 유지한다.

---

# 18. RFC 상세 Block

```text
┌─ RFC #1 : 설비정보 조회 ────────────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 호출 경로
│   EquipmentServiceImpl.searchEquipment()
│     ↓
│   SapService.getEquipment()
│     ↓
│   SapAdapter.call()
│     ↓
│   RFC Client
│
│ RFC Function
│   Z_PM_EQUIPMENT_SEARCH
│
│ 호출 조건
│   sapUseYn == "Y"
│
│ Request 생성
│   SapEquipmentRequest 생성
│
│ Input Mapping
│   request.plantCode
│     ↓
│   SapEquipmentRequest.plant
│     ↓
│   I_WERKS
│
│ Response
│   RFC Result
│
│ Response Mapping
│   E_RESULT
│     ↓
│   SapEquipmentResponse.result
│     ↓
│   Service 결과 판단
│
│ Error 처리
│   실제 Source 기준으로 기록
│
│ 호출 후 처리
│   RFC 결과를 변환한 후 DB #2 실행
└──────────────────────────────────────────────
```

RFC 상세 Block만 Excel에 복사해도
호출 목적과 전후 관계를 이해할 수 있어야 한다.

---

# 19. RFC Parameter Mapping

Parameter가 많거나 Mapping이 중요한 경우
별도 Block을 사용한다.

```text
┌─ RFC #1 Parameter Mapping ───────────────────
│ plantCode
│
│ Backend
│   request.plantCode
│     ↓
│ SAP Request
│   plant
│     ↓
│ RFC
│   I_WERKS
│
│ equipmentId
│
│ Backend
│   equipmentId
│     ↓
│ SAP Request
│   equipment
│     ↓
│ RFC
│   I_EQUNR
└──────────────────────────────────────────────
```

---

# 20. REST / HTTP 상세 Block

```text
┌─ REST #1 : 외부 상태 조회 ──────────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 호출 경로
│   EquipmentServiceImpl.searchEquipment()
│     ↓
│   InterfaceService.getStatus()
│     ↓
│   ExternalStatusClient.getStatus()
│
│ HTTP Method
│   POST
│
│ URL
│   실제 Source에서 확인된 URL
│
│ Header
│   실제 Source에서 확인된 Header
│
│ Query / Path
│   실제 Source 기준
│
│ Request Body
│   실제 Request 객체 기준
│
│ Response
│   실제 Response 객체 기준
│
│ Status 처리
│   실제 Source 기준
│
│ Error 처리
│   실제 Source 기준
│
│ Timeout
│   실제 Source에서 확인되는 경우 기록
│
│ 호출 후 처리
│   Response 상태를 확인한 후 다음 Business Logic 수행
└──────────────────────────────────────────────
```

---

# 21. 기타 외부 연동

SOAP, Message, File 등의 연동도
동일한 원칙으로 독립 Block을 생성한다.

예:

```text
┌─ SOAP #1 ────────────────────────────────────
│ 호출 위치
│   ...
│
│ 호출 경로
│   ...
│
│ Request
│   ...
│
│ Response
│   ...
│
│ Error 처리
│   ...
│
│ 호출 후 처리
│   ...
└──────────────────────────────────────────────
```

실제 기능에 존재하지 않는 연동 Block은 생성하지 않는다.

---

# 22. 반복 처리

반복문 안에서 DB 또는 외부 연동이 발생하면
반복 구조를 명확히 표시한다.

```text
┌─ 반복 처리 #1 ───────────────────────────────
│ 반복 대상
│   equipmentList
│
│ 각 equipment 처리
│
│   equipment
│     ↓
│   DB #2
│     ↓
│   조건 확인
│     ↓
│   RFC #2
│     ↓
│   결과 저장
│
│ 반복 횟수
│   Runtime 값에 따라 결정
└──────────────────────────────────────────────
```

Source에서 실제 반복 횟수를 알 수 없다면
숫자를 임의로 생성하지 않는다.

---

# 23. Exception 처리

```text
┌─ Exception #1 : RFC 호출 실패 ──────────────
│ 발생 위치
│   SapAdapter.call()
│
│ 발생 조건
│   실제 Source에서 확인되는 조건
│
│ 처리
│   catch
│     ↓
│   Error Mapping
│     ↓
│   BusinessException
│
│ 후속 흐름
│   현재 처리 중단
│
│ Source
│   gipms-api-common/src/main/java/.../
│   SapAdapter.java
└──────────────────────────────────────────────
```

Framework 일반 동작만으로
Exception 흐름을 추측하지 않는다.

---

# 24. Transaction

```text
┌─ Transaction ────────────────────────────────
│ 적용 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 설정
│   @Transactional
│
│ 실제 실행 순서
│   DB #1 SELECT
│     ↓
│   DB #2 UPDATE
│     ↓
│   RFC #1
│     ↓
│   DB #3 INSERT
│
│ Rollback
│   실제 Source / Transaction 설정에서
│   확인되는 범위만 기록
└──────────────────────────────────────────────
```

외부 연동 실패 시 Rollback 여부를
추측하지 않는다.

---

# 25. Response 생성

```text
┌─ Response 생성 ──────────────────────────────
│ 입력
│   DB 조회 결과
│   RFC 결과
│   REST 결과
│
│ 처리
│   결과 데이터 변환
│     ↓
│   Response DTO 생성
│     ↓
│   필드 Mapping
│
│ Response
│   EquipmentSearchResponse
│
│ Controller Return
│   ResponseEntity<EquipmentSearchResponse>
└──────────────────────────────────────────────
```

Backend 분석은 SQL이나 외부 연동에서 끝내지 않고
최종 Response까지 연결한다.

---

# 26. Response Mapping

필요한 경우 상세 Mapping을 독립 Block으로 작성한다.

```text
┌─ Response Mapping ───────────────────────────
│ equipmentId
│
│ DB
│   EQUIPMENT_ID
│     ↓
│ VO
│   equipmentId
│     ↓
│ Response
│   equipmentId
│
│ status
│
│ RFC
│   E_STATUS
│     ↓
│ Service
│   status
│     ↓
│ Response
│   status
└──────────────────────────────────────────────
```

---

# 27. Source Evidence

```text
┌─ Source Evidence ────────────────────────────
│ Controller
│   Path:
│   gipms-api-equipment/src/main/java/.../
│   EquipmentController.java
│
│   Method:
│   search()
│
│   Line:
│   40-65
│
│ Service
│   Path:
│   gipms-api-equipment/src/main/java/.../
│   EquipmentServiceImpl.java
│
│   Method:
│   searchEquipment()
│
│   Line:
│   55-140
│
│ Mapper
│   Path:
│   gipms-api-equipment/src/main/java/.../
│   EquipmentMapper.java
│
│   Method:
│   selectEquipment()
│
│ MyBatis
│   Path:
│   gipms-api-equipment/src/main/resources/.../
│   EquipmentMapper.xml
│
│   Namespace:
│   com.example.EquipmentMapper
│
│   Statement:
│   selectEquipment
│
│ RFC
│   Path:
│   gipms-api-common/src/main/java/.../
│   SapAdapter.java
│
│   Method:
│   call()
└──────────────────────────────────────────────
```

Source Path는 Project Root 기준 상대경로를 사용한다.

Line Range를 확인할 수 없다면:

```text
Line Range:
확인되지 않음
```

으로 기록한다.

---

# 28. 미확인 항목

```text
┌─ 미확인 항목 ────────────────────────────────
│ REST #1 Base URL
│   현재 Project Source에서 확인되지 않음
│
│ RFC #2 특정 Return Field 의미
│   현재 Source에서 확인되지 않음
│
│ External Client 내부 구현
│   외부 Dependency 경계로 인해 분석하지 않음
└──────────────────────────────────────────────
```

확인되지 않은 값을 추측해서 채우지 않는다.

---

# 29. 분석 경계

```text
┌─ 분석 경계 ──────────────────────────────────
│ 분석 대상
│   API-001
│
│ 포함
│   Controller
│   Service / ServiceImpl
│   Internal Method
│   Other Service / Common Service
│   Mapper
│   MyBatis
│   SQL
│   Oracle Metadata
│   RFC
│   REST / HTTP
│   기타 실제 외부 연동
│   Exception
│   Transaction
│   Response
│
│ 제외
│   다른 API
│   다른 화면
│   관련 없는 Service
│   관련 없는 Mapper / SQL
│   관련 없는 Oracle Object
│   Project 전체 RFC
│   Project 전체 외부 연동
│   JAR 내부
│   Decompiled Source
│   Project Root 외부 Source
└──────────────────────────────────────────────
```

---

# 30. Excel 단일 셀 복사 규칙

Backend 문서는 사용자가 필요한 영역을
Excel 한 셀에 직접 복사하는 것을 고려하여 작성한다.

따라서 기본적으로 Markdown Table을 사용하지 않는다.

다음 형식을 우선한다.

```text
┌─ 제목 ─────────────────────────────
│ 항목
│   값
│
│ 항목
│   값
└────────────────────────────────────
```

또는:

```text
조건
 ↓
처리
 ↓
결과
```

각 Block은 독립적으로 복사 가능해야 한다.

---

# 31. Excel Block 독립성

각 Block은 다른 Block을 함께 복사하지 않아도
내용을 이해할 수 있어야 한다.

피해야 할 표현:

```text
위와 동일
앞에서 설명
이전 SQL 참고
상기 RFC 참고
동일 Mapper 사용
```

대신 필요한 핵심 Context를
현재 Block 안에 다시 포함한다.

단, 의미 없는 장문 반복은 피한다.

---

# 32. Markdown Table 사용 제한

Backend 최종 문서에서는 기본적으로:

```text
| 항목 | 내용 |
|---|---|
```

형태의 Markdown Table을 사용하지 않는다.

Excel 단일 셀 복사와
ASCII 실행 흐름의 가독성을 우선한다.

---

# 33. 대량 항목 표시 규칙

Parameter, Column, Mapping 등이 많으면
임의의 일부만 표시하지 않는다.

먼저 전체 규모를 설명한다.

예:

```text
Request Parameter
총 27개

그룹
- 검색 조건: 8개
- 설비 정보: 11개
- 상태 정보: 5개
- 제어 정보: 3개
```

그 후 필요한 상세 내용을 아래 Block에서 계속 작성한다.

`대표 5개만 표시`처럼
근거 없이 일부를 생략하지 않는다.

---

# 34. 다중 DB 호출 표시

DB 호출이 많아도 하나의 DB Block으로 합치지 않는다.

예:

```text
DB 호출 총 5개

DB #1
  SELECT

DB #2
  SELECT

DB #3
  UPDATE

DB #4
  INSERT

DB #5
  SELECT
```

상세 내용은 각각:

```text
DB #1 상세
DB #2 상세
DB #3 상세
...
```

로 작성한다.

---

# 35. 다중 외부 연동 표시

외부 연동 역시 합치지 않는다.

예:

```text
외부 연동 총 4개

RFC #1
RFC #2
REST #1
RFC #3
```

전체 실행 Tree에서는 반드시
실제 호출 순서를 유지한다.

예:

```text
DB #1
 ↓
RFC #1
 ↓
DB #2
 ↓
RFC #2
 ↓
REST #1
 ↓
DB #3
 ↓
RFC #3
```

---

# 36. 실제 실행 순서 우선

다음처럼 Layer 기준으로 문서를 해석하지 않는다.

잘못된 예:

```text
Controller
 ↓
Service
 ↓
Mapper
 ↓
SQL
 ↓
RFC
 ↓
Response
```

실제 Source가 다음이라면:

```text
Service
 ↓
DB #1
 ↓
조건
 ↓
RFC #1
 ↓
RFC 결과 처리
 ↓
DB #2
 ↓
REST #1
 ↓
REST 결과 처리
 ↓
RFC #2
 ↓
DB #3
 ↓
Response
```

그 순서를 그대로 문서에 반영한다.

---

# 37. 호출 복귀 표현

하위 호출 이후 원래 Business Logic으로 돌아오는 구조를
가능하면 실행 Tree에서 표현한다.

```text
Service A
 │
 ├─ Service B
 │    │
 │    ├─ DB #1
 │    └─ return result
 │
 ├─ Service B 결과 처리
 │
 ├─ RFC #1
 │    │
 │    └─ return RFC result
 │
 ├─ RFC 결과 처리
 │
 └─ Response
```

호출된 하위 기능만 설명하고
원래 Caller의 후속 처리를 빠뜨리지 않는다.

---

# 38. Source와 Runtime 구분

Source에서 확인한 사실과
Runtime에서 관찰한 값을 혼동하지 않는다.

예:

```text
Source
  RFC 호출 조건:
  sapUseYn == "Y"

Runtime 관찰
  이번 실행에서 sapUseYn = "Y"
  → RFC 호출 발생
```

Runtime에서 한 번 관찰된 값이
전체 Business Rule인 것처럼 표현하지 않는다.

---

# 39. Oracle Metadata와 SQL 구분

SQL Source와 Oracle Metadata는 역할이 다르다.

```text
MyBatis SQL
  실제 Query 및 조건 확인

Oracle MCP
  실제 Object / Column / Type / Nullable / PK 확인
```

Oracle Metadata만 보고
Business Logic을 추측하지 않는다.

---

# 40. Reference 적용 규칙

이 Reference는 다음을 결정한다.

```text
문서 구조
표현 형식
상세 수준
ASCII Tree 형식
Excel 복사 구조
Block 독립성
```

이 Reference는 다음을 결정하지 않는다.

```text
실제 Class
실제 Method
실제 Source Path
실제 SQL
실제 Table
실제 RFC Function
실제 Endpoint
실제 Parameter
실제 Response
```

실제 값은 분석 대상 Source에서 확인한다.

---

# 41. SAMPLE 데이터 사용 금지

이 Reference에 있는 모든 이름과 값은 Sample이다.

예:

```text
EquipmentController
EquipmentServiceImpl
EquipmentMapper
TB_EQUIPMENT
Z_PM_EQUIPMENT_SEARCH
/api/equipment/search
plantCode
```

최종 문서에 Sample 값을
Evidence처럼 복사하지 않는다.

실제 Source에서 확인되지 않은 경우:

```text
확인되지 않음
```

으로 기록한다.

---

# 42. 최종 문서 작성 원칙

최종 Backend 문서는 다음 조건을 만족해야 한다.

```text
실제 Source 기반
실제 실행 순서 기반
Controller부터 Response까지 연결
중간 Method / Service 호출 보존
DB 호출 위치 보존
Mapper → MyBatis → SQL 연결
Dynamic SQL 보존
Oracle Metadata 교차 확인
RFC / REST / 기타 외부 연동 위치 보존
외부 연동 후 Caller 복귀
조건 / 분기 / 반복 보존
Exception / Transaction 확인
Source Evidence 포함
확인되지 않은 정보 추측 금지
Project Root 상대경로 사용
JAR / 외부 Dependency 경계 준수
Excel 단일 셀 복사 가능
Markdown Table 기본 사용 금지
각 상세 Block 독립적으로 이해 가능
```

문서를 보기 좋게 만드는 것보다
실제 Backend Business Logic을 정확하게 보존하는 것을 우선한다.